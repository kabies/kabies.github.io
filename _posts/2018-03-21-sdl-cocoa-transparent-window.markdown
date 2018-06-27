---
title:  "SDL 2.0.8 + macOS で半透明ウインドウ"
date:   2018-03-21 21:00:00 +0900
categories: macOS SDL minimeter mruby cocoa
---

SDLのバージョンを2.0.8へ上げたところ、 [mruby-sdl2-cocoa](https://github.com/mruby-sdl2/mruby-sdl2-cocoa)
で半透明ウインドウが出せなくなっていた。

SDLのこの変更 <https://hg.libsdl.org/SDL/changeset/64ae77236054> で、ViewのdrawRectが自分を黒く塗りつぶすようになっているのが原因。

メソッドの差し替えで対応したのだが、このSDLViewが非公開クラスなので若干ややこしい。
まず `NSClassFromString` で非公開クラスを取得して `class_addMethod` で自前のメソッドを追加し、
そのあと `method_exchangeImplementations` で元のメソッドと入れ替えるという手順。

ともかくうまくいったので、サンプルも兼ねて [minimeter](https://github.com/kabies/minimeter) を公開した。
