
💡H2脆弱性の一種であるCRLF注入テクニックの悪用は一番最後に取り組む
🚨: Param Minerは永遠に終わらない！リクエストキャプチャ時にParam Minerのリクエストをキャプチャしてしまう。初回はこれで落ちた

## Smugglerを使用する場合

### HTTP /1.1 × Smuggler

###### 基本の流れ

1. Issues -> Request 1をRepeaterへ
2. Issuedで報告された脆弱性をSmuggle attackで選択する
	- CL.TE等
	- TE.TEの場合は、TE.CLを選択

3. 開かれたTurbo Intruderの`prefix`を目的のアクションに変更してAttack
```http
prefix = '''GET /アクセス先。admin等 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x='''
```

#### フロントエンドのセキュリティのバイパス

- [[4. HRS脆弱性の悪用#フロントエンドのセキュリティのバイパス]]

#### & Reflected XSS

- [[4. HRS脆弱性の悪用#HRS × 反射型XSS悪用ステップ]]

---
### HTTP/2 × Smuggler

- ==⚠️==: [[1. HTTP2 request smuggling#HTTP/2 HRS 設定]]を事前にしておく
#### H2.TE脆弱性

- 密輸するリクエスト(prefix)はHTTPは1.1で良いが、カバーするリクエストはHTTP/2にchange attributeしておくこと
- Smuggle attack(H2.TE)を選択

---
## Smugglerを使用しない場合

### HTTP/1.1

🚨[[2. 基本的なHRSの実行#手動実行時のBurp Suiteの注意点]]

#### フロントエンドによる書き換えのキャプチャ

- [[4. HRS脆弱性の悪用#フロントエンドによる書き換えを明らかにするステップ]]
- ==⚠️==書き換え内容を確認したらSmugglerで悪用すること

#### 被害者ユーザのリクエストキャプチャ

- [[4. HRS脆弱性の悪用#リクエストキャプチャのステップ]]

#### CL.0

- [[CL.0 request smuggling#CL.0のエクスプロイト]]
- (CL.0はHTTP/2でも悪用可能だが、LABにはない)

---
### HTTP/2

- ==⚠️==：[[1. HTTP2 request smuggling#HTTP/2 HRS 設定]]を事前にしておく

#### H2.CLによるXSSの実行

- [[1. HTTP2 request smuggling#H2.CL悪用ステップ]]のステップ2~

#### CRLF注入による被害者ユーザのリクエストキャプチャ

H2.TEの場合のCRLF注入方法はこれを参考にする
- [[2. HTTP2専用のベクターとCRLF注入によるHRS#CRLF注入によるHRS]]

#### Response queue poisoning

- 被害者のリクエストに対するレスポンスを窃取
- (HTTP/1.1でも悪用可能)

###### & H2.TE

- [[3.  Response queue poisoning#Response queue poisoningステップ]]

###### & CRLF注入

H2.CLの場合のCRLF注入方法はこれを参考にする
- [[4. HTTP2リクエスト分割#CRLF注入によるHTTP/2リクエスト分割でのResponse queue poisoning]]