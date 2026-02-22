# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp    open   http           Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: OSCP
```

---

# Whatweb -> nothing

```zsh
WhatWeb report for http://192.168.104.110
Status    : 200 OK
Title     : OSCP
IP        : 192.168.104.110
Country   : RESERVED, ZZ

Summary   : Apache[2.4.52], Bootstrap[3.3.1], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], JQuery, Meta-Author[webthemez], Script

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and 
	maintain an open-source HTTP server for modern operating 
	systems including UNIX and Windows NT. The goal of this 
	project is to provide a secure, efficient and extensible 
	server that provides HTTP services in sync with the current 
	HTTP standards. 

	Version      : 2.4.52 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ Bootstrap ]
	Bootstrap is an open source toolkit for developing with 
	HTML, CSS, and JS. 

	Version      : 3.3.1
	Version      : 3.3.1
	Website     : https://getbootstrap.com/

[ HTML5 ]
	HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	OS           : Ubuntu Linux
	String       : Apache/2.4.52 (Ubuntu) (from server string)

[ JQuery ]
	A fast, concise, JavaScript that simplifies how to traverse 
	HTML documents, handle events, perform animations, and add 
	AJAX. 

	Website     : http://jquery.com/

[ Meta-Author ]
	This plugin retrieves the author name from the meta name 
	tag - info: 
	http://www.webmarketingnow.com/tips/meta-tags-uncovered.html
	#author

	String       : webthemez

[ Script ]
	This plugin detects instances of script HTML elements and 
	returns the script language/type. 


HTTP Headers:
	HTTP/1.1 200 OK
	Date: Sun, 22 Feb 2026 02:38:24 GMT
	Server: Apache/2.4.52 (Ubuntu)
	Last-Modified: Thu, 24 Jul 2025 13:42:20 GMT
	ETag: "8218-63aacfe9a734d-gzip"
	Accept-Ranges: bytes
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 4498
	Connection: close
	Content-Type: text/html
	

```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
┌──(koshi㉿kali)-[~/Exam/Bless]
└─$ gobuster dir -u http://$TargetIP:/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster.txt -x 'html,htm,txt,sh,php,cgi,asp,aspx,jsp,pl,py,pdf'

...
/images               (Status: 200) [Size: 2718]
/index.html           (Status: 200) [Size: 33304]
/js                   (Status: 200) [Size: 3032]
/css                  (Status: 200) [Size: 1992]
/.                    (Status: 200) [Size: 33304]
/fonts                (Status: 200) [Size: 2849]
/readme.txt           (Status: 200) [Size: 1644]
```

---

# Nikto

```zsh

```

---

# Enumeration


---

# Screenshot

- index
