---
title: "Google Fitと連携するサービスを利用してみる : 事例 FitnessSyncer"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["fitness", "google"]
published: false
---
# Google Fitの利用と同期

## 動機

自分は健康管理の一環で、毎日血圧を測っています。
今までは紙管理でしたが、Google Fit上にも登録を始めました。

ただ、古いデータの登録をスマートフォンからとか面倒すぎるので、何かしらのサービスを利用して登録したく探したところ、FitnessSyncerが悪くないのではとなり、利用しています。

## [FitnessSyncer](https://www.fitnesssyncer.com/)

このサービスおよびアプリケーションでは、各種フィットネスアプリケーション(含むGoogle Fit管理の本体の情報)との入出力による同期ができます。
血圧、アクティビティ、血糖値などの各項目それぞれで入力、出力が作れます。
なお、手動でFitnessSyncer Notebookとして項目ごとにデータ入力もできます。

そして、これらの入力-出力の組み合せ(片方はFitnessSyncer)による処理セットでカウントして、無料だと5枠まで処理が作れます。

Google Drive上のCSVファイルからの入力で、FitnessSyncerに血圧の過去データを入力し、出力としてGoogle Fitへデータを流すことで初期データの流し込みを行いました。
その後は、次のような流れの設定のみ行っています。

外部からの入力

- アクティビティ : 入力 Google Fit : 出力 FitnessSyncer : 日々のスマートフォンでの歩数カウントの同期
- 血圧 : 入力 Google Fit : 出力 FitnessSyncer : 日々のスマートフォンでの手動入力の同期
 
外部への出力

- 血圧 : 入力 FitnessSyncer : 出力 Google Drive CSV ファイル : 血圧情報の結果バックアップ
- 身体情報 : 入力 FitnessSyncer : 出力 Google Fit : 身長体重など、定期もしくは不定期に測定した結果を FitnessSyncer Notebookから入力して同期

正直もっとよいサービスがあれば変更するのもよいかなと思っている程度です。
ちょっとした情報として記事にしました。
