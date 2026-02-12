# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-title: APEX Hospital
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

---

# Whatweb

```zsh
┌──(koshi㉿kali)-[~/ProvingGrounds/Apex]
└─$ whatweb -v -a3 --log-verbose WebEnum/whatweb_80.txt http://$TargetIP                                        
WhatWeb report for http://192.168.154.145
Status    : 200 OK
Title     : APEX Hospital
IP        : 192.168.154.145
Country   : RESERVED, ZZ

Summary   : Apache[2.4.29], Bootstrap, Email[awilliam@apex.offs,awilliam@apex.offsec,contact@apex.offs,contact@apex.offsec,jamanda@apex.offs,jamanda@apex.offsec,jsarah@apex.offs,jsarah@apex.offsec,wwalter@apex.offs,wwalter@apex.offsec], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], JQuery, Script

Detected Plugins:
[ Apache ]
        The Apache HTTP Server Project is an effort to develop and 
        maintain an open-source HTTP server for modern operating 
        systems including UNIX and Windows NT. The goal of this 
        project is to provide a secure, efficient and extensible 
        server that provides HTTP services in sync with the current 
        HTTP standards. 

        Version      : 2.4.29 (from HTTP Server Header)
        Google Dorks: (3)
        Website     : http://httpd.apache.org/

[ Bootstrap ]
        Bootstrap is an open source toolkit for developing with 
        HTML, CSS, and JS. 

        Website     : https://getbootstrap.com/

[ Email ]
        Extract email addresses. Find valid email address and 
        syntactically invalid email addresses from mailto: link 
        tags. We match syntactically invalid links containing 
        mailto: to catch anti-spam email addresses, eg. bob at 
        gmail.com. This uses the simplified email regular 
        expression from 
        http://www.regular-expressions.info/email.html for valid 
        email address matching. 

        String       : awilliam@apex.offs,contact@apex.offs,jamanda@apex.offs,jsarah@apex.offs,wwalter@apex.offs
        String       : awilliam@apex.offsec,contact@apex.offsec,jamanda@apex.offsec,jsarah@apex.offsec,wwalter@apex.offsec

[ HTML5 ]
        HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
        HTTP server header string. This plugin also attempts to 
        identify the operating system from the server header. 

        OS           : Ubuntu Linux
        String       : Apache/2.4.29 (Ubuntu) (from server string)

[ JQuery ]
        A fast, concise, JavaScript that simplifies how to traverse 
        HTML documents, handle events, perform animations, and add 
        AJAX. 

        Website     : http://jquery.com/

[ Script ]
        This plugin detects instances of script HTML elements and 
        returns the script language/type. 


HTTP Headers:
        HTTP/1.1 200 OK
        Date: Thu, 12 Feb 2026 14:03:36 GMT
        Server: Apache/2.4.29 (Ubuntu)
        Last-Modified: Mon, 17 May 2021 15:00:14 GMT
        ETag: "711d-5c287d9d2c6e3-gzip"
        Accept-Ranges: bytes
        Vary: Accept-Encoding
        Content-Encoding: gzip
        Content-Length: 4777
        Connection: close
        Content-Type: text/html


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


