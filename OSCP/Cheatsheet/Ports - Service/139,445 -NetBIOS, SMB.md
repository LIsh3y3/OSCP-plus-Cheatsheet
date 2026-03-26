# 目的・シチュエーション

目的：
- SMB共有で興味深いファイルや書き込み可能な場所を見つける
- ユーザー、グループ、サーバー情報を収集・列挙する

シチュエーション：
- ターゲットはWindowsもしくはSamba利用のLinux
- Port: 139（NetBIOS）または445（SMB）がopen

> [!WARNING]
> ツールごとに使用するプロトコルなどが変わるため、結果に差異が出る可能性がある（NetExecでは失敗したがsmbclientなら成功、など）。

---
---

# 基本情報

## SMBとは

- SMB (Server Message Block) はファイル／プリンタ共有、RPC、ユーザー認証などを担うアプリケーション層プロトコル
- Windows 系 OS で標準実装であり、Linux では *Samba* で実装され、クロスプラットフォーム通信が可能
- クライアント–サーバーモデルで、最新 OS と旧 OS 間でも後方互換性あり
- 共有 (Share) はローカル FS の任意ディレクトリを公開可能
- ACL で Read／Write／Execute など細かい制御が可能

---

## NetBIOSとSMBの関係

| サービス名   | ポート       | 役割・特徴                                                  |
| ------- | --------- | ------------------------------------------------------ |
| NetBIOS | 139 (TCP) | ・Windowsの古い名前解決・ファイル共有（NBT over TCP/IP）<br>・レガシーSMBで使用 |
| SMB     | 445 (TCP) | Windowsのファイル/プリンタ共有（SMB over TCP/IP）                   |

- 歴史的にはNetBIOS上でSMBが動作した
- 現代は445単独で通信可能（DNSで名前解決）
- 古い環境では139＋445併用、139開放は脆弱性リスク高
- ==139, 445を両方調査するのがテストの基本==

---

## 代表的な共有名

- `$`（ドルマーク）：隠し共有で、アクセスには原則管理者権限が必要

