Windowsがどのように動作しているか、そのコアを理解する。

- 詳細：[Windows Internals - TryHackMe](https://tryhackme.com/room/windowsinternals)

---

# プロセス

## プロセスとは

プログラムの実行を維持・表現するもので、1つのアプリケーションが複数のプロセスを持つこともある。アプリケーションの実行によって作成され、Windowsの機能のコアをなしている。

Windowsのほとんどの機能はアプリケーションと対応するプロセスを持っている。例：
- `MsMpEng`：Microsoft Defender
- `wininit`：キーボードとマウス
- `lsass`：認証情報の保存

## 攻撃者の視点

攻撃者は検知を回避しマルウェアを合法的なプロセスに見せかけるためにプロセスを標的にする。

|攻撃手法|ATT&CK|
|---|---|
|プロセスインジェクション|[T1055](https://attack.mitre.org/techniques/T1055/)|
|プロセスホロウィング|[T1055.012](https://attack.mitre.org/techniques/T1055/012/)|
|プロセスのなりすまし|[T1055.013](https://attack.mitre.org/techniques/T1055/013/)|

## プロセスの構成要素

**高レベルな構成要素：**

|コンポーネント|説明|
|---|---|
|Private Virtual Address Space|プロセスに割り当てられた仮想メモリアドレス空間|
|Executable Program|仮想アドレス空間に格納されるコードとデータを定義する|
|Open Handles|プロセスからアクセス可能なシステムリソースへのハンドル|
|Security Context|アクセストークン（ユーザ・セキュリティグループ・権限などを定義）|
|Process ID|プロセスのユニークな数値識別子|
|Threads|実行が予定されているプロセスのセクション|

**メモリレベルの構成要素：**

|コンポーネント|説明|
|---|---|
|Code|プロセスで実行されるコード|
|Global Variables|グローバル変数|
|Process Heap|データが格納されるヒープ|
|Process Resources|プロセスの追加リソース|
|Environment Block|プロセス情報を定義するデータ構造|

## タスクマネージャーで見えるプロセス情報

|項目|説明|例|
|---|---|---|
|Name|プロセスの名前（通常アプリケーションから継承）|conhost.exe|
|PID|プロセスを識別するユニークな数値|7408|
|Status|プロセスの実行状態（running, suspended など）|Running|
|User name|プロセスを開始したユーザ（プロセスの特権を示す）|SYSTEM|

## プロセス可視化ツール

- 🔗[Process Hacker 2](https://github.com/processhacker/processhacker)
- 🔗[Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer)
- 🔗[Process Monitor（Procmon）](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)

---

# スレッド

## スレッドとは

プロセスに採用され、デバイスの要因（CPU・メモリ仕様・優先順位など）に基づいてスケジュールされる実行可能なユニット。簡単にいうと「プロセスの実行を制御するもの」。

実行を制御するため、攻撃者に狙われやすいコンポーネント。スレッドの悪用は単独でコード実行を補助するほか、他のAPIコールと組み合わせて使われることが多い。

スレッドは親プロセスとコード・グローバル変数などのリソースを共有しつつ、以下の独自データも持つ：

|コンポーネント|説明|
|---|---|
|Stack|スレッド固有のデータ（例外・プロシージャコールなど）|
|Thread Local Storage|固有のデータ環境にストレージを割り当てるためのポインタ|
|Stack Argument|各スレッドに割り当てられたユニークな値|
|Context Structure|カーネルが保持するマシンレジスタの値|

---

# 仮想メモリ

## 仮想メモリとは

アプリケーション間のメモリ衝突リスクなしに、物理メモリであるかのようにメモリと対話できる仕組み。各プロセスにプライベートな仮想アドレス空間を提供し、メモリマネージャが仮想アドレスを物理アドレスに変換する。

物理メモリに直接書き込まないことでプロセスが損害を受けるリスクが減る。また、アプリケーションが割り当てられた物理メモリより多くの仮想メモリを使用する場合、メモリマネージャは仮想メモリをディスクにページングして対応する。

![](../../../Images/Pasted%20image%2020230618092948.png)

$$仮想メモリのイメージ$$

## アドレス空間のレイアウト

|                | 32ビット（x86）                     | 64ビット  |
| -------------- | ------------------------------ | ------ |
| 理論上の最大仮想アドレス空間 | 4 GB                           | 256 TB |
| ユーザ空間（プロセス用）   | 0x00000000 〜 0x7FFFFFFF（下位2GB） | 比率は同じ  |
| カーネル空間（OS用）    | 0x80000000 〜 0xFFFFFFFF（上位2GB） | 比率は同じ  |

32ビット環境でより大きなアドレス空間が必要な場合、管理者は `increaseUserVA` の設定またはAWE（Address Windowing Extensions）で割り当てを変更できる。64ビット環境ではアドレス空間が大幅に増加したため、こうした設定変更が必要なケースは大幅に減少している。

![](../../../Images/Pasted%20image%2020230618093936.png)

$$アドレス空間のイメージ$$

> [!NOTE] 
> この概念はWindowsの内部機能に直接関係するわけではないが、内部機能を悪用する際の理解に役立つ。

---

# DLL（Dynamic Link Library）

## DLLとは

複数のプログラムで同時に使用できるコードとデータを含むライブラリ（Unixの🔗[Shared Objects](https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/6n33n7f8u/index.html)に相当）。コードのモジュール化・再利用・効率的なメモリ使用・ディスクスペース削減を目的としており、OSとプログラムの読み込み・実行を高速化する。

代表的なDLL：
- `kernel32.dll`：Windows APIのコア。メモリ管理・プロセス管理など、カーネルレベルの機能を提供する
- `user32.dll`：ウィンドウの作成・描画・イベント処理など、ユーザーインタラクションに関連する機能を提供する

## 攻撃者の視点

プログラムはDLLに依存しているため、攻撃者はアプリケーション本体ではなくDLLを標的にして実行・機能の一部を制御できる。

|攻撃手法|ATT&CK|
|---|---|
|DLLハイジャッキング|[T1574.001](https://attack.mitre.org/techniques/T1574/001/)|
|DLLサイドローディング|[T1574.002](https://attack.mitre.org/techniques/T1574/002/)|
|DLLインジェクション|[T1055.001](https://attack.mitre.org/techniques/T1055/001/)|

## DLL Search Order と DLL Hijacking

DLL Search Orderとは、Windowsがアプリがフルパスを指定せずに`LoadLibrary()`などでDLLをロードする際の探索順序。DLLハイジャッキングはこの仕組みを悪用する。

**悪用の流れ：**

1. ターゲットアプリが `abc.dll` をロードしようとするが存在しない
2. 攻撃者が同名の悪意ある `abc.dll` を探索順序の早い場所に配置する
3. アプリ起動時に攻撃者のDLLが先に読み込まれる

### Safe DLL Search Mode の探索順序

Safe DLL Search Mode が **ON**（デフォルト）の場合、カレントディレクトリは5番目（**OFF**の場合は2番目）に探索される。

|順序|探索場所|
|---|---|
|1|アプリケーションの実行ディレクトリ（.exeが置かれているフォルダ）|
|2|システムディレクトリ（通常 `C:\Windows\System32`）|
|3|16-bitシステムディレクトリ（`C:\Windows\System`、互換性目的）|
|4|Windowsディレクトリ（通常 `C:\Windows`）|
|5|カレントディレクトリ（サービスの場合はシステムディレクトリ）|
|6|System PATH環境変数に記載されたディレクトリ|

```powershell
# System PATH環境変数の確認
[Environment]::GetEnvironmentVariable("Path", "Machine")
```

### 探索順序の例外

**① KnownDLLs（既知のDLL）**

Windowsには探索順序を無視して強制的に `System32` から読み込むDLLのリスト（KnownDLLs）が存在する。これらは偽物を配置しても意味がない。

- 登録場所：`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs`
- 例：`kernel32.dll`、`user32.dll`、`ole32.dll` など

**② フルパスでのロード**

`LoadLibrary("C:\Windows\System32\example.dll")` のようにフルパスで指定されている場合は探索順序が無視される。

## DLLのリンク方式

### ロードタイム・ダイナミックリンク

アプリケーションがヘッダ（`.h`）とインポートライブラリ（`.lib`）を使ってDLLの関数を明示的に呼び出す方式。

```cpp
#include "stdafx.h"
#include "sampleDLL.h"
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
    HelloWorld();
    return 0;
}
```

### ランタイム・ダイナミックリンク

`LoadLibrary` または `LoadLibraryEx` を使って実行中にDLLをロードし、`GetProcAddress` で呼び出す関数を特定する方式。

```cpp
...
typedef VOID (*DLLPROC) (LPTSTR);
...
HINSTANCE hinstDLL;
DLLPROC HelloWorld;
BOOL fFreeDLL;

hinstDLL = LoadLibrary("sampleDLL.dll");
if (hinstDLL != NULL)
{
    HelloWorld = (DLLPROC) GetProcAddress(hinstDLL, "HelloWorld");
    if (HelloWorld != NULL)
        (HelloWorld);
    fFreeDLL = FreeLibrary(hinstDLL);
}
```

> [!NOTE] 
> 悪意のあるコードではロードタイム方式よりランタイム方式が多く使われる。
> 悪意あるプログラムがメモリ領域間でファイルを転送する必要がある場合、ランタイム方式は単一のDLLを転送するだけで済み、ロードタイム方式のように複数の依存ファイルを必要としないため管理しやすい。

---

# Portable Executable Format（PE形式）

## PE形式とは

Windows上の実行ファイル（`.exe`）やDLL（`.dll`）のファイル構造を定義するフォーマット。実行ファイルの構成要素とデータの格納方法を規定する。PEファイルとCOFF（Common Object File Format）ファイルで構成される。

PEデータは16進ダンプで確認するのが一般的。

## PEの構造

PEデータは以下の7つの要素で構成される：

|要素|説明|
|---|---|
|DOS Header|ファイルの種類を定義。先頭の `MZ` マジックナンバーで `.exe` であることを示す|
|DOS Stub|互換性のためのプログラム。DOSモードで実行されると「This program cannot be run in DOS mode」と表示する|
|PE File Header|バイナリのPEヘッダー情報（ファイルフォーマット・署名・画像ファイルヘッダなど）|
|Image Optional Header|PE File Headerの重要な部分。Data Dictionary（画像データのディレクトリ構造へのポインタ）を含む|
|Section Table|実行ファイル内で利用可能なセクションとその情報を定義する|
|Sections|ファイルの内容・データを格納する各セクション|

![](../../../Images/Pasted%20image%2020230618104414.png)

$$PEの構造イメージ$$

## セクションの種類

|セクション|内容|
|---|---|
|`.text`|実行コードとエントリポイント|
|`.data`|初期化済みの変数（文字列・変数など）|
|`.rdata`|読み取り専用データ（定数・文字列リテラルなど）|
|`.idata`|インポートテーブル（Windows API・DLLの参照情報）|
|`.reloc`|リロケーション情報|
|`.rsrc`|アプリケーションリソース（画像など）|
|`.debug`|デバッグ情報|

> [!NOTE] `.rdata` と `.idata` は混同されやすいが別のセクション。`.rdata` は読み取り専用データ全般を含み、`.idata` がインポートテーブルを含む。

## Detect It Easy（DIE）

🔗[DIE](https://github.com/horsicq/Detect-It-Easy)：バイナリファイルの解析・ファイル種類の識別・パッカー/コンパイラの検出に使用するツール。

主な確認事項：

- Entry Point：実行可能バイナリの実行開始アドレス
- Sections：セクション数と各セクションの情報
- PE：仮想アドレス・オフセット（ファイル内の特定位置を示す相対的な位置）

![](../../../Images/Pasted%20image%2020260325124505.png)

$$DIEイメージ$$

---

# Windows Internalsとのやり取り（Windows API）

## Windows APIとは

Windows OSと対話するためのネイティブ機能を提供するインターフェイス。64bit環境でもWin32 APIが使われる（Win64 APIという独立したAPIは存在しない）。

## ユーザモードとカーネルモード

Windowsの内部コンポーネントの多くは物理的なハードウェアやメモリと対話する必要がある。このため、Windowsカーネルがすべてのプログラムとプロセスを制御し、ソフトウェアとハードウェアの相互作用を橋渡しする。

通常のアプリケーションはカーネルに直接アクセスできないため、**プロセッサモード**によってアクセスレベルが制御される。

|            | ユーザモード                 | カーネルモード     |
| ---------- | ---------------------- | ----------- |
| ハードウェアアクセス | 直接アクセス不可               | 直接アクセス可能    |
| アドレス空間     | プロセスごとのプライベートな仮想アドレス空間 | 共有の仮想アドレス空間 |
| メモリアクセス    | 「所有されたメモリ位置」のみ         | 物理メモリ全体     |

ユーザモードで開始されたアプリケーションは、**システムコール**または**APIコール**が発生するまでそのモードに留まる。システムコールが発生するとカーネルモードに切り替わる。

![](../../../Images/Pasted%20image%2020230618114052.png)

$$ユーザーモードからカーネルモードへの切り替えフロー$$

C#などの言語はCLR（共通言語ランタイム）などのランタイムを経由してWin32 APIとやり取りするため、この流れはさらに複雑になる。

## 実装例：ローカルプロセスへのメッセージボックス注入

メモリとやり取りするProof-of-Conceptとして、ローカルプロセスのメモリ上にメッセージボックスを書き込んで実行する例を示す。

手順：
1. メッセージボックスのためにローカルプロセスのメモリを確保する
2. 確保したメモリにメッセージボックスを書き込む
3. メモリからメッセージボックスを実行する

```cpp
// 1. プロセスのハンドルを取得
HANDLE hProcess = OpenProcess(
    PROCESS_ALL_ACCESS,         // アクセス権の定義
    FALSE,                      // ハンドルの継承なし
    DWORD(atoi(argv[1]))        // コマンドライン引数で指定したプロセスID
);

// 2. メモリ領域を確保
remoteBuffer = VirtualAllocEx(
    hProcess,                   // 対象プロセス
    NULL,
    sizeof payload,             // 確保するメモリサイズ
    (MEM_RESERVE | MEM_COMMIT), // メモリを予約・コミット
    PAGE_EXECUTE_READWRITE      // 実行・読み書き権限を付与
);

// 3. ペイロードをメモリに書き込む
WriteProcessMemory(
    hProcess,                   // 対象プロセス
    remoteBuffer,               // 確保したメモリ領域
    payload,                    // 書き込むデータ
    sizeof payload,             // データのバイトサイズ
    NULL
);

// 4. メモリからペイロードを実行
remoteThread = CreateRemoteThread(
    hProcess,                               // 対象プロセス
    NULL,
    0,                                      // スタックサイズ（デフォルト）
    (LPTHREAD_START_ROUTINE)remoteBuffer,   // 実行開始アドレス
    NULL,
    0,                                      // 作成後すぐに実行
    NULL
);
```