このポートの見逃しがあった。。
PowerShellWebAccessTestWebsiteとあるが、これはWindowsの機能で、ブラウザからPowerShellセッションを開始できる。

# Nmap

```
PORT   STATE SERVICE REASON  VERSION
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2023-08-24T08:19:45
|_Not valid after:  2023-11-22T08:19:45
|_ssl-date: 2023-09-15T15:30:17+00:00; -2y116d11h54m27s from scanner time.
|_http-server-header: Microsoft-IIS/10.0
| tls-alpn: 
|   h2
|_  http/1.1
```

---

# Whatweb

```zsh

```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh

```

---

# Nikto

```zsh

```

---

# Enumeration


---

# Screenshot

![[Pasted image 20260110193956.png]]
