---
title: "2020 Vim アドベンド day 18: 今年のVim関係とそれ以外も含めた活動"
emoji: "🎅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['vim']
published: false
---
この記事は[Vim Advent Calendar 2020](https://qiita.com/advent-calendar/2020/vim)の18日目の記事です。

## 今年のVim活動の総括

何を書こうか悩みましたが、2020年のVim関係の活動をまとめてそこから何か抽出しよう、と考えました。

## 今年のVim関係だけどVimじゃない活動の総括

それ以外にも関係する活動が多少あったので、それも合わせて書きます。

## 活動内容

- <https://github.com/tsuyoshicho/vim-efm-langserver-settings> 作成
- <https://github.com/tsuyoshicho/lightline-lsp> 作成
- <https://github.com/tsuyoshicho/vim-fg> 作成
- <https://github.com/tsuyoshicho/asdf-vim> 作成

## 詳細

### vim-efm-langserver-settings

[efm-langserver](https://github.com/mattn/efm-langserver)の
「lintツールをlanguage serverにしてしまう」という汎用性の良さと、
[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)による
[lsp](https://microsoft.github.io/language-server-protocol/)
という(そのころはまだ敷居が高かった)ものの簡便化のよさに感激して作りました。
(どちらも作者は[mattn](https://zenn.dev/mattn)さん)
vim-lsp-settingsを真似てツールが使えるようなってさえいれば、
zero configで利用できるようにしたのがポイント。

### vim プラグインとしての工夫: vim-efm-langserver-settings

to be continued

