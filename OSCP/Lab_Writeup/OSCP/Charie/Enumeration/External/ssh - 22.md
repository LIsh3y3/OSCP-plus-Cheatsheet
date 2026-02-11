# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d5:32:4b:71:2d:31:2b:4a:ba:10:16:0b:f0:bd:3d:8b (ECDSA)
|_  256 49:6b:01:37:10:e5:6f:49:37:4e:65:14:79:31:da:79 (ED25519)
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
└─$ hydra -C may_creds.txt ssh://192.168.115.112 -t1
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-10 22:29:33
[DATA] max 1 task per 1 server, overall 1 task, 28 login tries, ~28 tries per task
[DATA] attacking ssh://192.168.115.112:22/
[STATUS] 14.00 tries/min, 14 tries in 00:01h, 14 to do in 00:02h, 1 active
[STATUS] 13.50 tries/min, 27 tries in 00:02h, 1 to do in 00:01h, 1 active
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-10 22:31:40
```

- Jasonを含めてもう一度？- >失敗
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
Jason:Jason
Jason:jason
jason:jason
Vickens:Vickens
vickens:vickens
root:mysql
root:root
root:chippc
admin:admin
root:
root:nagiosxi
root:usbw
cloudera:cloudera
root:cloudera
root:moves
moves:moves
root:testpw
root:p@ck3tf3nc3
mcUser:medocheck123
root:mktt
root:123
dbuser:123
asteriskuser:amp109
asteriskuser:eLaStIx.asteriskuser.2oo7
root:raspberry
root:openauditrootuserpassword
root:vagrant
root:123qweASD#
```

- jane.perkins:secretではログインできず


---

# Screenshot


