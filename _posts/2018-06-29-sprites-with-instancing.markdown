---
title:  "Instancing を使ったスプライト描画とシーングラフ"
date:   2018-06-29 08:00:00 +0900
tags: emscripten SDL OpenGL WebGL
---

[承前]({{ site.baseurl }}{% link _posts/2018-06-28-emscripten-webgl-instancing.markdown %})

OpenGL で Instancing を使った高速なスプライト描画の実験用のコードを公開した <https://github.com/kabies/sprite-instancing>

ついでに動作するWebGL版をここに置いておく（重いので注意）

 - Instancing
   - [1024 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_1024.html %})
   - [2048 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_2048.html %})
   - [4096 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_4096.html %})
   - [8192 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_8192.html %})
   - [16384 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_16384.html %})
   - [32768 sprites]({{ site.baseurl }}{% link sprite-instancing/instancing_32768.html %})
 - SDL
   - [1024 sprites]({{ site.baseurl }}{% link sprite-instancing/sdl_1024.html %})
   - [2048 sprites]({{ site.baseurl }}{% link sprite-instancing/sdl_2048.html %})
   - [4096 sprites]({{ site.baseurl }}{% link sprite-instancing/sdl_4096.html %})

MacBook Air 11inch (Intel HD Graphics 5000)での参考値はこんなものであった。

| sprites | SDL@Firefox | SDL@Safari | instancing@Firefox | instancing@Safari |
|---------|-------------|------------|--------------------|-------------------|
|    1024 | 60          | 60         |                    |                   |
|    2048 | 35          | 33         |                    |                   |
|    4096 | 20          | 20         | 60                 | 60                |
|    8192 |             |            | 60                 | 60                |
|   16384 |             |            | 55                 | 60                |
|   32768 |             |            | 30                 | 32                |


`texture_z` としてテクスチャの指定を渡しているのは、当初テクスチャ配列を使おうと思っていた名残。
OpenGL ES 2.0 / WebGL 1.0 にテクスチャ配列が見当たらなかったのでテクスチャユニットを代替にしている。

このバージョンでは、シェーダー内でスプライトの座標変換を行なっている。
シーングラフベースのシステムを組む場合、これをC側に持って来る必要があり、行列と三角関数の計算でCPU負荷が上がる。

実際シーングラフを実装してみたところ、8,192スプライトで160FPS、16,384スプライトで100FPS 程度まで性能が落ちた。

| sprites | in shader  | in cpu |
|---------|------------|--------|
|    4096 |            | 190FPS |
|    8192 | 250FPS     | 160FPS |
|   16384 | 200FPS     | 100FPS |

CPU負荷の問題は WebGL+WASM でさらに顕著で、例えば角度計算が入るかどうかによってかなり負荷が変わる。
角度が全て0度の場合と、全スプライトを10度傾けた場合の比較が以下。

| sprites | 0 deg      | 10 deg |
|---------|------------|--------|
|    2048 | 60FPS      |  60FPS |
|    4096 | 60FPS      |  35FPS |
|    8192 | 35FPS      |  18FPS |

この辺はWASMにSIMDが導入されると変わりそう。

ここにさらにmrubyでロジックを実装できるようにしたら、結構遅くなりそうだなぁ…。
