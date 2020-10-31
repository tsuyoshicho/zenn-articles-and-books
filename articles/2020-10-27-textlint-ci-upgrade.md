---
title: "GitHub Action textlint ツールでの reviewdog キャッチアップ : 修正サジェスト"
emoji: "🐶"
type: "tech" 
topics: ['GitHub', 'GitHubAction', 'textlint', 'reviewdog']
published: false
---
# action-textlint v2 available

どうも、[action-textlint](https://github.com/tsuyoshicho/action-textlint) を作りました作者でございます。
いくつかの記事
([1](https://zenn.dev/serima/articles/4dac7baf0b9377b0b58b),
[2](https://zenn.dev/srz_zumix/articles/cb21af1a86fc01cb829d),
[3](https://zenn.dev/srz_zumix/articles/9404b45e22cdf0f65ddd))
でも利用例を出していただいてうれしい限りです。

今回 v2 系へのアップデートと、それに伴う機能拡充があったので、どんなことをしたのかも含めて記事にします。

## what is textlint

改めてちょっとだけ textlint について紹介。

[textlint](https://github.com/textlint/textlint)
はnpmパッケージとして提供されている、テキストドキュメントをlint (プログラミング方言かな?
検査・検証)できるツールです。

プラグインでの拡張、ruleやその集合のpresetを導入することで柔軟な検査が行えます。
逆になにも入れてないときは、なにもしないですが。

## what is reviewdog

reviewdogにもたいへんお世話になっているので、紹介を……。
と思うところですが、下で更新を説明していますので、そこで。

## what is action-textlint

そして action-textlint ですが、これを GitHub Action として実行し、結果をコミット/PRに付与できるというものです。

## reviewdog support suggest now

action-textlint は reviewdog というツールを利用して GitHub のコミット/PRへ lint 結果を出力しています。

- [reviewdog/reviewdog: 🐶 Automated code review tool integrated with any code analysis tools regardless of programming language](https://github.com/reviewdog/reviewdog)

先日、この reviewdog が [v0.11.0](https://github.com/reviewdog/reviewdog/releases/tag/v0.11.0)
へバージョンアップしました。
変更の内容はリンク先を見ていただくとして、重要なものの1つに「差分による修正サジェストに対応」があります。

ご承知のように、GitHubのPRでは、レビューワーによる変更方法のサジェストができ、一部ツールでも出してくれるものがあります。
今回の reviewdog の対応で、通常の指摘内容より拡張された新フォーマットのrdjson、もしくは差分のdiff形式での修正サジェストをPRに出せるようになりました。

後述しますが、今回これに対応しています。

## node.js 15 / npm 7 release

参照

- [Node\.js v15 の主な変更点 \- 別にしんどくないブログ](https://shisama.hatenablog.com/entry/2020/10/21/004612)
- [npm v7の主な変更点まとめ \| watilde's blog](https://blog.watilde.com/2020/10/14/npm-v7%E3%81%AE%E4%B8%BB%E3%81%AA%E5%A4%89%E6%9B%B4%E7%82%B9%E3%81%BE%E3%81%A8%E3%82%81/)

これも先日ですが、node.js 15 およびそれに同梱の npm 7 がリリースされています。

これ自体は GitHub Action とは直接は関係しませんが(環境が固定的(ubuntu-latestが[20.04](https://github.blog/changelog/2020-10-29-github-actions-ubuntu-latest-workflows-will-use-ubuntu-20-04/)になりましたが)なので、意図して使わなければ使うことがない)、その中に

> その他、npxでの破壊的変更

にある内、この action に影響のあるものがありました。
これの対応が必要だったので、それにも対応しています。

## changelog

変更ですが、これらについて v2 系として対応しました。
最新は [v2.2.0](https://github.com/tsuyoshicho/action-textlint/releases/tag/v2.2.0)
です。

やった内容です。

### development param deprecate and remove

まず先に非互換について。

旧来のv1のパラメータですが、reviewdogとそれを利用するGitHub Actionとして不慣れな面があったため、引数の一部があまり好ましいものではありませんでした。
開発系(v0相当)からリリース(v1)になった時点で、実質利用しなくなっていたのですが、残っていましたがこれを削除しています。

### catch up npm 7

また、上に上げている npm(というかnpxの引数)での非互換になりうる箇所が実装にあり、これに対応するにあたって、v2という明確な切れ目に合せました。
なお、現在のv2でもnpm7以前でも正常に動作しますので、特に問題がなければv2系へ移行することをおすすめします。

### treat more options

また、これらと合せる形で、 reviewdog プロジェクトリポジトリ管理の GitHub Action (eslintを掛けるものなど多数あります) の内容をパク……もとい参考にさせていただきました。
reviewdogへ与えるパラメータについて、これらと同等になるように修正しています。

### hello suggestion

そして目玉ですが、textlint が提供する自動修正について、PRでのサジェストとして出す機能を入れています。
これは reviewdog チームが提供している汎用の差分サジェストサポート Action
[reviewdog/action-suggester](https://github.com/reviewdog/action-suggester) を参考にして出すようにしました。

例:

- [テスト実行](https://github.com/tsuyoshicho/action-test-repo/pull/3)での結果キャプチャ
  [![Image from Gyazo](https://i.gyazo.com/77649ba3fb4087667bad2e0079404df6.png)](https://gyazo.com/77649ba3fb4087667bad2e0079404df6)

利点としては、ここまでくれば、手元で `npm ci && npm run lint` などしなくても、textlintでの修正結果までGitHub上だけで受け取れることでしょうか。
