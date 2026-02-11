
# 本ノートの記法

- Attacker：攻撃者もしくは攻撃者のマシンのこと。（自分のこと。）
- Target：被害者もしくは被害者のマシンのこと。
- $TargetIP：ターゲットのIPアドレスを環境変数で定義しているもの
```zsh
export TargetIP=192.168.10.10
echo $TargetIP
192.168.10.10
```

- `[xxxx]`：ターゲットの環境に応じた値が入る箇所

# 注意

- 画像は同期できないかも。
	- Obsidian gitを使って同期を取れるかもしれないが、容量が大きいので失敗するかも。
- マシンはKali Linux（Debian 12ベース）を使っているので、使用環境に応じてコマンドを変更する必要がある
	- 例：`sudo apt install...`→`sudo apt-get`

# Tips

- モバイル端末の動機は、[GitSync](https://viscouspotenti.al/posts/gitsync-all-devices-tutorial)を使う
- test
test