| 共有名        | 用途・役割                                         | 備考                                                    |
| ---------- | --------------------------------------------- | ----------------------------------------------------- |
| `C$`       | ローカルドライブ (`C:\`)に対応する管理共有                     |                                                       |
| `ADMIN$`   | Windows システムディレクトリ（通常 `C:\Windows`）に対応する管理用共有 | 管理ツールがリモートで操作するために使用。                                 |
| `IPC$`     | 名前付きパイプ（Inter-Process Communication）共有        | 認証や RPC、サービス通信の基盤となる特殊共有。<br>OS情報・ローカルユーザー・グループ列挙に利用。 |
| `Print$`   | プリンタドライバなどの共有リソース を格納                         | ネットワークプリンタ利用時に使用されることが多い。                             |
| `SYSVOL`   | ドメインコントローラがポリシーやスクリプトを配布するための共有               | ドメイン参加クライアントがログオン時にアクセス。                              |
| `NETLOGON` | ログオンスクリプトや認証関連ファイルを保存する共有                     | 認証時に使用。ドメイン参加端末とDC間で重要な役割。                            |

---

## Toolについて

### SMBclientの結果の読み方

- ①：ファイル名 or ディレクトリ名
- ②：属性名（※）
- ③：サイズ
- ④：最終更新日時

※属性には以下の種類がある

| 文字  | 属性名           | 意味                                        |
| --- | ------------- | ----------------------------------------- |
| D   | Directory     | ディレクトリ（フォルダ）<br>サイズは必ず0となるが、中身が空という意味ではない |
| A   | Archive       | アーカイブ属性（バックアップの対象）                        |
| H   | Hidden        | 隠しファイル（エクスプローラ等で非表示）                      |
| S   | System        | システムファイル                                  |
| R   | Read-only     | 読み取り専用                                    |
| n   | Normal        | 通常のファイル（他の属性が無効）                          |
| r   | Reparse point | シンボリックリンクやマウントポイント等                       |
| T   | Temporary     | 一時ファイル                                    |

![](../../Images/Pasted%20image%2020250525181048.png)

$$smbclientで接続後ls実施した画面$$

### NetExecの結果の読み方

| プレフィックス  | 意味                        |
| -------- | ------------------------- |
| `[*]`（青） | 情報メッセージ（情報提供用でエラーや成功ではない） |
| `[-]`（赤） | 失敗・エラー（認証失敗、接続エラーなど）      |
| `[+]`（緑） | 成功（認証成功、特権取得成功など）         |

---

## 用語

- RID: Relative Identifier（[SID・RID](../../Misc/用語.md#SID・RID)）
- NULLセッション: 認証なし接続
- UNCパス（`\\example\ADMIN$\`）：ネットワーク上のリソースを指定するWindowsの標準形式

---
---

# Enumeration

- 使用ツールドキュメント🔗：
	- [NetExec](https://www.netexec.wiki/?q=)
	- [smbclient](https://www.samba.org/samba/samba/docs/man/manpages/smbclient.1.html)
	- [SMBMap](https://github.com/ShawnDEvans/smbmap)
	- [enum4linux](https://github.com/CiscoCXSecurity/enum4linux)
	- [Impacket](https://github.com/fortra/impacket)

## SMB スキャン & 基本列挙

### Nmap

基本スキャン
```zsh
nmap -p 139,445 <target_IP>
```

OS, コンピュータ名,  domain, workgroupを見当づける
```zsh
nmap -v -p 139,445 --script "smb-os-discovery" <target_IP>
```

脆弱性の列挙
```zsh
nmap --script "smb-vuln*" -p 139,445 <target_IP>
```

### NetExec

SMB経由でOS, コンピュータ名,  domain, workgroupを見当つける
```zsh
netexec smb <target_IP>
```

### nbtscan

NetBIOSの名前確認
```zsh
sudo nbtscan -r <target_IP>
```
- →ホストの"役割"を推測

### nc

バナーグラブ
```zsh
nc -nv <target_IP> 445
```

### 総合列挙

- enum4linux よりもSMBMapの方がメンテナンス状況や機能面で推奨される（🔗[HTB "Active" - ippsec - Youtube](https://www.youtube.com/watch?v=jUc1J31DNdw)）
- ターゲット環境によってはenum4linuxでのみ有用な結果が得られることもある

smbmap
```zsh
# 基本列挙（Null Session / ゲスト確認）
smbmap -H <target_IP>
```

enum4linux
```zsh
# 基本列挙
enum4linux <target_IP>

# 詳細列挙
enum4linux -aMld <target_IP> | tee enum4linux.log

# Guestユーザーとして列挙
enum4linux -u guest -aMld <target_IP> | tee enum4linux.log
```

#### enum4linuxの補足

- `enum4linux -w <WORKGROUP> ...` でワークグループ指定が必要な場合有り（ワークグループ名は`smbmap -v -H <target_IP>` で取得可能）
- `Known Usernames`：一般的なユーザー名というだけで、ターゲットに存在するとは限らない

---

## 共有の詳細列挙 & アクセス

### NetExec

Nullセッション
```zsh
netexec smb <target_IP> --shares
```

Guestもしくはanonymous認証
```zsh
netexec smb <target_IP> -u guest -p '' --shares
netexec smb <target_IP> -u anonymous -p '' --shares
```

認証あり
```zsh
netexec smb <target_IP> -u <username> -p '<password>' --shares
```

読み書き可能なSMB共有
```zsh
netexec smb <target_IP> -u <username> -p '<password>' --shares --filter-shares READ WRITE
```

### smbmap

Nullセッション
```zsh
smbmap -H <target_IP>
```

Guestもしくはanonymous認証
```zsh
smbmap -u guest -p '' -H <target_IP> -r
smbmap -u anonymous -p '' -H <target_IP> -r
```

認証あり
```zsh
# ドメインの指定: -d <domain>
smbmap -u <username> -p "<password>" -H <target_IP>
```

PtH
```zsh
smbmap -u <username> -p "<LMHash>:<NTHash>" -H <target_IP>
```

再帰探索
```zsh
smbmap -H <target_IP> -r <share>
```

よく使う追加オプション

| コマンド                                               | 内容                  |
| -------------------------------------------------- | ------------------- |
| `smbmap -H <target_IP> -L`                           | ドライブ（ボリューム）の一覧を表示   |
| `smbmap -H <target_IP> -d $DOMAIN -u $USER -p $PASS` | ドメイン指定でのログイン試行      |
| `smbmap -H <target_IP> --download "path\to\file"`    | 指定したファイルを直接ダウンロード   |
| `smbmap -H <target_IP> -x "whoami"`                  | OSコマンド実行（管理権限がある場合） |

### smbclient

Nullセッション
```zsh
smbclient --no-pass -L //<target_IP>
```

認証あり
```zsh
smbclient -L //<target_IP> -W <domain> -U '<username>'
```

PtH
```zsh
smbclient //<target_IP> -U '<username>' --pw-nt-hash <NTHash>
```

### Windowsマシン上から

ローカル共有
```powershell
net share
```

リモート共有
```powershell
# 具体例:net view \\dc01 /all
net view \\<target_host> /all
```

### Impacket

Kerberos認証を使ったファイルの列挙
```zsh
impacket-smbclient <domain>/<username>:<password>@<target_IP|host> -k -no-pass
```

---

## SMB 共有との対話 & 取得

### smbclient インタラクティブ

```zsh
# 接続
smbclient -W <domain> -U '<username>' \\\\<target_IP>\\<share>

# ファイル名のフィルターを解除（すべて対象に）
smb:> mask ""

# ディレクトリを再帰的に処理
smb:> recurse on

# ファイルごとの確認を無効にする
smb:> prompt off

# すべてのファイルを取得（TIMEOUTとなる場合は使わない）
smb:> mget *
```

### mount

> [!TIP]
> smbclientよりもファイルの列挙が簡単にできる。

基本コマンド
```zsh
mkdir <mount_dir>
sudo mount -t cifs //<target_IP>/<share_name> <mount_dir>/ -o username=<username>,password=<password>
cd <mount_dir>
# マウント解除：sudo umount -l <mount_dir>
```

具体例
```zsh
mkdir mount_point
sudo mount -t cifs //192.168.1.1/share mount_point/ -o guest
cd mount_point
```

### NetExec

- すべてのデータを収集
	- 出力はjsonファイルに保存されるが、ファイルがない場合は何も保存されずSMB shareの列挙に留まる
```zsh
netexec smb <target_IP> -u <username> -p '<password>' -M spider_plus
```

---

## ユーザーの列挙

- enum4linuxでも列挙可能

ユーザー一覧の取得
```zsh
netexec smb <target_IP> -u <username> -p '<password>' --users
```

ユーザーのブルートフォース（[RID（Relative Identifier）](../../Misc/用語.md#RID（Relative%20Identifier）)）
```zsh
netexec smb <target_IP> -u guest -p '' --rid-brute
```

入手済み認証情報の有効性確認
```zsh
netexec smb <target_IP> -u <username wordlist> -p <password wordlist> --continue-on-success
```

---
---

# Exploit

## SMB 資格情報ブルートフォース

NSE
```zsh
nmap --script smb-brute -p 445 <target_IP>
```

Hydra（[🐉THC-Hydra](../../Tools/🐉THC-Hydra.md)）
```zsh
hydra -V -f -l <username> -P <wordlist> <target_IP> smb
```
- SMBv1が無効なときは、`invalid reply`と表示されるので、NetExecで認証情報を検証する

SMBリレー攻撃
```zsh
netexec smb <target_IP> --gen-relay-list <output.txt>
```
- [Password Attack](../Common/Password%20Attack.md#Net-NTLMv2のリレーアタック)へ

## ファイルアップロード

- 条件：WRITE Permissions
- アタックベクター：PHP WebShellペイロードをアップロード→ブラウザで同ファイルにアクセスしてWebShell獲得

```zsh
smbclient //<target_IP>/<share> -U <username>%<password>
smb: > put <local> <remote>
```

## Misc

| 攻撃手法          | Tool                  | Command例                                          |
| ------------- | --------------------- | ------------------------------------------------- |
| Pass-the-Hash | NetExec               | `netexec smb <target_IP> -u <username> -H <NTLM>`   |
| SMB Relay     | Impacket (ntlmrelayx) | `impacket-ntlmrelayx -tf <targets.txt>`           |
| 任意コマンド実行      | NetExec / Impacket    | `nxc smb <target_IP> -u <user> -p <password> -x 'whoami'` |

---
---

# トラブルシューティング

## smbclient 

### プロトコルエラー対策

最近の Kali では NTLMv1 が無効化されており、以下のエラーが表示されることがある
```
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```

対処例：
1. `/etc/samba/smb.conf` の `[global]` に追記
```zsh
client min protocol = CORE
client max protocol = SMB3
```

2. コマンドラインで `-m SMB2` または `-m SMB3` を指定（enum4linux など一部ツールでは無効）

### NT_STATUS_IO_TIMEOUT listing

原因：ディレクトリにファイルが大量にあるか、ファイルサイズが大きい

対処例：`-t`でタイムアウト時間を延ばす
```zsh
smbclient \\\\<target_IP>\<share> -W <domain> -U '<username>' -t 60
```
- →これでもTIMEOUTの場合は、フォルダ構成を理解し、直でファイルを指定する
	- 具体的には、アプリケーションのバックアップフォルダがあり、ファイルがTIMEOUTで取得できない場合は、アプリケーションの公式ドキュメントを読み、passwordのあるファイルをダイレクトに指定して取得する

---
---

# 🔗参考リンク

- [HackTricks: Pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [0xdf SMB cheatsheet](https://0xdf.gitlab.io/cheatsheets/smb-enum)
- [NetExec: smb-protocol enumeration](https://www.netexec.wiki/smb-protocol/enumeration)
