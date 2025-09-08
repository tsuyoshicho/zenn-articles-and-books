---
title: "Gitのコミットメッセージ上の種別を簡単に変更する"
emoji: "📜"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "vim"]
published: false
published_at: 2025-09-15 00:00 # need published: true
publication_name: "vim_jp"
---

この記事は Vim 駅伝の 2025-09-15 の記事です。
前回の記事は [kawarimidoll](https://zenn.dev/kawarimidoll) さんの「[Neovim luaのlsとしてemmyluaを使ってみる](https://zenn.dev/vim_jp/articles/56bc5db545f47b)」でした。

Vim 駅伝は常に参加者を募集しています。詳しくは[こちら](https://vim-jp.org/ekiden/about/)のページをご覧ください。

# Gitのコミットメッセージ上の種別を簡単に変更する

Git のデフォルトエディタは Vim で、コミットメッセージの編集をすることも多いかと思います。
コミットメッセージ自体もですが、リベース時のコミットの順序変更や状態種別変更もよくあります。
