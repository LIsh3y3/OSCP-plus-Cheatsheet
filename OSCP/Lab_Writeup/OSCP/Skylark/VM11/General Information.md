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
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINPREP
|   NetBIOS_Domain_Name: WINPREP
|   NetBIOS_Computer_Name: WINPREP
|   DNS_Domain_Name: WINPREP
|   DNS_Computer_Name: WINPREP
|   Product_Version: 10.0.22000
|_  System_Time: 2026-02-14T11:53:26+00:00
|_ssl-date: 2026-02-14T11:53:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=WINPREP
| Not valid before: 2025-10-10T00:21:00
|_Not valid after:  2026-04-11T00:21:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
### full scan

```
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINPREP
|   NetBIOS_Domain_Name: WINPREP
|   NetBIOS_Computer_Name: WINPREP
|   DNS_Domain_Name: WINPREP
|   DNS_Computer_Name: WINPREP
|   Product_Version: 10.0.22000
|_  System_Time: 2026-02-14T12:08:09+00:00
| ssl-cert: Subject: commonName=WINPREP
| Not valid before: 2025-10-10T00:21:00
|_Not valid after:  2026-04-11T00:21:00
|_ssl-date: 2026-02-14T12:08:23+00:00; -1s from scanner time.
5040/tcp  open  unknown
49664/tcp open  msrpc         Microsoft Windows RPC
...
```

### udp scan

```
PORT      STATE         SERVICE        VERSION
135/tcp   open          msrpc          Microsoft Windows RPC
139/tcp   open          netbios-ssn    Microsoft Windows netbios-ssn
445/tcp   open          microsoft-ds?
3389/tcp  open          ms-wbt-server  Microsoft Terminal Services
123/udp   open|filtered ntp
...
```

## Rustscan

```
PORT      STATE SERVICE       REASON          VERSION
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 125
3389/tcp  open  ms-wbt-server syn-ack ttl 125 Microsoft Terminal Services
| ssl-cert: Subject: commonName=WINPREP
| Issuer: commonName=WINPREP
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-10T00:21:00
| Not valid after:  2026-04-11T00:21:00
| MD5:   8211:24da:2e9f:fb0a:d36c:9783:d23d:64aa
| SHA-1: 6a29:c062:9040:a830:ca7f:a13d:1581:7802:794f:72f3
| -----BEGIN CERTIFICATE-----
| MIIC0jCCAbqgAwIBAgIQPGhpHDDXVLtLkjieuaE82jANBgkqhkiG9w0BAQsFADAS
| MRAwDgYDVQQDEwdXSU5QUkVQMB4XDTI1MTAxMDAwMjEwMFoXDTI2MDQxMTAwMjEw
| MFowEjEQMA4GA1UEAxMHV0lOUFJFUDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
| AQoCggEBALXfjtyRfs7mh2dBeWeW1e90EuEEPb0/ZUIX0eKI6XGGufnlsuacsBLV
| 1UmiAO38xAf9UlXeNWiBDB0Ykss+ePnUv/obnsSOPIWOO7XjZ3fzU4vuXspv9ggB
| Sx/c+rgQdnwVRtMMxPbySvBzCNgxovti5FKn3847mseLOdUTKjrvjeQpGbk82v6F
| hAbmbzZHbhysEHhyOVlUucdB9QtpHri/Fj8TiR/fZcA6vjIj2Y4BTaWVoC9I0HK/
| enry4aFCLugPbFLtfsVxjOS/ALq3OReEdOjmYDHDlspgb0shIt48QSABMh1H778f
| FgDZDFG8WSd/6qrnzFmT+n2CwUgVR0ECAwEAAaMkMCIwEwYDVR0lBAwwCgYIKwYB
| BQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBCwUAA4IBAQAlTjR4zS5MnZMa
| lHrg5Pi+eXlJfrpkS0fXeIkzpDhar2Tw31k9nNWe+U8AlduIq0I5/gQbVjsgGAuQ
| 5hLqpWZt4dMjbZL4K642jzrEAdMgHoEzb52rOPG2PPlm+PHAWOh+tB02Gr99jdyp
| h2MNRGvc18LVt4fU3bG4KvtwnSVffWEs+rHlRG3BMTjRgSHLynXqShBZq26CHusy
| Iv8QwP4idKkvHRPwSoUgVu94EpiNxB8QTxfd+vEtR8WSMoyo5exUgfULKtaYpQ+B
| +uJVk83fuiqWbsPRUqZJzPhJBkwp8zxC/oAUfNG4b5/8HV3lomA1uTAVJ8m465NY
| aHkxAWzB
|_-----END CERTIFICATE-----
|_ssl-date: 2026-02-14T12:19:12+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: WINPREP
|   NetBIOS_Domain_Name: WINPREP
|   NetBIOS_Computer_Name: WINPREP
|   DNS_Domain_Name: WINPREP
|   DNS_Computer_Name: WINPREP
|   Product_Version: 10.0.22000
|_  System_Time: 2026-02-14T12:18:57+00:00
5040/tcp  open  unknown       syn-ack ttl 125
```