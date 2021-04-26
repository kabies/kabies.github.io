---
title: "emscripten/Linux/macOS/mingw-w64 での GLEW を使わない OpenGL 拡張機能のロード"
date: 2021-04-25 00:00:00 +0900
tags: macOS SDL OpenGL WebGL
---

# 前提（emscriptenとGLEW）

emscripten では、 OpenGL ES 2.0 ベースのコードを WebGL 1.0 で動作させる（Safari で WebGL 2.0 が動作しないため）

emscripten の GLEW サポートは内部で SDL に依存しており、実際には `SDL_opengl.h` に `GL_GLEXT_PROTOTYPES` を追加してそれらしい動作を実現している様子。

OpenGL ES で GLEW を使用する場合、 `GLEW_USE_LIB_ES11` あるいは `GLEW_USE_LIB_ES20` を指定すると、
`glew.h` から `SDL_opengles.h`, `SDL_opengles2.h` がロードされるようになっている。

そもそもオリジナルの GLEW は OpenGL ES をサポートしないようなので、emscriptenでポートという観点からは GLEW を使わないのが正攻法という気もする。

かわりに [SDL_GL_ExtensionSupported](https://wiki.libsdl.org/SDL_GL_ExtensionSupported) や
[SDL_GL_GetProcAddress](https://wiki.libsdl.org/SDL_GL_GetProcAddress) を使うのが良いだろう。

# GLEWを使用しない場合のプラットフォーム別対応

LinuxおよびmacOS環境では `GL_GLEXT_PROTOTYPES` を有効にしてコンパイル、拡張機能は `SDL_GL_GetProcAddress` でロードする。

mingw-w64 では `GL_GLEXT_PROTOTYPES` を有効にすると `glActiveTexture` などがリンク時に失敗するため、
拡張機能以外であっても必要な関数は定義だけしておき `SDL_GL_GetProcAddress` を使い動的にロードする。

# インスタンシングで使う拡張機能

## Vertex Array Object

`GenVertexArrays(sizei n, uint *arrays)`, `BindVertexArray(uint array)` を提供する

| OpenGL  |           name            | url |
|---------|---------------------------|-----|
| 1.1     | APPLE_vertex_array_object | <https://www.khronos.org/registry/OpenGL/extensions/APPLE/APPLE_vertex_array_object.txt> |
| 2.1     | ARB_vertex_array_object   | <https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_vertex_array_object.txt> |
| ES 1.1  | OES_vertex_array_object   | <https://www.khronos.org/registry/OpenGL/extensions/OES/OES_vertex_array_object.txt> |
| Web 1.0 | OES_vertex_array_object   | <https://www.khronos.org/registry/webgl/extensions/OES_vertex_array_object/> |

## Instanced Arrays

`VertexAttribDivisor(uint index, uint divisor)` を提供する


| OpenGL  |           name         | url |
|---------|------------------------|-----|
| 1.1     | ARB_instanced_arrays   | <https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_instanced_arrays.txt> |
| ES 2.0  | EXT_instanced_arrays   | <https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_instanced_arrays.txt> |
| ES 2.0  | ANGLE_instanced_arrays | <https://www.khronos.org/registry/OpenGL/extensions/ANGLE/ANGLE_instanced_arrays.txt> |
| Web 1.0 | ANGLE_instanced_arrays | <https://www.khronos.org/registry/webgl/extensions/ANGLE_instanced_arrays/> |

## Draw Instanced

`DrawArraysInstanced(enum mode, int first, sizei count, sizei primcount)` を提供する

| OpenGL  |           name         | url |
|---------|------------------------|-----|
| 2.0     | ARB_draw_instanced     | <https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_draw_instanced.txt> |
| 2.0     | EXT_draw_instanced     | <https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_draw_instanced.txt> |
| ES 2.0  | EXT_draw_instanced     | <https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_draw_instanced.txt> |
| ES 2.0  | EXT_instanced_arrays   | <https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_instanced_arrays.txt> |
| ES 2.0  | ANGLE_instanced_arrays | <https://www.khronos.org/registry/OpenGL/extensions/ANGLE/ANGLE_instanced_arrays.txt> |
| Web 1.0 | ANGLE_instanced_arrays | <https://www.khronos.org/registry/webgl/extensions/ANGLE_instanced_arrays/> |
