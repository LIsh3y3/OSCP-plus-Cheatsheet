# Host Information

```zsh

```

---

# Scans

## Nmap

### quick scan

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.18.0 (Ubuntu)
8090/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: 403 Forbidden
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
### full scan

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.18.0 (Ubuntu)
8090/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: 403 Forbidden
```

### udp scan

```
PORT      STATE         SERVICE     VERSION
21/tcp    open          ftp         vsftpd 3.0.3
80/tcp    open          http        nginx 1.18.0 (Ubuntu)
69/udp    open|filtered tftp
...
```

## Rustscan

```
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 61 vsftpd 3.0.3
80/tcp   open  http    syn-ack ttl 61 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.18.0 (Ubuntu)
8090/tcp open  http    syn-ack ttl 61 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-title: 403 Forbidden
```