# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 6.0.2
|_http-title: The Stationery Warehouse &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-trane-info: Problem with XML parsing of /evox/about
```

---

# Whatweb

```zsh
WhatWeb report for http://192.168.104.112
Status    : 200 OK
Title     : The Stationery Warehouse &#8211; Just another WordPress site
IP        : 192.168.104.112
Country   : RESERVED, ZZ

Summary   : Apache[2.4.41], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], JQuery[3.6.0], MetaGenerator[WordPress 6.0.2], PoweredBy[WordPress], Script[text/javascript], UncommonHeaders[link], WordPress[6.0.2]

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and 
	maintain an open-source HTTP server for modern operating 
	systems including UNIX and Windows NT. The goal of this 
	project is to provide a secure, efficient and extensible 
	server that provides HTTP services in sync with the current 
	HTTP standards. 

	Version      : 2.4.41 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTML5 ]
	HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	OS           : Ubuntu Linux
	String       : Apache/2.4.41 (Ubuntu) (from server string)

[ JQuery ]
	A fast, concise, JavaScript that simplifies how to traverse 
	HTML documents, handle events, perform animations, and add 
	AJAX. 

	Version      : 3.6.0
	Website     : http://jquery.com/

[ MetaGenerator ]
	This plugin identifies meta generator tags and extracts its 
	value. 

	String       : WordPress 6.0.2

[ PoweredBy ]
	This plugin identifies instances of 'Powered by x' text and 
	attempts to extract the value for x. 

	String       : WordPress

[ Script ]
	This plugin detects instances of script HTML elements and 
	returns the script language/type. 

	String       : text/javascript

[ UncommonHeaders ]
	Uncommon HTTP server headers. The blacklist includes all 
	the standard headers and many non standard but common ones. 
	Interesting but fairly common headers should have their own 
	plugins, eg. x-powered-by, server and x-aspnet-version. 
	Info about headers can be found at www.http-stats.com 

	String       : link (from headers)

[ WordPress ]
	WordPress is an opensource blogging system commonly used as 
	a CMS. 

	Version      : 6.0.2
	Aggressive function available (check plugin file or details).
	Google Dorks: (1)
	Website     : http://www.wordpress.org/

HTTP Headers:
	HTTP/1.1 200 OK
	Date: Sun, 22 Feb 2026 02:38:12 GMT
	Server: Apache/2.4.41 (Ubuntu)
	Link: <http://192.168.104.112/wp-json/>; rel="https://api.w.org/"
	Link: <http://192.168.104.112/wp-json/wp/v2/pages/6>; rel="alternate"; type="application/json"
	Link: <http://192.168.104.112/>; rel=shortlink
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 6870
	Connection: close
	Content-Type: text/html; charset=UTF-8
	

