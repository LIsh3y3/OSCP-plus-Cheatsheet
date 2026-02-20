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

- VM18の[[OSCP/Lab_Writeup/OSCP/Skylark/VM18/Enumeration/Internal/Administrator|Administrator]]で入手した認証情報を使ってスキャン
```proxychains4.conf
# socks4 	127.0.0.1 9050
http 192.168.141.224 3128 ext_acc DoNotShare!SkyLarkLegacyInternal2008
```
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM16]
└─$ proxychains nmap -sT -n 172.16.141.30-33 -T4                              
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-20 20:05 JST
```

---

# Screenshot

- index
	- localhost 8080で何か動いてそう
![[Pasted image 20260215102128.png]]
