#### 注目ポイント

- `/login`リクエスト：
	- ログインリクエストを何度か試行すると"You have made too many incorrect login attempts. Please try again in 時間"と表示される -> [⚡️Race condition](#Brute-forceのrate%20limitをバイパスする)

- emailアドレスでアクセス制限している&update email機能:
	- Please click the link in your email to confirm the change of e-mail to:`carlos@ginandjuice.shop` ->[⚡️Race condition](#攻撃者のアカウントと高権限ユーザのemailをリンクさせることによるPrivEsc)

- Forgot password? (Please enter your *username* or email) or ユーザ登録.  `Cookie: phpsessionid=~` &トークンがtimestamp

- リクエストが複数のエンドポイントに一気に送られる->[⚡️Race condition](#Multi-endpoint)

---
## Stage1

### Brute-forceのrate limitをバイパスする

Stage1

- [2. Limit overrun](2.%20Limit%20overrun.md#Turbo%20IntruderによるLimit%20overrun%20race%20conditionの検出と悪用)

---
## Stage2

### 攻撃者のアカウントと高権限ユーザのemailをリンクさせることによるPrivEsc

- [5. Single-endpoint race condtion](5.%20Single-endpoint%20race%20condtion.md#攻撃者のアカウントと高権限ユーザのemailをリンクさせることによるPrivEsc)

---
### 被害者のパスワードを任意のものにリセットする

- [8. Time-sensitive attack](8.%20Time-sensitive%20attack.md)

---
### その他

###### Multi-endpointでの商品購入

LABの解説を見た方がわかりやすい
-  [4. Multi-endpoint race condtion](4.%20Multi-endpoint%20race%20condtion.md#①接続をあたためる)

###### 無限にお金を増大させる

[Lab: Infinite money logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-infinite-money)