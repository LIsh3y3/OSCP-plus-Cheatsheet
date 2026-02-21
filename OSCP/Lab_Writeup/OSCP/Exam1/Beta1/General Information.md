# Host Information

```zsh
TargetIP:192.168.115.111
```

---

# Scans

## Nmap

### quick scan

- Windowsっぽい
```
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ sudo nmap -sC -sV -oA Nmap/quickscan -T4 $TargetIP
[sudo] password for koshi: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 11:08 +0900
Nmap scan report for 192.168.115.111
Host is up (0.23s latency).
Not shown: 994 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-generator: Nicepage 5.13.1, nicepage.com
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Home
|_http-server-header: Microsoft-IIS/10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2023-08-24T08:19:45
|_Not valid after:  2023-11-22T08:19:45
|_ssl-date: 2023-09-15T14:15:39+00:00; -2y116d11h54m27s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=oscp
| Not valid before: 2023-09-14T12:17:25
|_Not valid after:  2024-03-15T12:17:25
|_ssl-date: 2023-09-15T14:15:39+00:00; -2y116d11h54m27s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: OSCP
|   NetBIOS_Domain_Name: OSCP
|   NetBIOS_Computer_Name: OSCP
|   DNS_Domain_Name: oscp
|   DNS_Computer_Name: oscp
|   Product_Version: 10.0.17763
|_  System_Time: 2023-09-15T14:14:22+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -847d11h54m27s, deviation: 0s, median: -847d11h54m28s
| smb2-time: 
|   date: 2023-09-15T14:14:25
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 125.46 seconds

```

![[Pasted image 20260110112558.png]]

### full scan

```
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ # openポートを詳細スキャン
sudo nmap -A -sV -p $ports -oA Nmap/fullscan $TargetIP
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 12:23 +0900
Nmap scan report for 192.168.115.111
Host is up (0.24s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-generator: Nicepage 5.13.1, nicepage.com
|_http-title: Home
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2023-08-24T08:19:45
|_Not valid after:  2023-11-22T08:19:45
|_ssl-date: 2023-09-15T15:30:17+00:00; -2y116d11h54m27s from scanner time.
|_http-server-header: Microsoft-IIS/10.0
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-09-15T15:30:18+00:00; -2y116d11h54m27s from scanner time.
| ssl-cert: Subject: commonName=oscp
| Not valid before: 2023-09-14T12:17:25
|_Not valid after:  2024-03-15T12:17:25
| rdp-ntlm-info: 
|   Target_Name: OSCP
|   NetBIOS_Domain_Name: OSCP
|   NetBIOS_Computer_Name: OSCP
|   DNS_Domain_Name: oscp
|   DNS_Computer_Name: oscp
|   Product_Version: 10.0.17763
|_  System_Time: 2023-09-15T15:29:02+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -847d11h54m27s, deviation: 0s, median: -847d11h54m27s
| smb2-time: 
|   date: 2023-09-15T15:29:05
|_  start_date: N/A

TRACEROUTE (using port 135/tcp)
HOP RTT       ADDRESS
1   245.13 ms 192.168.49.1
2   245.24 ms 192.168.115.111

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 106.22 seconds
                                                              
```

### udp scan

```
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ nmap -sU -sS -sV $TargetIP -T4 -oN Nmap/udpscan.nmap
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 12:31 +0900
Stats: 0:35:20 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 62.92% done; ETC: 13:27 (0:20:22 remaining)
Stats: 0:41:20 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 72.17% done; ETC: 13:28 (0:15:39 remaining)
Stats: 0:48:34 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 84.39% done; ETC: 13:28 (0:08:50 remaining)
Nmap scan report for 192.168.115.111
Host is up (0.24s latency).
Not shown: 1000 open|filtered udp ports (no-response), 994 filtered tcp ports (no-resp
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/s
Nmap done: 1 IP address (1 host up) scanned in 3523.24 seconds

```

## Rustscan

