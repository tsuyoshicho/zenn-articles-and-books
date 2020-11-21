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
- <https://github.com/tsuyoshicho/vital-codec> Math.Fraction作成

## 詳細

### vim-efm-langserver-settings

[efm-langserver](https://github.com/mattn/efm-langserver)の「lintツールをlanguage serverにしてしまう」という汎用性の良さと、[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)による[lsp](https://microsoft.github.io/language-server-protocol/) という(そのころはまだ敷居が高かった)ものの簡便化のよさに感激して作りました。
(どちらも作者は[mattn](https://zenn.dev/mattn)さん)

vim-lsp-settingsを真似てツールが使えるようなってさえいれば、zero configで利用できるようにしたのがポイント。

### Vim プラグインとしての工夫: vim-efm-langserver-settings

プラグインしては、導入するだけで動作する、vim-lsp/ale/cocに対応というのがあります。
おかげか、自分的には利用者が多いプラグイン(の様子)。

複数のプラグインへの設定追加には、

```text
after
 + plugin
   + coc
   + ale
   + lsp
```

という箇所へファイルを入れることで、各プラグインがロードされたら合せてロードされるVimの機能、[after directory](https://vim-jp.org/vimdoc-ja/options.html#after-directory)というものを使っています。

これは自分で入れるプラグインの追加設定をする場合にも有効なので(ex `~/vimfile/after`以下などに自分で設定を書いたファイルを入れる)、覚えて損はないです。

### lightline-lsp

小物プラグイン。
vim-lspから(一部非公開の関数を使って)lspの状態を取得し、[lightline](https://github.com/itchyny/lightline.vim)に表示するためのコンポーネントを定義する
プラグイン。

[maximbaz/lightline-ale](https://github.com/maximbaz/lightline-ale)というものを元に作成、あまり特別なことはしていないので、キャプチャを載せるにとどめる。

[![lightline-lsp : Image from Gyazo](https://i.gyazo.com/68933419ed2704286cd8d4fe39f2b6f3.png)](https://gyazo.com/68933419ed2704286cd8d4fe39f2b6f3)

efm-langserverとその左に状態インジケータがそれ。

### vim-fg

検索補助プラグイン。
[yegappan/grep](https://github.com/yegappan/grep)からインスパイアされたもの、非同期でgrep/pt/agほかの実行をハンドリングする。

上のでも期待した動きはするが、自分の環境とうまく合わないなどもあったのでこうしてある。

### Vim プラグインとしての工夫: vim-fg 

非同期に外部プロセスをハンドリングするのに[vim-jp/vital.vim](https://github.com/vim-jp/vital.vim)そして[lambdalisue/vital-Whisky](https://github.com/lambdalisue/vital-Whisky)による
Job(vim/neovimのjobの抽象化)とPromiseを利用している。

また、このプラグインも上の各種grepperを動かすための設定を抱えています。そしてその設定はtomlで書かれているが、これもvitalのライブラリとそれによる設定ファイル同梱機能が活躍(一部予定)。

いつか、この設定部分を独立したライブラリにしたい。

### asdf-vim

これについては[別記事](https://zenn.dev/tsuyoshicho/articles/2020-09-17-asdf-vim-plugin)を参照のこと。

### vital-codec Math.Fraction

今後こうしたいなー、という想定があるのだが説明すると長いので省略。
ただ、その中で、「分数をうまく扱いたい」というのが出てきた。

そういう機能なのでvitalの自分モジュールとして作成することにした。

プログラミングでの分数・有理数(Rational, Fraction)の扱いについてはいろいろあるが、今回はPythonの[fractions](https://docs.python.org/ja/3/library/fractions.html)を参照して作成。
また、その動作を考えると、文字からの数値への変換とその桁が十分にある精度のものである必要があるので、これもvital.vimのData.BigNum[^1][^2]が使えるので、これを利用。

### Vim プラグインとしての工夫: vital-codec Math.Fraction 

実はそこまでのことはしていないので、これ自体はvitalのモジュールを作る例としよいのではないかと思っている。
ので、ライブラリに切り出したい、みたいに考えている人が作業する際の参考になれば幸いです。

## まとめ

当初は記事のネタもなく、「今年は大したことしてないなー」と思ったものだが、案外活動しているものだ。
これからもいろいろと頑張りたいと思う。

## 注釈

- [^1]: [Vimで任意精度整数演算ができるライブラリを作った - チューリング不完全](https://aomoriringo.hateblo.jp/entry/2015/04/17/143053)
- [^2]: [vital.vim/BigNum.txt at master · vim-jp/vital.vim](https://github.com/vim-jp/vital.vim/blob/master/doc/vital/Data/BigNum.txt)
