# Nmap

```
PORT   STATE SERVICE REASON  VERSION
53/tcp open  domain? syn-ack
|_dns-nsec3-enum: Can't determine domain for host 10.10.11.174; use dns-nsec3-enum.domains script arg.
|_dns-nsec-enum: Can't determine domain for host 10.10.11.174; use dns-nsec-enum.domains script arg.

Host script results:
|_dns-brute: Can't guess domain of "10.10.11.174"; use dns-brute.domain script argument.
```

---

# Whatweb

```zsh
WhatWeb report for http://192.168.151.40
Status    : 200 OK
Title     : IIS Windows Server
IP        : 192.168.151.40
Country   : RESERVED, ZZ

Summary   : HTTPServer[Microsoft-IIS/10.0], Microsoft-IIS[10.0], X-Powered-By[ASP.NET]

Detected Plugins:
[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	String       : Microsoft-IIS/10.0 (from server string)

[ Microsoft-IIS ]
	Microsoft Internet Information Services (IIS) for Windows 
	Server is a flexible, secure and easy-to-manage Web server 
	for hosting anything on the Web. From media streaming to 
	web application hosting, IIS's scalable and open 
	architecture is ready to handle the most demanding tasks. 

	Version      : 10.0
	Website     : http://www.iis.net/

[ X-Powered-By ]
	X-Powered-By HTTP header 

	String       : ASP.NET (from x-powered-by string)

HTTP Headers:
	HTTP/1.1 200 OK
	Content-Type: text/html
	Content-Encoding: gzip
	Last-Modified: Sat, 25 Nov 2023 13:01:01 GMT
	Accept-Ranges: bytes
	ETag: "37dc3719f1fda1:0"
	Vary: Accept-Encoding
	Server: Microsoft-IIS/10.0
	X-Powered-By: ASP.NET
	Date: Wed, 11 Feb 2026 05:35:44 GMT
	Connection: close
	Content-Length: 609
```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
┌──(koshi㉿kali)-[~/ProvingGrounds/Hokkaido]
└─$ feroxbuster -u http://$TargetIP/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --auto-tune -o feroxbuster.txt -x 'aspx,asp,html,htm,txt,pdf,config,cs,ashx,asmx' -s 200,301,302

200      GET      334l     2089w   180418c http://192.168.151.40/iisstart.png
200      GET       32l       55w      703c http://192.168.151.40/
```

---

# Nikto

```zsh

```

---

# Enumeration


---

# Screenshot


