# ツールの基本

## Proxy

- 通信内容を傍受する

### 不要なリクエストがinterceptされないようにするには？

1. Burpを起動し、Interceptはoffのままで、診断対象のwebサイトにアクセスする
2. ある程度アクセスしたら、TargetのSitemapのエントリから診断したいエントリを右クリックで選択し、Add to scopeを選択する
3. Proxy Settings->Request|Responce Interception rulesの下記項目にチェックを入れる
![[Pasted image 20230311192214.png | 500]]

4. upボタンで追加したルールを最上位（最優先)に設定する

⚠️このとき、「scope外のアクセスをログに取らないように設定しますか？」といったメッセージが出た時、yesにしてしまわないようにする
→yesにするとscope外のサイトにアクセスしたとき*Target*に情報が記録されない

WebSocketの場合：
- デフォルトでは上2つにチェックが入っているが、一番下の項目にチェックをいれる
![[Pasted image 20231101164422.png]]



---

## Inspector

- HTTP historyやRepeaterなど、ほとんどの機能の右側に付属している
- 内容を自動でデコードしたり、デコードした平文を変更することで自動でエンコードしなおしたりしてくれる。
- Request Cookieなどの項目をクリックするとデコードした内容が見れるが、ユーザが内容を選択してハイライトすることで、ハイライトした部分のデコードが可能。

---

## Repeater

- リクエストを何度も変更して送信することを可能にするのでSQLiのように何度もトライする必要がある攻撃に便利
- 一番右下にレスポンスタイムやバイトが記載されているので、Time-based SQLi等に使用できる

---

## Intruder

-  エンドポイントに対してリクエストをスプレーができる
- ブルートフォースアタックやファジングに使われる
	- ⚠️: Request数が500を超えるときは、Community Editionの場合は分割して行わないと正しい結果が得られない事があるため、100-150くらいのレンジに分割する
	- →他のツールを使うか、Burp SuiteのTurbo Intrudeを使う

### Payload Settings

- Pro-only: Add from list
- Fuzzingなどのペイロードが用意されている

###### メッセージフィルタリングのやり方

- Intruder -> Settings -> Grep -Extract -> Add-> Fetch Responce -> エラーメッセージを選択
- Intruderの結果テーブルにカラムが追加されるので、そこをソートする
- optiionで特定の文字を含むリクエストにフラグを立てたり、リダイレクトレスポンス(3××）に対するBurpの対応を定義できる

### Attack type

- positionから以下の**attack type**を選択できる


1. sniper
	- 一つのワードリストや数値の範囲の一つのファイルを使用して攻撃する。リクエストに割り当てるこのようなアイテムのリストのことをBurpでは*Payload Set*と呼ぶ
	- 各ポジションに各ペイロードを一つずつ入れていく攻撃
	- リクエストの数はrequests = numberOfWords * numberOfPositions
	- シングルポジションの攻撃に非常に適している
		- →ユーザ名がわかってる時のパスワードブルートフォースやAPI endpointのファジングなど

2. Battering Ram
	- sniperと同じく一つのアイテム(payload set)を使用するが、違いは、各ポジションに一斉に同じ値を代入するところ
	- ↓下イメージみたいに、どこが壊れるかみるために一斉にぶつけるイメージ
	![[Pasted image 20230312164020.png | 150]]

3. Pitch fork
	- sniperの次によく使われるattack type
	- 多数のスナイパーが同時に稼働しているイメージ
	- 複数のpayload setを使用可能
	- credential stuffing（漏洩したアカウント情報で他のサービスへの攻撃をする）でよく使う
	- 理想的にはペイロードの数は同じくらいがいい
	- なぜなら、100行と90行のリストがあった時、intruderは90回しかリクエストしない。何か一つのリストが終了したらintruderはテストをやめてしまう
	- ポジションは上から下、左から右の順番でpayloadsetを選択してpayload settingsを変更する
	 ![[Pasted image 20230312164332.png | 150]]

 4. Cluseter Bomb
	 - pitch forkと同様に複数のpayload setを使用可能
	 - pitch forkと違ってアイテムのリストの全ての組み合わせを試すので、何も情報がないときのクレデンシャル・ブルートフォースには非常に有効
		 - めちゃトラフィックでかい

### payload

- payload setはsniper攻撃とbattering ramでは一つしか選択できないようになっている
- payload typeには種類がたくさんあってそれぞれpayload settingsが違ってくる
- payload processingはリスト内の各ペイロードに適用されるルール(正規表現や全部大文字など)を適用できる
- payload encodingではペイロードを安全に送信するために自動的に適用されるデフォルトのURLエンコーディングオプションをオーバーライドすることができる
	- 自動でURLエンコードされてしまうので、ペイロードの動作のために、エンコードしないようにすることも可能

- payloadに設定して攻撃し、以下の画像のようにStatusやLengthを見比べて他と異なるレスポンスを返すリクエストをみる。（この場合Length 592)
![[Pasted image 20230313134935.png]]

