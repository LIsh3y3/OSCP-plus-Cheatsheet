- 関連ノート：
	- [Tunneling Through Deep Packet Inspection](Tunneling%20Through%20Deep%20Packet%20Inspection.md)

---

# Port Redirectionの基礎

Port Redirection ＝ Port Forwarding

## Port Redirectionの必要性

- ネットワークは分割（セグメント化）され、サブネット間はFWなどで通信経路が制御されているため、攻撃者は「直接つながらないサービス」 にアクセスする必要が出る
- **想定シナリオ**
    - 内部のサーバーに外部から直接アクセスできない
    - FWが特定ポートをブロックしている
    - 踏み台サーバ（jump host）経由でしか内部リソースに触れないが、踏み台サーバには必要なツール（DBクライアント等）がインストールされていないし、インストールするための権限もない ...etc.

- そこで利用するのが **Port Redirection / Tunneling**  
	- →ネットワークの制限を「迂回」して、目的のサービスへ到達できるようにする

## Port Forwardingの基本原理

リモートポートフォワーディング
	リッスンポートで受信したパケットをリモート端末のポートに転送すること。

![](../../../画像ファイル/Pasted%20image%2020260314091225.png)

$$リモートポートフォワーディングのイメージ$$

ローカルポートフォワーディング：
	リッスンポートで受信したパケットをローカル端末のポートに転送すること

![](../../../画像ファイル/Pasted%20image%2020260314091325.png)

$$ローカルポートフォワーディングのイメージ$$

---

# LinuxツールによるPort Forwading w/Socat

