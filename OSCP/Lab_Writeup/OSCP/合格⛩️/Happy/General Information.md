# Host Information

```zsh
192.168.104.111
```
```
host: Windows
```

---

# Scans

## Nmap

### quick scan

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-23-22  07:13AM                  145 .env
| 02-23-22  07:13AM                 2056 Acq.dll
| 02-24-22  05:24AM                 4868 DVRParams.ini
| 02-23-22  07:13AM                35996 Manifest.dll
| 02-23-22  07:13AM                20455 program.exe
| 02-23-22  07:15AM                40229 verisign.png
|_02-23-22  07:14AM                11446 wab.dll
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp   open  http          Samsung AllShare httpd
|_http-title: Did not follow redirect to https://192.168.104.111/cbs/Logon.do
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache Tomcat 8.5.34
|_ssl-date: TLS randomness does not represent time
| http-title: Site doesn't have a title (text/html;charset=utf-8).
|_Requested resource was /cbs/Logon.do
| ssl-cert: Subject: commonName=Not Secure/organizationName=Ahsay System Corporation Limited/stateOrProvinceName=Hong Kong SAR/countryName=CN
| Not valid before: 2017-03-21T20:52:17
|_Not valid after:  2020-03-20T20:52:17
445/tcp  open  microsoft-ds?
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http-proxy
|_http-generator: Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]
|_http-title: Argus Surveillance DVR
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Connection: Keep-Alive
|     Keep-Alive: timeout=15, max=4
|     Content-Type: text/html
|     Content-Length: 985
|     <HTML>
|     <HEAD>
|     <TITLE>
|     Argus Surveillance DVR
|     </TITLE>
|     <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
|     <meta name="GENERATOR" content="Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]">
|     <frameset frameborder="no" border="0" rows="75,*,88">
|     <frame name="Top" frameborder="0" scrolling="auto" noresize src="CamerasTopFrame.html" marginwidth="0" marginheight="0"> 
|     <frame name="ActiveXFrame" frameborder="0" scrolling="auto" noresize src="ActiveXIFrame.html" marginwidth="0" marginheight="0">
|     <frame name="CamerasTable" frameborder="0" scrolling="auto" noresize src="CamerasBottomFrame.html" marginwidth="0" marginheight="0"> 
|     <noframes>
|     <p>This page uses frames, but your browser doesn't support them.</p>
|_    </noframes>
8443/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-generator: Nicepage 4.5.4, nicepage.com
|_http-title: Home
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
```
### full scan

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-23-22  08:13AM                  145 .env
| 02-23-22  08:13AM                 2056 Acq.dll
| 02-24-22  06:24AM                 4868 DVRParams.ini
| 02-23-22  08:13AM                35996 Manifest.dll
| 02-23-22  08:13AM                20455 program.exe
| 02-23-22  08:15AM                40229 verisign.png
|_02-23-22  08:14AM                11446 wab.dll
80/tcp    open  http          Samsung AllShare httpd
|_http-title: Did not follow redirect to https://192.168.104.111/cbs/Logon.do
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache Tomcat 8.5.34
|_ssl-date: TLS randomness does not represent time
| http-title: Site doesn't have a title (text/plain;charset=ISO-8859-1).
|_Requested resource was /cbs/Logon.do
| ssl-cert: Subject: commonName=Not Secure/organizationName=Ahsay System Corporation Limited/stateOrProvinceName=Hong Kong SAR/countryName=CN
| Not valid before: 2017-03-21T20:52:17
|_Not valid after:  2020-03-20T20:52:17
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7680/tcp  open  pando-pub?
8080/tcp  open  http-proxy
|_http-generator: Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]
|_http-title: Argus Surveillance DVR
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Connection: Keep-Alive
|     Keep-Alive: timeout=15, max=4
|     Content-Type: text/html
|     Content-Length: 985
|     <HTML>
|     <HEAD>
|     <TITLE>
|     Argus Surveillance DVR
|     </TITLE>
|     <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
|     <meta name="GENERATOR" content="Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]">
|     <frameset frameborder="no" border="0" rows="75,*,88">
|     <frame name="Top" frameborder="0" scrolling="auto" noresize src="CamerasTopFrame.html" marginwidth="0" marginheight="0"> 
|     <frame name="ActiveXFrame" frameborder="0" scrolling="auto" noresize src="ActiveXIFrame.html" marginwidth="0" marginheight="0">
|     <frame name="CamerasTable" frameborder="0" scrolling="auto" noresize src="CamerasBottomFrame.html" marginwidth="0" marginheight="0"> 
|     <noframes>
|     <p>This page uses frames, but your browser doesn't support them.</p>
|_    </noframes>
8443/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-generator: Nicepage 4.5.4, nicepage.com
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
```

### udp scan

```
PORT      STATE         SERVICE       VERSION
21/tcp    open          ftp           Microsoft ftpd
80/tcp    open          http          Samsung AllShare httpd
135/tcp   open          msrpc         Microsoft Windows RPC
139/tcp   open          netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open          ssl/http      Apache Tomcat 8.5.34
445/tcp   open          microsoft-ds?
5357/tcp  open          http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8080/tcp  open          http-proxy
8443/tcp  open          http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
123/udp   open|filtered ntp
137/udp   open|filtered netbios-ns
138/udp   open|filtered netbios-dgm
500/udp   open|filtered isakmp
999/udp   open|filtered applix
1027/udp  open|filtered unknown
1701/udp  open|filtered L2TP
1900/udp  open|filtered upnp
2222/udp  open|filtered msantipiracy
3456/udp  open|filtered IISrpc-or-vat
3703/udp  open|filtered adobeserver-3
4500/udp  open|filtered nat-t-ike
5353/udp  open|filtered zeroconf
10000/udp open|filtered ndmp
```

## Rustscan

```
PORT      STATE SERVICE REASON          VERSION
21/tcp    open  ftp     syn-ack ttl 127 Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-23-22  08:13AM                  145 .env
| 02-23-22  08:13AM                 2056 Acq.dll
| 02-24-22  06:24AM                 4868 DVRParams.ini
| 02-23-22  08:13AM                35996 Manifest.dll
| 02-23-22  08:13AM                20455 program.exe
| 02-23-22  08:15AM                40229 verisign.png
|_02-23-22  08:14AM                11446 wab.dll
80/tcp    open  http    syn-ack ttl 127 Samsung AllShare httpd
|_http-title: Did not follow redirect to https://192.168.104.111/cbs/Logon.do
| http-methods: 
|_  Supported Methods: GET HEAD POST
5357/tcp  open  http    syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
49664/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
...
```


![[Pasted image 20260222111945.png]]![[Pasted image 20260222112017.png]]