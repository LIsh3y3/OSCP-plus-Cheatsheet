# Host Information

```zsh
TargetIP=172.16.115.202
```

---

# Scans

## Nmap

### quick scan

```
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ sudo nmap -sC -sV -oA Nmap/quickscan -T4 $TargetIP
[sudo] password for koshi: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 11:27 +0900
Nmap scan report for 172.16.115.202
Host is up (0.060s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-01-10T02:27:57
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.93 seconds
                                                                
```
![[Pasted image 20260110113515.png]]

### full scan

```
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ # openポートを詳細スキャン   
sudo nmap -A -sV -p $ports -oA Nmap/fullscan $TargetIP
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 14:17 +0900
Nmap scan report for 172.16.115.202
Host is up (0.24s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-01-10T05:18:14
|_  start_date: N/A

TRACEROUTE
HOP RTT       ADDRESS
1   244.88 ms 172.16.115.202

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.07 seconds

```

### udp scan

```

```

## Rustscan

```

```