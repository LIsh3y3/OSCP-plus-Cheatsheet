# Host Information

```zsh

```

---

# Scans

## Nmap

### quick scan

```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
### full scan

```
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2994/tcp  open  veritas-vis2?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
24621/tcp open  ftp           FileZilla ftpd 1.5.1
| ssl-cert: Subject: commonName=filezilla-server self signed certificate
| Not valid before: 2022-11-29T21:31:27
|_Not valid after:  2023-11-30T21:36:27
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla.
|_ssl-date: TLS randomness does not represent time
24680/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: &#x30DB;&#x30FC;&#x30E0; - Umbraco&#x30B5;&#x30F3;&#x30D7;&#x3...
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
...
```

### udp scan

```
PORT      STATE         SERVICE       VERSION
135/tcp   open          msrpc         Microsoft Windows RPC
139/tcp   open          netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open          microsoft-ds?
111/udp   open|filtered rpcbind
...
```

## Rustscan

```
PORT      STATE SERVICE       REASON          VERSION
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 125
5985/tcp  open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
24621/tcp open  ftp           syn-ack ttl 125 FileZilla ftpd 1.5.1
| ssl-cert: Subject: commonName=filezilla-server self signed certificate
| Issuer: commonName=filezilla-server self signed certificate
| Public Key type: ec
| Public Key bits: 256
| Signature Algorithm: ecdsa-with-SHA256
| Not valid before: 2022-11-29T21:31:27
| Not valid after:  2023-11-30T21:36:27
| MD5:   b189:7fe7:445a:8dba:2e6f:fb88:9590:36e5
| SHA-1: d0c2:c6f9:8aa0:ea26:e950:9004:c1ce:269c:0aa9:4515
| -----BEGIN CERTIFICATE-----
| MIIBiTCCAS6gAwIBAgIUUd8G9i3xWy+Bi6Xenag6OrlUp3QwCgYIKoZIzj0EAwIw
| MzExMC8GA1UEAxMoZmlsZXppbGxhLXNlcnZlciBzZWxmIHNpZ25lZCBjZXJ0aWZp
| Y2F0ZTAeFw0yMjExMjkyMTMxMjdaFw0yMzExMzAyMTM2MjdaMDMxMTAvBgNVBAMT
| KGZpbGV6aWxsYS1zZXJ2ZXIgc2VsZiBzaWduZWQgY2VydGlmaWNhdGUwWTATBgcq
| hkjOPQIBBggqhkjOPQMBBwNCAAS9ZT1Nrzd4ylSsDYSugXgJX4ma1L38//bjzoiz
| RbsCeclhMYwfjBHiEK2ZXPiQQ9+OSZoyaGpeZyGBeluKmtSioyAwHjAOBgNVHQ8B
| Af8EBAMCBaAwDAYDVR0TAQH/BAIwADAKBggqhkjOPQQDAgNJADBGAiEA3TURgZOO
| ajm1Aadtr7ehlR9r6PQbP6GJbud5QPSY3qYCIQC1bA5LakNNP/54TPD84rRmAErm
| +n7+Af5x3IFfeKYmZA==
|_-----END CERTIFICATE-----
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla.
|_ssl-date: TLS randomness does not represent time
24680/tcp open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
|_http-favicon: Unknown favicon MD5: 492B0BD5C68472C87C73D847FBBA3106
|_http-title: &#x30DB;&#x30FC;&#x30E0; - Umbraco&#x30B5;&#x30F3;&#x30D7;&#x3...
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/10.0
47001/tcp open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
...
```