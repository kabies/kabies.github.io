---
title: "GitHub Actions での macOS 11 Big Sur 向け arm64/x86_64 ユニバーサルバイナリのコンパイル"
date: 2021-04-26 00:00:00 +0900
---

M1チップ（Apple Silicon）搭載 macOS では、 Rosetta 2 で x86_64 向けバイナリを実行可能（アクティビティモニタでアーキテクチャの列に Intel と表示される）となっている。

ただし、ライブラリは x86_64 でメインプログラムは arm64 といったちゃんぽんはできないようなので、SDLなどのライブラリはユニバーサルバイナリ（fatバイナリ）にしておきたい。

[SDL公式](https://www.libsdl.org) が配布しているバイナリでは、 [SDL](https://www.libsdl.org/download-2.0.php) はfatになっているものの [SDL_image](https://www.libsdl.org/projects/SDL_image/) や [SDL_mixer](https://www.libsdl.org/projects/SDL_mixer/) がユニバーサル化されていなかったため、自力でコンパイルすることに。

検証・実験に [GitHub Actions](https://docs.github.com/en/actions) の利用を検討したところ、現状 Intel かつ macOS 10.15 までの環境しか使用できないようだ。（ <https://github.com/actions/virtual-environments> ）

ただ、新しめのバージョンの Xcode が導入されているので、ユニバーサルバイナリのコンパイル事態は可能そう…ということで試してみた。

# fatバイナリ化

コンパイルとリンクで `-arch arm64 -arch x86_64` を指定すればfatになる。

あるいは、それぞれのアーキテクチャ向けにコンパイルしたライブラリやバイナリを `lipo` コマンドで合体させてもよい。

# SDL と mpg123 のfat化

SDL,SDL_image,SDL_mixerについては `LDFLAGS` と `CFLAGS` に前述のアーキテクチャ指定をすればok。

SDL2_mixer で mp3 再生をしたいので [mpg123](https://mpg123.org) もfat化したが、
こちらはアーキテクチャ別の最適化を行う都合上、別々にコンパイルしたライブラリを後で `lipo` コマンドで合体させた。

arm64 向けのコンパイルは、クロスコンパイル用に `--host=x86_64-apple-darwin --build=aarch64` と指定して成功。
M1 macOS の target triple は、 `gcc -dumpmachine` によれば `arm64-apple-darwin20.3.0` だったが、これを build に指定してもうまくいかないようだ。

arm向けの最適化指定は `--with-cpu=aarch64` を使った。これは `neon64` か `fpu` かを動的に選ぶ、らしい。

# GitHub Actions でのビルド

前述の通り macOS 11 が使えないため Catalina に Xcode 12.4 という環境ではあるが、
`SDKROOT` 環境変数に `/Library/Developer/CommandLineTools/SDKs/MacOSX11.1.sdk` と指定することで Big Sur 向けのSDKが利用可能だった。

という感じでできたものがこちら <https://github.com/bismite/SDL-macOS-UniversalBinaries>

# おまけ：mruby

mruby3.0で導入された [Preallocate Symbols](https://github.com/mruby/mruby/blob/3.0.0/doc/guides/symbol.md)
に関連してプリプロセッサの一次コンパイルが挟まるようで、これが `-arch arm64 -arch x86_64` のアーキテクチャ同時指定で失敗する。

なので、クロスコンパイルで arm64 と x86_64 それぞれをビルドして `lipo` で合体させるとうまくいく。
