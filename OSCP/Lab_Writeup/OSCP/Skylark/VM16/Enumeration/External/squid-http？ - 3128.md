- デフォルトの認証情報はない
- squid v4.1とのことだが、エクスプロイトはない

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
l.nguyen@skylark.com:ChangeMePlease__XMPPTest
j.jameson@skylark.com:ChangeMePlease__XMPPTest
j.jones@skylark.com:ChangeMePlease__XMPPTest
reynolds@testing.com:bj1ZzWEmJd870YHMsV+NdfsTuce+zsQRyvb7e5fIgY8=
```

---

# Screenshot

- index
	- localhost 8080で何か動いてそう
![[Pasted image 20260215102128.png]]
