# Nmap

```
PORT   STATE SERVICE REASON  VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-generator: Nicepage 5.13.1, nicepage.com
|_http-title: Home
| http-methods: 
|_  Potentially risky methods: TRACE
```

---

# Whatweb

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ whatweb -v -a3 --log-verbose WebEnum/whatweb_80.txt http://$TargetIP     
WhatWeb report for http://192.168.115.111
Status    : 200 OK
Title     : Home
IP        : 192.168.115.111
Country   : RESERVED, ZZ

Summary   : HTML5, HTTPServer[Microsoft-IIS/10.0], JQuery, MetaGenerator[Nicepage 5.13.1, nicepage.com], Microsoft-IIS[10.0], Open-Graph-Protocol[website], Script[application/ld+json,text/javascript], X-Powered-By[ASP.NET]

Detected Plugins:
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

[ MetaGenerator ]
        This plugin identifies meta generator tags and extracts its 
        value. 

        String       : Nicepage 5.13.1, nicepage.com

[ Microsoft-IIS ]
        Microsoft Internet Information Services (IIS) for Windows 
        Server is a flexible, secure and easy-to-manage Web server 
        for hosting anything on the Web. From media streaming to 
        web application hosting, IIS's scalable and open 
        architecture is ready to handle the most demanding tasks. 

        Version      : 10.0
        Website     : http://www.iis.net/

[ Open-Graph-Protocol ]
        The Open Graph protocol enables you to integrate your Web 
        pages into the social graph. It is currently designed for 
        Web pages representing profiles of real-world things . 
        things like movies, sports teams, celebrities, and 
        restaurants. Including Open Graph tags on your Web page, 
        makes your page equivalent to a Facebook Page. 

        Version      : website

[ Script ]
        This plugin detects instances of script HTML elements and 
        returns the script language/type. 

        String       : application/ld+json,text/javascript

[ X-Powered-By ]
        X-Powered-By HTTP header 

        String       : ASP.NET (from x-powered-by string)

HTTP Headers:
        HTTP/1.1 200 OK
        Content-Type: text/html
        Last-Modified: Thu, 20 Jul 2023 16:08:08 GMT
        Accept-Ranges: bytes
        ETag: "064865f24bbd91:0"
        Server: Microsoft-IIS/10.0
        X-Powered-By: ASP.NET
        Date: Sat, 10 Jan 2026 09:28:01 GMT
        Connection: close
        Content-Length: 54212

```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ gobuster dir -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -x "html,htm,txt,sh,php,cgi,asp,aspx,jsp,cgi,pl,py,sh,pdf" -o WebEnum/gobuster_80.txt -u http://$TargetIP/ --no-error                         
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.115.111/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,txt,cgi,aspx,pl,pdf,htm,sh,php,asp,jsp,py
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 153] [--> http://192.168.115.111/images/]
/index.html           (Status: 200) [Size: 54212]
/aspnet_client        (Status: 301) [Size: 160] [--> http://192.168.115.111/aspnet_client/]
/home.html            (Status: 200) [Size: 54157]
/Images               (Status: 301) [Size: 153] [--> http://192.168.115.111/Images/]
/.                    (Status: 200) [Size: 54212]
/Home.html            (Status: 200) [Size: 54157]
/Index.html           (Status: 200) [Size: 54212]
/IMAGES               (Status: 301) [Size: 153] [--> http://192.168.115.111/IMAGES/]
/launch               (Status: 301) [Size: 153] [--> http://192.168.115.111/launch/]
/HOME.html            (Status: 200) [Size: 54157]
/Aspnet_client        (Status: 301) [Size: 160] [--> http://192.168.115.111/Aspnet_cli
/aspnet_Client        (Status: 301) [Size: 160] [--> http://192.168.115.111/aspnet_Cli
/iisstart.htm         (Status: 200) [Size: 703]
/INDEX.html           (Status: 200) [Size: 54212]
/ASPNET_CLIENT        (Status: 301) [Size: 160] [--> http://192.168.115.111/ASPNET_CLI
/Launch               (Status: 301) [Size: 153] [--> http://192.168.115.111/Launch/]
Progress: 820144 / 820144 (100.00%)
===============================================================
Finished
===============================================================

```

---

# feroxbuster

- Launch配下の再帰探索
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ feroxbuster -u http://192.168.115.111/Launch/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x html,htm,txt,sh,php,cgi,asp,aspx,jsp,cgi,pl,py,sh,pdf -t 100 -o WebEnum/feroxbuster_Launch.txt -s 200,301
                                                                                      
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.115.111/Launch
 🚩  In-Scope Url          │ 192.168.115.111
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
 👌  Status Codes          │ [200, 301]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💾  Output File           │ WebEnum/feroxbuster_Launch.txt
 💲  Extensions            │ [html, htm, txt, sh, php, cgi, asp, aspx, jsp, cgi, pl, py, sh, pdf]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        2l       10w      153c http://192.168.115.111/Launch => http://192.168.115.111/Launch/
200      GET       86l      281w     2675c http://192.168.115.111/Launch/index.html
200      GET       86l      281w     2675c http://192.168.115.111/Launch/
200      GET       86l      281w     2675c http://192.168.115.111/Launch/Index.html
200      GET       86l      281w     2675c http://192.168.115.111/Launch/INDEX.html
[####################] - 56m   946410/946410  0s      found:5       errors:14     
[####################] - 56m   946335/946335  281/s   http://192.168.115.111/Launch/   
```

- iisstart.htm配下の再帰探索
```zsh

```

- aspnet_clientの再帰探索
```zsh

```

---

# Nikto

```zsh

```

---

# Enumeration

## index

- ==静的ページ==
- 開発者ツールでコードを閲覧するも、何もなし
- usernameのリストになりそう
- ユーザー名を収集しておく
```zsh
cewl -w usernames.txt -d 5 -m 5 http://192.168.115.111/
curl \
  -H "X-Forwarded-Host: localhost" \
  -H "X-Forwarded-For: 127.0.0.1" \
  http://192.168.115.111/Launch/
```

launch souce
```
```

![[Pasted image 20260110183818.png]]

## launch

- Visit Launch6!に飛ぶと、localhostとなるため、内部用かと思われる。
![[Pasted image 20260110184814.png]]

```
curl -H "Host: localhost" http://192.168.115.111/Launch/
```
---

# Screenshot

- indexページ
![[Pasted image 20260110185057.png]]

- launchページ
![[Pasted image 20260110184751.png]]


- iisstart.htm
![[Pasted image 20260110185510.png]]
