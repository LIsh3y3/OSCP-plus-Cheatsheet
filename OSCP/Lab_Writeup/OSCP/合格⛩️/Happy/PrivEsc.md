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

