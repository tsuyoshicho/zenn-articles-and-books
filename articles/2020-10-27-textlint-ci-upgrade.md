---
title: "GitHub Action textlint ツールでの reviewdog キャッチアップ : 修正サジェスト"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GitHub', 'GitHubAction', 'textlint', 'reviewdog']
published: false
---

# [action-textlint](https://github.com/tsuyoshicho/action-textlint) v2 available!

どうも、action-textlint を作りました作者でございます。
いくつかの記事 ([1](https://zenn.dev/serima/articles/4dac7baf0b9377b0b58b), [2](https://zenn.dev/srz_zumix/articles/cb21af1a86fc01cb829d), [3](https://zenn.dev/srz_zumix/articles/9404b45e22cdf0f65ddd)) でも利用例を出していただいて嬉しい限りです。

今回 v2 系へのアップデートと、それに伴う機能拡充があったので、どんなことをしたのかも含めて記事にします。

## what is textlint

改めてちょっとだけ textlint について紹介。

[textlint](https://github.com/textlint/textlint) は npm パッケージとして提供されている、テキストドキュメントを lint (プログラミング方
言かな? 検査・検証)できるツールです。

プラグインでの拡張、ruleやその集合のpresetを導入することで柔軟な検査が行えます。
逆になにも入れてないときは、なにもしないですが。

## what is action-textlint

そして

## reviewdog support suggest now

action-textlint は reviewdog というツールを利用して GitHub のコミット/PRへ lint 結果を出力しています。

- [reviewdog/reviewdog: 🐶 Automated code review tool integrated with any code analysis tools regardless of programming language](https://github.com/reviewdog/reviewdog)

先日、この reviewdog が [v0.11.0](https://github.com/reviewdog/reviewdog/releases/tag/v0.11.0) へバージョンアップしました。
変更の内容はリンク先を見ていただくとして、重要なものの1つに「差分による修正サジェストに対応」があります。

ご承知のように、GitHubのPRでは、レビューワーによる変更方法のサジェストがあり、一部ツールでも出してくれるものがあります。
今回の reviewdog の対応で、通常の指摘内容より拡張された新フォーマットのrdjsonもしくは差分のdiff形式での修正サジェストをPRに出せるよう
になりました。

後述しますが、今回これに対応しています。

## node.js 15 / npm 7 release

参照

- [Node\.js v15 の主な変更点 \- 別にしんどくないブログ](https://shisama.hatenablog.com/entry/2020/10/21/004612)
- [npm v7の主な変更点まとめ \| watilde's blog](https://blog.watilde.com/2020/10/14/npm-v7%E3%81%AE%E4%B8%BB%E3%81%AA%E5%A4%89%E6%9B%B4%E7%82%B9%E3%81%BE%E3%81%A8%E3%82%81/)

これも先日ですが、node.js 15 およびそれに同梱の npm 7 がリリースされています。

これ自体は GitHub Action とは直接は関係しませんが(環境は固定的なので、意図して使わなければ使うことがない)、その中に

> その他、npxでの破壊的変更

にある内、この action に影響のあるものがありました。
これの対応が必要だったので、それにも対応しています。


https://github.com/tsuyoshicho/action-textlint/releases/tag/v2.1.1
v2 系でこれに対応


やった内容

例
https://github.com/tsuyoshicho/action-test-repo/pull/3
[![Image from Gyazo](https://i.gyazo.com/77649ba3fb4087667bad2e0079404df6.png)](https://gyazo.com/77649ba3fb4087667bad2e0079404df6)

利点



今後と課題


