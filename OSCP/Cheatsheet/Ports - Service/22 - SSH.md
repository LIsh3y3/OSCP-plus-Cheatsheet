- 関連ノート：
	- [SSH private keyの抽出](../Web%20App/Path%20traversal.md#SSH%20private%20keyの抽出)
	- [🐉THC-Hydra](../../Tools/🐉THC-Hydra.md)

---

# ssh-audit

🔗[ssh-audit - GitHub](https://github.com/jtesta/ssh-audit)

- ssh-auditとは
	- SSH接続を分析するツールであり、バナー、OS/ソフトウェアの識別、圧縮検出、アルゴリズム情報、セキュリティ推奨事項に関する詳細情報を取得する
	- 凡例：赤🟥：危険(脆弱)、黄色🟨：注意、緑🟩：安全
```zsh
ssh-audit <target_IP> > ssh_audit_result.txt
```
	↓出力例抜粋
```zsh
# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2020.79+
(gen) compression: enabled (zlib@openssh.com)
```

## 補足：ssh-auditの結果活用法

- `(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5` → OSのバージョンが推測できる
	- OSでは標準のSSHバージョンが決まっている
	- 🔗[Ubuntu - DistroWatch.com](https://distrowatch.com/table.php?distribution=ubuntu)などでsshのバージョンを検索すると、どのOSバージョンかがわかる

![](../../Images/Pasted%20image%2020251227113634.png)

$$UbuntuバージョンごとのSSHバージョン$$

- OpenSSH **4.x 系以前**であれば脆弱な可能性があるため、CVEを検索する

---

# 基本情報

- **脆弱でないことが多いのでエクスプロイトの優先度は低い**（ソースはPEN-200のModule13: Locating Public Exploits）
- ハイブリット方式の暗号化、もしくはパスワード方式で認証する
- known_hosts (`~/.ssh/known_hosts`)とは、SSHクライアント側に保存されるデータで、過去に接続したSSHサーバーのホストキー(公開鍵)を保存している

## 公開鍵認証

![ 350](../../Images/Pasted%20image%2020250330100126.png)

$$公開鍵認証フロー$$

| 項目                                | 意味                      | 保存場所                          |
| --------------------------------- | ----------------------- | ----------------------------- |
| 秘密鍵 (id_rsa / id_ed25519)         | SSHクライアントが保持。外に出してはいけない | クライアント（`~/.ssh/id_rsa`）       |
| 公開鍵 (id_rsa.pub / id_ed25519.pub) | サーバがこれを使って署名検証          |                               |
| authorized_keys                   | 正規のSSHクライアントの公開鍵一覧      | サーバ(`~/.ssh/authorized_keys`) |
| パスフレーズ                            | 秘密鍵ファイルをローカルで暗号化するもの    |                               |

## sshd_config

- コメントアウトされている設定値は「無効」ではなく、OpenSSH のデフォルト値が適用される
- そのため、コメントアウト＝安全／無効と誤解しないこと

| 設定値                    | 意味・確認ポイント                                |
| ---------------------- | ---------------------------------------- |
| AllowUsers             | SSH ログインを許可するユーザーを明示的に制限。指定されたユーザー以外は全拒否 |
| AllowGroups            | SSH ログインを許可するグループを指定                     |
| DenyUsers              | 特定ユーザーを明示的に拒否                            |
| DenyGroups             | 特定グループを明示的に拒否                            |
| PermitRootLogin        | root ユーザーの SSH ログイン可否。yes の場合、侵入後即フル権限   |
| PasswordAuthentication | パスワード認証が有効かどうか。yes の場合、Hydra 等が成立        |
| PermitEmptyPasswords   | 空パスワードの許可可否。yes なら致命的                    |
| PubkeyAuthentication   | 公開鍵認証の有効可否。コメントアウトでもデフォルトは yes           |
| MaxAuthTries           | 認証試行回数の上限。値が大きいとブルートフォース耐性が低下            |
| X11Forwarding          | X11 転送の可否。ローカル情報漏洩リスク                    |
| Match                  | 条件付き設定。グローバル設定より優先されるため見落とし厳禁            |
| LogLevel               | SSH のログ詳細度。侵入検知や痕跡調査に影響                  |
| SyslogFacility         | SSH ログの出力先                               |

---

# 基本操作

## SSH Private keyを使ったアクセス

- Keyの見た目
```zsh
-----BEGIN OPENSSH PRIVATE KEY-----
xxxx
-----END OPENSSH PRIVATE KEY-----

```

>[!WARNING]
>一番最後の行には`\n`（改行）が必須

- Keyの権限は、ユーザー/オーナーだけがREAD可能と設定しないと使えない
```zsh
chmod 400 <private_key>
```

- SSHアクセス
```zsh
ssh -i <private_key> -p <port> <username>@<target_IP|Domain>
```

>[!NOTE]
>- sshd.configで、`PubkeyAuthentication no`となっていると、パスワードの入力を求められる


過去に接続したことのあるSSHサーバーのホストキーを上書きすると、以下のエラーが表示される
```zsh
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
...
```
- →対策：`known_hosts`を削除する
```zsh
rm ~/.ssh/known/hosts
```

### SSH Private Keyの種類

ホームディレクトリ配下の`/.ssh/`ディレクトリにある

| 種類      | ファイル名（デフォルト）  | 暗号方式                 | 備考                           |
| ------- | ------------- | -------------------- | ---------------------------- |
| RSA     | `id_rsa`      | RSA                  | 伝統的。今も広く使われるが ED25519 推奨に移行中 |
| ECDSA   | `id_ecdsa`    | ECDSA（P-256/384/521） | 楕円曲線暗号。互換性高い                 |
| ED25519 | `id_ed25519`  | Ed25519              | いま最も推奨されるキー。高速かつセキュア         |
| DSA     | `id_dsa`      | DSA                  | 非推奨／OpenSSH ではデフォルト無効        |
| RSA1    | `id_identity` | SSH1 用               | SSH1 自体が廃止。使用禁止              |

#### 補足：Private Keyのリスト

`/etc/passwd`からシェル環境のあるユーザーを抽出し、Intruderでcluster bomb攻撃で使うために使う
```
id_rsa
id_ecdsa
id_ed25519
id_dsa
id_identity
```

![](../../Images/Pasted%20image%2020251202065433.png)

$$IntruderでホームディレクトリとPrivate Keyを変数に抽出を試みる$$

---

# SSH Exploit

## パストラバーサル x 秘密鍵

- 初期侵入できていない段階で、パストラバーサルの脆弱性を用いてクライアントから秘密鍵を抽出し、その秘密鍵でSSH接続する：
	- [SSH private keyの抽出](../Web%20App/Path%20traversal.md#SSH%20private%20keyの抽出)
	- [SSH Private Keyの種類](#SSH%20Private%20Keyの種類)

- パスフレーズがある場合：ssh2john([ハッシュ値の整形](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ハッシュ値の整形))

## 公開鍵一覧の書き換え

- `authorized_keys` を書き換えて、攻撃者が持つ秘密鍵 (`id_rsa`等) でSSH接続できるようにする

1. 自分の秘密鍵と公開鍵を生成
```zsh
# -tでアルゴリズム、-fでアウトプットファイル指定可能
ssh-keygen
```
- ↓出力
```zsh
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): <ファイル名>
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in <secretkey> #秘密鍵
Your public key has been saved in <publickey>.pub #公開鍵
...
```

2. 自信の公開鍵をターゲットのauthorized_keysに書き込み
```zsh
cat <ファイル名>.pub > authorized_keys
```

3. sshアクセス
```zsh
ssh -i <private_key> <target_user>@<target_IP>
```

## パスワード辞書攻撃

- 最終手段
- パスワード認証が可能なときは、ログイン失敗メッセージに、`...Permission denied (password,...).`とある
```zsh
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt <target_IP> ssh -t 1
```
- 参考ノート：[🐉THC-Hydra](../../Tools/🐉THC-Hydra.md#サービスに対する辞書攻撃)

---

## SSHをSOCKSプロキシ経由で動かす

### 用途

- [HTTP Tunneling w/ Chisel](../Port%20Redirection/HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md#HTTP%20Tunneling%20w/%20Chisel)等の方法(ポートフォワーディング)で確立した*既存の*SOCKSプロキシ経由でSSH接続したいとき
	- [SSH Local Dynamic Port Forwarding](../Port%20Redirection/Port%20Redirection%20&%20SSH%20Port%20Forwarding.md#SSH%20Local%20Dynamic%20Port%20Forwarding)の`-D`のように、SSHがSOCKSプロキシを*新規で*用意して接続はできるが、既存のSOCKSプロキシ経由でのアクセスは不可
- 代わりに `ProxyCommand` オプションを使い、外部コマンド(`ncat`)でプロキシ経由の通信を実現する

### SSHを既存SOCKSプロキシ経由で動かす手順

1. Kaliに`ncat`をインストールする
```zsh
sudo apt install ncat
```

2. SOCKSプロキシが動作していることを確認する
```zsh
ss -tulpn | grep <port>
```
- portはポートフォワーディングの文脈では、SOCKSプロキシを用意したリッスンポートを指定する
	- `chisel`の場合、1080を指定

3. SSH の `ProxyCommand` オプションで ncat を指定して接続する
	- `ncat`がSOCKSプロキシに接続し、そのトンネルをSSHに渡す
```zsh
ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:[port] %h %p' [ssh接続先username]@[IP]
```
- `%h` ：接続先ホスト名が代入されるプレースホルダー
- `%p` ：接続先ポート番号が代入されるプレースホルダー

---

# Misc

- sshpass は、「スクリプトでSSHのパスワード入力を自動化するためのツール」であり、パスワードを平文で記録して使うことが多い（ssh秘密鍵での認証のほうが安全）

## トラブルシューティング

- sshログインが成功してもハングするとき
	- →`~/.bashrc`もしくは` ~/.profile`が問題であるため、読み込ませない
```zsh
# 成功してもプロンプトは表示されないが、コマンドは実行できる
ssh hoge@192.168.219.245 "bash --noprofile --norc"

# ログイン後、プロンプトを表示できるようにする
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

- エラー：`Permission denied (publickey)` 
	- →パスワード認証を許可しておらず、公開鍵認証のみを受け付けている

- エラー：`Permission denied (publickey,keyboard-interactive).`
	- →パスワード認証もしくは公開鍵認証を受け付けている

- エラー：`Unable to negotiate with xxx port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss`
	- →接続先が脆弱な古い暗号化アルゴリズムを要求しているので、合わせる
```zsh
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa <username>@<target_IP>
```