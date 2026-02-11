[[⚡️Access control]]

#### 注目ポイント

- サーバーのレスポンスを見て情報を確認するしかない脆弱性。以下はあくまでも参考程度に！
- select role機能: [Lab: Authentication bypass via flawed state machine](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-flawed-state-machine)
- `/admin`ページ：Error: "Admin interface only available if logged in as an administrator"
- Change email機能(`roleid`等)
- Live chat機能のView transcriptでファイルダウンロードするとき(XSSを検知することもあるが多分違う)

---
## Detect

#### IDOR & PrivEsc: Access control用パラメタはないか？

APPERENTICE

- hidden field
- Cookie
- クエリストリングパラメタ(i.e.`http://TARGET_NET/?admin=true`)
-  blogの著者に被害者ID
- JSファイルにURLが記載(Targetに載っている)
- 具体例：
	- `roleid`等
	- Live chat機能で`[num].txt`形式のファイルをダウンロードする場合
-> [[⚡️Access control#リクエストパラメタでのIDOR悪用例]]

### PrivEsc

前提条件：administratorとしてアクセスしたい機能のURLがわかってるか？
###### A. URL上書きヘッダをサポートしているか？

- Param MinerのGuess headerでわかる
```http
X-Original-URL: /hoge
```
```http
X-Rewrite-URL: /hoge
```
-> [[⚡️Access control#URL-based access control bypass]]

###### B. 他のリクエストメソッドが使用できないか

- アプリの機能を実行するPOSTリクエストのメソッドをGETに変更してみる
- "Missig parameter xxx"等のエラーが返ればOK
- (または`OPTIONS`リクエストする)
-> [[⚡️Access control#Method-based access control bypass]]