```
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ rustscan -a $TargetIP --ulimit 5000 -- -sC -sV -oN Nmap/rustscan.nmap
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
TCP handshake? More like a friendly high-five!

[~] The config file is expected to be at "/home/koshi/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 192.168.115.111:80
Open 192.168.115.111:135
Open 192.168.115.111:139
Open 192.168.115.111:443
Open 192.168.115.111:445
Open 192.168.115.111:3389
[~] Starting Script(s)
[>] Running script "nmap -vvv -p {{port}} -{{ipversion}} {{ip}} -sC -sV -oN Nmap/rustscan.nmap" on ip 192.168.115.111
Depending on the complexity of the script, results may take some time to appear.
[~] Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 12:27 +0900
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:27
Completed NSE at 12:27, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:27
Completed NSE at 12:27, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:27
Completed NSE at 12:27, 0.00s elapsed
Initiating Ping Scan at 12:27
Scanning 192.168.115.111 [4 ports]
Completed Ping Scan at 12:27, 0.26s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:27
Completed Parallel DNS resolution of 1 host. at 12:27, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 12:27
Scanning 192.168.115.111 [6 ports]
Discovered open port 443/tcp on 192.168.115.111
Discovered open port 80/tcp on 192.168.115.111
Discovered open port 135/tcp on 192.168.115.111
Discovered open port 445/tcp on 192.168.115.111
Discovered open port 139/tcp on 192.168.115.111
Discovered open port 3389/tcp on 192.168.115.111
Completed SYN Stealth Scan at 12:27, 0.40s elapsed (6 total ports)
Initiating Service scan at 12:27
Scanning 6 services on 192.168.115.111
Completed Service scan at 12:27, 20.81s elapsed (6 services on 1 host)
NSE: Script scanning 192.168.115.111.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:27
NSE Timing: About 99.06% done; ETC: 12:28 (0:00:00 remaining)
NSE Timing: About 99.53% done; ETC: 12:28 (0:00:00 remaining)
Completed NSE at 12:28, 81.18s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:28
Completed NSE at 12:28, 2.72s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:28
Completed NSE at 12:28, 0.00s elapsed
Nmap scan report for 192.168.115.111
Host is up, received echo-reply ttl 127 (0.32s latency).
Scanned at 2026-01-10 12:27:08 JST for 105s

PORT     STATE SERVICE       REASON          VERSION
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Home
|_http-generator: Nicepage 5.13.1, nicepage.com
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
|_ssl-date: 2023-09-15T15:34:26+00:00; -2y116d11h54m27s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Issuer: commonName=PowerShellWebAccessTestWebSite
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2023-08-24T08:19:45
| Not valid after:  2023-11-22T08:19:45
| MD5:     ce5f 2697 4f6f 429e f052 4186 6f96 948c
| SHA-1:   ba7e 919b 1a3f 953c 475e 9408 75c1 8b41 9f15 8293
| SHA-256: 988f 0d98 ce6c 5f0d bbfc 3093 5309 191c 16c6 9dea 2394 ed1f cb8a 8cc3 934f 0f15
| -----BEGIN CERTIFICATE-----
| MIICHTCCAYagAwIBAgIQPwcY3kt2BKxINSOAggUo2TANBgkqhkiG9w0BAQUFADAp
| MScwJQYDVQQDDB5Qb3dlclNoZWxsV2ViQWNjZXNzVGVzdFdlYlNpdGUwHhcNMjMw
| ODI0MDgxOTQ1WhcNMjMxMTIyMDgxOTQ1WjApMScwJQYDVQQDDB5Qb3dlclNoZWxs
| V2ViQWNjZXNzVGVzdFdlYlNpdGUwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGB
| AMTGjoZtdwqi6nqF+XzQSWRXkVePPu2O3UWzCXn6X5qEQEZStYbjV8OI1BBhqAOQ
| OtucumQFLoxtNGfrdx5M6+gXljmkyfG3UHfkh3Zdpbmxd+Uk97yNgLPKZGI3vGZf
| xcfjA08PUQ7DQJpW5YvIOo0yGu1fZI8oANhgKoHjoqYdAgMBAAGjRjBEMBMGA1Ud
| JQQMMAoGCCsGAQUFBwMBMB0GA1UdDgQWBBQ3fuSrOgxTJuBN5skRGJaaerG2AjAO
| BgNVHQ8BAf8EBAMCBSAwDQYJKoZIhvcNAQEFBQADgYEAA0vRc8TaxAK/L30dwwNq
| 1UtRAFMxATzKluP3YK4Z0d1M2cUclthMwh04dADLELwPpbcoP7vEN0gCeqvvH2lj
| s7AxlAB+ffGW96RiFMwfXMmQUfoQXYFdFBEXzwXVdyZKDsm9G1+FeALz/9NQBfQv
| kyn5rOPd057gPmYj0JEJ/+k=
|_-----END CERTIFICATE-----
|_http-server-header: Microsoft-IIS/10.0
445/tcp  open  microsoft-ds? syn-ack ttl 127
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
|_ssl-date: 2023-09-15T15:34:26+00:00; -2y116d11h54m27s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: OSCP
|   NetBIOS_Domain_Name: OSCP
|   NetBIOS_Computer_Name: OSCP
|   DNS_Domain_Name: oscp
|   DNS_Computer_Name: oscp
|   Product_Version: 10.0.17763
|_  System_Time: 2023-09-15T15:33:04+00:00
| ssl-cert: Subject: commonName=oscp
| Issuer: commonName=oscp
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-09-14T12:17:25
| Not valid after:  2024-03-15T12:17:25
| MD5:     7097 491f 21f4 69e3 56de d4b8 aec7 36ec
| SHA-1:   5506 2ea6 0d5f 5301 3b4d 3c56 6073 e78a 2b7c 99e3
| SHA-256: 11bd a6d1 ec59 a47e e4db ba16 4131 b3eb b4fc 0deb 1ac2 fc5a 5c6c 8c13 3e44 a139
| -----BEGIN CERTIFICATE-----
| MIICzDCCAbSgAwIBAgIQU5EQd1GKA6lE63xNv9lypzANBgkqhkiG9w0BAQsFADAP
| MQ0wCwYDVQQDEwRvc2NwMB4XDTIzMDkxNDEyMTcyNVoXDTI0MDMxNTEyMTcyNVow
| DzENMAsGA1UEAxMEb3NjcDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
| AKCqHa9wo2zFySbh+up8PQ/PMuFATPPcfxzMh/lMVkE2qAoCcqAivgVcWan5m2lJ
| z57C0nfCRQuwoVGnW2qtiiiX0FsPcsEXQJksp2JWVNqBpyIsvxg1gRukHJIl5Wy5
| pZQt16kRGBSFzYGBUgNYbew8LIDk74c6ySVD/+qrbnJanwI90vm9AvV8DV/J+dmo
| y0rdXOcdlfMgvGDg5WErpRcWfiSTDwJF/hR4whW8gGrVKQe6reU67Fedsjx5QLwB
| zIqQk2lz7tkpxSnR48coEyk/CE+inE7q87/77NErUneP3uuXR20OZLNNO6fRXR6+
| LoNtevYfJMTd8q1KOiRFADECAwEAAaMkMCIwEwYDVR0lBAwwCgYIKwYBBQUHAwEw
| CwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBCwUAA4IBAQA5LKrTy8Ov2hO9fpgZLMfh
| qRGaptUM2rg09I0sSN9W68utkllbFo4+P+fl40aDq8vf8YmqfaV/OonSSrRPqnR9
| 2yNUx8RrD7UVZDMQ6+0Y+0HS9dLFPImevQ0y81I9q2whcck+kRwPuicIE/lD7RHs
| 3VlOYvOJ+CM/69cssn5zv/rsHFbV+N0JqNCekItKXkNDnz4ghLTMJX3RMM6/FK8z
| udPje7NTrvBI4hSXPHQ6WSNohhHD3O6v2RJqFYvMoCwlewKHaeGDyRmWkRK0edU4
| CRS+93l9Zep2U6KIyyYOpVw6z/P+a6H4sCfIanTG8VWErzGiKt1YftrCp2EMsGvB
|_-----END CERTIFICATE-----
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -847d11h54m27s, deviation: 0s, median: -847d11h54m27s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 54672/tcp): CLEAN (Timeout)
|   Check 2 (port 27970/tcp): CLEAN (Timeout)
|   Check 3 (port 60014/udp): CLEAN (Timeout)
|   Check 4 (port 62023/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2023-09-15T15:33:04
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:28
Completed NSE at 12:28, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:28
Completed NSE at 12:28, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:28
Completed NSE at 12:28, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 106.91 seconds
           Raw packets sent: 10 (416B) | Rcvd: 7 (292B)

```