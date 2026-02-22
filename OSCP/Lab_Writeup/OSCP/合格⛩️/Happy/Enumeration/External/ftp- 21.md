# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-23-22  07:13AM                  145 .env
| 02-23-22  07:13AM                 2056 Acq.dll
| 02-24-22  05:24AM                 4868 DVRParams.ini
| 02-23-22  07:13AM                35996 Manifest.dll
| 02-23-22  07:13AM                20455 program.exe
| 02-23-22  07:15AM                40229 verisign.png
|_02-23-22  07:14AM                11446 wab.dll
| ftp-syst: 
|_  SYST: Windows_NT
```

---

# Enumeration

- Anonymousログインが可能
- ファイルをアップロードしてweb shell？
	- →書き込みは不可

## env

- MySQLのパスワードがわかった（initial access後）
```sh
lftp 192.168.104.111:/> cat .env
STATUS = development                        
DEV_PORT = 7500
PROD_PORT = 7600
HOST = localhost
DATABASE = db.dev
USER = Sandra
PASSWORD = Nj82@1Waqk90$
DIALECT = MSSQL 
145 bytes transferred in 1 second (125 B/s)

```

## program.exe -> nothing

- `strings`および`strings -e l`で列挙すｒも読み取り可能な文字列はなし
- そのほかpngも特になし

## Misc

- `ls -la` としたらMyMusicやMyVideoがあったので、ｄ

---

# Screenshot


