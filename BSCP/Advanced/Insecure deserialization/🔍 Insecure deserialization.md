[[⚡️Insecure deserialization]]

⭐️必ずExtension: Java Deserialization Scannerをloadしておくこと。loadしている状態でJavaオブジェクトをScanするとライブラリ、エンコーディング方式などすべてわかる(信頼性低いが)

#### 注目ポイント

- ログイン後のCookie
- アプリ特有の機能：LABだとDelete accountだが、試験ではsecretを読み出す必要があるので、異なる機能になると思う

---
## Detect

###### シリアライズされたオブジェクトの検出

- ログイン後、Live auditで自動的にシリアライズされたオブジェクトを検出できる
	- Issue: Serialized object in HTTP message

- Issue detail: 言語(PHP, Java, Ruby, Python)のobjectと表示

###### 🚨 Issueに検出されないケース

- デコードしたオブジェクトの中身に、さらにまだエンコードされているデータがある場合がある。
 ![[Pasted image 20240113171335.png | 200]]
 - この場合、エンコードされている値だけ別途デコードする必要がある


- もしくはGzipなど特殊な形式でエンコーディングされている場合は検出できない
	- -> オブジェクト候補をIntruderで選択した後Scannerにかけて検出できる

---
#### ライブラリの判別

- Issueに表示される。(探索して2分くらい経つと表示されることが多い)
- または怪しいCookieをIntruderで選択してScan。ライブラリ・言語・エンコーディング方式がわかる(==⚠️==: Java Deserialization Sccanerによって検出された場合はライブラリの番号の信頼性は低い)
![[Pasted image 20240213100450.png]]

###### Java

- Java Deserialization sccanerを使用(ライブラリの番号の信頼性は低い)
- ソースコードを閲覧 -> [[#ソースコードや設定ファイルの検査]]

###### PHP &  Rubyなどその他言語

- シリアライズドオブジェクトを適当な値(`aaa`等)に変更し、Internal Server errorを引き起こすことで、レスポンスにどのフレームワークを使用しているかが表示される
	or
- ソースコードを閲覧 -> [[#ソースコードや設定ファイルの検査]]

---
#### ソースコードや設定ファイルの検査

Engagement tool -> Find Commentsを使用してエンドポイントを見つけられるか？
```
TODO: Refactor once /libs/CustomTemplate.php is updated
```
-> NO↓
	Java: [[⚡️Insecure deserialization#Java]]
	 PHP:[[⚡️Insecure deserialization#データ型操作によるアカウント窃取(Stage 2)]]
	 PHP: [[⚡️Insecure deserialization#アプリケーションの機能を利用する]]
	 Ruby: [[⚡️Insecure deserialization#Ruby]]
	 

->Yes: Gadgetを発見する

- 通常通りアクセス
	 or
- `~`(チルダ)を末尾に付与してアクセス
```
https://TARGET_NET/libs/CustomTemplate.php~
```

-> [[⚡️Insecure deserialization#任意のオブジェクトを注入する]]
	or
-> [[⚡️Insecure deserialization#PHPGGCを使用する]]

