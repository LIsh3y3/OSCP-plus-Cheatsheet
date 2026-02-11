[[⚡️GraphQL API vuln]]

#### 注目ポイント

- GraphQLが実行されている
	- HTTP Historyに`graphql`と記載がある等

- [[🔍CSRF#CSRFかもしれない条件]]が見られる

---
##### GraphQLをBurpで使用する際の注意点

GraphQLタブがないときは、以下のようにすることでGraphQLタブから編集できるようになる
- GET
```http
?query=
```

- POST✖︎ `application/x-www-form-urlencoded`。body部に
```http
query=
```

---
## Detect

- [[🔍 Recon#2. 隠されたエンドポイントの検出]]で"Query not present"と検出されたエンドポイントがあるか？
	 or
- IssueのInformationに"GraphQL endpoint found"が表示されるか？

---
## スキーマの情報取得

1. GraphQLエンドポイントを検出 -> Extensionsの*InQL*をloadする
2.  GraphQL endpointへのURLを入力(HTTP HisotryのCopy URLから) -> Analyze
![[Pasted image 20240214125411.png]]

3. 以下のように使用可能なクエリやミューテーション、スキーマが表示される
![[Pasted image 20240214125526.png | 300]]

4. `shema.json`のテキストをコピーし、[GraphQL voyager](https://graphql-kit.com/graphql-voyager/)でビジュアライズする
	- 使い方は[[3. GraphQLスキーマの情報窃取と防御バイパス#⭐️③ [GraphQL voyager](https //graphql-kit.com/graphql-voyager/)を使って、結果をわかりやすくする]]

#### スキーマ取得妨害バイパス

InQLでAnalyzeすると以下のようにエラーが起きる場合
![[Pasted image 20240214150426.png | 300]]

1. Query not presentと表示されるエンドポイントに対してUniversalクエリを投げる

- POSTの場合body部に
```http
query{__typename}
```
- GETの場合クエリパラメタに(必要であればURLエンコード)
```http
?query=query{__typename}
```

- 以下のレスポンスが返ればOK
```
{
  "data": {
    "__typename": "query"
  }
}
```

2. RepeaterのRawタブでGraphQLリクエストを右クリック->GraphQL -> Set Introspection query
3. GraphQLタブに移動し、 `__schema`の後ろに以下の文字のいずれかを注入してリクエスト
	- スペース
	- 改行
	- コンマ(`,`)

4. 入手したschemaをビジュアライズする
5. schemaをファイルに保存してInQLの"2. (Optional)~"に貼り付ける
![[Pasted image 20240214152507.png | 500]]

- [Lab: Finding a hidden GraphQL endpoint](https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint)

###### (ないとは思うが)これでもうまくいかないとき

- [[2. GraphQL APIの脆弱性とエンドポイントの見つけ方#2. Repeaterの"Change request method"で以下の順に遷移するので、それぞれのリクエストメソッドを試す]]