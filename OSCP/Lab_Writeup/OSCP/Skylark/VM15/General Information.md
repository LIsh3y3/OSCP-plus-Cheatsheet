# Host Information

```zsh

```

---

# Scans

## Nmap

### quick scan

```
PORT    STATE  SERVICE VERSION
80/tcp  closed http
443/tcp closed https
```
### full scan

```
PORT      STATE  SERVICE VERSION
80/tcp    closed http
443/tcp   closed https
60001/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 5.0 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
```

### udp scan

```
PORT    STATE  SERVICE VERSION
80/tcp  closed http
443/tcp closed https
80/udp  closed http
443/udp closed https
```

## Rustscan

```
PORT      STATE SERVICE REASON         VERSION
60001/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
```