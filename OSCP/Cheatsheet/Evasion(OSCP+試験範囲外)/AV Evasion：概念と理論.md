> [!NOTE] 
> AV回避は**OSCP試験において主眼ではない**が、現実のテストやOSEPでは非常に重要なテクニック。

- 関連ノート：
    - [PE構造とシェルコード](PE構造とシェルコード.md)
    - [AV Evasionのテクニック](AV%20Evasionのテクニック.md)
    - [Web攻撃の難読化](Web攻撃の難読化.md)
- 参考🔗：[HackTricks - AV Bypass](https://book.hacktricks.wiki/en/windows-hardening/av-bypass.html)
- 画像ソース：[tryhackme](https://tryhackme.com/)

---

# AVのDetection Technique と回避策

| 手法            | 説明                    | 回避策               |
| ------------- | --------------------- | ----------------- |
| 静的シグネチャ分析     | 既知のバイト列と照合するブラックリスト方式 | コード暗号化・バイナリ改変で回避可 |
| 静的ヒューリスティック分析 | よくある悪意あるコードのパターンを検出   | スタブに怪しい処理を含めない    |
| 動的解析          | サンドボックス環境で実行し、挙動を観察   | スタブを実行させないように騙す   |

- 参考：🔗[Bypass Antivirus Dynamic Analysis](https://web.archive.org/web/20210317102554/https://wikileaks.org/ciav7p1/cms/files/BypassAVDynamics.pdf)

---

# On-Disk Evasion vs In-Memory Evasion

| 比較項目    | On-Disk Evasion               | In-Memory Evasion            |
| ------- | ----------------------------- | ---------------------------- |
| ファイルの扱い | ファイルを難読化・暗号化して**ディスク上に保存**    | ファイルを一切書き込まず**メモリ上で展開・実行**   |
| 主な手段    | パッカー・エンコーダ・スタブ・署名改変           | メモリ空間の操作・プロセス注入              |
| 対策されやすさ | 静的検知（シグネチャ）やヒューリスティックで検出されやすい | メモリスキャンが必要なため検出が困難           |
| AVの防御対象 | ファイルシステム（on-access scan）      | 実行中のプロセスやメモリ空間（runtime scan） |

> [!TIP]
> 最近のEvasionテクニックはIn-Memory Evasionが主流。

## In-Memory Evasionの主な手法

| 技術                       | 説明                                                                                             |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| Remote Process Injection | `OpenProcess` → `VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread` の流れでペイロードを挿入・実行 |
| Reflective DLL Injection | `LoadLibrary` を使わず独自コードでDLLをメモリ上に読み込む（DLLをファイルとして書き出さない）                                       |
| Process Hollowing        | 無害なプロセスを一時停止状態で起動 → メモリ上の実行イメージを削除 → 悪意あるコードを上書き → プロセス再開                                      |
| Inline Hooking           | 正規の関数にジャンプ命令を挿入し任意の悪意コードへリダイレクト。その後元関数に戻るため正規コードに見せかけた実行が可能                                    |

**Rootkitsへの応用：** Hooking技術でシステムコールやカーネル関数を改ざん。権限昇格後にインストールされるケースが多く、ユーザーモードからハイパーバイザー領域までフック可能。

PowerShellを使ったIn-Memory InjectionはWindows APIをラップしており、より実装が簡単でAVバイパスに有効。上記の多くはファイルレス攻撃（Fileless Attack）とも呼ばれる。

---

# Staged vs Stageless Payloads

| 項目         | Staged                                         | Stageless                       |
| ---------- | ---------------------------------------------- | ------------------------------- |
| 構成         | 小さなstagerと本体の2段階構成                             | 単一ファイルに全コードを含む                  |
| 接続方式       | stagerがリスナーへ接続後、本体をダウンロード・実行                   | 即座にリバースシェルなどを実行                 |
| 検知         | ディスク上の痕跡が少なくブルーチームはstagerしかキャプチャできない           | FWやIPS充実時に最初の一撃で完結できる           |
| 通信要件       | stagerが再接続・ライブラリロードが必要なため低帯域・遅延に弱い             | 一度で接続が完結するため遅延環境に強い             |
| 検出率        | 本体はメモリ上にロードされるため検出されにくい。ただしAMSIなどで検出されやすくなっている | サイズが大きく静的解析で検出されやすい             |
| msfvenom命名 | `shell/reverse_tcp`（スラッシュ区切り）                  | `shell_reverse_tcp`（アンダースコア区切り） |

- 関連：[☠️Msfvenom](../../Tools/☠️Msfvenom.md)


Staged
![](../../Images/Pasted%20image%2020230620101919.png)
![](../../Images/Pasted%20image%2020230620102134.png)

$$Staged　Payloadの流れ$$

Stageless
![](../../Images/Pasted%20image%2020230620101358.png)
$$Stageless　payloadの流れ$$

---

# エンコーディングと暗号化

|項目|エンコーディング|暗号化|
|---|---|---|
|主な目的|可読性・互換性・保存性の向上|機密性の確保|
|可逆性|あり（簡単に元に戻せる）|あり（鍵が必要）|
|セキュリティ性|低い（回避技術としては補助的）|高い（鍵がなければ復号できない）|
|主な使用例|ファイル保存・通信・データ処理・難読化|セキュア通信・データ保護・機密情報の送受信|

**なぜ必要か：** AVは公開ツール（Metasploitなど）で生成されたシェルコードを静的・動的解析で高確率で検出する。エンコードや暗号化でシェルコードを難読化することで検出を回避できる。関数名・変数名の難読化にも応用できる。

単純なエンコードはAVにデコードされて検出される可能性がある。**複数のアルゴリズムを組み合わせる**（例：XOR + Base64）ことで検出を困難にできる。

- 参考：[TryHackMe - Encryption: Crypto 101](https://tryhackme.com/room/encryptioncrypto101)