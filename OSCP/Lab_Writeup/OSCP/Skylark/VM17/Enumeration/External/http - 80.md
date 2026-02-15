# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.18.0 (Ubuntu)
```

---

# Whatweb

```zsh

```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
gobuster dir -u http://$TargetIP/ -r w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster 80.txt -x 'html,ht m, txt, sh, php, cgi, asp, aspx, jsp, pl, py, pdf -s 200,301,302 -b ""
...
/. 200
```

---

# Nikto

```zsh

```

---

# Enumeration


---

# Screenshot


