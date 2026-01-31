---
title: "GitHub Copilotを使う (事例: git のエイリアスの改善)"
emoji: 🤖
type: tech
topics:
  - githubcopilot
  - git
  - vscode
  - agent
published: true
published_at: 2026-02-01 00:00
---
# Summary (概要)

この記事は、次のような経緯の雑文です。

1. 仕事でちょっと Agent 活用(VSCode + GitHub Copilot)してみた
2. それで慣れたので、自分のローカルでも使ってみた(GitHub の Free版)
3. 感想と結果

一応 tech にはなってますが、限りなく idea 寄り(でも思想はない)です。

# Detail (詳細)

## Reason (開発経緯)

最近、AI 連係での `git worktree` 連係の話題とかもありました。
また、普段の作業で、「あ、worktree だといいかも」という場面もちょっとありました。

それも合せて、自分の `git` の alias に worktree 系の便利なの用意したいな、というがありました。

それに、最近業務で VSCode での GitHub Copilot (ただエンタープライズ的に契約したもの: 詳細は伏せ)を利用する機会がありました。
それがけっこう便利だし、活用もできそうだというところでしたので、これでやってみました。

まあ、alias のシェル関数、とにかく手間で面倒なだけで、既存と同じ感じになる予定でした。
それはそれなりに面倒くささもあったので、やらせて楽したいというのがあったりします。

## Link (リンク)

すみませんがプライベートに置いているため、ファイルへのリンクはありません。
追加分については Result 節に記載はしているので、参考まで。

# Workflow (作業の流れ)

## Plan (計画)

作りたいと思っていたものですが。最終的な結果から逆算で記載します。
作っている途中であれこれ考えが変化したのもありますが、それをしっかりは覚えてはいないので。

仕様は成果として最後に記載します。

* `git wt` : `git worktree` のショートカット (これは手動で作成)
* `git wtc` : `git clone` の worktree 向け便利化版
* `git wtbr` : `git worktree add` のリモートブランチ直処理、便利版
* `git wtbl` : `git worktree add` のローカルブランチ処理、便利版
* `git wtrm` : `git worktree remove` の便利版

作りについては、自分が以前に作っていた各種エイリアスに有用な機能があります。
それを Copilot 君の参考にさせつつ実装させる感じです。

## Issues (課題点)

対話しつつ、設計(Plan)と実装と改良(Agent)、テストは手動で進めました。
問題として以下のようなことがありました。

順序はあまり覚えていないですが、Q&Aというか、問題と回答形式で記述します。

作業は前述する GitHub Copilot (Free 版のもの) です。
対比として出ている業務で利用したものは GitHub Copilot (だけど AI モデルは Claude Sonnet 4.5 (token rate x1)だったはず) です。

### Issue 1: テストは手動

これは、自分の dotfiles をじかに修正する都合もあります。(別でやることも不可能ではないでしょうけど、面倒だった)

ブランチ操作系ですので、その場で試させると現状を壊してしまいます。
そのため、これは別リポジトリで手動の確認をしました。
それによってアシストというかコワークする感じになりました。

### Issue 2: `git alias` の関数の都合 1

これについては、仕様が面倒というのが前提ですが...。

`git` の alias は、つまるところ、設定ファイルのエントリに設定した文字列 ("...") です。
その中にシェルスクリプトとして関数を押し込み、処理を実施させています。

この都合、複数行表記に見えるように書けますが "\"" (バックスラッシュ) を末尾に置き、行を継続させて
1つの行(文字列)にする必要があります。

Copilot は最初に読んだ時点で理解してやってくれましたが、改良の途中で忘れたりしました。

