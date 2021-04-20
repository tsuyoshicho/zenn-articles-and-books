---
title: "GitHub Actions aciton-textlintv3 が出てています"
emoji: "🐶"
type: "tech" 
topics: ['GitHub', 'GitHubActions', 'textlint', 'reviewdog']
published: false
---
# action-textlint v3 available

以前話題にしたaction-textlintですが、v3になりました。

[こんな要望](https://zenn.dev/serima/articles/4dac7baf0b9377b0b58b#comment-a9468b580bdbf3)がありましたので、簡単に解説します。

## action-textlint change from v2 to v3

あまり難しい説明はしないですが、まずv2で大きな問題がありました。

reviewdogプロジェクトのactiionに類似するなら、開発環境のlintの条件でlintして判定する、みたいなのが基本方針なのです。
が、Docker Actionとして作っていたため、開発環境のtextlintの設定が適切に引き継げていないというのがありました。

(このあたり、自分の状態の把握の不足です)

そのため、いろいろ見直した結果

- 実装をDocker actionからComposition actionに変換
- ユーザー環境のtextlintの設定をなるべくそのまま適用
- ログ出力をモダンに調整

### Composition action

Composition actionは新しいGitHub Acitonの形態です。

いままで、GitHub Actionには

- JavaScript Aciton
- Docker Aciton
 
というのがありました。
前者は、環境が切り離されたJavaScriopt(node)での処理実行、後者は所定のDockerでの処理実行です。

ただ、この処理には不便なことがあり

- 実行に時間がかかる(仮想的環境の構築)
- 元環境と条件が変ることがある(Dockerはむしろそれが狙い)

これらについて、開発環境と乖離するのはreviewdog系ではかなりうれしくない状態でした。
ですので、最近(?)になり、reviewdogプロジェクトが持つActionはCompositionになっています。

Composition Actionはリポジトリの状態のまま、JavaScriptが走る、と考えると分りやすいかと思います。

textlintもこれに倣いました。

### Result

結果として、速度の向上、ユーザー設定の尊守が行えるようになっています。

## epilogue

v3、オススメです。(ただ、安全のため自分の設定で問題ないかと動作の検証はしてください)
