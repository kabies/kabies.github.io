---
title:  "emscripten, SDL2, OpenGL, WebGL, instanced rendering!"
date:   2018-06-28 05:00:00 +0900
categories: emscripten SDL OpenGL WebGL
---

# SDL2 のスプライト描画はもうちょい速くならないか？

SDL2 で愚直に大量のスプライト描画すると結構遅い。

OpenGLでの描画システムをゼロから作ることになるが、 instanced rendering を使えば速くなるのでは！というアイデアである。
（メモ：整形してあとでコード公開すること）

いくつかの制限はあるものの実際うまくいった。ついでにWebGLにも対応した。

WebGL 版の性能を iPhone7 と MacBook Air 11inch (Intel HD Graphics 5000) で測ってみた。

| Deviece        | 16,384 sprites | 32,768 sprites |
|----------------|----------------|----------------|
| Safari@iPhone7 | 60FPS          | 40FPS          |
| Safari 11.1.1  | 60FPS          | 30FPS          |
| Firefox 61.0   | 55FPS          | 30FPS          |

iPhone7 の WebGL 1.0 でこんなに性能出ると思わずびっくり。何かの間違いではないか？

macOS 10.13.4 では [アクティビティモニタでGPU使用率を確認できるようになった](https://support.apple.com/ja-jp/HT208544)ので、
眺めてみたらものすごい使用率だった。GPUはたらいてる。

同じコードでデスクトップ環境のOpenGLでも動作するが、200FPS以上出ていて超速い。

インスタンシングは普通、3Dのフィールドにある岩とか樹木とか空き瓶みたいな、同じ物体を大量に描画するための機能であり、スプライト描画用ではない。
この辺で若干制限があるので、後に述べる。

しかしこれ インスタンス描画, インスタンス化, instanced rendering, instancing とかいろいろ呼び方あって検索しにくいな…。

# emscripten で WebGL

[emscripten](http://emscripten.org/) で C を WebAssembly に変換する。

OpenGL ES としてコードを書くと、だいたい WebGL に変換されて動作する。
OpenGL / OpenGL ES / WebGL の対応は _だいたい_ 以下の通り。

| OpenGL     | OpenGL ES |     WebGL |
|------------|-----------|-----------|
| OpenGL 2.X |    ES 2.0 | WebGL 1.0 |
| OpenGL 3.X |    ES 3.0 | WebGL 2.0 |

OpenGL ES 3.0 の機能を使うと、当然 WebGL 2.0 になる。
WebGL 1.0 までしか対応していないブラウザ（Safari@macOSとか、Safari@iOSとか）で動かしたい場合は、OpenGL ES 2.0 相当のコードを書く。

OpenGL ES 2.0 と 3.0 の違いはいくつかあるが、大きいのは以下のようなとこだろうか。

 - VAO がない
 - glDrawArraysInstanced がない（インスタンス描画機能がない）
 - GLSL のバージョンが古い
   - in/out ではなく attribute/varying, layoutもない
   - attributeに整数値を渡せない（uniformはいける）
   - switch-case 構文がない
   - gl_VertexID がない

OpenGL 2.X / OpenGL ES 2.0 / WebGL 1.0 の世代であっても、 VAO とインスタンス描画については、OpenGL拡張機能によって有効にできる。

# 拡張機能によるインスタンシング

emscripten には [GLEW](https://github.com/nigels-com/glew) が含まれているので、
拡張機能は（使用中のデバイスでサポートされていれば）簡単に利用できる。

OpenGL ES 3.0 でいう VAO, VertexAttribDivisor, DrawArraysInstanced あたりが欲しいわけだが、環境によって微妙に機能提供元が異なる。

ARB に以下の拡張機能があるので、まずはこれら。

 - VAO を提供する [ARB_vertex_array_object](https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_vertex_array_object.txt)
 - VertexAttribDivisor の [ARB_instanced_arrays](https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_instanced_arrays.txt)
 - DrawArraysInstanced の [ARB_draw_instanced](https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_draw_instanced.txt)

 emscripten を使う場合、以下の二つ。

 - VAO は [OES_vertex_array_object](https://www.khronos.org/registry/OpenGL/extensions/OES/OES_vertex_array_object.txt)
 - VertexAttribDivisor / drawArraysInstanced が [ANGLE_instanced_arrays](https://www.khronos.org/registry/webgl/extensions/ANGLE_instanced_arrays/)

さらについでに、 macOS のデスクトップで *普通の* OpenGL で動作させる場合（おそらくiOSもだろう）、VAO は Apple 提供のものを使う。

 - [APPLE_vertex_array_object](https://www.khronos.org/registry/OpenGL/extensions/APPLE/APPLE_vertex_array_object.txt)

というわけで、

```
if(GLEW_ARB_instanced_arrays) printf("ARB_instanced_arrays ok\n");

if(GLEW_ARB_draw_instanced) printf("ARB_draw_instanced ok\n");

if(GLEW_ARB_vertex_array_object) printf("ARB_vertex_array_object ok\n");

if(GLEW_ANGLE_instanced_arrays) printf("ANGLE_instanced_arrays ok\n");

if(GLEW_OES_vertex_array_object) printf("OES_vertex_array_object ok\n");

if(GLEW_APPLE_vertex_array_object) printf("APPLE_vertex_array_object ok\n");
```

こんな感じである。さらに関数名も微妙に違ったりするのでよしなにラッピングして使うのがよかろう。


## 制限と対策

 1. インスタンス描画中にテクスチャを切り替えることはできない
 2. インスタンス毎に値を渡すことはできるが、インスタンスの頂点毎に値を渡す機能が（たぶん）ないので、UV値を個別に指定できない
 3. Blending を切り替えられない

### 1. テクスチャ切り替え

テクスチャの切り替えはできない。

切り替えできないが、テクスチャ座標さえ送れるなら [テクスチャアトラス](https://en.wikipedia.org/wiki/Texture_atlas) を使えば良い。
テクスチャ座標が個別に送れるかどうかについては、次の節で述べる。

さらに、テクスチャユニットが少なくとも [8個くらいある](https://webglstats.com/webgl/parameter/MAX_TEXTURE_IMAGE_UNITS) ので、
各ユニットに別々の画像をセットしておけば、8枚は選択できることになる。

テクスチャサイズはだいたい [4096px くらいまで使える](https://webglstats.com/webgl/parameter/MAX_TEXTURE_SIZE)ので、
4096x4096x8 からテクスチャを選択できるようになる。十分じゃなかろうか。

### 2. 頂点毎に情報を送りたい

たぶんできない。

ただ、OpenGL 3.0 / ES 3.0 のシェーダーでは、 [gl_VertexID](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/gl_VertexID.xhtml) で、
いま何番目の頂点か把握できる。

これさえあれば、`vec4(Left,Right,Bottom,Top)` みたいな vec4 としてインスタンス毎にテクスチャ座標（UV値）を渡しておいて、シェーダー内で参照すればいい。

OpenGL 2.0 / ES 2.0 世代のシェーダーでは `gl_VertexID` が使えないので、自力で頂点の番号を送らねばならない。

インスタンシングでは、 [VertexAttribDivisor](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glVertexAttribDivisor.xhtml) で
`divisor` に `0` を指定したものが、インスタンスの雛形になる。
普通はこれが単純に3Dモデルの頂点になるわけだが、ここで（若干間抜けではあるが） `0,1,2,3` という情報を渡しておけば、頂点の番号を知ることができる。

こうして頂点の番号とテクスチャ座標の配列から、テクスチャアトラスを使うことができるようになる。

なんかもうちょっといい方法ないもんだろうか…

### 3. ブレンディング

一回のドローコール (`DrawArraysInstanced`) 内でブレンディングを切り替えることはできない。諦める。

別の `DrawArraysInstanced` を呼べば、その際にはブレンディングを変更できる。
なので、例えば、

 - `glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);` で通常のブレンディングに変更
 - `DrawArraysInstanced` で背景やキャラクターを描画
 - `glBlendFunc(GL_SRC_APLHA, GL_ONE);` でaddtiveなブレンディングに変更
 - `DrawArraysInstanced` で炎やキラキラしたパーティクルを描画

みたいな感じになる。

## おまけ

使用中のブラウザのWebGLのパラメータ（テクスチャの最大サイズとか）を知りたい場合、 <http://webglreport.com> をみると良い。

パラメータの統計データ（テクスチャサイズ4096がどれくらいの割合で使えるか？とか）を知りたい場合、 <https://webglstats.com> をみると良い。

