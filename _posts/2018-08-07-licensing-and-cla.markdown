---
title: "ライセンスとCLA"
date: 2018-08-07 21:00:00 +0900
---

# 配布時のライセンス

パッケージャーで諸々のDLLを同梱して配布する場合など、各ライブラリのライセンス表記が必要な場合がある。

`SDL` や `zlib` の [zlib license](https://www.zlib.net/zlib_license.html) なんかは表記不要。

`mruby` 本体はMITライセンス, mgemにもMITライセンスが多い。MITライセンスの場合はライセンス本文と copyright holders の表記がいる。
あと、 `mruby-yaml` のmgem ( [オリジナル](https://github.com/AndrewBelt/mruby-yaml) じゃなくて [mgem-listに入ってる方](https://github.com/hone/mruby-yaml) ) が [libyaml](https://github.com/yaml/libyaml) を内々でダウンロードしてきてビルドして静的リンクしてるみたいなのに注意。この場合 libyaml のライセンス表記も必要。

# 貢献を受ける側としてのライセンス

自分でプロジェクトを主催する場合、配布のためのライセンスと別に CLA (contributor license agreement) について考える必要がある。

`Apache License Version 2.0` [(原文)](http://www.apache.org/licenses/LICENSE-2.0.txt) [(和訳)](https://ja.osdn.net/projects/opensource/wiki/licenses%2FApache_License_2.0)
には一般的なCLAで行われる取り決めの一部（著作権ライセンスと特許ライセンスの付与）も含まれており、
とりあえずこれでいいんじゃないか感がある。

[FSFのライセンス選択ガイド](https://www.gnu.org/licenses/license-recommendations.html) でも結構おすすめされている。
githubの公開している [Open Source Guides](https://opensource.guide) 内の [The Legal Side of Open Source](https://opensource.guide/legal/) も参考になる。

`Apache License Version 2.0` にはコントリビュート内容の正当性（パクリじゃないです的な）を宣言する内容が含まれてない気がするんだけど、ContributionやContributorの定義周りに書いてある内容でカバーされてることになるのかな…？

このへん、Linuxカーネルでは [DCO](https://developercertificate.org) (Developer Certificate of Origin) というのが使われている様子。
[パッチ投稿に関する説明](https://www.kernel.org/doc/Documentation/translations/ja_JP/SubmittingPatches) も参照。
DCOへの同意を実名で宣言しつつメールでパッチ送る、というやり方らしい。

jQueryなどを含む [js foundationのCLA](https://js.foundation/CLA) とかも、内容はLinuxのDCOとそっくり。

[Android](https://source.android.com/setup/start/licenses) も `Apache License Version 2.0` を採用していて、CLAを要求している。
[個人向け](https://cla.developers.google.com/about/google-individual) そしてこっちが
[企業向け](https://cla.developers.google.com/about/google-corporate) のCLA。

[Apacheのライセンスに関するページ](https://www.apache.org/licenses/) には [個人向けCLA](http://apache.org/licenses/icla.pdf) と [企業向け](https://www.apache.org/licenses/cla-corporate.pdf) のがある。


[producing oss](https://producingoss.com) に [CLAについてのセクション](https://producingoss.com/ja/copyright-assignment.html) がある。
FSFが行うような（今もなの？）著作権の譲渡は、他のプロジェクトへ似たようなコードを提供することを困難にするはずなので、コントリビューター側として考えるとCLAの内容はよくチェックしないとまずい感じはある。

どうもCLAの和訳というのがあんまり見当たらない。国内法とのすり合わせがどうとか実例がどうなってるのかとかよくわからん雰囲気。

あんまこの辺でハードル上げてくのはFLOSSに好意的な人たちにとって本意ではなかろうなという気もするので、ほどほどが良いと思うんだが…
[Open Source Guides](https://opensource.guide) でも [Does my project need an additional contributor agreement?](https://opensource.guide/legal/#does-my-project-need-an-additional-contributor-agreement) に対して "Probably not." と書かれている。

貢献を受けた際、コントリビューターの名前をどう扱うのが標準的かというのもよくわからない。
`Apache License Version 2.0` 採用プロジェクトをちょろっと調べてみたが、コントリビューター一覧的なものは必ずしも含まれてない。機械的にやってるだけ（でかいプロジェクトだとザラに1000人超えてたりするしなあ）のパターンも多いようなので、gitのログがあればとりあえず良さそうかなあ…。

- LICENSE ファイルにコピーライト表記自体してない
  - [apache spark](https://github.com/apache/spark/blob/master/LICENSE)
    - ミラーなのでどっか別で管理されているのかもしれないが、リポジトリ内には貢献者リストはないぽい
    - PR見ると `Authored-by` や `Signed-off-by` といったLinuxのDCOめいた文言があるのでそういったプロセスで進められているんだろうたぶん
  - [kubernetes](https://github.com/kubernetes/kubernetes/blob/master/LICENSE)
    - [CHANGELOG](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.2.md) がかなり詳細で貢献者名もここから追える感じではあるが…
  - [google/material-design-icons](https://github.com/google/material-design-icons/blob/master/LICENSE)
  - [apple/swift](https://github.com/apple/swift/blob/master/LICENSE.txt)
- LICENSE ファイルにプロジェクトや企業の名前でcopyright表記している
  - `Copyright 2018 The TensorFlow Authors.` [TensorFlow](https://github.com/tensorflow/tensorflow/blob/master/LICENSE)
  - `Copyright 2013-2017 Docker, Inc.` [moby](https://github.com/moby/moby/blob/master/LICENSE)
    - [AUTHORS](https://github.com/moby/moby/blob/master/AUTHORS) にgitのログから機械的に生成された一覧がある

MITライセンスだがjQueryは `Copyright JS Foundation and other contributors, https://js.foundation/` であり、
[AUTHORS.txt](https://github.com/jquery/jquery/blob/master/AUTHORS.txt) に全員載せてる様子。これもgitのログから機械的にとりました感ある。

前述の [Open Source Guides](https://opensource.guide) の [Building Welcoming Communities](https://opensource.guide/building-community/#share-ownership-of-your-project) では以下のように書かれている。

> Start a CONTRIBUTORS or AUTHORS file in your project that lists everyone who’s contributed to your project, like Sinatra does.


ライセンスとコントリビューターといえば、 [OpenSSLがApacheライセンス2.0へのライセンス変更を試みた件](https://license.openssl.org) 、変更自体はアナウンスされてるはずだけど、
[ブログ](https://www.openssl.org/blog/blog/categories/license/) とか見るとまだ完了してないんだろうか。
googleがフォークした [BoringSSL](https://github.com/google/boringssl) ではやはりgoogleのCLAを要求してるようだ。 (https://github.com/google/boringssl/blob/master/CONTRIBUTING.md)
[LibreSSL portable](https://github.com/libressl-portable/portable) だとOpenBSDのMLにパッチを送れ、と書いているので、そっちでCLAをあれこれするのであろうか。

（ググってると *"inbound=outbound"* というのが頻出したが国内のテキストではあまり見なかったのはなんでだろ）

おまけ: [tl;drLegal](https://tldrlegal.com) というサイトがちょっと面白かった
