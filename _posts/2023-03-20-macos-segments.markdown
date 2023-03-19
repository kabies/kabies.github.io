---
title: "macOS(Mach-O)のセグメント探索とmruby-3.2.0"
date: 2023-03-20 00:00:00 +0900
tags: macOS mruby
---

macOS で mruby-3.2.0 を共有ライブラリ化した際、`mrb_ro_data_p` の挙動がおかしいので調査した。
結論としてはコンパイル時に `MRB_NO_DEFAULT_RO_DATA_P` を定義し、
`mrb_ro_data_p` 関数をオフ(常時FALSEを戻す)にして解決としたのだが、
実行ファイルとセグメントに関してのメモを残しておく。

## mrb_ro_data_p 関数について
`mrb_ro_data_p` 関数は以下のように説明されている。

> Return `TRUE` if `ptr` is in the read-only section, otherwise return `FALSE`.

渡されたアドレスが Read-Only かをチェックする関数で、
Linuxでの実装は `&etext < p && p < &edata` というロジックであり、
だいたい 「`.text` セグメント後から `.data` セグメント終端までの間にあるか？」というチェックが行われている。

## Linuxでの話
Linux での実行ファイルのセグメントは、 `size -A foo.exe` や
`objdump -t bar.exe | sort` などのコマンドで見ることができるが、
.text から .data の間に、.rodata とか色々と挟まっている。
実行中のプロセスからは, `/proc/self/maps` を読むことで自分のメモリマップを見ることができる。

ちなみに `objdump` で見ると `etext` `edata` `end` 以外に変数 `_start` が定義されており、
これがテキストセグメントの開始アドレスになっているようなので、
`end` と合わせるとセグメントの始まり〜終わりなども読み取れそうである。

ところで、 `etext` は `.text` セグメントの次のアドレスのようなので、
`&etext <= p && p < &edata` というように範囲に含んでいいんじゃないかという気もする(以下はmanページの引用)

```
etext  This is the first address past the end of the text segment (the program code).
edata  This is the first address past the end of the initialized data segment.
end    This is the first address past the end of the uninitialized data segment (also known as the BSS segment).
```

## macOSでの話
macOSでも `size hoge.exe` で `__TEXT` `__DATA` `__OBJC` その他セグメントの情報がみられる。
より詳細な情報は `otool -lV hage.exe` とすると見ることができる。

さて、mruby-3.2.0 では macOS 向けに `getsegmentdata` 関数から得られたアドレスとサイズを使って、
`text + textsize < p && p < data + datasize` というロジックが使われている。(参考 : <https://github.com/mruby/mruby/pull/5885>)

しかしながら macOS では定数はおおむね `__TEXT` セグメントにあり、`__DATA` セグメントをチェックしても発見できない。
Linuxでの実装をそのまま持ってきているためにあまり意味のない実装になってしまっているようだ。
（`__DATA,__const` というセクションもあるようだが、今はあまり使われていない？）
他にも `__DATA_CONST` というセグメントもある。

- 参考1(若干古い資料) : <https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/MachOOverview.html>
- 参考2: <https://developer.apple.com/documentation/xcode-release-notes/xcode-11-release-notes>

mrubyを共有ライブラリ化して実行ファイルを薄くしている場合などには `__DATA` セグメントが空であり、
`getsegmentdata` は `NULL` つまり 0 を戻す。
さらにこの時、 `datasize` に大きな数値が入っており、結果、
`__TEXT` セグメントより後ろのすべてのアドレス(ヒープやスタックも含む)が誤判定により TRUE を返してしまい、
動作に異常を引き起こす…というのが今回の事態の流れのようだ。

## macosでの動作を修正するなら？
セグメント構造体 `segment_command_64` の flags に `SG_READ_ONLY` というのがあるが、これは頼りにならなそう。
とりあえず `__DATA_CONST` と `__TEXT` をチェックするとして、さらに、外部ライブラリまで調査に含めるにはどうすればいいだろうか？

セグメント取得に使える `_mh_execute_header` 変数は、実行ファイルのリンク時に定義されるものなので使えない。
mruby-3.2.0 でも使っている `_NSGetMachExecuteHeader` は dylib 内部でも使用できるが、得られるのは実行ファイルのヘッダなので dylib の検証はできない。
となると、 `_dyld_get_image_header` で使用中のイメージ全てを対象とした走査を行うことになる。
`_dyld_get_image_header` で `mach_header_64` を取り出し、そこから `segment_command_64` を取り出し `cmd` が `LC_SEGMENT_64` のものを全てチェックして最初のセグメントとの `vmaddr` の差を `mach_header_64` に足してセグメント先頭のアドレスを得て、セグメントの終端はそこに `vmsize` を足して…という感じになると思われる。

こんな感じ？ : <https://gist.github.com/kabies/70dee8d9af15862511261016e29ccdf1>

## おまけ
`mrb_ro_data_p` をコールする箇所はざっと見たところ以下のような感じだった。

- `mrb_generate_code` (codegen.c)
  - eval.c で `eval` 系関数からも呼び出す
  - `new_lit_str`(mrbgems/mruby-compiler/core/codegen.c) で IREP_TT_SSTR か IREP_TT_STR か選ぶのに使う
  - IREP_TT_STR の場合 `mrb_irep_free` で解放されるようだが、誤判定した際どういった影響あるかは不明。
  - 動作を見た感じ、そもそもここでRO判定されることがあまりない？
- `mrb_proc_read_irep` (load.c)
  - read_irep に対して `FLAG_SRC_STATIC` を渡すか `FLAG_SRC_MALLOC` を渡すか決める
  - `mrb_load_irep` から使用されるので、ヘッダファイル化したコードの実行時などに通る。
- `mrb_str_new_cstr`
  - `RSTRING_EMBED_LEN_MAX` より短い文字列の場合は構造体に直接入るため判定されない
  - str_new (string.c) で `str_init_nofree` を使うかどうかの判定に使う。
- `mrb_intern_cstr`
  - `find_symbol` で既存のシンボルが発見された場合は判定されない
  - sym_intern (symbol.c) で、ポインタをそのまま使う or mallocするかの判定に使う
