---
title:  "UTF-8 を有効にすると mruby-io の read の動作がおかしくなる"
date:   2018-04-02 18:00:00 +0900
categories: mruby
---

`MRB_UTF8_STRING` を指定して `mruby-io` を使うと、正常にデータを読み込めない。

おそらく [このへん](https://github.com/mruby/mruby/blob/1.4.0/mrbgems/mruby-io/mrblib/io.rb#L209) が原因で、
`read` メソッド内で文字列のサイズを `bytesize` でなく `size` で読んでいるため、IOから読んだ実際のバイト数と齟齬が起きているぽい。

たぶん `read` を使うと `tell` や `pos` もちゃんと動かないはず。代わりに `sysread` すると良い。

## 参考

`MRB_UTF8_STRING` を有効にした状態で試すと以下のように、sizeは文字数を返す

```
> "あいうえお".size
 => 5
> "あいうえお".bytesize
 => 15
```

`MRB_UTF8_STRING` を有効にしてないとこんなん↓

```
> "あいうえお".size
 => 15
> "あいうえお".bytesize
 => 15
```
