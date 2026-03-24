OSCP（PEN-200）学習を軸に、ペネトレーションテストの技術・手法をまとめた日本語ノート集。

## 対象読者
- OSCP・OSEP取得を目指している方
- ペネトレーションテストの技術を体系的に学びたい方
- 日本語で攻撃的セキュリティを学びたい方
# 本ノートの記法

- Attacker：攻撃者もしくは攻撃者のマシンのこと（自分のこと）
- Target：被害者もしくは被害者のマシンのこと
- `<xxxx>`：ターゲットの環境に応じた値が入る箇所
- `[xxxx]`: オプションで使用

# 注意

- マシンはKali Linux（Debian 12ベース）を使っているので、使用環境に応じてコマンドを変更する必要がある
	- 例：`sudo apt install...`→`sudo apt-get`

# Tips

- モバイル端末の動機は、[GitSync](https://viscouspotenti.al/posts/gitsync-all-devices-tutorial)を使う
	- GitSyncとObsidian Gitの併用は競合の原因となるので、.gitignoreに、./obsidian/plugin/を追加したうえで、AndroidはGitSyncで、Windows/MacはObsidian.gitで管理

---

# Todo

- [x] AttackerIP、TargetIPを、attacker_IP、target_IPに変える（snakeにする）
- [x] pwとしているのを、passwordに変更する
- [x] ファイルのバックアップを取る
- [x] 不要なファイル（OSCP LABのwriteup、note_template、tryhackme、マインドなど）を削除
- [ ] 個人情報を削除
- [ ] claude, chatgptの文字列で検索かけてそんざいしないか確認する
- [ ] 著作物は改変

---

- ノート整理の現状
	- Cheatsheetまで修正済み
	- Toolsも修正済み
	- Miscも修正済み
	- あとはPEN-200、Windows Internal, READMEの整備、BSCPノートの削除もしくは整理