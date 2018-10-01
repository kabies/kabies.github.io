---
title: "mruby を emscripten で WebAssembly にする場合の例外の設定"
date: 2018-10-01 00:00:00 +0900
---

mruby の例外は C の setjmp/longjmp を使うものと C++ の例外を使う実装がある。
これは基本的には C++ のための機能で、つまり setjmp/longjmp を使うと C++ のデストラクタが発動しないためにそうなっている。

emscripten は setjmp/longjmp と C++ の例外の双方を一応サポートしているが、どっちも推奨されていない。

>Code that uses low-level features of the native environment, for example native stack manipulation in conjunction with setjmp/longjmp (we support proper setjmp/longjmp, i.e., jumping down the stack, but not jumping up to an unwound stack, which is undefined behavior).

from: <http://kripken.github.io/emscripten-site/docs/porting/guidelines/portability_guidelines.html#code-that-cannot-be-compiled>

>Catching C++ exceptions (specifically, emitting catch blocks) is turned off by default in -O1 (and above).

from: <http://kripken.github.io/emscripten-site/docs/optimizing/Optimizing-Code.html#other-optimization-issues>

WebAssembly にした mruby での setjmp/longjmp を使った例外は、コードの規模が大きくなってくると動作が怪しくなる。
デスクトップの Firefox と Safari では大丈夫だったのだが、 Safari@iOS だと実行中に突然Webページがクラッシュする。
例外処理を全然使ってないコードであっても発生する。
（SafariのWebページのクラッシュ、ログが残らないようで、調査がしづらい…）

mruby の例外の実装を、 C++ の例外をつかったものにすると、 Safari@iOS でも動作が安定した。
これには `build_config.rb` で `conf.enable_cxx_exception` を指定する。

さらに、 emscripten はデフォルトだと C++ の例外を動作させないので、 `emcc` に `-s DISABLE_EXCEPTION_CATCHING=0` オプションを渡す。
mruby で例外が起きた途端に死ぬことになるが、それでも構わないのなら、C++の例外を無効化したままだと、より安定しそうな気もする。

WebAssemblyに例外の導入が検討されているようなので、実現したらこの辺もちょっと変わるのかもしれない。
