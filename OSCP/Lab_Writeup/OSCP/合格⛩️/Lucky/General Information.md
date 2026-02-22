# Host Information

```zsh
192.168.104.112
```
```
host: Linux
```

---

# Scans

## Nmap

### quick scan

```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0         3557581 Nov 25  2021 2d5ef5a0f0c9579458c9
| -rw-r--r--    1 0        0         1258508 Nov 25  2021 4835e976619690ae006e
| -rw-r--r--    1 0        0         1617905 Nov 25  2021 4e8cce46d6abec9a9d9a
| -rw-r--r--    1 0        0          438095 Nov 25  2021 77cfe070405f6ca327a5
|_-rw-r--r--    1 0        0          841392 Nov 25  2021 c5237630ef40e2585d35
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0e:84:80:bd:8f:b6:51:7d:c1:87:db:8c:f4:f3:15:9e (RSA)
|   256 8c:98:44:30:1c:37:53:84:32:22:eb:e1:9c:06:68:06 (ECDSA)
|_  256 1b:db:c7:c9:36:54:b8:cf:ff:1a:2f:9a:91:b1:56:e4 (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-title: The Stationery Warehouse &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-generator: WordPress 6.0.2
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: Host: the; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### full scan

```
PORT    STATE SERVICE     VERSION
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
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0e:84:80:bd:8f:b6:51:7d:c1:87:db:8c:f4:f3:15:9e (RSA)
|   256 8c:98:44:30:1c:37:53:84:32:22:eb:e1:9c:06:68:06 (ECDSA)
|_  256 1b:db:c7:c9:36:54:b8:cf:ff:1a:2f:9a:91:b1:56:e4 (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 6.0.2
|_http-title: The Stationery Warehouse &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-trane-info: Problem with XML parsing of /evox/about
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4
OS details: Linux 4.19 - 5.15
Network Distance: 2 hops
Service Info: Host: the; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### udp scan

```
PORT      STATE         SERVICE      VERSION
21/tcp    open          ftp          vsftpd 2.0.8 or later
22/tcp    open          ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open          http         Apache httpd 2.4.41 ((Ubuntu))
139/tcp   open          netbios-ssn  Samba smbd 4
445/tcp   open          netbios-ssn  Samba smbd 4
19/udp    open|filtered chargen
80/udp    open|filtered http
88/udp    open|filtered kerberos-sec
137/udp   open          netbios-ns   Samba nmbd netbios-ns (workgroup: WORKGROUP)
138/udp   open|filtered netbios-dgm
631/udp   open|filtered ipp
996/udp   open|filtered vsinet
2000/udp  open|filtered cisco-sccp
4500/udp  open|filtered nat-t-ike
5353/udp  open|filtered zeroconf
49154/udp open|filtered unknown
```

## Rustscan

```
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 2.0.8 or later
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
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0         3557581 Nov 25  2021 2d5ef5a0f0c9579458c9
| -rw-r--r--    1 0        0         1258508 Nov 25  2021 4835e976619690ae006e
| -rw-r--r--    1 0        0         1617905 Nov 25  2021 4e8cce46d6abec9a9d9a
| -rw-r--r--    1 0        0          438095 Nov 25  2021 77cfe070405f6ca327a5
|_-rw-r--r--    1 0        0          841392 Nov 25  2021 c5237630ef40e2585d35
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0e:84:80:bd:8f:b6:51:7d:c1:87:db:8c:f4:f3:15:9e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC9nHUN7lQ1VshjtRmnS2YHv5LnvtGDgMTUftd0YUWr2LRwigorhGPAiPTq/Tlw6P8XzkoZS/8bhK76OcMWMKm3zo7fisFQIFy3sH35mIo46aWvACqoKmF2mlXimNgQVtM6g2aWoxJ+Qlpci8TwcAvrO1em8vYgOa5oE//o7LIdpItoUhm7R1iNLwl9lOQpbJfUXIhAQZAzrURlfWm11WpmEP7ILtsjhYS4SQOwwll/0sbxp+i4YXyu34oTCn6OA0TZgxBIc47S7gz22qpT74rdbLnUzZm00wcZxGobSpr5qNQC+pfNtvtqt8d91anNkQAVZ1C8U0h5DaIRQmpEqMwJvsEZh/yMAiFEpn9s7DvXLT1aK8D5NS8VtDCK5O420ViLeGBWXIs6avIYxQisuQYA3Cip0hWJ79ai10pb1lER+43nMesriJQkIkdauEaH7IhNk42ub2TZGWxn874n4xKkdzPbLZOTxzUmpRotpGBHwKdchUcRZJOMafrFJ01YDLc=
|   256 8c:98:44:30:1c:37:53:84:32:22:eb:e1:9c:06:68:06 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCtZT7ZFuLJw/DZPlGRJkofY7SC8CTXL3DMmQjdOKGqJLtghqbWZIjVKmraQt8LCm7JHVCSUDrxP70S2MrxZfBw=
|   256 1b:db:c7:c9:36:54:b8:cf:ff:1a:2f:9a:91:b1:56:e4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPc0QcPocKaY+OjfHFNGX4JlKQ2KHKmGD76UuPRWrXvt
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 6.0.2
|_http-title: The Stationery Warehouse &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 38BC6973F189E1FBB976EB4864D93528
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4
Service Info: Host: the; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

![[Pasted image 20260222111845.png]]