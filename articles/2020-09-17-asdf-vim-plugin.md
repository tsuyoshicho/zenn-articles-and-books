---
title: "asdf プラグインの作成 : Vimの場合"
emoji: 🛠
type: "tech"
topics: ["vim", "asdf"]
published: true
---

この記事は [asdf プラグインの作成 : Vimの場合 - Qiita](https://qiita.com/tsuyoshi_cho/items/495f081117253f0b23bb) と同じものです。

----

# asdf-vm プラグインの作成

[asdf-vm](https://github.com/asdf-vm/asdf) はバージョニング管理ツール（でよいのかな）で、[プラグイン](https://github.com/asdf-vm/asdf-plugins) による複数種のツールの管理、また同機構で拡張できるのが特徴です。

## asdf-vim プラグイン

さて、このプラグインの一覧を見たときに、[Vim](https://github.com/vim/vim) がないことに気付きました。

Vimはパッチバージョンの更新が頻繁で、リリースという明確な区切りがないので、ときどき不具合ありにあたってしまうことがあります。

このときに回避したり、どこで混入したか調べたりするのに、古いバージョンを簡単に導入・切り替えできるとうれしいところがあります。

（とはいえ、作った者は普段は[Linuxbrew](https://docs.brew.sh/Homebrew-on-Linux)で更新していっているので、これであれこれすることは少なそうですが……）

そこで、練習と環境充実を兼ねて[asdf-vim](https://github.com/tsuyoshicho/asdf-vim)を作成しました。

追記:マージされました。

## asdfのプラグインとして

下で作成自体については詳細に説明しますが、asdfのプラグインがどんなことをすればよいのか、簡単に説明します。

* bin/install、bin/list-allというコマンド（基本シェルスクリプトだが）が入ったリポジトリを作る（これは最低限）
* bin/list-allはどうにかやって、対象ツールのインストールとして対応するバージョンリストを出力する
* bin/installは引数で上のバージョン、インストール先などの情報をもらうので、それをもってインストール処理を行う
* `asdf plugin-install hoge <git https path>` でプラグインをインストールして使う
* asdf-pluginsへPRするには、
    * CIテストの追加、GitHub Acitonsならインストールテストをしてくれるactionが用意されている
    * CIテストがGreenであること
    * pluginsディレクトリのhogeファイルにターゲットリポジトリを記載
    * READMEに名前、リンク、Statusなどを入れ、更新
    * これをローカルチェック
    * PRのステータスもGreenであること

    を確認しこれらをもってPRとして追加依頼をする
    [Adds vim plugin by tsuyoshicho · Pull Request #289 · asdf-vm/asdf-plugins](https://github.com/asdf-vm/asdf-plugins/pull/289)

オフィシャルへの追加がありうるようなプラグインとしてはかなり簡素ですね。

## よくあるOSS派生物作成として

作り方というか、わりとへろっと作ったので、その軌跡は残しておきます。
今回のもの固有というよりは、何らかのOSSの派生（forkというよりは類似品作成的なもの）として参考になるよう凡例的に書いている（つもり）。

1. 既存のプラグインのコードを眺めてみる
2. オフィシャルドキュメント [docs at asdf-vm.com](https://asdf-vm.com/) を読む
3. ある程度目処がついたら、自分が作りたい内容に近いものを探す（今回は[srivathsanmurali/asdf\-cmake](https://github.com/srivathsanmurali/asdf-cmake)を派生元にさせてもらった）
4. ライセンスをチェックし、派生できるか確認
5. 大丈夫なので派生する
6. 自分向けに必要箇所を変更（cmake->vim）
7. ビルドするところは完全に新規で作る……が、これも実績あるものを参考にする（継続的にビルドをやっているものとして [homebrew\-core/vim\.rb at master · Homebrew/homebrew\-core](https://github.com/Homebrew/homebrew-core/blob/master/Formula/vim.rb)を参考にした）
8. これらをもとに自分が変えたいところも修正する（travis -> GitHub Actions化、同処理でshell scriptのlint）
9. 動作確認
10. 完成

こんな感じで既存の動いているものを参考にすることで、初動の良さ、あまり必要のない詰りをうまくやれたりします。


## いかがでしたか

って書くとアレですね……ほかのソフトのプラグインでもですが、1本まるまるソフトを作るに比べると敷居は下るので、挑戦するに良い題材なのではないでしょうか。
