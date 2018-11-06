---
title: "Retinaディスプレイで OpenGL や WebGL の表示がぼやける"
date: 2018-11-07 00:00:00 +0900
---

Retinaディスプレイに SDL+OpenGL でレンダリングしていると表示がぼやける。

![macos-no-care]({{ "/images/2018-11-06/macos-no-care.png" | absolute_url }})

この場合 `SDL_CreateWindow` に `SDL_WINDOW_ALLOW_HIGHDPI` を渡すと以下のように綺麗になる。

![macos-sdl]({{ "/images/2018-11-06/macos-highdpi.png" | absolute_url }})

emscripten + SDL + WebGL の環境でも同様に動作する。

大きめの解像度 (1280x720で試した) にすると、iPhoneのSafariには負荷が高いのか、Webページがクラッシュしてしまった。
macOS上でも若干負荷がかかっている感じ。
あと、iOSだとそもそもぼやけがあまり気にならなかったので、この辺は設定で切り替えられると良さげだろうか。
