---
title: "Vim Advent記念 Vim Plugin作成RTA"
emoji: "⏱"
type: "tech"
topics: ["vim", "advent"]
published: true
published_at: "2022-12-08 09:00"
publication_name: "vim_jp"
---
:::message

この記事は[Vim Advent Calendar 2022 (part 2)](https://qiita.com/advent-calendar/2022/vim)の8日目の記事です。

- 7日目は (未確定) です。
- 9日目は (未確定) です。
 
:::

# Vim Advent記念 Vim Plugin作成RTA

今回特にAdvent記事のネタはなかったのですが、先日発作的にプラグインを作ったので、その記録を上げます。
技術的な点はあまりないかもですね(一応がんばります)。

## 免責事項

この記事はわりと勢いで書いています。文章が乱雑でもご容赦ください。

## 発端と結果

2022/11/28 17:24、とあるslackに発端となるコメントが投下されました。
<!-- https://vim-jp.slack.com/archives/CLKR04BEF/p1669623859416269 -->

![発端のコメント](/images/20221130/slack1.png)

これを受けて、自分は唐突に「よし、作ってみるか」と思い立ちました。

そしてその後、 2022/11/28 21:54に同チャンネルへコメントを投下します。
<!-- https://vim-jp.slack.com/archives/CLKR04BEF/p1669640079723019?thread_ts=1669623859.416269&cid=CLKR04BEF -->

![結果のコメント](/images/20221130/slack2.png)

その時間差、実に4時間30分！
自分的に最速でのプラグイン[^1]作成と相成りました。
https://github.com/tsuyoshicho/asyncomplete-mr.vim

## 動機とその前提知識

さて、この突発のプラグイン作成、なにが動機だったか、の前に前提知識を。

### [mr.vim](https://github.com/lambdalisue/mr.vim)

`mr.vim` は上の画像の主である[ありすえさん](https://github.com/lambdalisue)によるMRU(Most Recently Used)[^2]のファイルリスト管理するプラグインです。
このプラグインではMRU/Used(つまり開いたファイル)だけではなく、MRW/Write(書き込みをしたファイル)、MRR/Repository(Gitリポジトリ)の履歴も取得しており、簡素ながら利便性の高い直近のファイル/ディレクトリ情報を利用できます。

このプラグインはシンプルなAPIになっており、各結果のリストを返す機能があります。
このコメントは、それを利用した各種補完機能での利用例を募集したということになります。

### [asyncomplete.vim](https://github.com/prabirshrestha/asyncomplete.vim)

ここで自分が対応しようと考えたのは、`asyncomplete.vim` というVimの自動補完プラグインの補完ソースとして、`mr.vim` を利用する方法についてです。

Vimは補完として基本の補完やomni補完、ほかに類語辞書での補完などが手動で呼べる機能があります。
自動補完プラグインは、これを越えて挿入モード中の文字入力時に補完を開始するようなプラグインになります。

`asyncomplete.vim` は補完ソースプラグインを追加(および設定)することで自動補完の情報ソースを増やせるプラグインです(類似のプラグインは多数あります)。
このソースとして `mr.vim` の履歴ファイル/ディレクトリを出すものを作ろうと考えました。

逆にちょっと不便(?)なのですが、補完ソースとしてのプラグインを作らないといけないという制約があります。
設定上にちょろっと補完元データを書くことで補完ソースとして動くというものではないんですね。

### [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)のソース作成経験と `asyncomplete.vim` のソース修正経験

さて、これだけだと作成のハードルはなかなか高そうですし、なぜ手を上げた(上げずに結果報告したんですが)のかも謎です。

実は `mr.vim` を情報ソースとしたプラグイン[ctrlp-mr.vim](https://github.com/tsuyoshicho/ctrlp-mr.vim)というのを作っていました。
これは `ctrlp.vim` というファジーファインダーの選択ソース(選択するとそのファイル/ディレクトリを開く)というものです。
ファジーファインダーについては解説記事[^3]を見てください。

この経験と、あと `asyncomplete.vim`の 補完ソースのプラグイン複数[^4][^5][^6]についてcontributeした経験がありました。

### 上記を受けての作成

これらがあることで、`asyncomplete.vim` で `mr.vim` の補完ソースを作るにはまあなんとかなる知識があり、適役な状態になってました。

ので、やってみようとなったわけです。

## 作成の流れ

さて作ると考えましたが、どんなことをやったのか、なるべく生のままの結果を出します。(コミット順は多少前後する)

なお、このプラグイン完成の報告時コミット数については、squash mergeしたこともあり initial commit -> setup commit (merge) と2つしかない状態だったりしました。
現在はREADMEの微調整、ファイル移動などでもうちょっと増えましたが。

### リポジトリとプラグインの枠組み作成

まず、リポジトリを用意しなくてはなりません。

GitHub上に新規作成し、ライセンスは無難にMITとしておきました。

なお、ここでミスって、自分用のテンプレートから生成しそこねていました。
このため後から `.gitignore` とか `.editorconfig` などのファイルを準備したりしました。

プラグインの枠組みですが、[layoutplugin.vim](https://github.com/mopp/layoutplugin.vim)というプラグインにより、プラグイン全体のスケルトンを作成しました。
ただ、`asyncomplete.vim` の補完ソースプラグインとして `/autoload/xxx.vim` や `/plugin/xxx.vim` は不要ですので、後で削除しましたが……。

helpを入れるなら、[vimhelpgenerator](https://github.com/LeafCage/vimhelpgenerator)というプラグインもあります。
このプラグインだと、実コードからhelpの中身を埋めてくれたりもするので重宝しますが、補完ソースプラグインという都合、出番はありませんでした。
(READMEに設定を記述すれば十分だった)

### PoC/プロトタイプ/初版の作成

実装の手順としてはあんまりよくはないんですが、動作プロトタイプを作り、それをブラッシュアップして本プラグイン化しました。
内容の詳細は後で行うとして、ある程度動く状態まではこれで確認しています。

### リファクタリング

さて、最初にmrr(Git リポジトリのリスト)で作成しましたが、全部を1つのソースとして作るとコントロールが効かなくなってうれしくありません。
ですので、mrwそしてmruについて同じファイルをコピーしてそれぞれをソースとして持つようにしました。

ただ、DRYの原則[^7]から考えてもこれも良くはありません。
動くための初手としてはよいのですが、これについては期待する動作が担保できた後にリファクタリングを行い、共通処理をまとめたりしました。

以降では、それを含めた改善箇所の説明をちょっとします。

### 工夫した点

#### 共通化

上にもありますが、内部的に重複したコードになっていました。
なので共通の関数として処理をまとめ、各補完ソースはそれを利用して処理を行うようにしています。

#### 構造化

構造化というとちょっと違うところはありますが。

`asyncomplete.vim` は補完情報を通知するとき、「文字列のリスト」か「辞書データのリスト」を送れます。
この辞書データのリストに対応するための修正をしました。

Vimの辞書はほかの言語でいうところの連想配列みたいなものです(Pythonのdictというべきか)。
`mr.vim` は単純な文字列リストを返していましたし、これを直に設定していましたが、これに構造を持せます。
辞書に変換して、補完ソースが何なのか、あとプロパティをいくつかもたせるようにしました。

この辞書データには、その補完元から補完データがなにをするのかなどの情報を追加で出すための `info` などもあるのですが、今回は出番はありませんでした。
辞書からの単語補完とかなら訳を出したり、言語の機能を補完するのであればそのドキュメントを出すなどできるんですけどねー。

#### マッチルール

`asyncomplete.vim` では補完ソースが呼ばれるとき、補完時の文字を自前で確認できます。
ここで不一致なら空の終了とすることで高速化されているわけです。

この処理の所を初期に `\w+` (任意の英数字1文字以上)としていましたが、`mr.vim` がパスを取り扱うので、あまり適切ではないものでした。
ここを `\f*` としました(1文字は必須でもよかったかも)。
こちらはVimの `isfname` というオプション依存ながら、パスに使う文字列が適合するので、より適切です。
これにより `:` や `/` や `\` でもマッチします。

ただし、補完のトリガや終了はVimの `iskeyword` というオプションに依存しています。
こちらでは、デフォルトでは `\` が含まれていません。
コーディングでのキーワードなどを識別するためですが、単語を認識するという概念はプラグインなどで、いろいろ使われてしまう(使うことになるというべきか?)のです。
このためWindowsでの補完はいまいちなものになっています……。

## 最終的なコード

最終的にできたものを載せます。
これ自体を見てもあまりピンとはこないと思いますが、参考までに。

mrrの実装。

https://github.com/tsuyoshicho/asyncomplete-mr.vim/blob/a924ef0612d5f6e98cdb6e2045706ab0ea3f2018/autoload/asyncomplete/sources/mrr.vim#L7-L15

`asyncomplete.vim` のエントリの関数を用意しています。
処理の本体は共通処理に一任。


こちらは共通処理。

https://github.com/tsuyoshicho/asyncomplete-mr.vim/blob/a924ef0612d5f6e98cdb6e2045706ab0ea3f2018/autoload/asyncomplete/mr/util.vim#L7-L23

共通の処理として:

1. 補完の開始文字の処理(ファイルパス可能な文字でマッチ)
2. もらってきているパスのリストについて、先頭が補完とマッチするかフィルタ
3. フィルタしたリストを構造のあるデータのリストに変換
4. 結果報告関数をコール

という体裁になっています。

この流れ自体はおおよそどの `asyncomplete.vim` 補完ソースでも同じようなものです。

## むすび

こんな感じで、勢いで作ったわけです。

`mr.vim` / `asyncomplete.vim` にのっかるだけだったので技術的な難易度はありませんが、 構築、修正という意味でやりがいあるものでした。

[^1]: [asyncomplete-mr.vim](https://github.com/tsuyoshicho/asyncomplete-mr.vim)
[^2]: [キャッシュアルゴリズム - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0)
[^3]: [Vimにたくさんあるファジーファインダー系プラグインを比較してみる](https://zenn.dev/yutakatay/articles/vim-fuzzy-finder)
[^4]: [asyncomplete-look.vim](https://github.com/gonzoooooo/asyncomplete-look.vim)
[^5]: [asyncomplete-tags.vim](https://github.com/prabirshrestha/asyncomplete-tags.vim)
[^6]: [asyncomplete-dictionary](https://github.com/yuki-yano/asyncomplete-dictionary)
[^7]: [Don't repeat yourself - Wikipedia](https://ja.wikipedia.org/wiki/Don%27t_repeat_yourself)
