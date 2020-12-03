---
title: "今年のVim関係とそれ以外も含めた活動での結果とエッセンス"
emoji: "🎅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['vim', 'tips', 'advent']
published: false
---
この記事は[Vim Advent Calendar 2020](https://qiita.com/advent-calendar/2020/vim)の18日目の記事です。

- 17日目は (あとで) ( [hrsh7th@github](https://qiita.com/hrsh7th@github)さん )
- 19日目は (あとで) ( [yutakatay](https://qiita.com/yutakatay)さん ) 

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

プラグインしては、導入するだけで動作する、ale/coc/vim-lspに対応というのがあります。
おかげか、自分的には利用者が多いプラグイン(の様子)。

複数のプラグインへの設定追加には、

```text
after
 + plugin
   + ale
   + coc
   + lsp
```

という箇所へファイルを入れることで、各プラグインがロードされたら合せてロードされるVimの機能、[after directory](https://vim-jp.org/vimdoc-ja/options.html#after-directory)というものを使っています。

これは自分で入れるプラグインの追加設定をする場合にも有効なので(ex `~/vimfile/after`以下などに自分で設定を書いたファイルを入れる)、覚えて損はないです。

### lightline-lsp

小物プラグイン。
vim-lspから(一部非公開の関数を使って)lspの状態を取得し、[lightline](https://github.com/itchyny/lightline.vim)に表示するためのコンポーネントを定義するプラグイン。

[maximbaz/lightline-ale](https://github.com/maximbaz/lightline-ale)というものを元に作成、あまり特別なことはしていないので、キャプチャを載せるにとどめます。

[![lightline-lsp : Image from Gyazo](https://i.gyazo.com/68933419ed2704286cd8d4fe39f2b6f3.png)](https://gyazo.com/68933419ed2704286cd8d4fe39f2b6f3)

efm-langserverとその左の状態インジケータがそれです。

### vim-fg

検索補助プラグイン。
[yegappan/grep](https://github.com/yegappan/grep)からインスパイアされたもので、非同期でgrep/pt/agほかの実行をハンドリングして、結果をquickfixへ出力します。

上のでも期待した動きはしますが、自分の環境とうまく合わないなどもあったので最終的には自作することになりました。

### Vim プラグインとしての工夫: vim-fg 

非同期に外部プロセスをハンドリングするのに[vim-jp/vital.vim](https://github.com/vim-jp/vital.vim)そして[lambdalisue/vital-Whisky](https://github.com/lambdalisue/vital-Whisky)による
Job(vim/neovimのjobの抽象化)[^1]とPromise[^2]を利用しています。

また、このプラグインも上の各種grepperを動かすための設定を抱えています。そしてその設定はtomlで書かれていますが、これもvitalのライブラリとそれによる設定ファイル同梱機能が活躍(一部予定)。

まだまだ作りかけですが、通常生活では満足する程度には動作します。
いつか、この設定部分を独立したライブラリにしたいのですね。

### asdf-vim

これについては[別記事](https://zenn.dev/tsuyoshicho/articles/2020-09-17-asdf-vim-plugin)を参照のこと。

### vital-codec Math.Fraction

今後こうしたいなー、というプラグインの構想があるのだが詳細を説明するとなると長いので省略します。 とはいえ、ほかの人の作ったものが開発停止になり、そのうち自分用にforkするだろうというものですが。
ただ、その中で、「分数をうまく扱いたい」というのが出てきました。

そういう機能なのでvitalの自分モジュールとして作成することにしました。

プログラミングでの、分数・有理数(Rational, Fraction)の扱いについてはいろいろあります。今回はPythonの[fractions](https://docs.python.org/ja/3/library/fractions.html)を参照して作成しました。
またその動作を考えると、文字からの数値への変換とその桁が十分な精度である必要があります。これについてはvital.vimのData.BigNum[^3][^4]が使えるので、これを利用しています。

### Vim プラグインとしての工夫: vital-codec Math.Fraction 

実はそこまでのことはしていないので、これ自体はvitalのモジュールを作る例としてよい例になるのではないかと思っています。[vital-codec/Fraction.txt at master · tsuyoshicho/vital-codec](https://github.com/tsuyoshicho/vital-codec/blob/master/doc/vital/Math/Fraction.txt)
ですので、小さめのライブラリを作りたい、とかライブラリに切り出したいと考えている人が作業する際の参考になれば幸いです。

一応の工夫として以下があります。

- モジュールが生成するオブジェクトに識別子を与えて判定可能にしている。(is_Rational)
- テスト作成とエラーハンドリング(最大のものはzero divid関係)をがんばったので、エラー処理やエラーのテストの例としても悪くない。
- 生成するRationalオブジェクトでadd/sub/mul/divなどをメソッドとして持ち、メソッドチェインできるようになっている。
- また、ほぼ同じ処理をクラスのメソッドとしても持っており、そこは数値(整数)、文字列(整数)、BigNum、Rationalを入れて処理もで
  きるようにしてある。

などがあります。たとえvitalのモジュールにしないにしても、Vim scriptでオブジェクトとかそのための記述方法の参考にはなるかと。

## まとめ

当初は記事のネタもなく、「今年は大したことしてないなー」と思ったものだが、案外活動しているようでした。
これからもいろいろと頑張りたいと思います。

## 注釈

- [^1]: [vital-Whisky/Job.txt at master · lambdalisue/vital-Whisky](https://github.com/lambdalisue/vital-Whisky/blob/master/doc/Vital/System/Job.txt)
- [^2]: [vital.vim/Promise.txt at master · vim-jp/vital.vim](https://github.com/vim-jp/vital.vim/blob/master/doc/vital/Async/Promise.txt)
- [^3]: [Vimで任意精度整数演算ができるライブラリを作った - チューリング不完全](https://aomoriringo.hateblo.jp/entry/2015/04/17/143053)
- [^4]: [vital.vim/BigNum.txt at master · vim-jp/vital.vim](https://github.com/vim-jp/vital.vim/blob/master/doc/vital/Data/BigNum.txt)
