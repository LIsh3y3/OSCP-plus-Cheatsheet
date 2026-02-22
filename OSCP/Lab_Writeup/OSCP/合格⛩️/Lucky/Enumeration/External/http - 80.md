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


/login                (Status: 200) [Size: 6265]#←loginページ
/admin                (Status: 200) [Size: 6265]#←loginページ
/wp-content           (Status: 200) [Size: 0]
/wp-admin             (Status: 200) [Size: 6265]#←loginページ
/index.php            (Status: 200) [Size: 33221]
/feed                 (Status: 200) [Size: 1934]#←feedファイルのダウンロード
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
/sitemap-xml          (Status: 200) [Size: 532]#←nothing
```

---

# WPScan(長文注意)

```sh
$ wpscan --url http://$TargetIP --no-banner --enumerate p,t --plugins-detection aggressive -o WebEnum/wpscan --api-token svP1khwitgUTmxDwLDuZZRTDg6ms875XmsU8MZVLqYs

[+] URL: http://192.168.104.112/ [192.168.104.112]
[+] Started: Sun Feb 22 20:15:23 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://192.168.104.112/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.104.112/xmlrpc.php
 | Found By: Link Tag (Passive Detection)
 | Confidence: 100%
 | Confirmed By: Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.104.112/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.104.112/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.0.2 identified (Insecure, released on 2022-08-30).
 | Found By: Rss Generator (Passive Detection)
 |  - http://192.168.104.112/feed/, <generator>https://wordpress.org/?v=6.0.2</generator>
 |  - http://192.168.104.112/comments/feed/, <generator>https://wordpress.org/?v=6.0.2</generator>
 |
 | [!] 32 vulnerabilities identified:
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via wp-mail.php
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/713bdc8b-ab7c-46d7-9847-305344a579c4
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/abf236fdaf94455e7bc6e30980cf70401003e283
 |
 | [!] Title: WP < 6.0.3 - Open Redirect via wp_nonce_ays
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/926cd097-b36f-4d26-9c51-0dfab11c301b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/506eee125953deb658307bb3005417cb83f32095
 |
 | [!] Title: WP < 6.0.3 - Email Address Disclosure via wp-mail.php
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/c5675b59-4b1d-4f64-9876-068e05145431
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/5fcdee1b4d72f1150b7b762ef5fb39ab288c8d44
 |
 | [!] Title: WP < 6.0.3 - Reflected XSS via SQLi in Media Library
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/cfd8b50d-16aa-4319-9c2d-b227365c2156
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/8836d4682264e8030067e07f2f953a0f66cb76cc
 |
 | [!] Title: WP < 6.0.3 - CSRF in wp-trackback.php
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/b60a6557-ae78-465c-95bc-a78cf74a6dd0
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/a4f9ca17fae0b7d97ff807a3c234cf219810fae0
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via the Customizer
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/2787684c-aaef-4171-95b4-ee5048c74218
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/2ca28e49fc489a9bb3c9c9c0d8907a033fe056ef
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via Comment Editing
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/02d76d8e-9558-41a5-bdb6-3957dc31563b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/89c8f7919460c31c0f259453b4ffb63fde9fa955
 |
 | [!] Title: WP < 6.0.3 - Content from Multipart Emails Leaked
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/3f707e05-25f0-4566-88ed-d8d0aff3a872
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/3765886b4903b319764490d4ad5905bc5c310ef8
 |
 | [!] Title: WP < 6.0.3 - SQLi in WP_Date_Query
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/1da03338-557f-4cb6-9a65-3379df4cce47
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/d815d2e8b2a7c2be6694b49276ba3eee5166c21f
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via RSS Widget
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/58d131f5-f376-4679-b604-2b888de71c5b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/929cf3cb9580636f1ae3fe944b8faf8cca420492
 |
 | [!] Title: WP < 6.0.3 - Data Exposure via REST Terms/Tags Endpoint
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/b27a8711-a0c0-4996-bd6a-01734702913e
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/ebaac57a9ac0174485c65de3d32ea56de2330d8e
 |
 | [!] Title: WP < 6.0.3 - Multiple Stored XSS via Gutenberg
 |     Fixed in: 6.0.3
 |     References:
 |      - https://wpscan.com/vulnerability/f513c8f6-2e1c-45ae-8a58-36b6518e2aa9
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/gutenberg/pull/45045/files
 |
 | [!] Title: WP <= 6.2 - Unauthenticated Blind SSRF via DNS Rebinding
 |     References:
 |      - https://wpscan.com/vulnerability/c8814e6e-78b3-4f63-a1d3-6906a84c1f11
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-3590
 |      - https://blog.sonarsource.com/wordpress-core-unauthenticated-blind-ssrf/
 |
 | [!] Title: WP < 6.2.1 - Directory Traversal via Translation Files
 |     Fixed in: 6.0.4
 |     References:
 |      - https://wpscan.com/vulnerability/2999613a-b8c8-4ec0-9164-5dfe63adf6e6
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-2745
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.1 - Thumbnail Image Update via CSRF
 |     Fixed in: 6.0.4
 |     References:
 |      - https://wpscan.com/vulnerability/a03d744a-9839-4167-a356-3e7da0f1d532
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.1 - Contributor+ Stored XSS via Open Embed Auto Discovery
 |     Fixed in: 6.0.4
 |     References:
 |      - https://wpscan.com/vulnerability/3b574451-2852-4789-bc19-d5cc39948db5
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.2 - Shortcode Execution in User Generated Data
 |     Fixed in: 6.0.5
 |     References:
 |      - https://wpscan.com/vulnerability/ef289d46-ea83-4fa5-b003-0352c690fd89
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-2-security-release/
 |
 | [!] Title: WP < 6.2.1 - Contributor+ Content Injection
 |     Fixed in: 6.0.4
 |     References:
 |      - https://wpscan.com/vulnerability/1527ebdb-18bc-4f9d-9c20-8d729a628670
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP 5.6-6.3.1 - Contributor+ Stored XSS via Navigation Block
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/cd130bb3-8d04-4375-a89a-883af131ed3a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-38000
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP 5.6-6.3.1 - Reflected XSS via Application Password Requests
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/da1419cc-d821-42d6-b648-bdb3c70d91f2
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Denial of Service via Cache Poisoning
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/6d80e09d-34d5-4fda-81cb-e703d0e56e4f
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Subscriber+ Arbitrary Shortcode Execution
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/3615aea0-90aa-4f9a-9792-078a90af7f59
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Contributor+ Comment Disclosure
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/d35b2a3d-9b41-4b4f-8e87-1b8ccb370b9f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-39999
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Unauthenticated Post Author Email Disclosure
 |     Fixed in: 6.0.6
 |     References:
 |      - https://wpscan.com/vulnerability/19380917-4c27-4095-abf1-eba6f913b441
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-5561
 |      - https://wpscan.com/blog/email-leak-oracle-vulnerability-addressed-in-wordpress-6-3-2/
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.4.3 - Deserialization of Untrusted Data
 |     Fixed in: 6.0.7
 |     References:
 |      - https://wpscan.com/vulnerability/5e9804e5-bbd4-4836-a5f0-b4388cc39225
 |      - https://wordpress.org/news/2024/01/wordpress-6-4-3-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.4.3 - Admin+ PHP File Upload
 |     Fixed in: 6.0.7
 |     References:
 |      - https://wpscan.com/vulnerability/a8e12fbe-c70b-4078-9015-cf57a05bdd4a
 |      - https://wordpress.org/news/2024/01/wordpress-6-4-3-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.5.2 - Unauthenticated Stored XSS
 |     Fixed in: 6.0.8
 |     References:
 |      - https://wpscan.com/vulnerability/1a5c5df1-57ee-4190-a336-b0266962078f
 |      - https://wordpress.org/news/2024/04/wordpress-6-5-2-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in HTML API
 |     Fixed in: 6.0.9
 |     References:
 |      - https://wpscan.com/vulnerability/2c63f136-4c1f-4093-9a8c-5e51f19eae28
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in Template-Part Block
 |     Fixed in: 6.0.9
 |     References:
 |      - https://wpscan.com/vulnerability/7c448f6d-4531-4757-bff0-be9e3220bbbb
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Path Traversal in Template-Part Block
 |     Fixed in: 6.0.9
 |     References:
 |      - https://wpscan.com/vulnerability/36232787-754a-4234-83d6-6ded5e80251c
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WP < 6.8.3 - Author+ DOM Stored XSS
 |     Fixed in: 6.0.11
 |     References:
 |      - https://wpscan.com/vulnerability/c4616b57-770f-4c40-93f8-29571c80330a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58674
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-cross-site-scripting-xss-vulnerability
 |      -  https://wordpress.org/news/2025/09/wordpress-6-8-3-release/
 |
 | [!] Title: WP < 6.8.3 - Contributor+ Sensitive Data Disclosure
 |     Fixed in: 6.0.11
 |     References:
 |      - https://wpscan.com/vulnerability/1e2dad30-dd95-4142-903b-4d5c580eaad2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58246
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-sensitive-data-exposure-vulnerability
 |      - https://wordpress.org/news/2025/09/wordpress-6-8-3-release/

