[[⚡️HTTP Host header attack]]

###### まとめ

- [[#Detect]]
- [[#該当脆弱性]]

---
#### 注目ポイント

- "Forgot password?"機能 : [[Password reset poisoning]]
- `/admin`等、高権限ページ

## Detect

 [[🔍 Recon#3. Param Miner -> Guess Headers]]でHHI攻撃に使用できそうなヘッダは見つかった
- yes->[[#3. Host上書き用特殊ヘッダを使用する]]を試す

- No -> 下記のテクニックを試す。Invalid Host Headerなどのエラーが吐かれたりレスポンスが返らなければ他のテクニックへ進む
	- それぞれのテクニックの==エラーstatusの違いにも着目==すること
	- 400系はアクセスコントロール系のブロック。500はアクセスコントロールなし

###### 1. Hostヘッダに適当な値を入力する

```http
GET /example HTTP/1.1
Host: COLLABORATOR_DOMAIN
```

###### 2. ドメイン検証の欠陥を探す

- ドメイン名のみチェックしポートをチェックしないケース
```http
GET /example HTTP/1.1
Host: target.com:COLLABORATOR_DOMAIN
```

- whitelistがあり、前方一致のケース
```http
GET /example HTTP/1.1
Host: target.com.COLLABORATOR_DOMAIN
```

- その他のドメイン検証の欠陥バイパス: [[2.  一般的なSSRF防御の回避#SSRF ホワイトリスト回避]]の難読化以外

###### 3. Host上書き用特殊ヘッダを使用する
```http
GET /example HTTP/1.1
Host: target.com
...
X-Forwarded-Host: COLLABORATOR_DOMAIN
```

##### 4. あいまいなリクエストを送る

###### 1. 2つのHostヘッダの注入

```http
GET /example HTTP/1.1
Host: target.com
Host: COLLABORATOR_DOMAIN　// 順不同
```

###### 2. Hostヘッダの前にスペースを追加する

```http
GET /example HTTP/1.1
    Host: COLLABORATOR_DOMAIN
Host: target.com //順不同
```

###### 3. リクエスト行に絶対URLを指定する

- リクエスト行には絶対URLを指定 (`http` or `https`)
```http
GET https://TARGET_NET/ HTTP/1.1
Host: COLLABORATOR_DOMAIN
```

---
## 該当脆弱性

###### [[⚡️HTTP Host header attack#& Routing-based SSRF]]

- Collaboratorで名前解決される
- &レスポンスに反映されない(blind)

---
###### [[⚡️HTTP Host header attack#& Web cache poisoning]]
- [ ] 
- HOSTヘッダの値が
	- HTMLエンコードされずにレスポンスに反映される
	- & スクリプトの読み込みに直接使用されている
-  レスポンスに[[2. 実装の欠陥の探索#Cacheされている兆候]]がある

---
[[⚡️HTTP Host header attack#Connection state attack]]

- Routing-based SSRFに脆弱だが、1つ目のリクエストHostヘッダの値が検証される

