[[⚡️OAuth]]

#### 注目ポイント

- Sign-in機能 : Host: oauth-...
- `/authorization`へのリクエスト
- アカウント連携機能: パスワードでログインするアカウントとSNSのアカウント

---
## Detect

- [[#注目ポイント]]があればOAuth使用している

### OIDC

1. [[3. OpenID Connectとその脆弱性#OIDC利用しているかの特定方法4つ]]のどれかで検出できたか？
2. 下記のエンドポイントにリクエストし`registration_endpoint`などOIDC登録用エンドポイントがある
	- Yes: -> [[⚡️SSRF#&OpenID]]
```http
outh-hostname/.well-known/oauth-authorization-server
```
```http
oauth-hostname/.well-known/openid-configuration
```

### OAuth

#### & CSRF

- `client_id~&redirect_uri`などがパラメタにあるのに、`state`パラメタがない場合(`state`パラメタ≒CSRFトークン)
- アカウント連携機能(パスワードでログインするアカウントとSNSのアカウント)がある場合
- -> [[⚡️OAuth#& CSRFによるadminアカウント乗っ取り]]

#### 認可コード横取り攻撃(Implicitのアクセストークンも可)

1. Interceptをonにして、OAuthフローを傍受。`redirect_uri`パラメタをエクスプロイトサーバURLへ変更する
```
redirect_uri=https://EXPLOIT_DOMAIN
```

2. エクスプロイトサーバのlogを確認し、攻撃者自身の`code`(認可コード)が送られていることを確認する
	- 認可コードの流出が可能だとわかる
- -> [[⚡️OAuth#redirect_uri検証無し]]

##### redirect_uriが検証されてうまくいかない場合

- -> [[⚡️OAuth#redirect_uri検証あり]]