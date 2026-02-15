# Nmap

```
PORT   STATE SERVICE REASON  VERSION
3128/tcp open  squid-http?
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

## index

- Basic認証の画面で以下のように言われる
```
The proxy moz-proxy://localhost:8080 is requesting a username and password. The site says: “SKYLARK LEGACY”
```

- 以下でBasic認証施行を実施するも失敗
```
SKYLARK:LEGACY
skylark:skylark
```

---

# Screenshot

- index
![[Pasted image 20260215102128.png]]
