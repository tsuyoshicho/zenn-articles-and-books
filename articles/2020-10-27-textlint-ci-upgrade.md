---
title: "GitHub Action textlint ツールでの reviewdog キャッチアップ : 修正サジェスト"
emoji: "🐶"
type: "tech" 
topics: ['GitHub', 'GitHubAction', 'textlint', 'reviewdog']
published: false
---
# action-textlint v2 available

どうも、[action-textlint](https://github.com/tsuyoshicho/action-textlint) を作りました作者でございます。 いくつかの記事 ([1](https://zenn.dev/serima/articles/4dac7baf0b9377b0b58b), [2](https://zenn.dev/srz_zumix/articles/cb21af1a86fc01cb829d), [3](https://zenn.dev/srz_zumix/articles/9404b45e22cdf0f65ddd)) でも利用例を出していただいてうれしい限りです。

今回v2系へのアップデートと、それに伴う機能拡充があったので、どんなことをしたのかも含めて記事にします。

## What is textlint

あらためてちょっとだけ[textlint](https://github.com/textlint/textlint)について紹介。

textlintはnpmパッケージとして提供されている、テキストドキュメントをlint(プログラ
ミング方言かな? 検査・検証)できるツールです。

プラグインでの拡張、ruleやその集合のpresetを導入することで柔軟な検査が行えます。
逆になにも入れてないときは、なにもしないですが。

## What is reviewdog

[reviewdog](https://github.com/reviewdog/reviewdog)にもたいへんお世話になっているので、紹介を……。
と思うところですが、下で更新を説明していますので、そこで。

## What is action-textlint

そしてaction-textlintですが、これをGitHub Actionsとして実行し、結果をコミット/PRに付与できるというものです。

## reviewdog support suggestion feature

action-textlintはtextlintの結果をreviewdogというツールを利用してGitHubのコミット/PRへlint結果を出力しています。

先日、このreviewdogが[v0.11.0](https://github.com/reviewdog/reviewdog/releases/tag/v0.11.0)へバージョンアップしました。
変更の内容はリンク先を見ていただくとして、重要なものの1つに「差分による修正サジェストに対応」があります。

ご承知のように、GitHubのPRでは、レビュアーよる変更方法のサジェストができ、一部ツールでも出してくれるものがあります。
今回のreviewdogの更新によって、通常の指摘内容より拡張された新フォーマットのrdjson、もしくは差分のdiff形式での修正サジェストをPRに出せるようになりました。

後述しますが、今回これに対応しています。

## Node.js 15 / npm 7 release

参照:

- [Node\.js v15 の主な変更点 \- 別にしんどくないブログ](https://shisama.hatenablog.com/entry/2020/10/21/004612)
- [npm v7の主な変更点まとめ \| watilde's blog](https://blog.watilde.com/2020/10/14/npm-v7%E3%81%AE%E4%B8%BB%E3%81%AA%E5%A4%89%E6%9B%B4%E7%82%B9%E3%81%BE%E3%81%A8%E3%82%81/)

これも先日ですが、Node.js 15およびそれに同梱のnpm 7がリリースされています。

これ自体は GitHub Action とは直接は関係しませんが(環境が固定的(ubuntu-latestが[20.04](https://github.blog/changelog/2020-10-29-github-actions-ubuntu-latest-workflows-will-use-ubuntu-20-04/)になりますが)なので、意図して使わなければ使うことがない)、その中に

> その他、npxでの破壊的変更

にある内、このactionに影響のあるものがありました。
これの対応が必要だったので、それにも対応しています。

## Changelog

変更ですが、これらについてv2系として対応しました。
最新は[v2.2.0](https://github.com/tsuyoshicho/action-textlint/releases/tag/v2.2.0)です。

やった内容です。

### Deprecated parameters adopted during development

まず先に非互換について。

旧来のv1のパラメータですが、reviewdogとそれを利用するGitHub Actionsとして不慣れな面があったため、引数の一部があまり好ましいものではありませんでした。
開発系(v0相当)からリリース(v1)になった時点で実質利用しなくなっていましたが、残ってしまっていました。
これについて削除しています。

### Catch up npm 7

また、上に上げているnpm(というかnpxの引数)での非互換になりうる箇所が実装にあり、これに対応するにあたってv2という明確な切れ目に合せました。
なお、現在のv2でもnpm7以前で正常に動作しますので、特に問題がなければv2系へ移行することをお勧めします。

### Treat more options

また、これらと合せる形で、 reviewdogプロジェクトリポジトリ管理のGitHub Actions(eslintを掛けるものなど多数あります)の内容をパク……もとい参考にさせていただきました。
reviewdogへ与えるパラメータについて、これらと同等に設定可能になるように修正しています。

### Hello suggestion

そして目玉ですが、textlintが提供する自動修正の結果にもとづいて、PRでのサジェストとして出す機能を入れています。
これはreviewdogチームが提供している汎用の差分サジェストサポートAction、[reviewdog/action-suggester](https://github.com/reviewdog/action-suggester)を参考にして出すようにしました。

実行例:

- [テスト実行](https://github.com/tsuyoshicho/action-test-repo/pull/3)での結果キャプチャ
  [![Image from Gyazo](https://i.gyazo.com/77649ba3fb4087667bad2e0079404df6.png)](https://gyazo.com/77649ba3fb4087667bad2e0079404df6)

利点としては、ここまでくれば手元で `npm ci && npm run lint` などをしなくても、textlintでの修正結果までGitHub上だけで受け取れることでしょうか。

実施内容ですが、以下のようなことをしています。
詳細を述べるにしても内容はシンプルなので、ソースを見たりしてもらえればと思います。

1. v1とそう変らない、通常のlintによる確認と通知動作を行う
2. reporter形式が`github-pr-review`であるときにサジェスト追加処理にはいる(3以降)
3. `textlint --fix`で修正を行う
4. `git diff`で差分ファイルを取り出す
5. reviewdogに差分ファイルを渡すことでサジェストをPRへ出力
6. 差分ファイルの削除、作業環境への修正適用を巻き戻すなどのクリーンナップ

見ていただくと分りますが、新形式のrdjsonを生成するのはそうそうに諦めたこともあり、指摘とサジェストは分離してしまうこととなりました。

## epilogue

現在もまだ改善の余地はありますが、かなり使えるようになる修正ではないかと自負しています。
ただ、無条件にサジェストも出しているので、これをオプション制御にするかなど、考えるべき所はまだまだあります。

これからもこのツールをよろしくお願いします。
