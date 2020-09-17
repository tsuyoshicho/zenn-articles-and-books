---
title: "Gitのaliasを晒す revised"
emoji: "🐧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---

この記事は  [Gitのaliasを晒す - Qiita](https://qiita.com/tsuyoshi_cho/items/f615dbd4631957334ef3) を改訂したものです。

----

# gitのalias

各所で提案されているaliasを収集、一部修正やら改良やらしたもの。
公開して更新を維持する方法がなかった（プライベートリポジトリでの運用のため）ので、適当なタイミングで公開することにする

なお、個別のファイルにしてあって、`.gitconfig`の`include`設定で取得するようにしている。

`peco`を利用するaliasは、引数があれば短縮ワードとして、そうじゃないなら、`peco`での選択になるようにしてある。
`fzf`にも対応（Windowsでwinptyなしのインタラクティブ動作が安定したこともあり）
あと、その選択もローカルのブランチだけ、とか追跡ブランチも込みとか、ログからのハッシュ選択とかある。

改良が必要な箇所としては、`elif`にしてあるのは`else`でもいいんじゃないか（`peco`がないなら常に短縮動作）とか考えてますが、まだ未着手。

## alias定義

```gitconfig:.gitconfig.aliases
# -*- coding: utf-8 -*-
# vim:fenc=utf-8 ff=unix ft=gitconfig noexpandtab

# fzf/peco support
# need backslash escape? ... currently not use backslash(in branch name)

[alias]
	# based on
	# http://qiita.com/su_k/items/1cd9597eacc20121a5b0
	# https://developers.eure.jp/tech/git-alias/
	aliases = !git config --get-regexp '^alias\\.' | sed 's/alias\\.\\([^ ]*\\) \\(.*\\)/\\1\\\t => \\2/'
	includes = !git config --get-regexp '^include\\.path' | sed 's/include\\.path \\(.*\\)/\\1/'
	# http://qiita.com/takayukioda/items/4fc142aadb3ee7e081c0
	me = !git config --get-regexp '^user' | sed -e 's/^user\\.//' | sed -e 's/ /|/' | column -t -s '|'
	pager-settings = !git config --get-regexp '^pager' | sed -e 's/^pager\\.//' | sed -e 's/ /|/' | column -t -s '|'
	info = "!f () {\
		branch=$(git current-branch);\
		if [ \"${#}\" -eq 0 ]; then\
			git config         branch.${branch}.description 2> /dev/null || echo \"not description yet\";\
		else\
			if [ \"${#}\" -eq 1 ] && [ \"${*}\" == \"\" ]; then\
				git config --local --unset branch.${branch}.description;\
			else\
				git config --local         branch.${branch}.description \"${*}\";\
			fi;\
		fi;\
	};f"
	# basic
	# co = checkout
	# co = like sw checkout alias
	sw = "!f () {\
		checkouted=0;\
		if [ \"${#}\" -eq 1 -a \"${1}\" = \"-\" ]; then\
			git switch -;\
			checkouted=1;\
		elif [ \"${#}\" -eq 1 ]; then\
			match_num=$(git branch | sed 's/^..//' | grep -x \"${*}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				git switch ${*};\
				checkouted=1;\
			fi;\
		fi;\
		common_opt=\"--prompt=switch> --query=${*} --select-1\";\
		if [ 1 -eq \"${checkouted}\" ]; then\
			:;\
		elif which fzf >/dev/null 2>&1; then\
			add_opt=\"--header=local-branch\";\
			if which bat >/dev/null 2>&1; then\
				git branch --sort=authordate | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | bat -l gitlog --color always --style plain'\
					| xargs -r -n 1 git switch;\
			else\
				git branch --sort=authordate | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | cat'\
					| xargs -r -n 1 git switch;\
			fi;\
		elif which peco >/dev/null 2>&1; then\
			git branch --sort=authordate | tr -d ' \\*'\
				| peco ${common_opt}\
				| xargs -r -n 1 git switch;\
		elif [ \"${#}\" -ge 1 ]; then\
			git switch ${*};\
		fi;\
	};f"
	rco = "!f () {\
		checkouted=0;\
		if [ \"${#}\" -eq 1 ]; then\
			match_num=$(git branch -r | sed 's/^..//' | grep -x \"${*}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				git checkout ${*};\
				checkouted=1;\
			fi;\
		fi;\
		common_opt=\"--prompt=checkout> --query=${*} --select-1\";\
		if [ 1 -eq \"${checkouted}\" ]; then\
			:;\
		elif which fzf >/dev/null 2>&1; then\
			add_opt=\"--header=remote-branch\";\
			if which bat >/dev/null 2>&1; then\
				git branch -r --sort=authordate | grep -v 'HEAD' | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | bat -l gitlog --color always --style plain'\
					| sed -e 's/\\([^/]*\\)\\/\\(.*\\)/\\2/g' | xargs -r -n 1 git checkout;\
			else\
				git branch -r --sort=authordate | grep -v 'HEAD' | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | cat'\
					| sed -e 's/\\([^/]*\\)\\/\\(.*\\)/\\2/g' | xargs -r -n 1 git checkout;\
			fi;\
		elif which peco >/dev/null 2>&1; then\
				git branch -r --sort=authordate | grep -v 'HEAD' | tr -d ' \\*'\
					| peco ${common_opt}\
					| sed -e 's/\\([^/]*\\)\\/\\(.*\\)/\\2/g' | xargs -r -n 1 git checkout;\
		elif [ \"${#}\" -ge 1 ]; then\
			git checkout ${*};\
		fi;\
	};f"
	# create branch
	cbr = switch -c
	# merge(branch) fzf/peco selectable
	# support fuzzy match
	# old fzf if [ $# -eq 0 -a `which fzf` ]; then\
	mb = "!f () {\
		merged=0;\
		if [ \"${#}\" -eq 1 ]; then\
			match_num=$(git branch | sed 's/^..//' | grep -x \"${*}\" | wc -l);\
			if [ 1 -eq \"${match_num}\" ]; then\
				git merge ${*};\
				merged=1;\
			fi;\
		fi;\
		common_opt=\"--prompt=merge> --query=${*}\";\
		if [ 1 -eq \"${merged}\" ]; then\
			:;\
		elif which fzf >/dev/null 2>&1; then\
			add_opt=\"--header=all-branch\";\
			if which bat >/dev/null 2>&1; then\
				git branch -a | grep -v '\\->' | sed -e 's/remotes\\///g' | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | bat -l gitlog --color always --style plain'\
					| xargs -r -n 1 git merge;\
			else\
				git branch -a | grep -v '\\->' | sed -e 's/remotes\\///g' | tr -d ' \\*'\
					| fzf  ${common_opt} ${add_opt} --preview='git show {} | cat'\
					| xargs -r -n 1 git merge;\
			fi;\
		elif which peco >/dev/null 2>&1; then\
			git branch -a | grep -v '\\->' | sed -e 's/remotes\\///g' | tr -d ' \\*'\
				| peco ${common_opt}\
				| xargs -r -n 1 git merge;\
		elif [ \"${#}\" -ge 1 ]; then\
			git merge ${*};\
		fi;\
	};f"
	# based on
	# http://qiita.com/ryusukefuda/items/98e7e8d6c3b7486a407d
	# delete branch(es) fzf/peco selectable
	db = "!f () {\
		common_opt=\"--prompt=delete-branch(es)>\";\
		if [ \"${#}\" -eq 0 ] && which fzf >/dev/null 2>&1; then\
			add_opt=\"-m --header=local-branch\";\
			if which bat >/dev/null 2>&1; then\
				git branch --sort=authordate | tr -d ' \\*'\
				| fzf  ${common_opt} ${add_opt} --preview='git show {} | bat -l gitlog --color always --style plain'\
				| xargs -r git branch -d;\
			else\
				git branch --sort=authordate | tr -d ' \\*'\
				| fzf  ${common_opt} ${add_opt} --preview='git show {} | cat'\
				| xargs -r git branch -d;\
			fi;\
		elif [ \"${#}\" -eq 0 ] && which peco >/dev/null 2>&1; then\
			git branch --sort=authordate | tr -d ' \\*'\
			| peco ${common_opt}\
			| xargs -r git branch -d;\
		elif [ \"${#}\" -ge 1 ]; then\
			git branch | sed 's/^..//' | grep \"${*}\" | xargs git branch -d;\
		fi;\
	};f"
	# cherry-pick fzf/peco selectable
	cp = "!f () {\
		common_opt=\"--prompt=cherry-pick>\";\
		if [ \"${#}\" -eq 0 ] && which fzf >/dev/null 2>&1; then\
			add_opt=\"-m --header=commit-log\";\
			if which bat >/dev/null 2>&1; then\
				git log --branches --oneline\
					| fzf  ${common_opt} ${add_opt} --preview='echo {} | awk {print $1} | xargs git show | bat -l gitlog --color always --style plain'\
					| awk '{print $1}' | xargs -r git cherry-pick;\
			else\
				git log --branches --oneline\
					| fzf  ${common_opt} ${add_opt} --preview='echo {} | awk {print $1} | xargs git show | cat'\
					| awk '{print $1}' | xargs -r git cherry-pick;\
			fi;\
		elif [ \"${#}\" -eq 0 ] && which peco >/dev/null 2>&1; then\
			git log --branches --oneline\
				| peco ${common_opt}\
				| awk '{print $1}' | xargs -r git cherry-pick;\
		elif [ \"${#}\" -ge 1 ]; then\
			git cherry-pick ${*};\
		fi;\
	};f"
	# https://developers.eure.jp/tech/git-alias/
	drag = pull --rebase
	# based on
	# https://developers.eure.jp/tech/git-alias/
	refresh = fetch --prune
	# basic
	ci = commit
	# ci interactive
	# below not work
	# ci = "!f () {\
	#		if [ $# -eq 0 -a -e `~/bin/cii.sh` ]; then\
	#			export GIT_EDITOR='bash ~/bin/cii.sh';\
	#			git commit -t ~/bin/cii-empty.txt;\
	#		elif [ $# -ge 1 ]; then\
	#			git commit ${*};\
	#		fi;\
	# };f"
	# based on
	# http://qiita.com/tettekete/items/7019bf7d1bfa883d8549
	st = status --short --branch
	untracked = status --untracked-files=normal
	# based on
	# https://qiita.com/usamik26/items/38bd5b99349d0ae9c4ca
	ignored = status --ignored
	br = branch --sort=authordate
	# log decorate
	# https://developers.eure.jp/tech/git-alias/
	ll        = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --numstat
	# variant
	logline   = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --branches --no-merges --stat
	logstatus = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --branches --no-merges --name-status
	# https://qiita.com/hirotsugu_kawa/items/41afaafe477b877b5b73
	# https://doruby.jp/users/yokian/entries/git-log---graph%E3%81%A7%E3%83%AD%E3%82%B0%E3%82%92%E8%A6%8B%E3%82%84%E3%81%99%E3%81%8F%E3%81%99%E3%82%8B
	# tree = log --graph --all --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(bold white)? %an%C(reset)%C(bold yellow)%d%C(reset)' --abbrev-commit --date=relative
	# tree = log --graph --pretty=format:'%x09%C(auto) %h %Cgreen %ar %Creset%x09by"%C(cyan ul)%an%Creset" %x09%C(auto)%s %d'
	# tree = log --graph --oneline --decorate=full -20 --date=short --format='%C(yellow)%h%C(reset) %C(magenta)[%ad]%C(reset)%C(auto)%d%C(reset) %s %C(cyan)@%an%C(reset)'
	tree  = log --graph --oneline --date=iso                --pretty=format:'%C(yellow)%h%C(reset) %C(magenta)[%ad]%C(reset)%C(auto)%d%C(reset) %C(cyan)@%an%C(reset) %s'
	# http://postd.cc/human-git-aliases/
	# graph = log --graph -10 --branches --remotes --tags --format=format:'%Cgreen%h %Creset? %<(75,trunc)%s (%cN, %ar) %Cred%d' --date-order
	graph = log --graph --oneline --date=iso                --pretty=format:'%h %ad | %s%d [%an]'
	# variant
	hist  = log --graph --oneline --date=short --date-order --pretty=format:'%h %ad | %s%d [%an]'
	# based on
	# https://qiita.com/skoji/items/28f1d6582cf81638cd3f
	# http://neos21.hatenablog.com/entry/2018/05/28/080000
	# https://stackoverflow.com/questions/52123651/how-to-show-whitespace-differences-with-git-word-diff
	wdiff = diff --word-diff-regex='[ ]+|[^ ]+'
	# wdiff = diff --color-words='[ ]+|[^ ]+'

	conflicts = diff --name-only --diff-filter=U

	# based on
	# https://developers.eure.jp/tech/git-alias/
	dr = "!f () {\
		echo \"diff from $1^ to $1\";\
		git diff \"$1\"^..\"$1\";\
	};f"
	# based on
	# https://developers.eure.jp/tech/git-alias/
	find = "!git ls-files | grep -i"
	ls = ls-files
	# http://qiita.com/Yinaura/items/cc10fbc83b4d6bb1ef0c
	branch-root = merge-base master HEAD
	remember = !git diff $(git branch-root)
	# based on
	# http://qiita.com/yaotti/items/0d5364eae36ad1bb8e01
	fix = commit --amend
	merged = "!f () {\
		git branch --merged | grep -v master | grep -v '*' | sed 's/^..//';\
	};f"
	# http://postd.cc/human-git-aliases/
	unmerged = branch --no-merged
	# delete merged branch(es) fzf/peco selectable
	merged-delete = "!f () {\
		common_opt=\"--prompt=delete-branch(es)>\";\
		if [ \"${#}\" -eq 0 ] && which fzf >/dev/null 2>&1; then\
			add_opt=\"-m --header=merged-branch\";\
			if which bat >/dev/null 2>&1; then\
				git merged | tr -d ' \\*'\
				| fzf  ${common_opt} ${add_opt} --preview='git show {} | bat -l gitlog --color always --style plain'\
				| xargs -r git branch -d;\
			else\
				git merged | tr -d ' \\*'\
				| fzf  ${common_opt} ${add_opt} --preview='git show {} | cat'\
				| xargs -r git branch -d;\
			fi;\
		elif [ \"${#}\" -eq 0 ] && which peco >/dev/null 2>&1; then\
				git merged | tr -d ' \\*'\
			| peco ${common_opt}\
			| xargs -r git branch -d;\
		elif [ \"${#}\" -ge 1 ]; then\
			git branch | sed 's/^..//' | grep \"${*}\" | xargs git branch -d;\
		fi;\
	};f"

	# based on
	# https://qiita.com/panti310/items/35d4b820029bab4fad3c
	current-branch = rev-parse --abbrev-ref HEAD
	# push/pull single ... limitation remote name fixed problem.
	# or
	# https://qiita.com/kmszk/items/3de61ef75e30dedd6f6e
	# current-branch = symbolic-ref --short HEAD
	# with lf

	# base on
	# https://qiita.com/upinetree/items/0b74b08b64442f0a89b9
	# use only non-single-branch
	# currently broken
	# parent-branch = "!f () {\
	#		git show-branch | grep '*' | grep -v '$(git current-branch)' | head -1 | awk -F'[]~^[]' '{print $2}'\
	# };f"

	pf = push --force-with-lease

	# http://postd.cc/human-git-aliases/
	branches = branch -a
	tags = tag
	stashes = stash list
	remotes = remote -v
	unstage = reset -q HEAD --
	discard = checkout --
	uncommit = reset --mixed HEAD~
	amend = commit --amend
	# http://postd.cc/git-undo/
	# undo	= "!f () {\
	#		git reset --hard $(git rev-parse --abbrev-ref HEAD)@{${1-1}};\
	# };f"
	undo	= "!f () {\
		git reset --hard $(git current-branch)@{${1-1}};\
	};f"

	# http://qiita.com/konweb/items/621722f67fdd8f86a017
	exclusion-gitignore = "!f () {\
		git rm --cached $(git ls-files --full-name -i --exclude-from=.gitignore)\
	};f"

	# https://itiskj.hatenablog.com/entry/2017/09/07/050000
	# alias fzf/peco selectable
	alex = "!f () {\
		common_opt=\"--prompt=alias>\";\
		if [ \"${#}\" -eq 0 ] && which fzf >/dev/null 2>&1; then\
			add_opt=\"-m --header=aliases\";\
			git aliases\
				| fzf  ${common_opt} ${add_opt}\
				| awk '{print $1}' | cut -f1 -d'=' | xargs -r git;\
		elif [ \"${#}\" -eq 0 ] && which peco >/dev/null 2>&1; then\
			git aliases\
				| peco ${common_opt}\
				| awk '{print $1}' | cut -f1 -d'=' | xargs -r git;\
		elif [ \"${#}\" -ge 1 ]; then\
			git aliases;\
		fi;\
	};f"
	shallow = "!f () {\
		if [ \"${#}\" -eq 0 ]; then\
			git fetch --depth=100  && git prune && git gc;\
		else\
			git fetch --depth=${*} && git prune && git gc;\
		fi;\
	};f"

	fixup = commit --fixup HEAD
	fixing = "!f () {\
		GIT_SEQUENCE_EDITOR=':' git rebase -i $(git branch-root);\
	};f"

	coauthoradd = "!f () {\
		if [ \"${#}\" -gt 1 ]; then\
			echo \"not support argment greater than 1\";\
		else\
			output=$(git log -1 --pretty='%b');\
			test -z \"${output}\";\
			hasbody=${?};\
			output=$(git log -1 --pretty='%(trailers:key=Co-authored-by)');\
			test -z \"${output}\";\
			hastrailers=${?};\
			outputformat=\"%s\";\
			newline=\"\n\n\";\
			if [ \"${hasbody}\" -eq 1 ]; then\
				outputformat=\"%s%n%n%b\";\
				if [ \"${hastrailers}\" -eq 1 ]; then\
					newline=\"\n\";\
				fi;\
			else\
				newline=\"\n\n\";\
			fi;\
			common_opt=\"--prompt=author> --query=${*}\";\
			if which fzf >/dev/null 2>&1; then\
				add_opt=\"--header=co-author\";\
				if which bat >/dev/null 2>&1; then\
					coauthor=$(git log -1000 --pretty=\"%an <%ae>, %ae\"\
						| sort -f -k 2 -t , | uniq -i -f 1 | cut -d, -f1\
						| fzf ${common_opt} ${add_opt} --preview='git show | bat -l gitlog --color always --style plain');\
				else\
					coauthor=$(git log -1000 --pretty=\"%an <%ae>, %ae\"\
						| sort -f -k 2 -t , | uniq -i -f 1 | cut -d, -f1\
						| fzf ${common_opt} ${add_opt} --preview='git show');\
				fi;\
				if [ -n \"${coauthor}\" ]; then\
					printf \"%s%sCo-authored-by: %s\" \"$(git log -1 --pretty=${outputformat})\" \"${newline}\" \"${coauthor}\"\
						| git commit --file - --amend;\
				fi;\
			elif which peco >/dev/null 2>&1; then\
				coauthor=$(git log -1000 --pretty=\"%an <%ae>, %ae\"\
					| sort -f -k 2 -t , | uniq -i -f 1 | cut -d, -f1\
					| peco ${common_opt});\
				if [ -n \"${coauthor}\" ]; then\
					printf \"%s%sCo-authored-by: %s\" \"$(git log -1 --pretty=${outputformat})\" \"${newline}\" \"${coauthor}\"\
						| git commit --file - --amend;\
				fi;\
			fi;\
		fi;\
	};f"
# EOF
```

## 注

statusは以下の設定があり、untrackedを普段は見ないように定義している。

```gitconfig:.gitconfig(部分)
[status]
        # http://qiita.com/tettekete/items/7019bf7d1bfa883d8549
        showuntrackedfiles = no
```

## 後記

この記事での[gist](https://gist.github.com/tsuyoshicho/0a8610e87f209f5e86bc)を利用するって手はあるけど、ファイルメンテナンスをローカルでもgistでもできなくて、記事自体で書き換えが必要になるのは、ちょっと辛いな...

## 後記2

Qiitaのgist連携が止りました。

## 後記3

Zennへ移植改訂しました。

