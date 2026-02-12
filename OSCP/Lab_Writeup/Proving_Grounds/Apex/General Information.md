# Host Information

```zsh
192.168.154.145
```

---

# Scans

## Nmap

### quick scan

```
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-title: APEX Hospital
|_http-server-header: Apache/2.4.29 (Ubuntu)
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MariaDB 5.5.5-10.1.48
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.48-MariaDB-0ubuntu0.18.04.1
|   Thread ID: 32
|   Capabilities flags: 63487
|   Some Capabilities: IgnoreSigpipes, Speaks41ProtocolOld, FoundRows, SupportsLoadDataLocal, ODBCClient, Support41Auth, DontAllowDatabaseTableColumn, LongPassword, IgnoreSpaceBeforeParenthesis, SupportsTransactions, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, LongColumnFlag, SupportsCompression, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: >)O3OrtoQ'jb/b!zgs01
|_  Auth Plugin Name: mysql_native_password
Service Info: Host: APEX
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