# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0         3557581 Nov 25  2021 2d5ef5a0f0c9579458c9
| -rw-r--r--    1 0        0         1258508 Nov 25  2021 4835e976619690ae006e
| -rw-r--r--    1 0        0         1617905 Nov 25  2021 4e8cce46d6abec9a9d9a
| -rw-r--r--    1 0        0          438095 Nov 25  2021 77cfe070405f6ca327a5
|_-rw-r--r--    1 0        0          841392 Nov 25  2021 c5237630ef40e2585d35
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.49.104
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

---

# Enumeration

- Anonymousログイン可能
- ファイルのアップロードは不可
- ダウンロードしたすべてのファイルを閲覧してみる
	- すべてPDFファイル

- 2d5e...と438...
	- 「Staedtler 」という実際にある会社のパンフレットっぽい

## 7cfe070...

- AuditorzRusという会社（多分このターゲット）の年間レポートっぽい
- パスワードが古く脆弱なので変えてくださいね、というレポート
- 




---

# Screenshot


![[Pasted image 20260222200501.png]]

![[Pasted image 20260222200759.png]]