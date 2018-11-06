---
title: "Retinaディスプレイで OpenGL や WebGL の表示がぼやける"
date: 2018-11-07 00:00:00 +0900
---

Retinaディスプレイに SDL+OpenGL でレンダリングしていると表示がぼやける。

![macos-no-care]({{ "/images/2018-11-06/macos-no-care.png" | absolute_url }})

この場合 `SDL_CreateWindow` に `SDL_WINDOW_ALLOW_HIGHDPI` を渡すと以下のように綺麗になる。

![macos-sdl]({{ "/images/2018-11-06/macos-highdpi.png" | absolute_url }})

emscripten + SDL + WebGL の環境でも同様に動作する。
