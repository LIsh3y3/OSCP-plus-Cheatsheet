# Nmap

- 80とは異なり、Tomcatが使用されている
```
PORT   STATE SERVICE REASON  VERSION
443/tcp  open  ssl/http      Apache Tomcat 8.5.34
|_ssl-date: TLS randomness does not represent time
| http-title: Site doesn't have a title (text/html;charset=utf-8).
|_Requested resource was /cbs/Logon.do
| ssl-cert: Subject: commonName=Not Secure/organizationName=Ahsay System Corporation Limited/stateOrProvinceName=Hong Kong SAR/countryName=CN
| Not valid before: 2017-03-21T20:52:17
|_Not valid after:  2020-03-20T20:52:17
```

---

# Whatweb

```zsh
┌──(koshi㉿kali)-[~/Exam/Happy]
└─$ whatweb -v -a3 --log-verbose WebEnum/whatweb_443.txt http://$TargetIP:443
WhatWeb report for http://192.168.104.111:443
Status    : 400 Bad Request
Title     : <None>
IP        : 192.168.104.111
Country   : RESERVED, ZZ

Summary   : 

Detected Plugins:
HTTP Headers:
        HTTP/1.1 400 
        Content-Type: text/plain;charset=ISO-8859-1
        Connection: close

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


