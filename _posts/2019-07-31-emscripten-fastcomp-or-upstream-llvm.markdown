---
title: "emscripten のバックエンドを fastcomp から llvm へ"
date: 2019-07-31 00:00:00 +0900
---

emscriptenでは現在、WebAssembly出力のバックエンドとして、 fastcomp を使う `latest` の他に、 LLVM を使う `latest-upstream` が選択できる。
将来的にデフォルトはllvmにしていく様子。

>Emscripten can currently (April 2019) use 2 backends to generate WebAssembly: fastcomp (the asm.js backend, together with asm2wasm) and the upstream LLVM wasm backend.
>
>Fastcomp is currently the default, but we hope to switch the default soon to the upstream backend.

（[Building to WebAssembly — Emscripten 1.38.39 documentation](https://emscripten.org/docs/compiling/WebAssembly.html#backends) より）

開発中のコードで試してみたところ、fastcompで最適化レベルを `-O1` 以上にすると、iOSでのみノードのscale設定時に値が0になる、という奇妙な問題が発生した。
 `-O0` を指定して最適化を無効にして出力すると解消するが今度は動作がかなり遅く、さらに不安定で、iOSではページがクラッシュしてしまう。

llvmで出力したものは高速かつ安定しているのだが、こちらには `va_list` に複数要素を含む構造体を渡すと値が壊れるバグ
[Invalid pointer/value from variadic function argument with aggregate variable type #9042](https://github.com/emscripten-core/emscripten/issues/9042)
があり、mrubyの
[mrb_funcall](https://github.com/mruby/mruby/blob/2.0.1/src/vm.c#L384)
が正常に動作しない問題があった。

今後デフォルトがllvmになるということもあるので、とりあえず `latest-upstream` を選択して開発をすすめることにする。
`mrb_funcall` の使用箇所については現状それほど多くないので、個別に対応することに。
