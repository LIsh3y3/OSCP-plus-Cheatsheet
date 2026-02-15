# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Home page - Skylark Partner Portal
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=SKYLARK
|_http-server-header: Microsoft-IIS/10.0
```

---

# Whatweb

## Basic認証情報なし

```zsh
──(koshi㉿kali)-[~/PEN-200/Skylark/VM12]
└─$ whatweb -v -a3 --log-verbose WebEnum/whatweb_80_nonauth.txt http://$TargetIP          
WhatWeb report for http://192.168.222.220
Status    : 401 Unauthorized
Title     : Home page - Skylark Partner Portal
IP        : 192.168.222.220
Country   : RESERVED, ZZ

Summary   : Bootstrap, HTML5, HTTPServer[Microsoft-IIS/10.0], JQuery, Microsoft-IIS[10.0], Script, WWW-Authenticate[SKYLARK][Basic]

Detected Plugins:
[ Bootstrap ]
        Bootstrap is an open source toolkit for developing with 
        HTML, CSS, and JS. 

        Website     : https://getbootstrap.com/

[ HTML5 ]
        HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
        HTTP server header string. This plugin also attempts to 
        identify the operating system from the server header. 

        String       : Microsoft-IIS/10.0 (from server string)

[ JQuery ]
        A fast, concise, JavaScript that simplifies how to traverse 
        HTML documents, handle events, perform animations, and add 
        AJAX. 

        Website     : http://jquery.com/

[ Microsoft-IIS ]
        Microsoft Internet Information Services (IIS) for Windows 
        Server is a flexible, secure and easy-to-manage Web server 
        for hosting anything on the Web. From media streaming to 
        web application hosting, IIS's scalable and open 
        architecture is ready to handle the most demanding tasks. 

        Version      : 10.0
        Website     : http://www.iis.net/

[ Script ]
        This plugin detects instances of script HTML elements and 
        returns the script language/type. 


[ WWW-Authenticate ]
        This plugin identifies the WWW-Authenticate HTTP header and 
        extracts the authentication method and realm. 

        Module       : Basic
        String       : SKYLARK

HTTP Headers:
        HTTP/1.1 401 Unauthorized
        Transfer-Encoding: chunked
        Content-Type: text/html; charset=utf-8
        Server: Microsoft-IIS/10.0
        WWW-Authenticate: Basic realm="SKYLARK"
        Date: Sun, 15 Feb 2026 00:23:55 GMT
        Connection: close

```

---

# Gobuster

errorが多ければ`-t 64`も試す

## Basic認証情報なし

```zsh

```

---

# Nikto

```zsh

```

---

# Enumeration

## index

- Basic認証を求められるが、ページの一部は表示される


---

# Screenshot

- index
![[Pasted image 20260215092433.png]]
