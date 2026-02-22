# Host Information

```

```
```zsh
Host:Linux
```

---

# Scans

## Nmap

### quick scan

```
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 3.0.5
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.49.104
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:5c:c0:36:35:2e:9d:17:7a:5d:62:e5:57:15:a0:c9 (ECDSA)
|_  256 e2:5e:b2:4e:13:3a:7a:8d:77:f1:a8:8a:b5:0c:14:85 (ED25519)
80/tcp   open  http       Apache httpd 2.4.52 ((Ubuntu))
|_http-title: OSCP
|_http-server-header: Apache/2.4.52 (Ubuntu)
3306/tcp open  mysql      MariaDB 5.5.5-10.6.22
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.6.22-MariaDB-0ubuntu0.22.04.1
|   Thread ID: 6
|   Capabilities flags: 63486
|   Some Capabilities: ConnectWithDatabase, Support41Auth, SupportsTransactions, DontAllowDatabaseTableColumn, InteractiveClient, Speaks41ProtocolNew, Speaks41ProtocolOld, ODBCClient, IgnoreSigpipes, FoundRows, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsLoadDataLocal, SupportsCompression, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: $n"Ws^\4+|Q_hlhDXnDE
|_  Auth Plugin Name: mysql_native_password
5432/tcp open  postgresql PostgreSQL DB 14.7 - 14.9
| ssl-cert: Subject: commonName=oscp
| Subject Alternative Name: DNS:oscp
| Not valid before: 2023-05-02T14:26:17
|_Not valid after:  2033-04-29T14:26:17
|_ssl-date: TLS randomness does not represent time
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
### full scan

```

```

### udp scan

```

```

## Rustscan

```

```

---
![[Pasted image 20260222112033.png]]