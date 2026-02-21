# Host Information

```zsh

```

---

# Scans

## Nmap

### quick scan

```
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ sudo nmap -sC -sV -oA Nmap/quickscan -T4 $TargetIP -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 14:39 +0900
Nmap scan report for 192.168.115.112
Host is up (0.24s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d5:32:4b:71:2d:31:2b:4a:ba:10:16:0b:f0:bd:3d:8b (ECDSA)
|_  256 49:6b:01:37:10:e5:6f:49:37:4e:65:14:79:31:da:79 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Offsec Insurance | We value your security!
|_http-server-header: Apache/2.4.52 (Ubuntu)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.92 seconds
```
![[Pasted image 20260110145447.png]]

### full scan

```
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ # openポートを詳細スキャン
sudo nmap -A -sV -p $ports -oA Nmap/fullscan $TargetIP
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 15:15 +0900
Nmap scan report for 192.168.115.112
Host is up (0.22s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d5:32:4b:71:2d:31:2b:4a:ba:10:16:0b:f0:bd:3d:8b (ECDSA)
|_  256 49:6b:01:37:10:e5:6f:49:37:4e:65:14:79:31:da:79 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Offsec Insurance | We value your security!
|_http-server-header: Apache/2.4.52 (Ubuntu)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router|firewall
Running (JUST GUESSING): Linux 4.X|5.X|6.X (97%), MikroTik RouterOS 7.X (91%), IPFire 2.X (88%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:ipfire:ipfire:2.27 cpe:/o:linux:linux_kernel:6.1
Aggressive OS guesses: Linux 4.19 - 5.15 (97%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%), Linux 4.15 (90%), IPFire 2.27 (Linux 5.15 - 6.1) (88%), Linux 5.4 (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   226.91 ms 192.168.49.1
2   227.01 ms 192.168.115.112

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.05 seconds

```

### udp scan

```
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ nmap -sU -sS -sV $TargetIP -T4 -oN Nmap/udpscan.nmap
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 15:36 +0900
Nmap scan report for 192.168.115.112
Host is up (0.23s latency).
Not shown: 1000 open|filtered udp ports (no-response), 996 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3569.47 seconds
                                                                  
```

## Rustscan

```
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
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
Open ports, closed hearts.

[~] The config file is expected to be at "/home/koshi/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 192.168.115.112:80
Open 192.168.115.112:3306
Open 192.168.115.112:22
Open 192.168.115.112:21
[~] Starting Script(s)
[>] Running script "nmap -vvv -p {{port}} -{{ipversion}} {{ip}} -sC -sV -oN Nmap/rustscan.nmap" on ip 192.168.115.112
Depending on the complexity of the script, results may take some time to appear.
[~] Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 15:33 +0900
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:33
Completed NSE at 15:33, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:33
Completed NSE at 15:33, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:33
Completed NSE at 15:33, 0.00s elapsed
Initiating Ping Scan at 15:33
Scanning 192.168.115.112 [4 ports]
Completed Ping Scan at 15:33, 0.24s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:33
Completed Parallel DNS resolution of 1 host. at 15:33, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 15:33
Scanning 192.168.115.112 [4 ports]
Discovered open port 22/tcp on 192.168.115.112
Discovered open port 80/tcp on 192.168.115.112
Discovered open port 3306/tcp on 192.168.115.112
Discovered open port 21/tcp on 192.168.115.112
Completed SYN Stealth Scan at 15:33, 0.25s elapsed (4 total ports)
Initiating Service scan at 15:33
Scanning 4 services on 192.168.115.112
Completed Service scan at 15:34, 10.14s elapsed (4 services on 1 host)
NSE: Script scanning 192.168.115.112.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 8.19s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 22.28s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 0.00s elapsed
Nmap scan report for 192.168.115.112
Host is up, received echo-reply ttl 63 (0.23s latency).
Scanned at 2026-01-10 15:33:51 JST for 41s

PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 63 vsftpd 3.0.5
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d5:32:4b:71:2d:31:2b:4a:ba:10:16:0b:f0:bd:3d:8b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIKTh1LWatixUsaOhMH9yWiUX27YS2y1Vpoxf1xMs6/1ME/c0jHLSb9kFD9SNAa/H+lLS3wFrg/uRA/2o+rIPaQ=
|   256 49:6b:01:37:10:e5:6f:49:37:4e:65:14:79:31:da:79 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP3XVKyaIavgKjqzuKG8iyMyI4T7Q7jPmCQi8RyZYud/
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.52 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 053F68E1BF8581477F85BA1DB568BE72
|_http-title: Offsec Insurance | We value your security!
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
3306/tcp open  mysql   syn-ack ttl 63 MariaDB 10.3.23 or earlier (unauthorized)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:34
Completed NSE at 15:34, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.35 seconds
           Raw packets sent: 8 (328B) | Rcvd: 5 (204B)

```