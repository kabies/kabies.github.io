---
title: "Retinaディスプレイで OpenGL や WebGL の表示がぼやける"
date: 2018-11-07 00:00:00 +0900
---

Retinaディスプレイに SDL+OpenGL でレンダリングしていると表示がぼやける。

![macos-no-care]({{ "/images/2018-11-06/macos-no-care.png" | absolute_url }})

この場合 `SDL_CreateWindow` に `SDL_WINDOW_ALLOW_HIGHDPI` を渡すと以下のように綺麗になる。

![macos-sdl]({{ "/images/2018-11-06/macos-sdl.png" | absolute_url }})

`SDL_WINDOW_ALLOW_HIGHDPI` を指定していても、 emscripten + SDL + WebGL には影響せずぼやける。
以下はsafariとfirefoxでのレンダリング結果。

![safari_macos-sdl]({{ "/images/2018-11-06/safari_macos-sdl.png" | absolute_url }})
![firefox-sdl]({{ "/images/2018-11-06/firefox-sdl.png" | absolute_url }})

`SDL_CreateWindow` で２倍サイズのウインドウを作ってレンダリングし、 `canvas` のサイズを半分にすると、以下のように綺麗になる。

![safari_macos-sdl-x2]({{ "/images/2018-11-06/safari_macos-sdl-x2.png" | absolute_url }})
![firefox-sdl-x2]({{ "/images/2018-11-06/firefox-sdl-x2.png" | absolute_url }})
