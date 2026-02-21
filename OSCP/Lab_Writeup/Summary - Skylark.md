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
331 Please, specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||63862|)
150 Starting data transfer.
drwxrwxrwx 1 ftp ftp               0 Nov 30  2022 aspnet_client
-rw-rw-rw- 1 ftp ftp             703 Nov 29  2022 iisstart.htm
-rw-rw-rw- 1 ftp ftp           99710 Nov 29  2022 iisstart.png
-rw-rw-rw- 1 ftp ftp              74 Dec 01  2022 security.txt
drwxrwxrwx 1 ftp ftp               0 Nov 29  2022 umbraco
226 Operation successful
ftp> 
```