また、それでエラーしたことを報告すると修正しますが、修正が失敗("\\\" にしてしまっていました)したり、それの再修正でも失敗したり漏れてましたね...。

### Issue 3: `git alias` の関数の都合 2

2 に付随してですが、処理中のクオート("...")についてもエスケープしないといけないのに、これ
も改良の途中でうっかりエスケープ漏れをやってくれたりしました。

2 と合せて修正させました。

ここらへんから、現状のエージェントの能力が、業務で使えてた Sonnet 4.5 と比べて低い(というか実用レベルに上りきっていない)というのは感じました。
まったく使えないということはないんですが。

### Issue 4: 編集ミスによる無関係な変更

修正をやりとりしてましたが、手法自体は Sonnet 4.5 と同じ正規表現検知と置換なのですが...。
どしても不十分なのか無関係なところまで修正してしまい、ファイルとして壊れてしまいました。

指示して直すにしてもいろいろ微妙な所がありますので、この時点の作成内容を退避させ、`git restore` 後に書き戻して続行しました。

さすがに一度しか発生してないし、次にその課題に対応した作業をしたので再発はなかったです。
(Issue x での対応がそれ)

### Issue 5: 複数の課題を伝えると、修正が不十分、あるは忘れる

Sonnet 4.5 と比べると弱いのはしょうがないのですが、コンテキスト的に厳しいのか、表題がそれなりに発生しました。

1. me: xx と yy という問題があるみたい、修正してください
2. co: xx について修正しました(不十分)

この時点で yy についてはあまり覚えておらず、「zzという感じで不十分、続けて修正して」といっても、yy については対応せず。

やり方で対応はできると考えますが、作成対象も大きい予定ではなかったので、泥縄で対応し続けて完了まで持っていきました。
次やるときは、もうちょっとうまくやりたいですね。

### Issue 6: 修正の内容と実施が場当たり気味

Sonnet 4.5 とかだと、良きに修正する感じで対応できてたのですが、どうにも部分修正すぎてうまく行っていませんでした。

ある程度作りができた時点で、それを PoC ということにして、Plan に完全に再検討させました。

再検討時は、次の要件で行いました。

* いままでのやりとりで、決まった仕様
* あらためて alias のファイル全体を参照
  * 既存の構造、機能をうまく利用するように
* 前の実装も参照

この内容で完全に作りなおさせ、Agent に修正を適用させるときも、部分変更ではなく Plan で決めた実装にまるまる差し替えるように指示しました。

これで微修正しようとして変になるのを回避しました。

# Result (成果)

## Product (生成物)

:::details 作った alias

```gitconfig
	# worktree short
	wt = worktree

	# worktree clone - clone repository into a worktree with default branch
	wtc = "!f () {\
		if [ \"${#}\" -ne 1 ]; then\
			echo \"not support argument not equal 1\";\
			return;\
		fi;\
		repo_url=\"${1}\";\
		folder_name=\"$(basename \"${repo_url}\" .git)\";\
		default_branch=\"$(git ls-remote --symref \"${repo_url}\" HEAD 2>/dev/null | head -1 | awk -F'/' '{print $NF}' | awk '{print $1}')\";\
		if [ -z \"${default_branch}\" ]; then\
			echo \"failed to get default branch\";\
			return;\
		fi;\
		mkdir -p \"${folder_name}\";\
		git clone -b \"${default_branch}\" \"${repo_url}\" \"${folder_name}/${default_branch}\";\
	};f"

	# worktree branch local - create a worktree for a local tracking branch
	wtbl = "!f () {\
		branch=\"${1}\";\
		if [ \"${#}\" -eq 1 ]; then\
			match_num=$(git branch --format \"%(refname:short)\" | grep -x \"${branch}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				current_branch=$(git current-branch);\
				if [ \"${branch}\" = \"${current_branch}\" ]; then\
					echo \"error: cannot create worktree for current branch '${branch}'\";\
					return;\
				fi;\
				if git worktree list | awk 'NR>1 {print $NF}' | grep -q \"^${branch}$\"; then\
					echo \"error: branch '${branch}' is already used by a worktree\";\
					return;\
				fi;\
				git worktree add \"../${branch}\" \"${branch}\" && echo \"worktree added: ../${branch}\";\
				return;\
			fi;\
		fi;\
		existing_branches=$(git worktree list | awk 'NR>1 {print $NF}' | grep -v '(detached)' | sort);\
		current_branch=$(git current-branch);\
		common_opt=\"--prompt=worktree-branch> --query=${branch} --select-1\";\
		if which fzf >/dev/null 2>&1; then\
			header_opt=\"--header=local-branch\";\
			if which bat >/dev/null 2>&1; then\
				branch=$(git branch --format \"%(refname:short)\" | grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") | grep -v \"^${current_branch}$\" | fzf ${common_opt} ${header_opt} --exit-0 --preview='git show {} | bat -l gitlog --color always --style full' --preview-window '~3' 2>/dev/null);\
			else\
				branch=$(git branch --format \"%(refname:short)\" | grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") | grep -v \"^${current_branch}$\" | fzf ${common_opt} ${header_opt} --exit-0 --preview='git show {} | cat' 2>/dev/null);\
			fi;\
		elif which peco >/dev/null 2>&1; then\
			branch=$(git branch --format \"%(refname:short)\" | grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") | grep -v \"^${current_branch}$\" | peco ${common_opt} 2>/dev/null);\
		else\
			return;\
		fi;\
		[ -z \"${branch}\" ] && return;\
		git worktree add \"../${branch}\" \"${branch}\" && echo \"worktree added: ../${branch}\";\
	};f"

	# worktree branch from remote - create a worktree from a remote tracking branch
	wtbr = "!f () {\
		branch=\"${1}\";\
		if [ \"${#}\" -eq 1 ]; then\
			match_num=$(git branch -r --format \"%(refname:short)\" | sed 's|^origin/||' | grep -x \"${branch}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				if git branch --format \"%(refname:short)\" | grep -q \"^${branch}$\"; then\
					echo \"error: local branch '${branch}' already exists\";\
					return;\
				fi;\
				if git worktree list | awk 'NR>1 {print $NF}' | grep -q \"^${branch}$\"; then\
					echo \"error: branch '${branch}' is already used by a worktree\";\
					return;\
				fi;\
				git worktree add -b \"${branch}\" \"../${branch}\" \"origin/${branch}\" && echo \"worktree added: ../${branch} (from origin/${branch})\";\
				return;\
			fi;\
		fi;\
		existing_branches=$(git worktree list | awk 'NR>1 {print $NF}' | grep -v '(detached)' | sort);\
		local_branches=$(git branch --format \"%(refname:short)\" | sort);\
		common_opt=\"--prompt=remote-branch> --query=${branch} --select-1\";\
		if which fzf >/dev/null 2>&1; then\
			header_opt=\"--header=remote-branch\";\
			if which bat >/dev/null 2>&1; then\
				branch=$(git branch -r --format \"%(refname:short)\" \
					| sed 's|^origin/||' | sed 's| -> .*||' | grep -v '^HEAD' | grep -v '^origin$' \
					| grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") \
					| grep -Fvxf <(printf \"%s\\n\" \"${local_branches}\") \
					| fzf ${common_opt} ${header_opt} --exit-0 --preview='git show origin/{} | bat -l gitlog --color always --style full' --preview-window '~3' 2>/dev/null);\
			else\
				branch=$(git branch -r --format \"%(refname:short)\" \
					| sed 's|^origin/||' | sed 's| -> .*||' | grep -v '^HEAD' | grep -v '^origin$' \
					| grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") \
					| grep -Fvxf <(printf \"%s\\n\" \"${local_branches}\") \
					| fzf ${common_opt} ${header_opt} --exit-0 --preview='git show origin/{} | cat' 2>/dev/null);\
			fi;\
		elif which peco >/dev/null 2>&1; then\
			branch=$(git branch -r --format \"%(refname:short)\" \
				| sed 's|^origin/||' | sed 's| -> .*||' | grep -v '^HEAD' | grep -v '^origin$' \
				| grep -Fvxf <(printf \"%s\\n\" \"${existing_branches}\") \
				| grep -Fvxf <(printf \"%s\\n\" \"${local_branches}\") \
				| peco ${common_opt} 2>/dev/null);\
		else\
			return;\
		fi;\
		[ -z \"${branch}\" ] && return;\
		git worktree add -b \"${branch}\" \"../${branch}\" \"origin/${branch}\" && echo \"worktree added: ../${branch} (from origin/${branch})\";\
	};f"

	# worktree remove - remove a worktree (excluding current) and associated local branch
	wtrm = "!f () {\
		search_path=\"${1}\";\
		current_worktree=$(git worktree list | awk 'NR==1 {print $1}');\
		worktree_list=$(git worktree list | awk 'NR>1 {print $1}');\
		if [ \"${#}\" -eq 1 ]; then\
			match_num=$(printf \"%s\\n\" \"${worktree_list}\" | grep -x \"${search_path}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				if [ \"${search_path}\" = \"${current_worktree}\" ]; then\
					echo \"error: cannot remove current worktree '${search_path}'\";\
					return;\
				fi;\
				branch_name=$(git -C \"${search_path}\" rev-parse --abbrev-ref HEAD 2>/dev/null);\
				git worktree remove \"${search_path}\" && echo \"worktree removed: ${search_path}\";\
				if [ -n \"${branch_name}\" ] && [ \"${branch_name}\" != \"HEAD\" ]; then\
					is_remote_tracking=$(git for-each-ref refs/remotes/origin/ --format=\"%(refname:short)\" | sed 's|^origin/||' | grep -x \"${branch_name}\" | wc -l);\
					if [ 1 -eq \"${is_remote_tracking}\" ]; then\
						git branch -D \"${branch_name}\" 2>/dev/null && echo \"branch deleted: ${branch_name}\";\
					else\
						echo \"local-only branch (not deleted): ${branch_name}\";\
					fi;\
				fi;\
				return;\
			fi;\
		fi;\
		common_opt=\"--prompt=remove-worktree> --query=${search_path}\";\
		if which fzf >/dev/null 2>&1; then\
			header_opt=\"--header=worktree-path\";\
			worktree_path=$(printf \"%s\\n\" \"${worktree_list}\" | grep -v \"^${current_worktree}$\" | fzf ${common_opt} ${header_opt} --exit-0 --preview='git -C {} rev-parse --abbrev-ref HEAD' --preview-window '~2' 2>/dev/null);\
		elif which peco >/dev/null 2>&1; then\
			worktree_path=$(printf \"%s\\n\" \"${worktree_list}\" | grep -v \"^${current_worktree}$\" | peco ${common_opt} 2>/dev/null);\
		else\
			return;\
		fi;\
		[ -z \"${worktree_path}\" ] && return;\
		branch_name=$(git -C \"${worktree_path}\" rev-parse --abbrev-ref HEAD 2>/dev/null);\
		git worktree remove \"${worktree_path}\" && echo \"worktree removed: ${worktree_path}\";\
		if [ -n \"${branch_name}\" ] && [ \"${branch_name}\" != \"HEAD\" ]; then\
			is_remote_tracking=$(git for-each-ref refs/remotes/origin/ --format=\"%(refname:short)\" | sed 's|^origin/||' | grep -x \"${branch_name}\" | wc -l);\
			if [ 1 -eq \"${is_remote_tracking}\" ]; then\
				git branch -D \"${branch_name}\" 2>/dev/null && echo \"branch deleted: ${branch_name}\";\
			else\
				echo \"local-only branch (not deleted): ${branch_name}\";\
			fi;\
		fi;\
	};f"
```

:::

## Specification (仕様)

特徴を整理すると、以下になります。

* `git wtc` : `git clone` するが、worktree 展開に便利なフォルダ構成で行う
* `git wtbr` : リモートのブランチをじかに `git worktree add` する
  * すでに worktree にしてれば対象外にする
  * ファジーマッチ(単一なら即座)
  * ジャストマッチ(exact match、すれば即座)
  * ファジーファインダーによるインタラクティブ選択
    * 選択するブランチのコミットログプレビュー
    * ESC すればなにもしない
* `git wtbl` : ローカルブランチを `git worktree add` する
  * 現状でカレントのブランチは対象外にする
  * すでに worktree にしてれば対象外にする
  * ファジーマッチ(単一なら即座)
  * ジャストマッチ(exact match、すれば即座)
  * ファジーファインダーによるインタラクティブ選択
    * 選択するブランチのコミットログプレビュー
    * ESC すればなにもしない
* `git wtrm` : 指定した worktree 化したブランチを `git worktree remove` する
  * worktree したものが対象
  * 現在のフォルダのカレントのブランチは対象外にする
  * ジャストマッチ(exact match、すれば即座)
  * 削除したとき、リモート追跡ブランチも合せて削除する
  * ファジーマッチ(しても選択画面へいく)
  * ファジーファインダーによるインタラクティブ選択
    * 選択するブランチのコミットログプレビュー
    * ESC すればなにもしない
    * 不用意な削除をしないようにしている

ファジーファインダーは自分の都合、`fzf` と `peco` のを記載はしています。
`peco` はほぼ使ってませんが...。

まだ問題は残っている可能性はありますが、いちおう確認した範囲で実装できてます。
まだ、コードレビュー的に見てもちゃんと入れているので、大丈夫なはず。

## Usage (使い方)

`git wtc` で clone すると、"<repo name>/<default branch name>/" へ clone します。 ("<repo name>/" は作成される)
こうすることで、worktree でブランチ名で別フォルダがどんどん作るのと、普通のリポジトリの混在を防げます。

この状態で、"<repo name>/<default branch name>/" で `git wtbr` でリモートのブランチを worktree として切り出したり、作ったブランチについて `git wtbl` で worktree 化したりできます。
位置は "<repo name>/<branch name>/" つまり "../<branch name>/" として作るようにしています。

`git wtrm` で worktree とともに作業フォルダの削除もできますが、このときは安全と自己責任両方がある設計としています。

ファジーに1つだけマッチする条件では、安全のために即座処理はせず、`fzf` 画面を挟むようにしました。

逆に削除するなら、`git worktree remove` の仕様でもありますが、フォルダごと消します。
rm してから `git worktree prune` を手動でやるより危険ですが、それは自己責任です。
また、残るブランチでのトラブル回避のため、リモート追跡ブランチも合せて削除します。

(なお、リモート追跡ブランチというとちょっと用語的に正しくないです、詳しくはその点について言及されている[記事](https://qiita.com/uasi/items/69368c17c79e99aaddbf)を参照)

# Additional notes (付記)

なお、この作業だけで、1期間の利用 limit に到達しました。
Free だと枠(というかトークン量)は小さいですね。
