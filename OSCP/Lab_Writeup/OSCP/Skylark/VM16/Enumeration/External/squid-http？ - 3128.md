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
└─$ proxychains nmap -sT -n -p $ports -A -sV -oN Nmap/scan_via_proxy localhost
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-20 18:30 JST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000093s latency).
Other addresses for localhost (not scanned): ::1

PORT      STATE SERVICE    VERSION
8080/tcp  open  http-proxy
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported: GET HEAD CONNECTION
|_http-title: Burp Suite Community Edition
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Connection: close
|     Cache-control: no-cache, no-store
|     Pragma: no-cache
|     X-Frame-Options: DENY
|     Content-Type: text/html; charset=utf-8
|     X-Content-Type-Options: nosniff
|     <html><head><title>Burp Suite Community Edition</title>
|     <style type="text/css">
|     body { background: #dedede; font-family: Arial, sans-serif; color: #404042; -webkit-font-smoothing: antialiased; }
|     #container { padding: 0 15px; margin: 10px auto; background-color: #ffffff; }
|     word-wrap: break-word; }
|     a:link, a:visited { color: #e06228; text-decoration: none; }
|     a:hover, a:active { color: #404042; text-decoration: underline; }
|     font-size: 1.6em; line-height: 1.2em; font-weight: normal; color: #404042; }
|     font-size: 1.3em; line-height: 1.2em; padding: 0; margin: 0.8em 0 0.3em 0; font-weight: normal; color: #404042;}
|_    .title, .navbar { color: #ffffff; background: #e06228; padding: 10px 15px;
40539/tcp open  http       Golang net/http server
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Date: Fri, 20 Feb 2026 09:31:09 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Date: Fri, 20 Feb 2026 09:30:54 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
43091/tcp open  unknown
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
```

---

# Screenshot

- index
	- localhost 8080で何か動いてそう
![[Pasted image 20260215102128.png]]
