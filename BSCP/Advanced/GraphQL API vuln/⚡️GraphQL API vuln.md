
- [[#Passwordの抽出]]
- [[#ミューテーションの実行]]
- [[#brute-force防御バイパス]]
- [[#& CSRF]]

---
### Passwordの抽出

1. GraphQL voyagerでユーザ情報を取得するクエリを発見する(`getUser`)
2. InQLから該当クエリを右クリック-> Batch attack with InQL AttackerでInQLのAttackerへ送る(TargetにはGraphQLエンドポイントまでのURLを入力)
3. Supported placeholdersを使用して有効な`id`を割り出すクエリを作成してSend
```graphQL
query {
    getUser(id:  $[INT:1:100]) {
        id
        password
        username
    }
}
```
- ==⚠️==: InQLのAttackerが使用できなければRepeaterのGraphQLタブから編集する

- [LAB: Accidental exposure of private GraphQL fields](https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure)

---
#### ミューテーションの実行

- ミューテーションでデータの操作が可能(PUT, DELETE, POST)
- 操作対象のデータを引数に指定する。少しややこしい

- 具体例: [[3. GraphQLスキーマの情報窃取と防御バイパス#mutationの具体例]]

---
### brute-force防御バイパス

- [[5. Aliaseを使用した総当たり攻撃に対するrate制限のバイパス]]

---
### & CSRF

[[6. GraphQL CSRF]]