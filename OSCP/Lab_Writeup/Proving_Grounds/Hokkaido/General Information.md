# Host Information

```zsh
Windows Server 2022 Build 20348
```
```sh
hokkaido-aerospace.com
```

---

# Scans

## Nmap

### quick scan

```
Nmap scan report for 192.168.151.40
Host is up (0.15s latency).
Not shown: 985 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-11 05:21:52Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2026-02-11T05:22:51+00:00; -4s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
|_ssl-date: 2026-02-11T05:22:51+00:00; -4s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-08-02T02:13:54
|_Not valid after:  2054-08-02T02:13:54
|_ssl-date: 2026-02-11T05:22:51+00:00; -1s from scanner time.
| ms-sql-info: 
|   192.168.151.40:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   192.168.151.40:1433: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
|_ssl-date: 2026-02-11T05:22:51+00:00; -4s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
|_ssl-date: 2026-02-11T05:22:51+00:00; -4s from scanner time.
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-02-11T05:22:51+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: HAERO
|   NetBIOS_Domain_Name: HAERO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hokkaido-aerospace.com
|   DNS_Computer_Name: dc.hokkaido-aerospace.com
|   DNS_Tree_Name: hokkaido-aerospace.com
|   Product_Version: 10.0.20348
|_  System_Time: 2026-02-11T05:22:40+00:00
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Not valid before: 2026-02-10T05:09:29
|_Not valid after:  2026-08-12T05:09:29
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
### full scan

```
# Nmap 7.95 scan initiated Wed Feb 11 14:27:19 2026 as: /usr/lib/nmap/nmap -A -sV -p 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,3389,5985,8530,8531,9389,47001,49664,49665,49666,49667,49668,49671,49675,49684,49685,49691,49700,49701,49712,49785,58538 -oA Nmap/fullscan 192.168.151.40
Nmap scan report for 192.168.151.40
Host is up (0.16s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-11 05:27:26Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2026-02-11T05:13:19
|_Not valid after:  2027-02-11T05:13:19
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2026-02-11T05:13:19
|_Not valid after:  2027-02-11T05:13:19
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   192.168.151.40:1433: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
| ms-sql-info: 
|   192.168.151.40:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-08-02T02:13:54
|_Not valid after:  2054-08-02T02:13:54
|_ssl-date: 2026-02-11T05:28:43+00:00; 0s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2026-02-11T05:13:19
|_Not valid after:  2027-02-11T05:13:19
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2026-02-11T05:13:19
|_Not valid after:  2027-02-11T05:13:19
|_ssl-date: TLS randomness does not represent time
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HAERO
|   NetBIOS_Domain_Name: HAERO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hokkaido-aerospace.com
|   DNS_Computer_Name: dc.hokkaido-aerospace.com
|   DNS_Tree_Name: hokkaido-aerospace.com
|   Product_Version: 10.0.20348
|_  System_Time: 2026-02-11T05:28:36+00:00
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Not valid before: 2026-02-10T05:09:29
|_Not valid after:  2026-08-12T05:09:29
|_ssl-date: 2026-02-11T05:28:43+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8530/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: 403 - Forbidden: Access is denied.
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
8531/tcp  open  unknown
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
...
58538/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   192.168.151.40:58538: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
|_ssl-date: 2026-02-11T05:28:44+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-08-02T02:13:54
|_Not valid after:  2054-08-02T02:13:54
| ms-sql-info: 
|   192.168.151.40:58538: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 58538
...
```

### udp scan

```
PORT      STATE         SERVICE        VERSION
53/tcp    open          domain         (generic dns response: SERVFAIL)
80/tcp    open          http           Microsoft IIS httpd 10.0
88/tcp    open          kerberos-sec   Microsoft Windows Kerberos (server time: 2026-02-11 05:30:56Z)
135/tcp   open          msrpc          Microsoft Windows RPC
139/tcp   open          netbios-ssn    Microsoft Windows netbios-ssn
389/tcp   open          ldap           Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
445/tcp   open          microsoft-ds?
1433/tcp  open          ms-sql-s       Microsoft SQL Server 2019 15.00.2000
3389/tcp  open          ms-wbt-server  Microsoft Terminal Services
53/udp    open          domain         Simple DNS Plus (generic dns response: SERVFAIL)
80/udp    open|filtered http
88/udp    open          kerberos-sec   Microsoft Windows Kerberos (server time: 2026-02-11 05:29:20Z)
123/udp   open          ntp            NTP v3
...
```

## Rustscan

```
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 125 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  syn-ack ttl 125 Microsoft Windows Kerberos (server time: 2026-02-11 05:33:17Z)
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-02-11T05:13:19
| Not valid after:  2027-02-11T05:13:19
| MD5:   2970:8b4f:da10:c35c:c2d3:1d7f:408a:efc5
| SHA-1: bd81:67c3:1000:12e7:3699:e4d9:692b:711b:2afe:b646
| -----BEGIN CERTIFICATE-----
| MIIGezCCBWOgAwIBAgITPwAAAAUew8YGRT0VFgAAAAAABTANBgkqhkiG9w0BAQsF
| ADBcMRMwEQYKCZImiZPyLGQBGRYDY29tMSIwIAYKCZImiZPyLGQBGRYSaG9ra2Fp
| ZG8tYWVyb3NwYWNlMSEwHwYDVQQDExhob2trYWlkby1hZXJvc3BhY2UtREMtQ0Ew
| HhcNMjYwMjExMDUxMzE5WhcNMjcwMjExMDUxMzE5WjAkMSIwIAYDVQQDExlkYy5o
| b2trYWlkby1hZXJvc3BhY2UuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
| CgKCAQEAzNZbPsmO/EMsNXSjS1ZwB1Ny3Vl9n+1cGgS6nstdoim7q+kwJkefEmZU
| kffrdlknSuZQy+TLkGqzAYzK/qPoWLh8kKA3CEbhVX44DquzpTwczUKF5RyLt9yV
| Rb2ACm34trXH1yjCQ/qpvyH0M1OtJbECE3rGzJi9UOEdvA5oVlWTTQZZVCY74LxG
| J6erfnyE2SQbeFDJR43ifB1ZCmZsEuGw2EFHVu33lBFVgoRsp5cLb2ZiFqW4JESI
| Ga+sxmgs7NDYWJ2eTt3KDRscCIZTF6Gkso72Q4ELq+3yGRJzBwiaYgU/W8Zu5fci
| U4H62AsMHBZ/K/Hur+2Jq3NHQDc5EQIDAQABo4IDbDCCA2gwLwYJKwYBBAGCNxQC
| BCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQG
| CCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkP
| BGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQB
| KjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMC
| BzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUZ1nqrBT+q/AIY2iz5M/XcN0pXKYwHwYD
| VR0jBBgwFoAUO3GDfJd+M4g9RXjy4dZ42J5MKa0wgdwGA1UdHwSB1DCB0TCBzqCB
| y6CByIaBxWxkYXA6Ly8vQ049aG9ra2FpZG8tYWVyb3NwYWNlLURDLUNBLENOPWRj
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPWhva2thaWRvLWFlcm9zcGFjZSxEQz1jb20/Y2Vy
| dGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3Ry
| aWJ1dGlvblBvaW50MIHVBggrBgEFBQcBAQSByDCBxTCBwgYIKwYBBQUHMAKGgbVs
| ZGFwOi8vL0NOPWhva2thaWRvLWFlcm9zcGFjZS1EQy1DQSxDTj1BSUEsQ049UHVi
| bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv
| bixEQz1ob2trYWlkby1hZXJvc3BhY2UsREM9Y29tP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEUGA1UdEQQ+MDyg
| HwYJKwYBBAGCNxkBoBIEEHPAV/nQryFLm5ZKYfkmFISCGWRjLmhva2thaWRvLWFl
| cm9zcGFjZS5jb20wTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEEAYI3GQIBoC8ELVMt
| MS01LTIxLTMyMjcyOTY5MTQtOTc0NzgwMjA0LTEzMjU5NDE0OTctMTAwMDANBgkq
| hkiG9w0BAQsFAAOCAQEAf6I4N0QOu2gYh7U6nlGORUHZYZjvaZ9WosXcRQhF+Axa
| bur4Q/+EnuhDTphhG74aP8xcl0YLQWjOlGSy1HBksdkD8cgX8ug/iO/dyjuoI5AP
| x8ldDhMg3CaITVtQiAJPyWtikrtbQGrIu0kRLaFmciRhTcEKF+Qo9yAbPGPvEzib
| oyLtdLNrca428g4/mHiZDTLjcqf4SOUPyPIBXQFTuMcpWZPOwq5BtuowdfDW/JCr
| 9Lhp/mWH4Cs0B4jMEhngKLmPKhhFCwHGn8aAKgFBUb9l1vDhuSUP6NL2qcu6rDXZ
| iq8dgg2YWKeGG56VzcDrmzMikklNOF4l4VsHmx/NzQ==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds? syn-ack ttl 125
464/tcp   open  kpasswd5?     syn-ack ttl 125
593/tcp   open  ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-02-11T05:13:19
| Not valid after:  2027-02-11T05:13:19
| MD5:   2970:8b4f:da10:c35c:c2d3:1d7f:408a:efc5
| SHA-1: bd81:67c3:1000:12e7:3699:e4d9:692b:711b:2afe:b646
| -----BEGIN CERTIFICATE-----
| MIIGezCCBWOgAwIBAgITPwAAAAUew8YGRT0VFgAAAAAABTANBgkqhkiG9w0BAQsF
| ADBcMRMwEQYKCZImiZPyLGQBGRYDY29tMSIwIAYKCZImiZPyLGQBGRYSaG9ra2Fp
| ZG8tYWVyb3NwYWNlMSEwHwYDVQQDExhob2trYWlkby1hZXJvc3BhY2UtREMtQ0Ew
| HhcNMjYwMjExMDUxMzE5WhcNMjcwMjExMDUxMzE5WjAkMSIwIAYDVQQDExlkYy5o
| b2trYWlkby1hZXJvc3BhY2UuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
| CgKCAQEAzNZbPsmO/EMsNXSjS1ZwB1Ny3Vl9n+1cGgS6nstdoim7q+kwJkefEmZU
| kffrdlknSuZQy+TLkGqzAYzK/qPoWLh8kKA3CEbhVX44DquzpTwczUKF5RyLt9yV
| Rb2ACm34trXH1yjCQ/qpvyH0M1OtJbECE3rGzJi9UOEdvA5oVlWTTQZZVCY74LxG
| J6erfnyE2SQbeFDJR43ifB1ZCmZsEuGw2EFHVu33lBFVgoRsp5cLb2ZiFqW4JESI
| Ga+sxmgs7NDYWJ2eTt3KDRscCIZTF6Gkso72Q4ELq+3yGRJzBwiaYgU/W8Zu5fci
| U4H62AsMHBZ/K/Hur+2Jq3NHQDc5EQIDAQABo4IDbDCCA2gwLwYJKwYBBAGCNxQC
| BCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQG
| CCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkP
| BGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQB
| KjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMC
| BzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUZ1nqrBT+q/AIY2iz5M/XcN0pXKYwHwYD
| VR0jBBgwFoAUO3GDfJd+M4g9RXjy4dZ42J5MKa0wgdwGA1UdHwSB1DCB0TCBzqCB
| y6CByIaBxWxkYXA6Ly8vQ049aG9ra2FpZG8tYWVyb3NwYWNlLURDLUNBLENOPWRj
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPWhva2thaWRvLWFlcm9zcGFjZSxEQz1jb20/Y2Vy
| dGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3Ry
| aWJ1dGlvblBvaW50MIHVBggrBgEFBQcBAQSByDCBxTCBwgYIKwYBBQUHMAKGgbVs
| ZGFwOi8vL0NOPWhva2thaWRvLWFlcm9zcGFjZS1EQy1DQSxDTj1BSUEsQ049UHVi
| bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv
| bixEQz1ob2trYWlkby1hZXJvc3BhY2UsREM9Y29tP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEUGA1UdEQQ+MDyg
| HwYJKwYBBAGCNxkBoBIEEHPAV/nQryFLm5ZKYfkmFISCGWRjLmhva2thaWRvLWFl
| cm9zcGFjZS5jb20wTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEEAYI3GQIBoC8ELVMt
| MS01LTIxLTMyMjcyOTY5MTQtOTc0NzgwMjA0LTEzMjU5NDE0OTctMTAwMDANBgkq
| hkiG9w0BAQsFAAOCAQEAf6I4N0QOu2gYh7U6nlGORUHZYZjvaZ9WosXcRQhF+Axa
| bur4Q/+EnuhDTphhG74aP8xcl0YLQWjOlGSy1HBksdkD8cgX8ug/iO/dyjuoI5AP
| x8ldDhMg3CaITVtQiAJPyWtikrtbQGrIu0kRLaFmciRhTcEKF+Qo9yAbPGPvEzib
| oyLtdLNrca428g4/mHiZDTLjcqf4SOUPyPIBXQFTuMcpWZPOwq5BtuowdfDW/JCr
| 9Lhp/mWH4Cs0B4jMEhngKLmPKhhFCwHGn8aAKgFBUb9l1vDhuSUP6NL2qcu6rDXZ
| iq8dgg2YWKeGG56VzcDrmzMikklNOF4l4VsHmx/NzQ==
|_-----END CERTIFICATE-----
1433/tcp  open  ms-sql-s      syn-ack ttl 125 Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2026-02-11T05:34:36+00:00; 0s from scanner time.
| ms-sql-info: 
|   192.168.151.40:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   192.168.151.40:1433: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-02T02:13:54
| Not valid after:  2054-08-02T02:13:54
| MD5:   594a:702f:d421:ff2f:411d:d1ea:73f1:2c3f
| SHA-1: f4ad:4152:6a70:f50b:ec47:5026:400f:8ffb:0dc3:5178
| -----BEGIN CERTIFICATE-----
| MIIDADCCAeigAwIBAgIQLhWUI307A4ROtwMgh+8WuDANBgkqhkiG9w0BAQsFADA7
| MTkwNwYDVQQDHjAAUwBTAEwAXwBTAGUAbABmAF8AUwBpAGcAbgBlAGQAXwBGAGEA
| bABsAGIAYQBjAGswIBcNMjQwODAyMDIxMzU0WhgPMjA1NDA4MDIwMjEzNTRaMDsx
| OTA3BgNVBAMeMABTAFMATABfAFMAZQBsAGYAXwBTAGkAZwBuAGUAZABfAEYAYQBs
| AGwAYgBhAGMAazCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJOM/LlJ
| n7WRALHhPH9JpPOUCprXzepna4LoyPFu7jf3ndos/OXkWEmhJXWbOmRgoBg+MYb9
| FcoA6cdcPQYVQMQ9sYIm9RnlwnpYHzdBCamVX9GEOcB7g05fJED+IDzZyIZSlvRb
| BJfUOJsHC47hkZuDSM8pucGG4aYwrLrqmH59xt4FfHnneSa7FIE0UTrtDQHuEiAa
| rmTSdS5VotclR99upSYGnySGgxeE4rojdz6RoCZO7MfVRKxuq2jzwyAw4KTrbDb5
| innLtClsPryZwnT/xwxY6JUmA2i9/0GKM9bphQXu4eWC23fhCh+Ej1lLeB1x6duS
| 0OcUD9ViEIaAIX0CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAHbiMlU++aiHToJuF
| ECtOjqW4vlEq9VKI57Q1CtsFfCxTsY4fSnS+0lylxOReAfU6GnXaIkAq1VSEml7r
| RBo6bYWxYwDWrmy4RoHBHzE2DKYfq8x7i09uDQ3E2ndWE83k2wbyJ6aXMld120D7
| 73S1hCFppHJwqBqjvOFr+HePxMG8XT4YXZvFjtgiL2cvPoAc4swU1VlMeFfPRrve
| 6D442eQMd+tVckMrS/3p9AavbUGwwyFUDgPS2pp3e4A/pQdPMN91gvoVaguWKY4J
| lAbWPCd5UKPsIz3QS/lXivFtCLBSkxsqlM6/dF1mIMfHMECyxL4esBifdGqYcnt5
| p9exOQ==
|_-----END CERTIFICATE-----
3268/tcp  open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-02-11T05:13:19
| Not valid after:  2027-02-11T05:13:19
| MD5:   2970:8b4f:da10:c35c:c2d3:1d7f:408a:efc5
| SHA-1: bd81:67c3:1000:12e7:3699:e4d9:692b:711b:2afe:b646
| -----BEGIN CERTIFICATE-----
| MIIGezCCBWOgAwIBAgITPwAAAAUew8YGRT0VFgAAAAAABTANBgkqhkiG9w0BAQsF
| ADBcMRMwEQYKCZImiZPyLGQBGRYDY29tMSIwIAYKCZImiZPyLGQBGRYSaG9ra2Fp
| ZG8tYWVyb3NwYWNlMSEwHwYDVQQDExhob2trYWlkby1hZXJvc3BhY2UtREMtQ0Ew
| HhcNMjYwMjExMDUxMzE5WhcNMjcwMjExMDUxMzE5WjAkMSIwIAYDVQQDExlkYy5o
| b2trYWlkby1hZXJvc3BhY2UuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
| CgKCAQEAzNZbPsmO/EMsNXSjS1ZwB1Ny3Vl9n+1cGgS6nstdoim7q+kwJkefEmZU
| kffrdlknSuZQy+TLkGqzAYzK/qPoWLh8kKA3CEbhVX44DquzpTwczUKF5RyLt9yV
| Rb2ACm34trXH1yjCQ/qpvyH0M1OtJbECE3rGzJi9UOEdvA5oVlWTTQZZVCY74LxG
| J6erfnyE2SQbeFDJR43ifB1ZCmZsEuGw2EFHVu33lBFVgoRsp5cLb2ZiFqW4JESI
| Ga+sxmgs7NDYWJ2eTt3KDRscCIZTF6Gkso72Q4ELq+3yGRJzBwiaYgU/W8Zu5fci
| U4H62AsMHBZ/K/Hur+2Jq3NHQDc5EQIDAQABo4IDbDCCA2gwLwYJKwYBBAGCNxQC
| BCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQG
| CCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkP
| BGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQB
| KjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMC
| BzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUZ1nqrBT+q/AIY2iz5M/XcN0pXKYwHwYD
| VR0jBBgwFoAUO3GDfJd+M4g9RXjy4dZ42J5MKa0wgdwGA1UdHwSB1DCB0TCBzqCB
| y6CByIaBxWxkYXA6Ly8vQ049aG9ra2FpZG8tYWVyb3NwYWNlLURDLUNBLENOPWRj
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPWhva2thaWRvLWFlcm9zcGFjZSxEQz1jb20/Y2Vy
| dGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3Ry
| aWJ1dGlvblBvaW50MIHVBggrBgEFBQcBAQSByDCBxTCBwgYIKwYBBQUHMAKGgbVs
| ZGFwOi8vL0NOPWhva2thaWRvLWFlcm9zcGFjZS1EQy1DQSxDTj1BSUEsQ049UHVi
| bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv
| bixEQz1ob2trYWlkby1hZXJvc3BhY2UsREM9Y29tP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEUGA1UdEQQ+MDyg
| HwYJKwYBBAGCNxkBoBIEEHPAV/nQryFLm5ZKYfkmFISCGWRjLmhva2thaWRvLWFl
| cm9zcGFjZS5jb20wTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEEAYI3GQIBoC8ELVMt
| MS01LTIxLTMyMjcyOTY5MTQtOTc0NzgwMjA0LTEzMjU5NDE0OTctMTAwMDANBgkq
| hkiG9w0BAQsFAAOCAQEAf6I4N0QOu2gYh7U6nlGORUHZYZjvaZ9WosXcRQhF+Axa
| bur4Q/+EnuhDTphhG74aP8xcl0YLQWjOlGSy1HBksdkD8cgX8ug/iO/dyjuoI5AP
| x8ldDhMg3CaITVtQiAJPyWtikrtbQGrIu0kRLaFmciRhTcEKF+Qo9yAbPGPvEzib
| oyLtdLNrca428g4/mHiZDTLjcqf4SOUPyPIBXQFTuMcpWZPOwq5BtuowdfDW/JCr
| 9Lhp/mWH4Cs0B4jMEhngKLmPKhhFCwHGn8aAKgFBUb9l1vDhuSUP6NL2qcu6rDXZ
| iq8dgg2YWKeGG56VzcDrmzMikklNOF4l4VsHmx/NzQ==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-02-11T05:13:19
| Not valid after:  2027-02-11T05:13:19
| MD5:   2970:8b4f:da10:c35c:c2d3:1d7f:408a:efc5
| SHA-1: bd81:67c3:1000:12e7:3699:e4d9:692b:711b:2afe:b646
| -----BEGIN CERTIFICATE-----
| MIIGezCCBWOgAwIBAgITPwAAAAUew8YGRT0VFgAAAAAABTANBgkqhkiG9w0BAQsF
| ADBcMRMwEQYKCZImiZPyLGQBGRYDY29tMSIwIAYKCZImiZPyLGQBGRYSaG9ra2Fp
| ZG8tYWVyb3NwYWNlMSEwHwYDVQQDExhob2trYWlkby1hZXJvc3BhY2UtREMtQ0Ew
| HhcNMjYwMjExMDUxMzE5WhcNMjcwMjExMDUxMzE5WjAkMSIwIAYDVQQDExlkYy5o
| b2trYWlkby1hZXJvc3BhY2UuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
| CgKCAQEAzNZbPsmO/EMsNXSjS1ZwB1Ny3Vl9n+1cGgS6nstdoim7q+kwJkefEmZU
| kffrdlknSuZQy+TLkGqzAYzK/qPoWLh8kKA3CEbhVX44DquzpTwczUKF5RyLt9yV
| Rb2ACm34trXH1yjCQ/qpvyH0M1OtJbECE3rGzJi9UOEdvA5oVlWTTQZZVCY74LxG
| J6erfnyE2SQbeFDJR43ifB1ZCmZsEuGw2EFHVu33lBFVgoRsp5cLb2ZiFqW4JESI
| Ga+sxmgs7NDYWJ2eTt3KDRscCIZTF6Gkso72Q4ELq+3yGRJzBwiaYgU/W8Zu5fci
| U4H62AsMHBZ/K/Hur+2Jq3NHQDc5EQIDAQABo4IDbDCCA2gwLwYJKwYBBAGCNxQC
| BCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQG
| CCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkP
| BGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQB
| KjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMC
| BzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUZ1nqrBT+q/AIY2iz5M/XcN0pXKYwHwYD
| VR0jBBgwFoAUO3GDfJd+M4g9RXjy4dZ42J5MKa0wgdwGA1UdHwSB1DCB0TCBzqCB
| y6CByIaBxWxkYXA6Ly8vQ049aG9ra2FpZG8tYWVyb3NwYWNlLURDLUNBLENOPWRj
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPWhva2thaWRvLWFlcm9zcGFjZSxEQz1jb20/Y2Vy
| dGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3Ry
| aWJ1dGlvblBvaW50MIHVBggrBgEFBQcBAQSByDCBxTCBwgYIKwYBBQUHMAKGgbVs
| ZGFwOi8vL0NOPWhva2thaWRvLWFlcm9zcGFjZS1EQy1DQSxDTj1BSUEsQ049UHVi
| bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv
| bixEQz1ob2trYWlkby1hZXJvc3BhY2UsREM9Y29tP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEUGA1UdEQQ+MDyg
| HwYJKwYBBAGCNxkBoBIEEHPAV/nQryFLm5ZKYfkmFISCGWRjLmhva2thaWRvLWFl
| cm9zcGFjZS5jb20wTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEEAYI3GQIBoC8ELVMt
| MS01LTIxLTMyMjcyOTY5MTQtOTc0NzgwMjA0LTEzMjU5NDE0OTctMTAwMDANBgkq
| hkiG9w0BAQsFAAOCAQEAf6I4N0QOu2gYh7U6nlGORUHZYZjvaZ9WosXcRQhF+Axa
| bur4Q/+EnuhDTphhG74aP8xcl0YLQWjOlGSy1HBksdkD8cgX8ug/iO/dyjuoI5AP
| x8ldDhMg3CaITVtQiAJPyWtikrtbQGrIu0kRLaFmciRhTcEKF+Qo9yAbPGPvEzib
| oyLtdLNrca428g4/mHiZDTLjcqf4SOUPyPIBXQFTuMcpWZPOwq5BtuowdfDW/JCr
| 9Lhp/mWH4Cs0B4jMEhngKLmPKhhFCwHGn8aAKgFBUb9l1vDhuSUP6NL2qcu6rDXZ
| iq8dgg2YWKeGG56VzcDrmzMikklNOF4l4VsHmx/NzQ==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
5985/tcp  open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8530/tcp  open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: 403 - Forbidden: Access is denied.
8531/tcp  open  unknown       syn-ack ttl 125
9389/tcp  open  mc-nmf        syn-ack ttl 125 .NET Message Framing
47001/tcp open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```