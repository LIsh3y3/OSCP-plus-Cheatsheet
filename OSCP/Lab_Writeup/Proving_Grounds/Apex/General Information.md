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
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-title: APEX Hospital
|_http-server-header: Apache/2.4.29 (Ubuntu)
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
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
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
```

### udp scan

```
PORT     STATE  SERVICE      VERSION
80/tcp   open   http         Apache httpd 2.4.29 ((Ubuntu))
445/tcp  open   netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open   mysql        MariaDB 5.5.5-10.1.48
445/udp  closed microsoft-ds
Service Info: Host: APEX
```

## Rustscan

```
PORT    STATE SERVICE     REASON         VERSION
80/tcp  open  http        syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: APEX Hospital
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
445/tcp open  netbios-ssn syn-ack ttl 61 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: APEX

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 42897/tcp): CLEAN (Timeout)
|   Check 2 (port 26505/tcp): CLEAN (Timeout)
|   Check 3 (port 43324/udp): CLEAN (Timeout)
|   Check 4 (port 11769/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2026-02-12T13:53:59
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: apex
|   NetBIOS computer name: APEX\x00
|   Domain name: \x00
|   FQDN: apex
|_  System time: 2026-02-12T08:54:01-05:00
|_clock-skew: mean: 1h40m00s, deviation: 2h53m15s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```