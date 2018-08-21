---
title: "emscripten で WebAssembly にした mruby の動作速度を見る"
date: 2018-08-22 00:00:00 +0900
---

emscripten で WebAssembly にした mruby で、シューティングやアクションでの当たり判定で行うような、矩形の衝突判定をやってみる。

矩形の配列を渡すと衝突しているものを探してくれるメソッドを、 全部mruby / C+mrb_iv_get / ほぼC の三パターンで実装して比べる。

弾と敵を想定した矩形をつくって、弾の方を配列として渡し、命中した弾は以後の判定から除いている。

MacBook Air(11inch, i7@1.7GHz)とiPhone7で1000発の弾と100体の敵の判定をやらせた結果が以下。

| browser       | all mruby | C with mrb_iv_get | almost C |
|---------------|-----------|-------------------|--------|
| Safari@iPhone | 922 msec  | 15 msec           | 3 msec |
| Safari@macOS  | 861 msec  |  9 msec           | 2 msec |
| Firefox       | 156 msec  |  7 msec           | 1 msec |

Firefox非常に速い。Safari遅い。デスクトップ版と大差ないiPhone版Safariは偉い気もする。

数を逆（弾が100発、敵が1000体）にしたのが以下。

| browser       | all mruby | C with mrb_iv_get | almost C |
|---------------|-----------|-------------------|---------|
| Safari@iPhone | 822 msec  | 21 msec           | 24 msec |
| Safari@macOS  | 880 msec  | 27 msec           | 23 msec |
| Firefox       | 144 msec  |  8 msec           |  2 msec |

Cの割合が大きいほど悪い影響を受けているぽいが、mruby側でのループ回数が増えるためであろうか。

弾100発、敵100体と少なめにしたのが以下。

| browser       | all mruby | C with mrb_iv_get | almost C |
|---------------|-----------|-------------------|--------|
| Safari@iPhone | 218 msec  |  4 msec           | 1 msec |
| Safari@macOS  | 216 msec  |  3 msec           | 2 msec |
| Firefox       |  44 msec  |  2 msec           | 1 msec |

ネイティブで実行するとこんな感じ。

| enemies | bullets | all mruby | C with mrb_iv_get | almost C |
|---------|---------|-----------|-------------------|----------|
|  100    | 1000    | 23.7 msec | 3.0 msec          | 0.3 msec |
| 1000    |  100    | 18.5 msec | 2.6 msec          | 0.4 msec |
|  100    |  100    |  5.2 msec | 0.8 msec          | 0.1 msec |

ネイティブでも Emscripten & WebAssembly でも、オブジェクト数が相応にある場合、大部分をCで処理しないとキツいな、という感じである。

コード: <https://github.com/kabies/emscripten-mruby-benchmark>