[+] WordPress theme in use: pencil
 | Location: http://192.168.104.112/wp-content/themes/pencil/
 | Last Updated: 2024-09-13T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/pencil/readme.txt
 | [!] The version is out of date, the latest version is 1.7.2
 | Style URL: http://192.168.104.112/wp-content/themes/pencil/style.css?ver=1.6.2
 | Style Name: Pencil
 | Style URI: https://blogonyourown.com/themes/pencil/
 | Description: Pencil is an amazing Simply & Clean WordPress Blog / Magazine Theme. It’s elegant and modern theme p...
 | Author: BlogOnYourOwn.com
 | Author URI: https://blogonyourown.com
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.6.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/pencil/style.css?ver=1.6.2, Match: 'Version: 1.6.2'


[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.104.112/wp-content/plugins/akismet/
 | Latest Version: 5.6
 | Last Updated: 2025-11-12T16:31:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.104.112/wp-content/plugins/akismet/, status: 500
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
 |     Fixed in: 3.1.5
 |     References:
 |      - https://wpscan.com/vulnerability/1a2f3094-5970-4251-9ed0-ec595a0cd26c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-9357
 |      - http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
 |      - https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
 |
 | The version could not be determined.


[i] Theme(s) Identified:

[+] pencil
 | Location: http://192.168.104.112/wp-content/themes/pencil/
 | Last Updated: 2024-09-13T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/pencil/readme.txt
 | [!] The version is out of date, the latest version is 1.7.2
 | Style URL: http://192.168.104.112/wp-content/themes/pencil/style.css
 | Style Name: Pencil
 | Style URI: https://blogonyourown.com/themes/pencil/
 | Description: Pencil is an amazing Simply & Clean WordPress Blog / Magazine Theme. It’s elegant and modern theme p...
 | Author: BlogOnYourOwn.com
 | Author URI: https://blogonyourown.com
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.6.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/pencil/style.css, Match: 'Version: 1.6.2'

[+] twentynineteen
 | Location: http://192.168.104.112/wp-content/themes/twentynineteen/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/twentynineteen/readme.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://192.168.104.112/wp-content/themes/twentynineteen/style.css
 | Style Name: Twenty Nineteen
 | Style URI: https://wordpress.org/themes/twentynineteen/
 | Description: Our 2019 default theme is designed to show off the power of the block editor. It features custom sty...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentynineteen/, status: 500
 |
 | Version: 2.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentynineteen/style.css, Match: 'Version: 2.1'

[+] twentytwenty
 | Location: http://192.168.104.112/wp-content/themes/twentytwenty/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 3.0
 | Style URL: http://192.168.104.112/wp-content/themes/twentytwenty/style.css
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwenty/, status: 500
 |
 | Version: 1.8 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwenty/style.css, Match: 'Version: 1.8'

[+] twentytwentyone
 | Location: http://192.168.104.112/wp-content/themes/twentytwentyone/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/twentytwentyone/readme.txt
 | [!] The version is out of date, the latest version is 2.7
 | Style URL: http://192.168.104.112/wp-content/themes/twentytwentyone/style.css
 | Style Name: Twenty Twenty-One
 | Style URI: https://wordpress.org/themes/twentytwentyone/
 | Description: Twenty Twenty-One is a blank canvas for your ideas and it makes the block editor your best brush. Wi...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwentyone/, status: 500
 |
 | Version: 1.4 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwentyone/style.css, Match: 'Version: 1.4'

[+] twentytwentytwo
 | Location: http://192.168.104.112/wp-content/themes/twentytwentytwo/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://192.168.104.112/wp-content/themes/twentytwentytwo/readme.txt
 | [!] The version is out of date, the latest version is 2.1
 | Style URL: http://192.168.104.112/wp-content/themes/twentytwentytwo/style.css
 | Style Name: Twenty Twenty-Two
 | Style URI: https://wordpress.org/themes/twentytwentytwo/
 | Description: Built on a solidly designed foundation, Twenty Twenty-Two embraces the idea that everyone deserves a...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwentytwo/, status: 200
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.104.112/wp-content/themes/twentytwentytwo/style.css, Match: 'Version: 1.2'

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 7
 | Requests Remaining: 18

[+] Finished: Sun Feb 22 20:17:31 2026
[+] Requests Done: 1960
[+] Cached Requests: 23
[+] Data Sent: 542.091 KB
[+] Data Received: 1.425 MB
[+] Memory used: 292.672 MB
[+] Elapsed time: 00:02:07

```


---


# WPScanユーザー列挙

```sh

```
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
admin
administrator
```
- emailのとき："Unknown email address. Check again or try your sername."
- ユーザー名のとき："The username lee is not registered on this site. If you are unsure of your username, try your email address instead."

---

# Screenshot


