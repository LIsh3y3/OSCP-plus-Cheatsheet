# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
3306/tcp open  mysql       MariaDB 5.5.5-10.1.48
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.48-MariaDB-0ubuntu0.18.04.1
|   Thread ID: 42
|   Capabilities flags: 63487
|   Some Capabilities: Speaks41ProtocolNew, LongPassword, Support41Auth, SupportsCompression, SupportsTransactions, FoundRows, LongColumnFlag, IgnoreSigpipes, ODBCClient, InteractiveClient, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOld, ConnectWithDatabase, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: qZO]gmlWNc'7}2!L:~.g
|_  Auth Plugin Name: mysql_native_password
```

---

# Enumeration

- sqlconf.phpで入手した認証情報とデータベース情報（openemr）を利用して機密情報を入手できないか検証




---

# Screenshot


