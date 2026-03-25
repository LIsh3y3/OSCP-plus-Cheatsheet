Windowsがどのように動作しているか、そのコアを理解する。

- 参照🔗：[Windows Internals - TryHackMe](https://tryhackme.com/room/windowsinternals)
- 関連ノート：[Windows APIのIntro](https://claude.ai/chat/2.%20Windows%20API%E3%81%AEIntro.md) / [Abusing Windows Internals](https://claude.ai/chat/3.%20Abusing%20Windows%20Internals.md)

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

## アドレス空間のレイアウト

|                | 32ビット（x86）                     | 64ビット  |
| -------------- | ------------------------------ | ------ |
| 理論上の最大仮想アドレス空間 | 4 GB                           | 256 TB |
| ユーザ空間（プロセス用）   | 0x00000000 〜 0x7FFFFFFF（下位2GB） | 比率は同じ  |
| カーネル空間（OS用）    | 0x80000000 〜 0xFFFFFFFF（上位2GB） | 比率は同じ  |

32ビット環境でより大きなアドレス空間が必要な場合、管理者は `increaseUserVA` の設定またはAWE（Address Windowing Extensions）で割り当てを変更できる。64ビット環境ではアドレス空間が大幅に増加したため、こうした設定変更が必要なケースは大幅に減少している。

> [!NOTE] 
> この概念はWindowsの内部機能に直接関係するわけではないが、内部機能を悪用する際の理解に役立つ。

---

# DLL（Dynamic Link Library）

## DLLとは

複数のプログラムで同時に使用できるコードとデータを含むライブラリ（Unixの[Shared Objects](https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/6n33n7f8u/index.html)に相当）。コードのモジュール化・再利用・効率的なメモリ使用・ディスクスペース削減を目的としており、OSとプログラムの読み込み・実行を高速化する。

**代表的なDLL：**

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

> [!NOTE] 悪意のあるコードではロードタイム方式よりランタイム方式が多く使われる。悪意あるプログラムがメモリ領域間でファイルを転送する必要がある場合、ランタイム方式は単一のDLLを転送するだけで済み、ロードタイム方式のように複数の依存ファイルを必要としないため管理しやすい。

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

**主な確認事項：**

- **Entry Point**：実行可能バイナリの実行開始アドレス
- **Sections**：セクション数と各セクションの情報
- **PE詳細**：仮想アドレス・オフセット（ファイル内の特定位置を示す相対的な位置）

---

# Windows Internalsとのやり取り（Windows API）

## Windows APIとは

Windows OSと対話するためのネイティブ機能を提供するインターフェイス。64bit環境でもWin32 APIが使われる（Win64 APIという独立したAPIは存在しない）。詳細は→[Windows APIのIntro](https://claude.ai/chat/2.%20Windows%20API%E3%81%AEIntro.md)

## ユーザモードとカーネルモード

Windowsの内部コンポーネントの多くは物理的なハードウェアやメモリと対話する必要がある。このため、Windowsカーネルがすべてのプログラムとプロセスを制御し、ソフトウェアとハードウェアの相互作用を橋渡しする。

通常のアプリケーションはカーネルに直接アクセスできないため、**プロセッサモード**によってアクセスレベルが制御される。

||ユーザモード|カーネルモード|
|---|---|---|
|ハードウェアアクセス|直接アクセス不可|直接アクセス可能|
|アドレス空間|プロセスごとのプライベートな仮想アドレス空間|共有の仮想アドレス空間|
|メモリアクセス|「所有されたメモリ位置」のみ|物理メモリ全体|

ユーザモードで開始されたアプリケーションは、**システムコール**または**APIコール**が発生するまでそのモードに留まる。システムコールが発生するとカーネルモードに切り替わる。

C#などの言語はCLR（共通言語ランタイム）などのランタイムを経由してWin32 APIとやり取りするため、この流れはさらに複雑になる。

## 実装例：ローカルプロセスへのメッセージボックス注入

メモリとやり取りするProof-of-Conceptとして、ローカルプロセスのメモリ上にメッセージボックスを書き込んで実行する例を示す。

**手順：**

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

---

# まとめ

Windowsの内部はOSの動作の中核をなしており、その仕組みを変えずに攻撃者に悪用されることが多い。プロセス・スレッド・仮想メモリ・DLL・PE形式・Windows APIの理解は、攻撃手法の理解と防御の両面で不可欠。

詳細な悪用手法については→[Abusing Windows Internals](https://claude.ai/chat/3.%20Abusing%20Windows%20Internals.md)




---
---


---

##  仮想メモリ

- 仮想メモリによって、他の内部コンポーネントは、アプリケーション間の衝突のリスクなしに、物理メモリであるかのようにメモリと対話する
- 仮想メモリは、各プロセスにプライベートな仮想アドレス空間を提供する
- 仮想アドレスを物理アドレスに変換するために、メモリマネージャが使用される
- プライベートな仮想アドレス空間を持ち、物理メモリに直接書き込まないことで、プロセスが損害を被るリスクが少なくなる
- また、メモリマネージャは、ページや転送を使用してメモリを処理する。
- アプリケーションが、割り当てられた物理メモリよりも多くの仮想メモリ使用する場合、メモリマネージャは、この問題を解決するために、仮想メモリをディスクに転送またはページングする。
- この概念を下図に表す

![ 400](../../../画像ファイル/Pasted%20image%2020230618092948.png)

###### 32ビット

- 32ビットのx86システムでは、理論上の最大仮想アドレス空間は4 GB
- このアドレス空間は半分に分割されており、下位半分（0x00000000 - 0x7FFFFFFF）はプロセスに割り当てらる。
- 上位半分（0x80000000 - 0xFFFFFFFF）はOSのメモリ利用に割り当てられる。
- 管理者は設定（increaseUserVA）またはAWE（Address Windowing Extensions）を介して、より大きなアドレス空間を必要とするアプリケーションのためにこの割り当てレイアウトを変更することができる

###### 64ビット

- 64ビットの現代のシステムでは、理論上の最大仮想アドレス空間は256 TB
- 32ビットシステムから64ビットシステムへの正確なアドレスレイアウトの比率が割り当てられる。つまり。アドレスの配置パターンが互換性を持って維持される。
- 設定やAWEが必要な多くの問題は、理論上の最大値の増加により解決される。なぜなら、仮想アドレス空間が256 TBに増加するため、通常のアプリケーションのメモリ要件や性能要件を満たす。したがって、多くの場合、設定やAWEを使用する必要がなくなり、問題が解決される

###### 図解

- 下の図には、両方のアドレス空間割り当てレイアウトを視覚化した。
- この概念は直接的にWindowsの内部機能や概念には関係しないが、正しく理解すれば、Windowsの内部機能を悪用するのに役立てられる

 ![ 300](../../../画像ファイル/Pasted%20image%2020230618093936.png)

##### Procmon

- Operation列のLoad ImageのPropertyにベースAddressが記載

![ 200](../../../画像ファイル/Pasted%20image%2020230618095158.png)

---

## DLL

### DLLの基本的な知識

- DLL（dynamic link library）は、複数のプログラムで同時に使用できるコードとデータを含むライブラリ（[Microsoft docs](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library#:~:text=A%20DLL%20is%20a%20library,common%20dialog%20box%20related%20functions.)）
	- Unixで同じようなものは、[Shared Objects](https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/6n33n7f8u/index.html)と呼ばれる.
- DLLの目的は、コードのモジュール化、コードの再利用、効率的なメモリ使用、ディスクスペースの削減をサポートすること。
- そのため、OSとプログラムの読み込みと実行が高速化され、コンピュータ上のディスクスペース利用も少なくなる

- 例えば、==kernel32.dllやuser32.dllがある==
- `kernel32.dll`は、WindowsOSにおいてカーネルレベルの機能を提供するDLL。Windows APIの一部であり、多くの重要なシステム関数やリソースへのアクセスを提供する
- `user32.dll`はWindows APIの一部であり、ウィンドウの作成、描画、イベント処理など、ユーザーインタラクションに関連する機能を提供

- プログラム内で関数としてDLLがロードされると、そのDLLは依存関係として割り当てられる。つまり、プログラムが正常に動作するためには、対応するDLLが存在し、正しくロードされている必要がある
- プログラムがDLLに依存しているため、==攻撃者はアプリケーションではなくDLLを標的にして、実行や機能の一部を制御することができる==
- 攻撃例：
	- DLLハイジャッキング（[T1574.001](https://attack.mitre.org/techniques/T1574/001/))
	- DLLサイドローディング（[T1574.002](https://attack.mitre.org/techniques/T1574/002/)）
	- DLLインジェクション（[T1055.001](https://attack.mitre.org/techniques/T1055/001/)） 

### DLL search orderとDLL Hijackingについて

- DLL search orderとはWindowsがDLLを読み込むときの探索順序を定めたルールのこと
- アプリが`LoadLibrary()`などでDLLをロードする際、ファイルパスをフルで指定しない場合、この順序で探す
- DLLハイジャッキング（[T1574.001](https://attack.mitre.org/techniques/T1574/001/))で悪用される仕組み
	1. ターゲットアプリが `abc.dll` をロードしようとするが、存在しない
	2. 攻撃者が同名の悪意ある`abc.dll`を、アプリの実行ディレクトリなど早い探索位置に置く
	3. アプリ起動時に攻撃者DLLが先に読み込まれる

#### Safe DLL Search Modeの探索順序

このモードがONだと、カレントディレクトリが５番目に探索される
（OFFだと２番目に探索される）

1. アプリケーションの実行ディレクトリ
    - 実行ファイル（.exe）が置かれているフォルダ
2. システムディレクトリ
    - 通常は `C:\Windows\System32`
3. 16-bitシステムディレクトリ
    - `C:\Windows\System`（ほぼ互換性目的で存在）
4. Windowsディレクトリ
    - 通常は `C:\Windows`
5. カレントディレクトリ
	- 実行ファイルを実行しているディレクトリ（危険なので後方に移動されている）
	- サービスの場合はシステムディレクトリ
6. System PATH環境変数に記載されたディレクトリ
    - `PATH`に並ぶフォルダを順に探索
```powershell
# System PATH環境変数の確認
[Environment]::GetEnvironmentVariable("Path", "Machine")
```

#### ⚠️探索順序の例外

1. KnownDLLs（既知のDLL）
	- Windowsには、セキュリティのために「探索順序を無視して強制的に `System32` から読み込むDLL」のリスト（KnownDLLs）が存在する
	- これらは、偽物を置いたとしても意味がない
	- 例： `kernel32.dll`, `user32.dll`, `ole32.dll` など

2. フルパスでのロード
	- もしアプリケーションのコード内で `LoadLibrary("C:\Windows\System32\example.dll")` のようにフルパスで指定されている場合は、探索順序は無視され、指定された場所のものが読み込まれる

### load-time・ダイナミック・リンク

- アプリケーションからDLLの関数を明示的に呼び出す。
- このタイプのリンクは、ヘッダー (.h) とインポートライブラリ (.lib) ファイルを提供することによってのみ実現できる。
- 以下は、アプリケーションからエクスポートされたDLL関数を呼び出す例
```cpp
#include "stdafx.h"
#include "sampleDLL.h"
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
    HelloWorld();
    return 0;
}
```

### run-time・ダイナミックリンク

- 別の関数（`LoadLibrary`または`LoadLibraryEx`）を使用して、実行中にDLLをロードする。読み込んだら、`GetProcAddress`を使用して、呼び出すエクスポートされたDLL関数を特定する必要がある。
- 以下は、アプリケーションでDLL関数をロードしてインポートする例
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
...
```

### 攻撃者の視点

- 悪意のあるコードでは、脅威者はロードタイム・ダイナミック・リンクよりもランタイム・ダイナミック・リンクを使用することが多い。
- これは、悪意のあるプログラムがメモリ領域間でファイルを転送する必要がある場合があり、単一のDLLを転送する方が、他のファイル要件を複数使用してインポートするload-timeよりも管理しやすいから。つまり、run-timeだと単一の要件を満たせばいい。


---

## Portable Executable Format

- 実行ファイルとアプリケーションは、Windowsの内部がより高いレベルで動作するための大きな部分を占めている。
- PE（Portable Executable）フォーマットは、実行ファイルと格納されたデータに関する情報を定義する。データ構成要素がどのように格納されるかという構造も定義している
- PEフォーマットは実行ファイルやオブジェクトファイルのための包括的な構造。
- PEファイルとCOFF（Common Object File Format）ファイルは、PEフォーマットを構成している
- PEデータは、実行ファイルの16進ダンプで見るのが最も一般的。
- 以下では、calc.exeの16進ダンプをPEデータのセクションに分解して説明する
  
- PEデータの構造は、7つの構成要素に分かれている
 
 ![ 300](../../../画像ファイル/Pasted%20image%2020230618104414.png)
  
###### DOSヘッダ

- ファイルの種類を定義
- `MZ` DOSヘッダでは、ファイル形式を`.exe`と定義。DOSヘッダーは、以下の16進ダンプの項目で確認できる
![ 300](../../../画像ファイル/Pasted%20image%2020230618104455.png)

###### DOS stub

- ファイルの冒頭でデフォルトで実行されるプログラムで、互換性メッセージを表示する。これは、ほとんどのユーザーにとって、ファイルの機能に影響を与えるものではない。
- DOSスタブは、`This program cannot be run in DOS mode`というメッセージを表示する。以下の16進ダンプのセクションで見ることができる
![ 400](../../../画像ファイル/Pasted%20image%2020230618104640.png)

###### PE File Header

- PE File Headerは、バイナリのPEヘッダー情報を提供。
- ファイルのフォーマットを定義し、署名と画像ファイルヘッダ、およびその他の情報ヘッダを含む
- PEファイルヘッダは、人間が読むことのできる出力が最も少ないセクション。PEファイルヘッダの開始は、以下の16進ダンプセクションの`PE`スタブから特定できる
![ 300](../../../画像ファイル/Pasted%20image%2020230618104839.png)

######  Image Optional Header

- PE File Headerの重要な部分である  
- Data Dictionaryは、Image Optional Headerの一部。
- これらは、画像データのディレクトリ構造を指す
###### Section Table

- セクションテーブルは、イメージの中で利用可能なセクションと情報を定義する。
- セクションはコード、インポート、データなど、ファイルの内容を保存する。以下の16進ダンプのセクションで、テーブルから各セクションの定義を確認することができる

![ 300](../../../画像ファイル/Pasted%20image%2020230618105156.png)

- 上の図にあるように、Header (PE format)でファイルの形式や機能を定義した後は、Sections (contents)でファイルの内容やデータを定義することができる

|Section|Purpose|
|---|---|
|.text|実行コードとエントリポイントを含む|
|.data|初期化されたデータ（文字列、変数など）を含む|
|.rdata or .idata|imports（Windows API）とDLLを含む|
|.reloc|リロケーション情報を含む|
|.rsrc|アプリケーションリソースを含む (images, etc.)|
|.debug|デバッグ情報を含む|

##### Ditect It Easy (die)

- [DIE](https://github.com/horsicq/Detect-It-Easy)はバイナリファイルの解析やファイルの種類の識別に使用されるツール

![](../../../画像ファイル/Pasted%20image%2020230618111621.png)

- 右上の`...`からファイルを開く。ここでは`notepad.exe`を使用
- Entry pointでエントリーポイントがわかる。==実行可能なバイナリファイル（通常はWindowsで使用される実行可能ファイル）の実行が開始される場所を指す==
- Sectionsでセクションナンバーがわかる
- `PE`を開くと左側にタブがたくさん表示され、仮想アドレスやOffset(ファイルやデータの特定の位置を示す相対的な位置)がわかる。

---

## Interacting with Windows Internals

- WindowsのInternalと対話するための最もアクセスしやすい選択肢は、Windows APIコールを介してインターフェイスすること。
- Windows APIは、Windows OSと対話するためのネイティブ機能(特定のplatformに固有の機能)を提供する。このAPIには、Win32 APIと、一般的ではないWin64 APIがある
- ここでは簡単な概要を説明するが、詳細は[2. Windows APIのIntro](2.%20Windows%20APIのIntro.md)

- Windowsの内部コンポーネントの多くは、物理的なハードウェアやメモリと対話する必要がある。  
- Windowsカーネルは、すべてのプログラムとプロセスを制御し、すべてのソフトウェアとハードウェアの相互作用を橋渡しする。多くのWindows内部コンポーネントが何らかの形でメモリとの相互作用を必要とするため、windowsカーネルは特に重要
- 通常、デフォルトのアプリケーションはカーネルと対話したり、物理的なハードウェアを変更することができないため、インターフェイスが必要。
- この問題は、プロセッサモードとアクセスレベルの使用によって解決される
  
- Windowsプロセッサには、ユーザモードとカーネルモードがある。プロセッサは、アクセスや要求されたモードに応じて、これらのモードを切り替える
- ユーザモードとカーネルモードの切り替えは、多くの場合、システムコールやAPIコールによって行われる。文書では、このポイントを "Switching Point" と呼ぶことがある

|User mode|Kernel Mode|
|---|---|
|直接的なハードウェアアクセスはできない|直接的なハードウェアアクセスが可能|
|プライベートな仮想アドレス空間でプロセスを作成|共有の仮想アドレス空間で実行|
|「所有されたメモリ位置」へのアクセス|物理メモリ全体へのアクセス|

- User modeまたは"Userland"で開始されたアプリケーションは、System Callが発生するかAPIを介してインターフェースされるまで、そのモードに留まる。システムコールが発生すると、アプリケーションはモードを切り替える。
- 下の図は、このプロセスを説明するフローチャート

![ 300](../../../画像ファイル/Pasted%20image%2020230618114052.png)

- 言語がWin32 APIとやり取りする方法を見ると、このプロセスはさらに複雑になる。
- アプリケーションは、Win32 APIとのやり取りする前に言語ランタイムを経由する。最も一般的な例は、C#がCLR(コンパイルなどする)を介して実行され、Win32 APIとシステムコールを行うこと

###### ローカルプロセスにメッセージボックスを注入する例

- メモリとやり取りするProof-of-Conceptを示す

- メモリにメッセージボックスを書き込むための手順は以下の通り

	1. メッセージボックスのためにローカルプロセスのメモリを確保する。
	2. 確保されたメモリにメッセージボックスを書き込む/コピーする。
	3. ローカルプロセスのメモリからメッセージボックスを実行する。

1. `OpenProcess`を使って、指定されたプロセスのハンドルを取得
```cpp
HANDLE hProcess = OpenProcess(
	PROCESS_ALL_ACCESS, // Defines access rights
	FALSE, // Target handle will not be inhereted
	DWORD(atoi(argv[1])) // Local process supplied by command-line arguments 
);
```
2. `VirtualAllocEx`を使ってペイロードバッファのあるメモリ領域を確保する
```cpp
remoteBuffer = VirtualAllocEx(
	hProcess, // Opened target process
	NULL, 
	sizeof payload, // Region size of memory allocation
	(MEM_RESERVE | MEM_COMMIT), // Reserves and commits pages
	PAGE_EXECUTE_READWRITE // Enables execution and read/write access to the commited pages
);
```
3. `WriteProcessMemory`を使用して、割り当てられたメモリの領域にペイロードを書き込む
```cpp
WriteProcessMemory(
	hProcess, // Opened target process
	remoteBuffer, // Allocated memory region
	payload, // Data to write
	sizeof payload, // byte size of data
	NULL
);
```
4. `CreateRemoteThread`を使用して、メモリからペイロードを実行する
```cpp
remoteThread = CreateRemoteThread(
	hProcess, // Opened target process
	NULL, 
	0, // Default size of the stack
	(LPTHREAD_START_ROUTINE)remoteBuffer, // Pointer to the starting address of the thread
	NULL, 
	0, // Ran immediately after creation
	NULL
); 
```

---

## Conclusion

- Windowsの内部は、WindowsOSがどのように動作するかの中核をなすもの。
- これらの内部は、OSが基本的にどのように動作するかを損なうことなく変更することはできない。このため、Windowsの内部は攻撃者にとって格好のターゲット。システムへの不正アクセス、データの盗み出し、破壊的な変更などを行うことができるので。
- 攻撃者はWindows内部コンポーネントの機能を簡単に悪用し、悪意ある理由で利用することができる。詳細は[3. Abusing Windows Internals](3.%20Abusing%20Windows%20Internals.md)