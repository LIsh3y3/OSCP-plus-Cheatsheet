# Host Information

```zsh
IP:192.168.115.206
Host: WS26.oscp.exam
```

---

# Scans

## Nmap

### quick scan

```
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26]
└─$ sudo nmap -sC -sV -oA Nmap/quickscan -T4 $TargetIP
[sudo] password for koshi: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 09:27 +0900
Nmap scan report for 192.168.115.206
Host is up (0.23s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=WS26.oscp.exam
| Not valid before: 2024-11-12T11:04:26
|_Not valid after:  2025-05-14T11:04:26
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: OSCP
|   NetBIOS_Domain_Name: OSCP
|   NetBIOS_Computer_Name: WS26
|   DNS_Domain_Name: oscp.exam
|   DNS_Computer_Name: WS26.oscp.exam
|   DNS_Tree_Name: oscp.exam
|   Product_Version: 10.0.22621
|_  System_Time: 2024-11-13T11:17:02+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=1/10%Time=69619D0D%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
![[Pasted image 20260110092927.png]]

### full scan

```
$ # openポートを詳細スキャン
sudo nmap -A -sV -p $ports -oA Nmap/fullscan $TargetIP
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 09:36 +0900
Nmap scan report for WS26.oscp.exam (192.168.115.206)
Host is up (0.24s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: OSCP
|   NetBIOS_Domain_Name: OSCP
|   NetBIOS_Computer_Name: WS26
|   DNS_Domain_Name: oscp.exam
|   DNS_Computer_Name: WS26.oscp.exam
|   DNS_Tree_Name: oscp.exam
|   Product_Version: 10.0.22621
|_  System_Time: 2024-11-13T11:26:08+00:00
| ssl-cert: Subject: commonName=WS26.oscp.exam
| Not valid before: 2024-11-12T11:21:32
|_Not valid after:  2025-05-14T11:21:32
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=1/10%Time=69619F2D%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 11|2008|7 (88%)
OS CPE: cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows 11 21H2 (88%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### udp scan

```

```

## Rustscan

```

```