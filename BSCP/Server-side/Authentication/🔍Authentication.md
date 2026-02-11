[[⚡️Authentication]]

#### 注目ポイント

- My account機能: `/login`リクエスト
- 2FA
- "Remember me"：Stay-logged-in
- "Change passowrd"
- Account Registration ->  [Github cheat sheet: Account registration](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/tree/main?tab=readme-ov-file#account-registration)

---
## Detect

### Username列挙

- Grep-Extractにloginエラーメッセージを指定した上でusernameのwordlist × 長いパスワードでIntruder。以下の反応を確認する
```http
username=§§&password=peterpeterpeterpeterpeter... //✖️ 100
```

↓反応

- IPアドレスブロック
- アカウントロック: 💡正規のアカウントにだけロックがかかるかもなので、intruderする
- User rate制限：ある程度の時間が経てばロックが解除される場合

##### 前提：レスポンスでクレデンシャルが正しいことを示す兆候

- Status Codeの違い
- Error messageの違い
	- 人間の目にはわからない違いもあるので、==必ず==Intruder -> Settings -> Grep Extractでエラーメッセージを選択しておくこと

- Response timeの違い
	- 違いを明確にするため、usernameの列挙時は==100文字程度のめっちゃ長いpasswordを使用==する

---
### 2FA

###### 認証コードバイパスの場合

- Cookie等に特定のユーザに紐づいた情報がある
```http
Cookie: verify=wiener;
```
- 認証コード入力画面で適当なクレデンシャルを使用してbrute-forceし、アカウントロックがかからないことを確認する

---
### Stay-logged-inがある場合

- [[⚡️XSS#Stay-logged-in Cookieを取得]]で被害者のcookieを取得できるか

---
### Change password

- パスワード変更には基本的に3つの入力欄がある
	- Current password
	- New password
	- Confirm new password
- 3つの入力欄に以下のパターンを試し、エラーメッセージなどの反応を確認する。Current passwordの値が正しい & ログアウトされない反応を引き出す

|  | A | B | C | D |
| ---- | ---- | ---- | ---- | ---- |
| Current password | ⭕️ | ❌ | ❌ | ⭕️ |
| New password | ⭕️ | hoge | foo | foo |
| Confirm new password | ⭕️ | hoge | hoge | hoge |

- -> [[⚡️Authentication#Change passwordを悪用して被害者パスワードのbrute-force]]
