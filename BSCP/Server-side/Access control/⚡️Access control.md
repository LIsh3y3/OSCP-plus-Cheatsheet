###### BurpにおいてのAccess control便利機能

- 詳細：[Using Burp's "Request in Browser" Function to Test for Access Control Issues](https://portswigger.net/support/using-burp-suites-request-in-browser-function-to-test-for-access-control-issues)
- Repeaterでリクエストを右クリック-> Request in Browser
	- -> In original session(これ使う)
	- -> In current browser session

- これにより、アクセス制御のセキュリティテストにおいて、アプリのアクセス制御が異なるユーザアカウントレベルで適切に機能するかどうかを確認できる

### パラメタを操作しPrivEsc可能か確認する

- もし`roleid`を発見して0~10に変更してもうまくいかなければ、0~100でbrute-forceする

---
### プラットフォームの誤設定によるアクセス制御の不具合を悪用する

- 詳細：[[2. Privilege Escalationの基本#プラットフォームの誤設定によるアクセス制御の不具合]]

###### URL-based access control bypass

- [[🔍Access control#URL上書きヘッダをサポートしているか？]]でわかったヘッダでURLを上書きしてアクセスする
```http
GET / HTTP/2
...
X-Original-URL: /admin
```

###### Method-based access control bypass

- [[🔍Access control#他のリクエストメソッドが使用できないか]]
- 利用可能なメソッドでリクエストしてみる

---
### multi-stepプロセスにおけるアクセス制御の脆弱性

- [[4. 様々なアクセス制御の脆弱性#multi-stepプロセスにおけるアクセス制御の脆弱性]]

---
### Refererヘッダによるアクセス制御バイパス

- [[4. 様々なアクセス制御の脆弱性#Refererヘッダによるアクセス制御バイパス]]

---
### リクエストパラメタでのIDOR悪用例

#### 固有の値で識別している場合

- 👉 victim(DBやユーザ等オブジェクト）の値に変更
- 例
	- `my-account?id=wiener` -> `my-account?id=administrator`
	- `GET /download-transcript/77.txt`  -> `GET /download-transcript/1.txt`

#### 固有の値で識別 & ランダムな値で識別 

- 👉victimのID等が流出している箇所を探す

#### 固有の値で識別& 非許可のvictimオブジェクト参照でリダイレクト

- 👉 victimユーザの値に変更し、リダイレクトレスポンスのボディ部に重要情報がないか探す