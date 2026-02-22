# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp   open  http          Samsung AllShare httpd
|_http-title: Did not follow redirect to https://192.168.104.111/cbs/Logon.do
```

---

# Whatweb

```zsh
WhatWeb report for http://192.168.104.111
Status    : 302 Found
Title     : <None>
IP        : 192.168.104.111
Country   : RESERVED, ZZ

Summary   : Cookies[JSESSIONID], HttpOnly[JSESSIONID], Java, RedirectLocation[/cbs/Logon.do]

Detected Plugins:
[ Cookies ]
	Display the names of cookies in the HTTP headers. The 
	values are not returned to save on space. 

	String       : JSESSIONID

[ HttpOnly ]
	If the HttpOnly flag is included in the HTTP set-cookie 
	response header and the browser supports it then the cookie 
	cannot be accessed through client side script - More Info: 
	http://en.wikipedia.org/wiki/HTTP_cookie 

	String       : JSESSIONID

[ Java ]
	Java allows you to play online games, chat with people 
	around the world, calculate your mortgage interest, and 
	view images in 3D, just to name a few. It's also integral 
	to the intranet applications and other e-business solutions 
	that are the foundation of corporate computing. 

	Website     : http://www.java.com/

[ RedirectLocation ]
	HTTP Server string location. used with http-status 301 and 
	302 

	String       : /cbs/Logon.do (from location)

HTTP Headers:
	HTTP/1.1 302 Found
	Set-Cookie: JSESSIONID=5F345069BA8961C47D3FF58E889BACF8; Path=/; HttpOnly
	Location: /cbs/Logon.do
	Content-Type: text/html;charset=ISO-8859-1
	Content-Length: 0
	Date: Sun, 22 Feb 2026 02:16:55 GMT
	Connection: close
	
WhatWeb report for http://192.168.104.111/cbs/Logon.do
Status    : 302 Found
Title     : <None>
IP        : 192.168.104.111
Country   : RESERVED, ZZ

Summary   : Cookies[JSESSIONID], HttpOnly[JSESSIONID], Java, RedirectLocation[https://192.168.104.111/cbs/Logon.do]

Detected Plugins:
[ Cookies ]
	Display the names of cookies in the HTTP headers. The 
	values are not returned to save on space. 

	String       : JSESSIONID

[ HttpOnly ]
	If the HttpOnly flag is included in the HTTP set-cookie 
	response header and the browser supports it then the cookie 
	cannot be accessed through client side script - More Info: 
	http://en.wikipedia.org/wiki/HTTP_cookie 

	String       : JSESSIONID

[ Java ]
	Java allows you to play online games, chat with people 
	around the world, calculate your mortgage interest, and 
	view images in 3D, just to name a few. It's also integral 
	to the intranet applications and other e-business solutions 
	that are the foundation of corporate computing. 

	Website     : http://www.java.com/

[ RedirectLocation ]
	HTTP Server string location. used with http-status 301 and 
	302 

	String       : https://192.168.104.111/cbs/Logon.do (from location)

HTTP Headers:
	HTTP/1.1 302 Found
	Set-Cookie: JSESSIONID=7A36490DDE720C3E15C7104E16E0BC70; Path=/cbs; HttpOnly
	Location: https://192.168.104.111/cbs/Logon.do
	Content-Type: text/html;charset=utf-8
	Content-Length: 0
	Date: Sun, 22 Feb 2026 02:16:55 GMT
	Connection: close
	
WhatWeb report for https://192.168.104.111/cbs/Logon.do
Status    : 200 OK
Title     : <None>
IP        : 192.168.104.111
Country   : RESERVED, ZZ

Summary   : Cookies[locale], HTML5, JQuery[1.7.1], PasswordField[systemPassword], Script[JavaScript,text/javascript]

Detected Plugins:
[ Cookies ]
	Display the names of cookies in the HTTP headers. The 
	values are not returned to save on space. 

	String       : locale

[ HTML5 ]
	HTML version 5, detected by the doctype declaration 


[ JQuery ]
	A fast, concise, JavaScript that simplifies how to traverse 
	HTML documents, handle events, perform animations, and add 
	AJAX. 

	Version      : 1.7.1
	Website     : http://jquery.com/

[ PasswordField ]
	find password fields 

	String       : systemPassword (from field name)

[ Script ]
	This plugin detects instances of script HTML elements and 
	returns the script language/type. 

	String       : JavaScript,text/javascript

HTTP Headers:
	HTTP/1.1 200 OK
	Set-Cookie: locale=en; Max-Age=31536000; Expires=Thu, 28-Mar-2024 08:46:07 GMT; Path=/cbs
	P3P: CP="NON DSP COR CURa ADMa DEVa CUSa TAIa OUR SAMa IND"
	Content-Type: text/html;charset=utf-8
	Transfer-Encoding: chunked
	Date: Sun, 22 Feb 2026 02:16:55 GMT
	Connection: close
```

---

# Gobuster

- jspが使われている

```zsh
┌──(koshi㉿kali)-[~/Exam/Happy]
└─$ gobuster dir -u http://$TargetIP:/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster.txt -x 'html,htm,txt,sh,php,cgi,asp,aspx,jsp,pl,py,pdf'

/index.jsp            (Status: 200) [Size: 51938]
/.                    (Status: 200) [Size: 51938]
/cbs                  (Status: 200) [Size: 210]

```

# Feroxbuster (cbs)

---

# Nikto

```zsh

```

---

# Enumeration

>AhsayCBSは、高度なクラウド/ローカルバックアップソフトウェア


---

# Screenshot

- index
![[Pasted image 20260222131928.png]]