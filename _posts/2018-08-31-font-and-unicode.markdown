---
title: "ビットマップフォントとUnicodeとcairo"
date: 2018-08-31 00:00:00 +0900
---

ビットマップフォントを使いたい。

 - pros
   - たぶん速い
   - グラデーションや縁取りで飾り付けできる
   - 複数のフォントの合成、あるいはsubstitutionがしやすい
   - オリジナル文字とか、絵文字も使いやすいかも
   - 依存ライブラリがほとんど不要
 - cons
   - 漢字を含めるとテクスチャサイズがすごくデカい
   - フォントサイズを綺麗に自由に変えられない
     - 多少汚くて良いなら拡大縮小で対応可能ではある

ビットマップフォントを作るプログラムはすでにいくつか存在するのだが、
以下のような条件を満たす良い感じのが見つけられなかったので自分で作ろうかな…。

 - substitution や合成ができる
 - SMP (Supplementary Multilingual Plane) の Unicode を扱える
 - 4096x4096 以上のサイズに対応
 - グリフ数が多くても平気
 - グラデーション、アウトラインその他の飾り付けが可能
 - コマンドラインで操作できる
 - FOSS

参考：ビットマップフォントを作るプログラム

 - [Bitmap Font Generator](http://www.angelcode.com/products/bmfont/)
 - [bmGlyph](https://www.bmglyph.com)
 - [Glyph Designer](https://www.71squared.com/glyphdesigner)


## テクスチャサイズについて

[JIS X 0208](https://en.wikipedia.org/wiki/JIS_X_0208) に6879グリフが含まれている。 [ユニコードとの対応](http://www.unicode.org/Public/MAPPINGS/OBSOLETE/EASTASIA/JIS/JIS0208.TXT) も参照。
これとASCIIやいくつかの記号をあわせて7000グリフとして、32pxのフォントでレンダリングすると、 `32×32×7000=7,168,000px` となる。
テクスチャサイズ 4096x4096 であれば、 `32×32×7000 / 4096^2 ≒ 0.43` というわけで四割強程度の占有率に収まる、はず。

参考: [JIS X 0208の漢字リスト](https://en.wiktionary.org/wiki/Index:Japanese_kanji_by_JIS_X_0208_kuten_code)

## レンダリング

### FreeType

[FreeType](https://www.freetype.org) を直接使うのは、飾り付け等を考えると結構大変そうである。
絵文字の場合分けをして、アウトラインは `FT_Glyph_Stroke` で作って `FT_Render_Glyph` でレンダリングして、
あとはグラデーションをなんとかして…という感じであろうか。

### SDL_ttf

[SDL_ttf](https://www.libsdl.org/projects/SDL_ttf/) は version2.0.14 で確認したとこ、以下二点が厳しい。

 1. フォントによってはレンダリング結果がクロップされた状態になる
 2. 4バイト使う Supplementary な領域のグリフを扱えない

(1)については、例えば "ᾆ" (`U+1F86`) のようなダイアクリティカルマーク盛り盛りのグリフなんかで上下が切れてしまう場合がある。
以下は [Miguフォント](http://mix-mplus-ipa.osdn.jp/migu/) をレンダリングしたもの。
上は macOSのレンダリング結果、下は SDL_ttf を使ったもので、ダイアクリティカルマークが結構途切れてしまっている。

![cropped]({{ "/images/2018-08-31-cropped.png" | absolute_url }})

(2)は例えば "👺" (`U+1F47A`) のようなカラー絵文字だとか、ヒエログリフだとかを表示できない。

### cairo

というわけで FreeTypeのフロントエンドとして [cairo](https://cairographics.org) を使ってみる。

以下は [mplusフォント](https://mplus-fonts.osdn.jp) をレンダリングしたもの。
`cairo_pattern_t` を使ったグラデーションと `cairo_stroke` を使ったアウトライン入り。

![rendered]({{ "/images/2018-08-31-font-rendered.png" | absolute_url }})

以下は [Notoフォント](https://www.google.com/get/noto/) の Noto Color Emoji を `cairo_show_glyphs` でレンダリングした結果。

![emoji]({{ "/images/2018-08-31-emoji.png" | absolute_url }})

cairoを使うとbdfフォントなんかもほぼ透過的にハンドリングしてくれる。 [Gohufont](http://font.gohu.org) をレンダリングしたのが以下。

![gohufont]({{ "/images/2018-08-31-gohufont.png" | absolute_url }})

素晴らしい結果なのだが、cairoは標準の配布用バイナリ（*.framework とか *.dll とか）が用意されておらず、
実行ファイルを作って配布しようと思うとそこが少々めんどくさそうである。
最低限 FreeType, libpng, libz, libbz2, pixman, cairo が必要そうかな…？

レンダリングするにあたっては、テキストから UTF-8 の文字を一文字ずつとりだす必要があり、これも自分で実装するとちょっとめんどくさい。
レンダリングするだけなので [Overlong encodings](https://en.wikipedia.org/wiki/UTF-8#Overlong_encodings) なんかは無視しても良いと思う。SDL_ttfは無視してた。
