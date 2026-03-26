- OSCPにおいてはImmunity debuggerを使わなければならない
- 試験では動作確認のためにWindows BOXが提供され、その中にImmunity debuggerがインストールされているし、Windows Boxはインターネット接続できないから
	- 他のコースではwindbgが使われるが...
- 参考🔗：[OSCPではなぜImmunity debuggerを使うのか - Reddit](https://www.reddit.com/r/oscp/comments/b8ya99/deleted_by_user/)

---

# 基本情報

## Immunity debuggerとは

- Windowsアプリケーションのデバッグや解析に使われるツール
- エクスプロイトの開発などに用いられる
	- マルウェア解析には使いにくいが、**BoF**には十分使える
- Windows
- 32bitアプリケーションのみ対応のデバッガ
	- ※ OSの32bit アーキテクチャとは全く関係ない
- 公式リポジトリからインストールする（[Immunity Debugger](https://github.com/kbandla/ImmunityDebugger/releases/tag/1.85)）

> [!WARNING]
> エクスプロイト開発に用いるこのようなツールは、ホストOSにインストールしてはならない。
> なぜなら、エクスプロイト開発中に自身のシステムがエクスプロイトされる恐れがあるから。

---

# 開き方

1. Admin権限で動作しているプロセスなどを選択できるようにするため、Run as administratorで開く
2. File -> Attach -> 分析対象のプロセスを選択 -> Attach、で以下のような画面が開く

![](../Images/Pasted%20image%2020250609065800.png)

$$Attach後のImmunityDebugger$$

- ①プロセスのアセンブリコードを表示するウィンドウ
- ②各種レジスタを表示するウィンドウ
- ③メモリの16進ダンプを表示するウィンドウ
- ④スタックを表示するウィンドウ
- ⑤コマンドバー

- 上部タブのさまざまなボタンのアクション内容は、下部に記載（画像赤枠同士対応）

![](../Images/Pasted%20image%2020250621090653.png)

$$アクションの内容が表示される様子$$

---

# モジュールの探し方

1. 上部タブView -> Executable modulesをクリック
2. Name列に実行可能ファイルやDLL等のPEフォーマットが記載されている
	- PEフォーマットとは：[Portable Executable Format（PE形式）](../Cheatsheet/🪟Windows/Windowsの基礎・前提知識/Windows%20Internal.md#Portable%20Executable%20Format（PE形式）)
3. Name列から目的のモジュールを探す

![](../Images/Pasted%20image%2020250609073042.png)

$$View->Executable　modules画面$$

4. 該当のモジュールをダブルクリックすると、モジュールのアセンブラ等が表示される

---

# セキュリティ機構が無効なモジュールの探し方とアドレス検索方法

1. ⑤のコマンドバーに以下のコマンドを入力
```python
!mona modules
```

![](../Images/Pasted%20image%2020250609065800.png)

$$Attach後のImmunityDebugger$$

2. ASLRなどのセキュリティ保護機能が有効（True）か無効（False）かが表示される
	- [!mona modulesの結果解説](#!mona%20modulesの結果解説)

![](../Images/Pasted%20image%2020250615092757.png)

$$ASLRなどのセキュリティ保護機構の有無表示$$

3. [モジュールの探し方](#モジュールの探し方)で該当のモジュールを開く

4. Ctrl + Fで任意の命令（多くの場合JMP ESP）を検索する

![](../Images/Pasted%20image%2020250615101645.png)

$$Find　Command$$

↓

![](../Images/Pasted%20image%2020250615101717.png)

$$該当する行がハイライトされる$$

### !mona modulesの結果解説

- すべてFalseでない場合はエクスプロイトが少し難しい
	- ASLR、Rebase、OS Dllについては、Falseであることが事実上*必須*

| 項目                          | 意味・役割                                                                                                                  | False の意味                           |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| *Rebase*                    | DLLがロード時にアドレスの変更を許可するか（≒ランダム配置を許す）                                                                                     | 固定アドレスでロードされる                       |
| SafeSEH                     | ・Structured Exception Handler保護が有効か（例外処理チェインを保護）<br>・例外ハンドラのリストに任意のコードを注入して実行させないようにする                                 | ・任意のSEHオーバーライドが可能                   |
| *ASLR*                      | ・Address Space Layout Randomization（アドレス空間のランダム化）<br>・有効だと、プログラム実行ごとに、ライブラリやヒープ領域の先頭アドレス、スタック領域の先頭アドレス（スタックポインタ）が毎度変わる | ・毎回同じアドレスでロード<br>・JMP ESPなどのアドレスが固定 |
| NXCompat                    | ・実行防止領域（Data Execution Prevention, DEP）に対応しているか<br>・DEP：実行命令は静的領域のテキスト領域にのみあるべきだから、ヒープやスタックの実行命令は実行しない                 | スタックなどでのコード実行が可能                    |
| *OS Dll*（＝system DLL）       | ・OS標準のDLLか（例：kernel32.dll など）<br>・OS標準の場合、起動毎にアドレスが変わる                                                                 | 独自/サードパーティ製のDLL。                    |
| Version / Modulename / Path | DLLのバージョン・モジュール名・実ファイルパス                                                                                               | -                                   |

---

# 検索

- Ctrl+G→Enter expression to follow：アドレス、アドレス式、シンボル名で検索
	- 存在すれば確実にヒット
	- アドレスはCPUに関わらずビッグエンディアンで検索

- Ctrl+F→ Find command：アセンブラで検索
	- 存在してもヒットしないことがある

>[!Warning]
>- 目的の命令とは限らない
>	- 例えば、JMP ESP命令が複数あり、利用できる命令Aを見つけたいのに、利用できない命令Bもヒットする可能性がある
>- 現在メモリ上にロードされていないモジュールはヒットしない
>	- 例えば、JMP ESPと検索しても見つからないが、Ctrl+Gでアドレスを検索するとメモリ上に仮でモジュールがロードされ、Ctrl+Fでも検索ヒットするようになる
>	- スクロールするとヒットした命令が消えることがあるが、これはモジュールがまだ読み込まれていないから

---

# ブレークポイントの使い方・メモリ破壊後のリスタート

1. f2（ファンクションキー）を押して設定（macはfnボタンを押しながらTouch barでf2）

![](../Images/Pasted%20image%2020250620074953.png)

$$ブレークポイント設定後の状態$$

2. 別の環境（linuxなど）からメモリ破壊のエクスプロイトを実行

3. Immunity debuggerに戻り、▶️（再生）ボタンを押して挙動を確認

- メモリ破壊後は⏪ボタンでRestart