```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
┌──(koshi㉿kali)-[~/Exam/Lucky]
└─$ gobuster dir -u http://$TargetIP:/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster.txt -x 'html,htm,txt,sh,php,cgi,pl,py,pdf' --no-error
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.104.112:/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,php,pl,py,htm,sh,cgi,pdf,html
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================


/login                (Status: 200) [Size: 6265]
/admin                (Status: 200) [Size: 6265]
/wp-content           (Status: 200) [Size: 0]
/wp-admin             (Status: 200) [Size: 6265]
/index.php            (Status: 200) [Size: 33221]
/feed                 (Status: 200) [Size: 1934]
/contact              (Status: 200) [Size: 27237]
/rss                  (Status: 200) [Size: 1934]
/blog                 (Status: 200) [Size: 27313]
/wp-login.php         (Status: 200) [Size: 6265]
/wp-register.php      (Status: 200) [Size: 6431]
/about                (Status: 200) [Size: 27225]
/1                    (Status: 200) [Size: 30813]
/2                    (Status: 200) [Size: 30796]
/a                    (Status: 200) [Size: 27225]
/c                    (Status: 200) [Size: 27237]
/s                    (Status: 200) [Size: 28346]
/.                    (Status: 200) [Size: 33173]
/readme.html          (Status: 200) [Size: 7401]
/wp-feed.php          (Status: 200) [Size: 1934]
/0                    (Status: 200) [Size: 33173]
/license.txt          (Status: 200) [Size: 19915]
/b                    (Status: 200) [Size: 27313]
/robots.txt           (Status: 200) [Size: 115]
/.1                   (Status: 200) [Size: 30814]
/atom                 (Status: 200) [Size: 1788]
/wp-config.php        (Status: 200) [Size: 0]
/wp-commentsrss2.php  (Status: 200) [Size: 1659]
/.2                   (Status: 200) [Size: 30797]
/dashboard            (Status: 200) [Size: 6265]
/h                    (Status: 200) [Size: 31942]
/Contact              (Status: 200) [Size: 27237]
/sample               (Status: 200) [Size: 28346]
/create               (Status: 200) [Size: 33173]
/comment-page-1       (Status: 200) [Size: 33173]
/wp-rss.php           (Status: 200) [Size: 1934]
/wp-rss2.php          (Status: 200) [Size: 1934]
/wp-cron.php          (Status: 200) [Size: 0]
/wp-rdf.php           (Status: 200) [Size: 1514]
/wp-atom.php          (Status: 200) [Size: 1788]
/Blog                 (Status: 200) [Size: 27313]
/wp-links-opml.php    (Status: 200) [Size: 239]
/About                (Status: 200) [Size: 27225]
/embed                (Status: 200) [Size: 33173]
/cont                 (Status: 200) [Size: 27237]
/co                   (Status: 200) [Size: 27237]
/A                    (Status: 200) [Size: 27225]
/rss2                 (Status: 200) [Size: 1934]
/sa                   (Status: 200) [Size: 28346]
/cr                   (Status: 200) [Size: 33173]
/C                    (Status: 200) [Size: 27237]
/S                    (Status: 200) [Size: 28346]
/wp-load.php          (Status: 200) [Size: 0]
/B                    (Status: 200) [Size: 27313]
/ab                   (Status: 200) [Size: 27225]
/H                    (Status: 200) [Size: 31942]
/he                   (Status: 200) [Size: 31942]
/wp-signup.php        (Status: 200) [Size: 6431]
/page2                (Status: 200) [Size: 33197]
/hello                (Status: 200) [Size: 31942]
/page1                (Status: 200) [Size: 33173]
/rdf                  (Status: 200) [Size: 1514]
/Sample               (Status: 200) [Size: 28346]
/sam                  (Status: 200) [Size: 28346]
/.c                   (Status: 200) [Size: 27237]
/comment-page-2       (Status: 200) [Size: 33173]
/hello-world          (Status: 200) [Size: 31942]
/wp-activate.php      (Status: 200) [Size: 6431]
/page3                (Status: 200) [Size: 33197]
/con                  (Status: 200) [Size: 27237]
/abo                  (Status: 200) [Size: 27225]
/.a                   (Status: 200) [Size: 27225]
/bl                   (Status: 200) [Size: 27313]
/SA                   (Status: 200) [Size: 28346]
/comment-page-3       (Status: 200) [Size: 33173]
/page4                (Status: 200) [Size: 33197]
/page5                (Status: 200) [Size: 33197]
/page6                (Status: 200) [Size: 33197]
/CO                   (Status: 200) [Size: 27237]
/0000                 (Status: 200) [Size: 27300]
/CR                   (Status: 200) [Size: 33173]
/.h                   (Status: 200) [Size: 31942]
/conta                (Status: 200) [Size: 27237]
/page7                (Status: 200) [Size: 33197]
/.s                   (Status: 200) [Size: 28346]
/comment-page-4       (Status: 200) [Size: 33173]
/.A                   (Status: 200) [Size: 27225]
/.b                   (Status: 200) [Size: 27313]
/.sample              (Status: 200) [Size: 28346]
/AB                   (Status: 200) [Size: 27225]
/cre                  (Status: 200) [Size: 33173]
/page10               (Status: 200) [Size: 33201]
/sample-page          (Status: 200) [Size: 28346]
/ABOUT                (Status: 200) [Size: 27225]
/.co                  (Status: 200) [Size: 27237]
/BL                   (Status: 200) [Size: 27313]
/BLOG                 (Status: 200) [Size: 27313]
/comment-page-6       (Status: 200) [Size: 33173]
/comment-page-5       (Status: 200) [Size: 33173]
/page9                (Status: 200) [Size: 33197]
/CONTACT              (Status: 200) [Size: 27237]
/creat                (Status: 200) [Size: 33173]
/page14               (Status: 200) [Size: 33201]
/page404              (Status: 200) [Size: 33205]
/page8                (Status: 200) [Size: 33197]
/.S                   (Status: 200) [Size: 28346]
/Create               (Status: 200) [Size: 33173]
/page22               (Status: 200) [Size: 33201]
/page17               (Status: 200) [Size: 33201]
/page26               (Status: 200) [Size: 33201]
/.C                   (Status: 200) [Size: 27237]
/.contact             (Status: 200) [Size: 27238]
/.sitemap.xml         (Status: 200) [Size: 532]
/2021                 (Status: 200) [Size: 27314]
/Co                   (Status: 200) [Size: 27237]
/Sam                  (Status: 200) [Size: 28346]
/comment-page-7       (Status: 200) [Size: 33173]
/crea                 (Status: 200) [Size: 33173]
/page11               (Status: 200) [Size: 33201]
/page20               (Status: 200) [Size: 33201]
/samp                 (Status: 200) [Size: 28346]
/stationery           (Status: 200) [Size: 30694]
/.C.                  (Status: 200) [Size: 27237]
/.about               (Status: 200) [Size: 27226]
/.blog                (Status: 200) [Size: 27313]
/.create              (Status: 200) [Size: 33173]
/HE                   (Status: 200) [Size: 31942]
/Stationery           (Status: 200) [Size: 30694]
/hel                  (Status: 200) [Size: 31942]
/page19               (Status: 200) [Size: 33201]
/page21               (Status: 200) [Size: 33201]
/page23               (Status: 200) [Size: 33201]
/page16               (Status: 200) [Size: 33201]
/page40               (Status: 200) [Size: 33201]
/page67               (Status: 200) [Size: 33201]
/page65               (Status: 200) [Size: 33201]
/page35               (Status: 200) [Size: 33201]
/page27               (Status: 200) [Size: 33201]
/.H                   (Status: 200) [Size: 31942]
/Conta                (Status: 200) [Size: 27237]
/Hello                (Status: 200) [Size: 31942]
/SAM                  (Status: 200) [Size: 28346]
/blo                  (Status: 200) [Size: 27313]
/comment-page-10      (Status: 200) [Size: 33173]
/comment-page-9       (Status: 200) [Size: 33173]
/page01               (Status: 200) [Size: 33173]
/page13               (Status: 200) [Size: 33201]
/page12               (Status: 200) [Size: 33201]
/page15               (Status: 200) [Size: 33201]
/page18               (Status: 200) [Size: 33201]
/page25               (Status: 200) [Size: 33201]
/page24               (Status: 200) [Size: 33201]
/page28               (Status: 200) [Size: 33201]
/page31               (Status: 200) [Size: 33201]
/page32               (Status: 200) [Size: 33201]
/page30               (Status: 200) [Size: 33201]
/page37               (Status: 200) [Size: 33201]
/page59               (Status: 200) [Size: 33201]
/page55               (Status: 200) [Size: 33201]
/page39               (Status: 200) [Size: 33201]
/page46               (Status: 200) [Size: 33201]
/page47               (Status: 200) [Size: 33201]
/page61               (Status: 200) [Size: 33201]
/page66               (Status: 200) [Size: 33201]
/pencil               (Status: 200) [Size: 30637]
/sitemap-xml          (Status: 200) [Size: 532]

```

---

# WPScan

```sh
wpscan --url http://$TargetIP --no-banner --enumerate p,t --plugins-detection aggressive -o WebEnum/wpscan --api-token svP1khwitgUTmxDwLDuZZRTDg6ms875XmsU8MZVLqYs
```


---

# Nikto

```zsh

```

---

# Enumeration

## wp-login.php

- 以下のユーザー名で試行
```sh
lee@auditorzr.us
sam@auditorzr.us
simon@auditorzr.us
lee
sam
simon
```
- emailのとき："Unknown email address. Check again or try your sername."
- ユーザー名のとき："The username lee is not registered on this site. If you are unsure of your username, try your email address instead."

---

# Screenshot