- Socatがインストールされていない場合は、🔗[socat - 3ndG4me](https://github.com/3ndG4me/socat/releases)からスタンドアロンバイナリを攻撃者のマシンにダウンロードし、ターゲットに転送する
- このような単純なポートフォワーディングは検知・遮断されやすい

1. Jump Host上でリモートポートフォワーディング
```zsh
# 下図例：socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
socat -ddd TCP-LISTEN:<listen_port>,fork TCP:<Dest_IP>:<port>
```
- `-ddd`：詳細出力
- `fork`：指定することで、１回きりの通信で接続断しないようになる
- Jump Hostのリッスンポートは1,024超えのwell-knownポート以外を指定すること

![](../../../画像ファイル/Pasted%20image%2020250920195209.png)

$$Socatポートフォワーディング成功時出力例$$

2. 攻撃者のマシンからJump Host上のリッスンポートにアクセスすることで、フォワーディング先のtarget_IPにアクセス可能

>[!NOTE]
>他のポートに対してポートフォワーディングをするためには、Ctrl + Zでターミナルを閉じ、再度ターミナルに接続後、既存のsocatプロセスを`kill`する。

---

# SSH Port Forwardingの基礎

SSHセッションで通信を==カプセル化（暗号化）==してポートフォワーディングする。

- **必要条件**：
	- FWにSSH通信が許可されている
	- インタラクティブなシェルが使える
		- WinRM接続中など、インタラクティブなシェルが使えない場合は、
			- (a) Unix系：`python3 -c 'import pty; pty.spawn("/bin/sh")'`
			- (b) Windows：[Plink](#Plink)や[Netsh](#Netsh)を使う

- ✅通常のネットワークトラフィックと見分けがつきにくい
- ❌近年のFWに実装されているDeep Packet Inspection(DPI)機能により、悪意ある通信は検知され遮断されてしまう
	- →HTTP通信が許可されている場合は[Tunneling Through Deep Packet Inspection](Tunneling%20Through%20Deep%20Packet%20Inspection.md)を使えば、DPIは回避可能

## SSHポートフォワーディングの種類と概要

単純なポートフォワーディングでは、パケットをリッスンするホスト自身(下図 Jump Host01 )がパケットを転送するが、SSHポートフォワーディングでは、リッスンする役割を担う機器（Jump Host 01）と、転送する役割を担う機器（Jump Host 02）の、合計２つのマシンを使ってポートフォワーディングをする

### SSHローカルポートフォワーディング

- 概要：ローカルマシン(SSHクライアント Jump Host01 )のポートを、SSH サーバー経由で別のサーバーのポートに転送する

![](../../../画像ファイル/Pasted%20image%2020260314095420.png)

$$SSHローカルポートフォワーディングのイメージ図$$

### SSHローカルダイナミックポートフォワーディング

- 概要：ローカルマシン(SSHクライアント)のポートを、SSH サーバー経由で別のサーバーの任意ポートに転送する

![](../../../画像ファイル/Pasted%20image%2020260314094003.png)

$$SSHローカルダイナミックポートフォワーディングのイメージ図$$

### SSHリモートポートフォワーディング

- 概要：SSH サーバー側のポートを、ローカルマシン(SSHクライアント)経由で別のサーバーのポートに転送する
- 用途：踏み台を介して社内NWにアクセスすることや、FWをバイパスして特定のポートにアクセスすることがある

- ポイント：
	1. SSHクライアント(Jump Host01）を足場のマシンとし、SSHサーバーを攻撃者のマシン(Attacker)とし、SSH接続開始
	2. 攻撃者のマシンのポートに送信されたパケットが、ステップ１で確立したSSH接続を介してSSHクライアントに送付され、そこから目的のマシンに到達

![](../../../画像ファイル/Pasted%20image%2020260314094640.png)

$$SSHリモートポートフォワーディングのイメージ図$$

### SSHリモートダイナミックポートフォワーディング

- 概要：SSHサーバー側のポートを、ローカルマシン(SSHクライアント)経由で別のサーバーの任意ポートに転送する

>[!TIP]
>SSH Port Forwardingの中では**最も成功率が高い**ので、これを優先的に使う。

![](../../../画像ファイル/Pasted%20image%2020260314094922.png)

$$SSHリモートダイナミックポートフォワーディングのイメージ図$$

---

# SSH Local Port Forwarding

>[!NOTE]
> - FWのルールは、アウトバウンドトラフィックには比較的寛容に設定されているが、インバウンドトラフィックには厳しいことが多い
> - Local Port Forwardingはインバウンドトラフィックが許可されていないと使えないので失敗することが多い(SSHクライアントで任意のリッスンポートを開いても、攻撃者のマシンからのリッスンポートへの通信がFWで遮断される)
> - →[SSH Remote Port Forwarding](#SSH%20Remote%20Port%20Forwarding)のほうが成功確率高い

1. SSHクライアント上で非インタラクティブシェルであれば、インタラクティブシェルへ切り替える
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

2. SSHクライアント上でローカルポートフォワーディング
	- 出力はなし
```zsh
# 例：ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215
ssh -N -L 0.0.0.0:<SSH_client_listen_port>:<target_IP>:<port> <SSH_server_username>@<SSH_server_IP>
```
- `-N`：ポートフォワーディングのために使用することを明示し、新たなシェルが開かないようにする

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

4. 攻撃者のマシンからSSHクライアントのリッスンポートに接続することで、TargetマシンのPortにアクセス可能

---

# SSH Local Dynamic Port Forwarding

- [SSH Local Port Forwarding](#SSH%20Local%20Port%20Forwarding)と同じく、現実では失敗する可能性が高い
	- →[SSH Remote Dynamic Port Forwarding](#SSH%20Remote%20Dynamic%20Port%20Forwarding)

1. SSHクライアント上で非インタラクティブシェルであれば、インタラクティブシェルへ切り替える
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

2. SSHクライアント上でローカルダイナミックポートフォワーディング（出力はなし）
```zsh
# 例：ssh -N -D 0.0.0.0:9999 database_admin@10.4.50.215
ssh -N -D 0.0.0.0:<SSH_client_listen_port> <SSH_server_username>@<SSH_server_IP>
```

3. 攻撃者のマシン上で、Proxychainsの設定を変更する
```zsh
sudo nano /etc/proxychains4.conf
```
	↓
```zsh
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 <SSH_client_IP> <SSH_client_listen_port>
```
- ProxyListはstrict_chainモード(デフォルト)では１つにしておく必要がある

>[!WARNING]
>Proxychainsは関数をフックする仕組みを使うので、静的リンクされたバイナリには使えない。

4. 攻撃者のマシンから、コマンドの先頭に`proxychains`をつけて実行することで、任意のポートにフォワーディングできる（SSHクライアントのIP/ポートは指定不要）
```zsh
# 例：Nmapの場合（sTでないとProxyChains経由で動作しない）
sudo proxychains nmap -vvv -sT --top-ports=20 -Pn <target_IP>
# 例：smbclientの場合
sudo proxychains smbclient -L //<target_IP>/ -U <username> --password=<password>
```

>[!TIP]
>`proxychains nmap`の速度が遅い場合は、Proxychainsの設定で以下の２つの値を小さくする。
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

2. SSHクライアント上で非インタラクティブシェルであれば、インタラクティブシェルへ切り替える
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

3. SSHクライアント上でリモートポートフォワーディング（出力はなし）
```zsh
ssh -N -R 127.0.0.1:<SSH_server_listen_port>:<target_IP>:<port><SSH_server_username(Attacker)>@<SSH_server_IP>
```

4. 攻撃者のマシンでリモートポートフォワーディングが成功しているかどうかを確認
```zsh
ss -tulpn
```
	↓
```zsh
Netid State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess
tcp   LISTEN 0      128        127.0.0.1:<listen_port>      0.0.0.0:*
```

5. 攻撃者のマシンから攻撃者マシンのリッスンポートに接続することで、Targetのportにアクセス可能
```zsh
# 例：postgresql
psql -h 127.0.0.1 -p 2345 -U postgres

# 例：ssh
ssh database_admin@127.0.0.1 -p 2222
```

---

# SSH Remote Dynamic Port Forwarding

1. 攻撃者のマシンでSSHサーバーをスタートする
```zsh
sudo systemctl start ssh
```

2. SSHクライアント上で非インタラクティブシェルであれば、インタラクティブシェルへ切り替える
```zsh
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

3. SSHクライアント上でリモートダイナミックポートフォワーディング（出力はなし）
```zsh
ssh -N -R <SSH_server_listen_port> <SSH_server_username(Attacker)>@<SSH_server_IP>
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

5. 攻撃者のマシン上で、Proxychainsの設定を変更する
```zsh
sudo nano /etc/proxychains4.conf
```
	↓
```zsh
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 127.0.0.1 <SSH_server_listen_port>
```
- ProxyListはstrict_chainモード(デフォルト)では１つにしておく必要がある

>[!WARNING]
>Proxychainsは関数をフックする仕組みを使うので、静的リンクされたバイナリには使えない。

6. 攻撃者のマシンから、コマンドの先頭に`proxychains`をつけて実行することで、任意のポートにフォワーディングできる
```zsh
# 例：Nmapの場合
sudo proxychains nmap -vvv -sT --top-ports=20 -Pn -n <target_IP>
```

>[!NOTE]
>`target_IP`には、Jump Host 01からネットワーク情報を列挙して発見した、**内部NW側**のインターフェースを使用する。たとえば、TargetがNICを2つもっていて、222.111.123.231という外部からアクセスできるIPと192.168.1.1という内部からのみアクセスできるIPをもっていたら、`target_IP`には192.168.1.1を指定する。


![](../../../画像ファイル/Pasted%20image%2020260314100618.png)

$$リモートダイナミックポートフォワーディングによってFWを回避するイメージ$$

>[!TIP]
>`proxychains nmap`の速度が遅い場合は、Proxychainsの設定で以下の２つの値を小さくする。
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

# Dynamic Port Forwardingの簡素化 w/🔗[sshuttle](https://github.com/sshuttle/sshuttle)

- sshuttleとは
	- SSH をトンネルとして利用し、特定のサブネットへのトラフィックをすべてそのトンネルに流す仕組みを持つ
	- ユーザーにとっては**VPN で接続したかのように内部ネットワークを列挙できる**ように見える

- 条件：
	- SSHクライアント(Jump Host)にroot権限：iptables操作やパケットのキャプチャのため
	- SSHサーバーにPython3：リモートでコードを実行するためのtty

1. Jump Hostでリモートポートフォワーディング
```zsh
socat -ddd TCP-LISTEN:<listen_port>,fork TCP:<target_IP>:<port>
```

2. 攻撃者のマシン上でsshuttleを使用し、トンネルをしたい**サブネット**を指定
```zsh
# 例：sshuttle -r database_admin@192.168.50.63:2222 10.4.50.0/24 172.16.50.0/24
sshuttle -r <jump_host_ssh_username>@<jump_host_IP>:<listen_port> <target_subnet>
```

3. 指定したサブネット内のIPに対し任意の操作
```zsh
# smbclientの場合
smbclient -L //<target_IP>/ -U <username> --password=<password>
```

---

# WindowsツールによるPort Forwaring

## ssh.exe

- 条件：windowsマシンにOpenSSHクライアント(ssh.exe)がインストールされているときに使用可能

1. 攻撃者のマシンでsshをスタート
```zsh
sudo systemctl start ssh
```

2. SSHクライアントであるwindowsマシンにssh.exeがあることを確認する
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
- リモートダイナミックポートフォワーディングが可能なバージョン は7.7以上

3. あとは[SSH Remote Dynamic Port Forwarding](#SSH%20Remote%20Dynamic%20Port%20Forwarding)のステップ３以降と同じ(`ssh -N...`)

## 🔗[Plink](https://github.com/chrchang/plink-ng)

- ssh.exeが使えない場合や、インタラクティブなシェルが使えないとき(WinRM接続中など）に使用を検討
- 条件：足場のマシンへのCLIアクセスが可能なとき使用可能
	- ❌リモートダイナミックポートフォワーディングは使えない
- PlinkはPuTTYのCLI版

>[!WARNING]
>Attackerのパスワードが記録されてしまう恐れがあるため、信頼できないNWに対してのペネトレにおいては、ポートフォワーディング用ユーザーを別途作成すべき。

1. 攻撃者のマシンでsshをスタート
```zsh
sudo systemctl start ssh
```

2. 攻撃者のマシンからPlinkを足場のマシンに転送
	- [Apache webサーバー](../../Common/ファイル操作、ユーティリティ.md#Apache%20webサーバー)
```zsh
# Kali上のplinkのパス
/usr/share/windows-resources/binaries/plink.exe
```

3. 足場のマシン上でリモートポートフォワーディング
	- このコマンドで表示されるシェルには何も入力しないこと
```powershell
# windowsシェルが非ttyシェルの場合は先頭に`cmd.exe /c echo y |`を追加
C:\Windows\Temp\plink.exe -ssh -l <attacker_ssh_username> -pw <password> -R 127.0.0.1:<SSH_server_listen_port(Attacker)>:127.0.0.1:<Plink_client_listen_port> <attacker_IP>
```

4. 任意の操作を実施
```zsh
# proxychainsは不要
# mysql接続例
mysql -h 127.0.0.1 -P <SSH_server_listen_port> -u <username> -p
```

## 🔗[Netsh](https://learn.microsoft.com/ja-jp/windows-server/administration/windows-commands/netsh)

- NetshはWindows OS標準の機能で、SSHクライアントに使えるツールが何もないときなど、特に制限が厳しいときに使用する
- 条件：
	- 足場のマシンへのCLIアクセスが可能なとき
	- **管理者権限**があるとき

1. 管理者権限でcmd.exeを開き、netshのportproxyルールを追加しポートフォワーディング
	- 出力はなし
```powershell
netsh interface portproxy add v4tov4 listenport=<Netsh_client_listen_port> listenaddress=<Netsh_client_IP> connectport=<connect_port> connectaddress=<target_IP>
```
- `v4tov4`：IPv4からIPv4へのポートフォワーディングを意味する

2. 攻撃者のマシンから足場のマシンへの通信がFWで遮断される場合は、netshでFWを操作し、リッスンポートへの通信を許可する（成功時はOk.と出力される）
```powershell
netsh advfirewall firewall add rule name="<rule_name>" protocol=TCP dir=in localip=<Netsh_client_IP> localport=<connect_port> action=allow
```

3. 攻撃者のマシンからNmap等でリッスンポート経由でtargetのソケットにアクセス
```zsh
# 例
sudo nmap -sS <Netsh_client_IP> -Pn -n -p <connect_port>
```

4. netshによる変更の取り消し
```powershell
# FWルールの削除
netsh advfirewall firewall delete rule name="[rule_name>"
# ポートフォワーディングの削除
netsh interface portproxy del v4tov4 listenport=[指定したリッスンポート] listenaddress=[NetshクライアントIP]
```

---

# トラブルシューティング

| 問題                                                                   | 原因                                                 | 対策                                         |
| -------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------ |
| `TLS/SSL error: SSL is required, but the server does not support it` | SSL/TLS接続を要求しているが、SSH tunnel経由ではSSLが不要（既に暗号化されている） | SSL接続を無効化するオプションを付与する（mysqlの場合：--skip-ssl） |
| ポートフォワーディングをctrl+Cしてもまだ接続されている                                       | sshサービスを停止しても、既存のセッションは切断されない                      | `sudo pkill sshd`                          |

