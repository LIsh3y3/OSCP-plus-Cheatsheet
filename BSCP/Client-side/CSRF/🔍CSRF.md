[[⚡️CSRF]]

- 目的： adminのemailをエクスプロイトサーバのドメインを持つメールアドレスに変更し、adminのパスワードをリセットしてadminを侵害

#### 注目ポイント

- Update email機能
- Login with social media機能：OAuth
- Live chat機能： WebSocket
- Forgot password機能 -> [[2. CSRF token検証のバイパス#CSRF tokenがユーザのセッションと結びついていない]]
![[Pasted image 20240222153212.png | 250]]

---
## Detect

- Update email機能、もしくは、Login with social media機能機能があれば、該当リクエスト全体をDo Active Scanする。
- (Live chat機能はscanしても意味ない）

##### CSRFかもしれない条件

1. ログインして実行するアクションが1つ以上のHTTPリクエストで可能
	- パスワード変更
	- Email変更

2. Cookieのみでセッションを管理している
	- セッションの追跡やアクション実行前の検証がない

 3. 予測不可能なリクエストパラメータがない
	- アクション実行前にパスワード入力を求められたらダメ

#### & OAuth

＊：Login with social media機能

1. Login with social mediaがあるか？
2. BurpのTargetのshow only in-scopeは☑️を外し、`/oauth-`から始まるドメインもscopeに
3. 侵害したアカウントでLogin with social media機能のログインをする
4. OAuthフロー中に`state`パラメタがないことを確認する

#### & OAuth & SameSite=Lax

1. Social loginに自動でリダイレクトされる
2. かつ、Update email機能がある

---
#### WebSocket

＊：Live chat機能がある

#### Referer CSRF

- Update emailリクエストのRefererヘッダの値を任意に変更し、"Invalid referer header"エラーがあるか？
