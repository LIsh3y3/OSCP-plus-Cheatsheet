- 関連ノート：
	- [コンパイル・ビルド](../../Misc/コンパイル・ビルド.md#Python%20Package%20Management%20(pip))
- 参照🔗：[Git's Reference page](https://git-scm.com/docs).

---

# Local Git Enumeration

- シチュエーション：ターゲット侵害後、ターゲットがGitでWebサイトを管理しているとき

- 知識：`.git`が保存されているディレクトリは、Gitで管理するディレクトリのルートディレクトリに相当する

gitのワーキングディレクトリのステータス表示
```zsh
git status
```

コミットログを表示する
```zsh
git log
```
- 凡例：🟥赤：リモートリポジトリ、🟩緑：ローカル

![](../../Images/Pasted%20image%2020260102123346.png)

$$log出力例$$

- 出力形式
```zsh
commit <commit_hash> (HEAD -> master)
Author: xx
Date: Fri Nov 18 16:58:34 2022 <UTCから±何時間>

  <コミットメッセージ>
```
- Dateに着目する
	- 上記の例では、リモートよりもローカルが進んでいる
	- ローカルは-0500とあるため、16:58:34に+5時間でUTCとなり、21:58:34
	- リモートは+0200とあるため、23:57:12に-2時間でUTCとなり、21:57:12
	- →つまり、ローカルの編集内容をリモートのmainブランチにpushしていない
- （`initial commit`は最初にコミットされたというメッセージであるだけなので無視）

コミット間の差分を表示する
```zsh
git diff <commit_hash1> <commit_hash2>
```

コミット内容を表示する
```zsh
git show <commit_hash>
```

ターゲットマシンのOS判定 w/git
```zsh
git version
```
- →Windowsなら出力の中に"Windows"と表示、Unixなら特になし。

---

# Remote Git Enumeration

- シチューエーション：NmapやGobusterの結果、WebサイトがGitを利用していることがわかったとき
```zsh
# Nmap出力例
...
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Last commit message: Security Update 
|     Remotes:
...
```

1. 🔗[git-dumper - GitHub](http://github.com/arthaud/git-dumper)のインストール
```zsh
pipx install git-dumper
```

2. dump
```zsh
# 例：git-dumper http://192.168.45.157/.git WebEnum/GitDumper
git-dumper <url> <output_dir>
```

3. [Local Git Enumeration](#Local%20Git%20Enumeration)の手法で列挙