- ログインフォームを突破しようとしたとき、ログインフォームに適当な値を入力してキャプチャすると、ユーザ名とパスワードだけでなく、*リクエスト毎に*異なるセッションクッキーやCSRFトークン（正規のページからアクセスが行われてることを証明するための値）がレスポンスに見られることがある。
	- この場合、ユーザ名とパスワードはpitch fork攻撃などで割り出せるが、セッションクッキーとCSRFトークンは割りだせない
	- ログイン試行のたびにセッションクッキーやCSRFトークンのユニークな値が必要
	- そのため、[[#Macro]]を使用する
		- ログインリクエスト毎に変わるセッションクッキーやCSRFトークンを値を毎回変えて入れてくれる

---

## Decoder

- 同じような機能をもつ、より便利なアプリケーションは他にもあるため、あまり使われないが、キャプチャした情報やペイロードをターゲットに送信する前の情報をエンコード/デコードできる
- データを平文に戻るまで再起的にデコードし続けるSmart Decode機能がある
- decodeでモード選択、encodeのモード選択、hashのアルゴリズム選択して何度も繰り返せる

---

## Sequencer

- 情報のランダム化がちゃんとできてるかチェックする（エントロピーの確保）
- セッションの値などに使う
- Web開発のテストなどに使われる
- （ランダム化できていないとき、壊滅的な脆弱性につながることがある）

### Live capture / Manual load

- Live capture（メジャー）：
	- 同じ値を何度もリクエストして、レスポンスのランダムな値を集計して、どれくらいランダムか確かめる（大量のメモリを消費）
	- リクエストをキャプチャしたらsend to sequencerで送ることができる
	- "token location within responce `セクションでエントロピーをテストしたい場所を選べる
	- "start live capture" で始める
	- 適当な数だけ値をゲットしたら"Pause"して、"Analyze now"をクリック
	- "auto analyze"にチェックを入れると2000リクエスト毎にエントロピー検査してくれるので、徐々に正確な計測結果が出る

- Manual load：
	- ランダムな値のリストを用意しておいて、ファイルなどをロードしてランダム性を図る。リソースは全然使わないけど、リストを用意するのが大変。

---

# Macro

## 基本

- [参考記事：Burp Suite Intruder THM](https://rahulk2903.medium.com/burp-suite-intruder-tryhackme-walkthrough-270489854c9d)

- タスクを自動化する
- 具体的には、複数のHTTPリクエストを組み合わせて1つのタスクを作成することができる。これは、例えばログインやログアウトのような複数のステップを要するプロセスを自動化するためなど
- Sessionトークンの取得やCSRFトークンの取得などを自動化するのに非常に有用。
- 設定しない限り、基本的にはMacroに加えたRequest のパラメータは1度のRequestごとに書き換えられる。
- ==IntruderやTurboで使用する場合は同時リクエストは1に設定すること==

- [LAB: Expert](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request)

## 基本的な使い方

1. Settings->Sessions-> Session handling rulesのAddをクリック
2. DetailsタブにてRule descriptionで適当な名前をつけ、Rule actionsのAddをクリックし、Run a macroを選択
3. ポップアップのSelect macro: のAddをクリック
4. Macro RecorderでproxyのHTTP Historyが表示されるので、Macroに含みたいリクエストを選択しOK
	- 基本はSessionやCSRF tokenの生成される==GETリクエスト==を選択する
	- 2FAのように、1段階目のログイン成功によりSessionやCSRFが割り当てられるときは、==1段階目のGET + 1段階目のPOST(username+passwordを送る) + 2段階目のログインページのGET==を選択する
	- (選択はctrl+optionで可能)
	- **自動化に最低限必要なリクエストのみ選択する**

5. ⭐️**Macro Editor**

- Macro items: 順番は==正しい順番==でないといけない。順番通りにRequestを送るから。
- "Custom parameter locations in responce"
	- パラメタと値のセットを作成する。Parameter name:を決定し、Parameter value をレスポンス上で指定する。
	- 作成したパラメタと値のセットを後のリクエストで選択し、"derive from previous responce"することで==リクエストごとに更新されるパラメータの値を取得できる==
	- ![[Pasted image 20231212214015.png | 400]]

 - "Configure item"
	 - 指定したパラメータの値を"derive from previous responce"とすることで、指定したパラメータに前回のリクエストのレスポンスから取得した値をセットできる
	 - ![[Pasted image 20231212212721.png | 300]]

6. Scopeタブに移動し、Tools scopeで使用するToolを選択
7. URL scopeを選択
	- Include all URLs：==こだわりなければこれ==。とりあえずRuleを適用したいときに
	- Use suite scope \[defined in Target tab\]: ScopeにAddしているURLに適用したいときに
	- Use custom scope: 1つのRequestのみにRuleを適用したいときに使用

- 🎥 以下一連の流れ
![[Burp Suite Macro How-to.mov]]


### トラブルシューティング

パラメータの値がRequestごとに変更されたり消えたりする

- Session handling rule editor -> Rule acrionts -> Edit -> Update all parameters and headers except for: {削除したくないパラメータ名}

BAPPのインストールが進まない

- オンラインのBAPP storeから対象のBAPPをダウンロードする（[BAPP store](https://portswigger.net/bappstore)）
- "Extensions > BApp Store > Manual install"よりインストールする

### 応用的な使い方

- [[#Turbo Intruder]]と合わせて使用可能
- 詳細は[[3. 多要素認証の脆弱性#CSRF Token bypass 一定回数の入力ミスで認証コードがリセットされる場合]]を確認

---

# WebSocket関連

- [[1. WebSocketsの基本と安全化]]
- 詳細は[記事](https://portswigger.net/web-security/websockets#manipulating-websocket-messages-to-exploit-vulnerabilities)

## WebSocketメッセージのinterceptと変更

- Proxyの*WebSockets history*を確認
- ProxyのSettingsからWebSocketのinterceptルールについて変更可能
![[Pasted image 20240109201356.png | 400]]
- 通常通りinterceptionをonにすれば傍受・変更可能

## WebSocketメッセージのリプレイと生成

- WebSocket historyをRepeaterへ
- ✏️アイコンで接続中のWSにアタッチしたり、クローンしたり、切断したWSに再接続
	- その際、ハンドシェイクリクエストの詳細が表示される
- ✏️アイコン左のスイッチをON(Connect)すると、ハンドシェイクリクエストを実行し、レスポンスも表示する。成功したらRepeaterで新しいメッセージを送信できる
- HistoryペインでこのRepeaterタブで送信した履歴を表示、改変して再送できる
![[Pasted image 20240109202821.png]]

---

# Extender

- 拡張機能をBapp storeからダウンロードできる
- ページ上部のBurp Extentionsのリストは上から順でburp suiteのトラフィックが通過するので、up downボタンで順番を並び替えることができる
- ページ下部のボックスには現在選択しているモジュールの内容が出る
- BApp storeでインストールすると新しいタブが出たり拡張機能によっては見た目に反映されなかったりする
- pythonでコーディングされた拡張機能を適用するにはJythonをインストールする必要がある
	- [Jython website](https://www.jython.org/download)　へアクセス
	- jaython standaloneを適当な位置に保存して、burpのoptionsサブタブを開く
	- python enviromentまでスクロールして、保存したファイルのパスを指定する

---

# トラブルシューティング

## Intercept

### Repeaterからレスポンスが返ってこない

- Proxy Settingsで[[#不要なリクエストがinterceptされてしまう]]の設定をしており、かつ、Repeaterへ送ったエントリがScopeに登録されていないから
	- →Scopeに対象サイトを登録する

- ヘッダの後に空行がないから
	- →2行の空白が必要
