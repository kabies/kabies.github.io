---
layout: post
title:  "macOS向け .app 形式での配布"
date:   2018-03-15 11:04:29 +0900
categories: bismuth macOS
---

macOS向けにアプリを配布する場合bundle(.app)にしたい。

[SDL](https://www.libsdl.org/)はframeworkを配布してるのだが、SDL2_imageなどを使おうとすると、
ヘッダファイルのパスが一般的なUNIX環境と変わってしまうため、 `#include <SDL2_image.h>` とかやると失敗する。

Linuxの場合やMacPortsでSDLを入れた場合なんかは、だいたい `/usr/include/SDL2/` みたいなとこに入り `sdl2-config --cflags` で
よしなにインクルードパスを渡してくれるわけだが、フレームワークを使う場合、 `フレームワーク名/hoge.h` つまり `SDL2_image/SDL2_image.h`
といったパスになってしまう。

というわけで、 `-I/path/to/framework/SDL2_image.framework/Headers/` のようにインクルードパスはフレームワークごとに渡すことにした。

あと、実行ファイルからフレームワークへのリンクを、 `install_name_tool` で `@rpath` から `@executable_path` を使ったものに変更してやる必要がある。
ちゃんと変更されてるか、妙なものとリンクしてないかなどは `otool -L mruby` で確認できる。

`mirb` で使ってる `ncurses` などに `mruby` もリンクしてるのがあまり適当でない感じがするので、 `mirb` のビルドは分けた方が良いかもしれない…。
