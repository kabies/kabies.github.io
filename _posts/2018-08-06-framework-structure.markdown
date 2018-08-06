---
title:  "フレームワークの構成について"
date:   2018-08-06 23:00:00 +0900
tags: bismite
---

bismuth というプロジェクトを見つけてしまったので名称を bismite に変えようかな…

## エンジン本体の構成

- bismite-core
  - C製
  - レンダリングとイベント処理
- bismite-ext
  - C製
  - フォント関連、アクションなどcoreから切り離し可能なもの
- mruby-bismite-core
  - coreのmrubyバインディング
- mruby-bismite-ext
  - extのmrubyバインディング

## 周辺ツール

- bismite-packager
  - 各プラットフォーム用の配布パッケージを作る
    - macOS Application Bundle
    - emscripten ( for browser, WASM & WebGL )
    - mingw-w64
- bismite-sprite-sheeter
  - スプライトシート（テクスチャアトラス）を作る
- bismite-font-sheeter
  - フォントのスプライトシートを作る
