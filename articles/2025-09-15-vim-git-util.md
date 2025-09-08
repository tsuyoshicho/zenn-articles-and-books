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

Git のデフォルトエディタは Vim で、コミットメッセージの編集をすることがあります。
コミットメッセージ自体もですが、リベース時のコミットの順序変更や状態種別変更もよくあります。

![コミットメッセージ編集画面](/images/20250915/git-commit-edit-vim.png)

このとき、次のような設定を入れていて、これによって、状態を簡単に切り替えられるようにしています。
「記事になるかも」というコメントもらったので、ここで設定や動作を見せてみます。

## プラグイン

* vim-clurin

このプラグインは、Ctrl-x/Ctrl-a の数値増減に類似した操作として、値変更操作を提供します。

元は <https://github.com/syngan/vim-clurin> ですが、fork した <https://github.com/uplus/vim-clurin> がお勧めです。

## 設定

キーマッピングは省略します。

```vim
  let g:clurin = {
        \ 'gitrebase': {
        \   'def': [
        \      ['pick', 'fixup', 'reword', 'edit', 'squash', 'drop', 'exec'],
        \   ]
        \ }
        \}
```

リベース時に利用するファイルに設定されるファイルタイプ gitrebase に対して、キーワードについての循環変更設定を与えています。

## 操作結果

![リベース編集画面](/images/20250915/git-rebase-edit-vim.gif)

# あとがき

プラグインを利用はしていますが、ちょっとした設定で、便利になる一例になれば幸いです。
