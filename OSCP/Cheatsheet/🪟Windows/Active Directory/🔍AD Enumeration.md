- 関連ノート：
	- [🐶Bloodhound](../../../Tools/🐶Bloodhound.md)

---

# ADの自動列挙 w/ ldapdomaindump

- 🔗[ldapdomaindump](https://www.kali.org/tools/python-ldapdomaindump/)
- 初期段階において有用だが、BloodHoundのほうが可読性が高い
```zsh
# anonymous
ldapdomaindump <target_IP> -n <DC_IP> -o <outputdir>

# 認証有り
ldapdomaindump -u '<domain>\<username>' -p '<pw | LM:NTLMhash>' <target_IP> -n <DC_IP> -o <outputdir>
```

---

# ADの自動列挙 w/BloodHound

- BloodHoundは、[ADの手動列挙](#ADの手動列挙)で確認できる内容とほぼ同じ内容を、短時間かつ視覚的に確認でき、PEN-200モジュールでも、まずはBloodHoundを実行するよう推奨されている
- **手動・自動の両方で列挙すること**
	- 例えば、手動列挙の`Find-LocalAdminAccess`でわかることが、BloodHoundではわからない

- 使い方：[🐶Bloodhound](../../../Tools/🐶Bloodhound.md)

---

# ADの手動列挙

>[!NOTE]
>`Get-ADUser`は🔗[RSAT](https://learn.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/remote-server-administration-tools)の一部であり、DCにのみインストールされていることが多いため、基本的に使えないと考えた方がよい

- 手動列挙の多くは、PowerViewを使った列挙となる（🔗[PowerViewコマンドリファレンス - PowerSploit](https://powersploit.readthedocs.io/en/latest/Recon/)）

## ユーザー・グループオブジェクトの列挙

[補足：windowsに存在するユーザーについて](#補足：windowsに存在するユーザーについて)

### 目的

1. ユーザー/グループの属性(Attribute)を把握し、リスト化する
2. 異質なユーザー/グループを探す
	- デフォルトで存在するグループ：🔗[Active Directory security groups - Microsoft](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups)

### ビルトインコマンドによるユーザー・グループの列挙

>[!Warning]
>特定の属性（Attribute）を表示できないため、見逃しが発生する可能性あり

現在のユーザーが保持するチケットを確認する
```cmd
klist
```

ADのドメインのユーザーをすべてEnum
```cmd
net user /domain
```

一つのユーザーをさらに深掘りする
```cmd
net user <username> /domain
```

ADのドメインのグループをすべてEnum
```cmd
net group /domain
```
- 見慣れないグループがあったら着目
- デフォルトで存在するグループ：🔗[Default AD security group - Microsoft](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#default-active-directory-security-groups)

一つのグループをさらに深掘りする
```cmd
net group <groupname> /domain
```

ADのパスワードポリシーを表示する
```cmd
net accounts /domain
```

### ⭐️PowerViewによるユーザー・グループの列挙列挙

1. kaliにはpowerviewが以下のパスに存在するため、ターゲットマシンに転送する
	- [ファイル操作、ユーティリティ](../../Common/ファイル操作、ユーティリティ.md#ファイルの転送)
```
/usr/share/powershell-empire/empire/server/data/module_source/situational_awareness/network/powerview.ps1
```

2. ターゲット上で関数を実行できるようにするために、execution policyを回避し、メモリに関数をインポートする
```powershell
powershell -ep bypass
Import-Module .\PowerView.ps1
```
- `Exception calling "FindAll" with "0" argument(s): "An operations error occurred.`：権限不足

3. 列挙する
```powershell
# ドメイン情報の列挙（💡ノートで<domain>とあるものはこの出力のNameプロパティを使用）
Get-NetDomain

# ドメイン内のすべてのユーザーとその属性を列挙（出力が膨大なので必要に応じてselectを使う）
Get-NetUser | select cn

# 特定ユーザーの属性を表示
Get-NetUser '<common_name>'

# パスワードの変更日と最終ログイン日を合わせて出力(※)
Get-NetUser | select cn,pwdlastset,lastlogon

# ドメイン内のすべてのグループとその属性を列挙（出力が膨大なので必要に応じてselectを使う）
Get-NetGroup | select cn

# 特定グループの属性を表示（member属性が存在しない場合はmemberがいない）
Get-NetGroup '<common_name>'

# Kerberosの事前認証が無効なユーザーを列挙→AS-REP Roasting
Get-DomainUser -PreauthNotRequired
```
- (※)パスワードアタックの対象としてパスワードが全然変更されていないユーザーや、もしくは侵害しても注意を引かないような全然使われていないユーザーを列挙するため

### PowerShell自作関数によるユーザー・グループの列挙

AMSIなどの影響で<u>PowerViewが使えないとき</u>に使う。

1. 以下の関数を用意し、ファイルに保存する（ファイル名：`function.ps1`とする）
	- [補足：powershell列挙関数の解説](#補足：powershell列挙関数の解説)
```powershell
function LDAPSearch {
    param (
        [string]$LDAPQuery
    )
	# ①LDAP pathの用意：LDAP://DC1.corp.com/DC=corp,DC=com
    $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
    $DistinguishedName = ([adsi]'').distinguishedName
	# ②DirectoryEntryクラスでオブジェクトをカプセル化する
    $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")
	# ③フィルタリングして検索
    $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)

    return $DirectorySearcher.FindAll()
}
```

2. 関数を実行できるようにするために、execution policyを回避し、メモリに関数をインポートする
```powershell
powershell -ep bypass
Import-Module .\function.ps1
```

3. 関数を使い検索する
```powershell
# ユーザーアカウントを列挙するため、
# samAccountType(*1)でユーザーに絞り検索
LDAPSearch -LDAPQuery "(samAccountType=805306368)"

# グループ一覧を取得して各グループのCNとmember属性を確認するため
# ObjectCategory(*2)でフィルター
foreach ($group in $(LDAPSearch -LDAPQuery "(objectCategory=group)")) {
$group.properties | select {$_.cn}, {$_.member}}

# 特定ユーザーのプロパティを表示
foreach ($user in $(LDAPSearch -LDAPQuery "(cn=<common name>)")) {
$user.properties

# 指定したグループのプロパティを確認するため、
# 特定のグループかつcn（グループ名）(*3)に絞って調査
$group = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=<common name>))"
$group.properties
```
- (※1)　[補足：samAccountType一覧](#補足：samAccountType一覧)
- (※2）ObjectCategory：オブジェクトの主要カテゴリ（グループ、ユーザー等）を表す
- (※3)  例えば：`Development Department*`（ワイルドカード使用可）

>[!NOTE]
>実行結果が`CN=hoge,CN=Users...`とあっても、hogeというオブジェクトがuserかgroupなのかは、そのプロパティを見る必要がある。
>nested group（グループの中にさらにグループがある）の場合は、重要なユーザーの見逃しに注意すること

---

## OS、ホストの列挙

### 目的

1. コンピューターオブジェクトをリスト化する
2. OS情報からドメイン内で相対的に古いOSを使っているコンピューターを探す
3. ホスト名からweb もしくは ファイルサーバーなどの*クラウンジュエル*を保有していそうなものに見当をつける

### コンピューターオブジェクトの列挙 w/ PowerView

OS情報とホスト名に絞った列挙
```powershell
Get-NetComputer | select dnshostname, operatingsystem, operatingsystemversion
```

特定ホストの属性列挙
```powershell
Get-NetComputer '<dnshostname>'
```
- `operatingsystemversion`は`10.0（20348）`のように、括弧内にビルド番号があり、ビルド番号を🔗[List of Microsoft Windows versions - Wikipedia](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)で検索することで、詳細なOSバージョンがわかる

特定ホストの名前解決
```powershell
# nslookup.exe '<dnshostname>'で可
Resolve-DnsName -ComputerName '<dnshostname>'
```

---

## パーミッションとログイン中のユーザーの列挙

### 目的

- ドメイン内の権限関係とログオン状況を把握して攻撃ベクター（足場→横展開→昇格）の候補を見つける（Chained Compromise）
	- 最終ゴールはDomain Adminだが、🔗[クラウンジュエル](https://www.sompocybersecurity.com/column/glossary/crown-jewel)を持つDBやファイルサーバーもゴールとなりえる
	- 足場のユーザーが封鎖されても、同等権限を持つユーザーを侵害しておけば、足場を維持できる

- 他のユーザーがどの端末にログオンしているかを把握することで、横展開や資格情報収集、「有利な踏み台」の発見につなげる
	- ログオン済み端末のメモリからTGT/TGSやNTLMハッシュを盗む
	- セッション乗っ取り

### パーミッションとログイン中のユーザーの列挙コマンド

1. ドメイン内で現在のアカウントが管理者権限をもつマシンを列挙する（PowerView)
```powershell
Find-LocalAdminAccess
```
- ドメインのサイズなどの環境によっては、完了までに数分かかる

2. 誰がどこからログオンしているかを確認（PowerView）
```powershell
Get-NetSession -ComputerName <dnshostname(Get-NetComputerの出力)>　-Verbose
```

3. `Get-NetSession`がAccess is deniedエラーで失敗した場合は、PsLoggedOnを試す
	（[PsLoggedOn](../../../Tools/🛠️Windows%20Sysintarnals.md#PsLoggedOn)）
	（[補足：Get-NetSessionがうまくいかない理由](#補足：Get-NetSessionがうまくいかない理由)）
```powershell
.\PsLoggedon.exe \\<dnshostname>
```
- `Users logged on locally:`もしくは`Users logged on via resource shares:`の場合は、<u>横展開のターゲットとして着目</u>
- 💥現在のユーザーがログオン中のユーザーとして表示された場合は、そのユーザーの認証情報を用いてマシンの侵害が可能

#### 補足：Get-NetSessionがうまくいかない理由

対応策はPsLoggedOnを使うか、諦めるか

- PowerViewのGet-NetSessionは🔗[NetSessionEnum](https://learn.microsoft.com/en-us/windows/win32/api/lmshare/nf-lmshare-netsessionenum)APIに依存する
	- NetSessionEnumは`SrvsvcSessionInfo`レジストリキーの状態に依存する
- たとえ管理者権限があったとしても、 Windows10 / Server 2019以降は、<u>デフォルトでリモートからの列挙が制限される</u>可能性がある
	- レジストリへの「ローカルな」アクセス権と「リモート列挙できるか」は別の話
	- ACL に ReadKey があるプリンシパルが「ローカルのプロセス」や「OSが認めたアプリ」なら、そのプリンシパルはローカルでログオンユーザーを読み取れる
	- しかし「リモート接続しているドメインユーザー」がその権限を利用して遠隔から読み取れるかは、ACL の対象（Identity）がリモート呼び出しを許すものであるかどうか次第

---

## SPNの列挙

### 目的

サービスアカウントとそれに紐づくSPN（Service Principal Name）を洗い出して、どのアカウントがどのサービス用SPNを持っているか（＝どの鍵で署名されるか）を把握する。
※ 実際の稼働有無は別途確認が必要
	SPN：[AD用語一覧表](ADの基本.md#AD用語一覧表)

### 背景

- アプリケーションはそのサービスアカウントのコンテキスト(≒権限)で動作するため、サービスアカウントは特定サーバ上のローカル管理者権限などの高い権限を持つ場合がある
- SPN は AD に登録されるため、DCに問い合わせるだけでわかる

### SPNの列挙コマンド

サービスアカウント名とそのSPNを表示（PowerView）
```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```
	↓出力例
```powershell
samaccountname serviceprincipalname
-------------- --------------------
krbtgt         kadmin/changepw
iis_service    {HTTP/web04.corp.com, HTTP/web04, HTTP/web04.corp.com:80}
```
- 読み方：`serviceprincipalname` に `HTTP/web04.corp.com` とあれば 、web04.corp.com 上で HTTP (80)サービスが動いている可能性が高いと判断
	- →💥[Silver Ticket](../../../Tools/🥝Mimikatz.md#Silver%20Ticket)

Windows標準インストールの`setspn.exe`
```powershell
# setspn -L iis_service
setspn -L <サービスアカウント>
```

>[!NOTE]
>SPN があっても必ずサービスが稼働中とは限らない（古い登録、誤登録の可能性）ため、実接続で確認すること。

---

## オブジェクトの権限列挙

- 関連ノート：
	- [ACL, DACL, ACE](../../../Misc/用語.md#ACL,%20DACL,%20ACE)
	- [SID, RID](../../../Misc/用語.md#SID,%20RID)

### 目的

オブジェクトのアクセス制御を列挙し、悪用可能な権限をもつプリンシパルをみつける。

### オブジェクトの権限列挙コマンド

1. 指定したオブジェクトのアクセス制御を列挙し、権限及びその権限をもつプリンシパルを列挙（PowerView）
```powershell
# 具体例：Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights
Get-ObjectAcl -Identity "<common_name>" | ? {$_.ActiveDirectoryRights -eq "<権限>"} | select SecurityIdentifier,ActiveDirectoryRights
```
- `SecurityIdentifier`：指定したオブジェクトに対して権限をもつプリンシパル
	- 具体例でいうと、Management Departmentオブジェクトに対して権限をもつプリンシパル（Adminなど）
- `ActiveDirectoryRights`：オブジェクトに対するプリンシパルの権限
	- ACLでは*GenericAll*が最強： [補足：攻撃者が着目する権限一覧](#補足：攻撃者が着目する権限一覧)
- 補足：`ObjectSID`：オブジェクト自身のSID(重要でない)

2. SIDの名前解決（PowerView）
```powershell
# 例：S-1-5-21-1987370270-658905905-1781884369-512","S-1-5-21-1987370270-658905905-1781884369-1104'| Convert-SidToName
'<SID>' | Convert-SidToName
```
	↓出力例
```
CORP\Domain Admins
CORP\stephanie
BUILTIN\Account Operators
Local System
CORP\Enterprise Admins
```
- GenericAllをもつプリンシパルを侵害できたら、そのオブジェクトを自由自在に操作可能

#### 補足：攻撃者が着目する権限一覧

[💥AD Exploit](💥AD%20Exploit.md)

| 権限名                   | 内容                |
| :-------------------- | :---------------- |
| *GenericAll*          | オブジェクトに対するフル権限    |
| GenericWrite          | 特定の属性を編集可能        |
| WriteOwner            | 所有者を変更可能          |
| WriteDACL             | ACLの編集が可能         |
| AllExtendedRights     | パスワード変更・リセットなど    |
| ForceChangePassword   | 他ユーザーのパスワード変更     |
| Self（Self-Membership） | 自分自身をグループなどに追加できる |

---

## Domain共有の列挙

### 目的

1. ドメイン全体で共有されているフォルダー（domain shares）を列挙し、  重要情報（パスワード、設定ファイル、スクリプトなど）を探索すること
2. 特に SYSVOL や非デフォルト共有（例: docshare）に注目する
	- **量は多いがすべて調査対象！**

### Domain共有の列挙コマンド

1. すべてのDomain共有一覧を取得する（PowerView）
	将来的に標的とする可能性があるDomain共有も含めた完全な一覧を取得するため、すべてのDomain共有一覧は必ず取得すること（少し時間がかかる）
```powershell
Find-DomainShare
```

2. 現在のユーザーがアクセス可能なDomain共有に絞って列挙する（PowerView）
```powershell
Find-DomainShare -CheckShareAccess
```
	↓出力例
```powershell
Name           Type Remark                 ComputerName
----           ---- ------                 ------------
ADMIN$   2147483648 Remote Admin           DC1.corp.com
C$       2147483648 Default share          DC1.corp.com
IPC$     2147483651 Remote IPC             DC1.corp.com
NETLOGON          0 Logon server share     DC1.corp.com
```

3. 現在のユーザーがアクセス可能なDomain共有を探索する
```powershell
# 例：ls \\dc1.corp.com\sysvol
ls \\<ComputerName>\<Name>
```
- ComputerNameとNameは上記コマンドの出力を使う
- 補足：権限が不足しているときは、"Access is denied"及び"Cannnot find path"エラーが出る

#### 💥機密情報が保存されているGPPとその復号

- GPP（Group Policy Preferences）：Windows Server 2008 以降に導入された機能で、ADドメイン環境でクライアント設定を自動配布・適用するための機能
- 古いGPPを`Policy`などの名称でDomain共有に保存していることがある
- 各クライアントのlocal administratorのパスワード(`cpassword`という名称)を同時に変更するため、GPPに暗号化して保存していたことがあった
- 攻撃者のマシン上で暗号化されたlocal administratorのパスワードを復号できる
```zsh
gpp-decrypt '<cpasswordの値>'
```

---

# 補足

## 補足：powershell列挙関数の解説

関連用語：[AD用語一覧表](ADの基本.md#AD用語一覧表)

### ①LDAP pathの用意

- 目的：Primaly Domain（PDC）をホスト名としたLDAP pathを用意する
	- PDCは１つのドメインに１つまでしか存在できず、しかも最も新しい情報をもっている
```powershell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
$DistinguishedName = ([adsi]'').distinguishedName
```

- 前提知識：<u>ADとの通信にはLDAP pathが必要</u>であり、LDAP provider（パス）の形式は以下となっている
```powershell
LDAP://HostName[:Port][/DistinguishedName]
```

- `$DistinguishedName = ([adsi]'').distinguishedName`：ADSIを使い、LDAPで使用可能な命名規則に則ったDNのDCを抽出する
```powershell
# DNの命名規則
CN=Stephanie,CN=Users,DC=corp,DC=com
```
- ※DNはDC(Domain Component)までを指定する。CNまで指定すると、探索範囲が狭まるから

### ②DirectoryEntryクラスでオブジェクトをカプセル化する

- LDAP pathをカプセル化している
- 目的：カプセル化することで、オブジェクトのプロパティ（属性）を参照したり、操作したりするのが統一的にできる
	- 別ドメイン／別フォレストの AD に接続してクエリを実行したいときなどに、認証情報をカプセル化して渡すこともできる：`System.DirectoryServices.DirectoryEntry($LDAP, $username, $password)`

### ③フィルタリングして検索

- `DirectorySearcher`クラスはLDAP pathを用いてADに対してクエリを実行できる
- 第二引数にフィルタリング条件をもてる

## 補足：samAccountType一覧

ソース：🔗[SAM-Account-Type 属性 - Microsoft](https://learn.microsoft.com/ja-jp/windows/win32/adschema/a-samaccounttype)
	16進数で記載されているが、LDAPで通信する際は10進数を使う

| 名前                      | 16進値         | 10進値      | 説明          |
| ----------------------- | ------------ | --------- | ----------- |
| SAM_NORMAL_USER_ACCOUNT | `0x30000000` | 805306368 | 通常ユーザーアカウント |
| SAM_MACHINE_ACCOUNT     | `0x30000001` | 805306369 | コンピュータアカウント |
| SAM_TRUST_ACCOUNT       | `0x30000002` | 805306370 | トラストアカウント   |
| SAM_GROUP_OBJECT        | `0x10000000` | 268435456 | グループ        |

## 補足：windowsに存在するユーザーについて

| ユーザー（人）            | 詳細                                                                                |
| ------------------ | --------------------------------------------------------------------------------- |
| **Administrators** | 最も多くの特権を持っている。システム構成パラメータを変更したり、システム内のあらゆるファイルにアクセスすることができる                       |
| Standard Users     | コンピューターにアクセスすることはできるが、限られたタスクしか実行できない。システムには永続的または重要な変更を加えることができず、変更はファイルだけに限定される |

| ユーザー（人以外）                | 説明                                                                                                        |
| ------------------------ | --------------------------------------------------------------------------------------------------------- |
| **SYSTEM / LocalSystem** | OSが内部タスクを実行するために使用するアカウント。ホスト上で利用可能なすべてのファイルとリソースに==フルアクセスでき、Adminよりもさらに高い特権を持つ==                         |
| Local Service            | Windowsサービスを「最小」権限で実行するために使用するデフォルトのアカウント。ネットワーク上で匿名接続を使用する                                               |
| Network Service          | Local Serviceと同じく、Windowsサービスを「最小」権限で実行するために使用されるデフォルトのアカウント。しかし、ネットワークを通じて認証するために、コンピュータの認証情報を使用するところが違う |

| AD ==Admin権限==アカウント    | 説明                          |
| ---------------------- | --------------------------- |
| BUILTIN\\Administrator | DCのローカル管理者権限                |
| Domain Admins          | ドメイン内のすべてのリソースにアクセスできる管理者権限 |
| Enterprise Admins      | forest rootのみで使用可能          |
| Schema Admins          | ドメイン/フォレストを変更可能。攻撃者に狙い目     |
| Server Operators       | ドメインサーバーの管理ができる             |
| Account Operators      | 特権グループに属さないユーザーを管理できる       |
