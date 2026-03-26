# Proxy

通信内容を傍受・変更するためのコアツール。

## スコープ外のリクエストをInterceptしないようにする

1. BurpでInterceptをOFFのまま、診断対象のWebサイトにアクセスする
2. ある程度アクセスしたら、Target → SitemapのエントリをInScope対象で右クリック → Add to scopeを選択する
3. Proxy Settings → Request/Response Interception rules で以下の項目にチェックを入れる

![](../../Images/Pasted%20image%2020230311192214.png)

$$ProxySettings$$

4. Upボタンで追加したルールを最上位（最優先）に設定する

> [!WARNING]
>  設定時に「scope外のアクセスをログに取らないように設定しますか？」というメッセージが出た場合、**Noにする**。Yesにするとscope外のサイトにアクセスしたとき、「Target」に情報が記録されなくなる。

**WebSocketの場合：**
- デフォルトでは上2つにチェックが入っているが、一番下の項目にもチェックを入れる

---

# Inspector

HTTP history・Repeaterなど、ほとんどの機能の右側に付属しているパネル。

- 内容を自動でデコードして表示する
- デコードした平文を変更すると自動で再エンコードしてくれる
- Request Cookieなどの項目をクリックするとデコードした内容を確認できる
- ユーザーが内容を選択してハイライトすると、その部分だけデコードすることも可能

---

# Repeater

リクエストを何度も変更して再送信できるツール。SQLiのように何度もトライする必要がある攻撃に便利。右下にレスポンスタイムやバイト数が表示されるため、Time-based SQLiの確認にも使える。

---

# Intruder

エンドポイントに対してリクエストをスプレーするツール。ブルートフォースやファジングに使われる。

> [!WARNING] Community Editionではスロットルがかかり、リクエスト数が多いと正確な結果が得られない場合がある。100〜150件ずつのレンジに分割するか、Turbo Intruderなど他のツールを使う。

## Attack Type

Positionsタブから以下のAttack Typeを選択できる。

|Attack Type|説明|用途|
|---|---|---|
|**Sniper**|1つのPayload Setを使い、各ポジションに1つずつペイロードを入れていく。リクエスト数 = ワード数 × ポジション数|シングルポジションの攻撃（ユーザ名がわかっているときのパスワードブルートフォース、APIエンドポイントのファジングなど）|
|**Battering Ram**|1つのPayload Setを使い、全ポジションに同じ値を一斉に代入する|複数の箇所に同じ値を当てて壊れるか確認するとき|
|**Pitch Fork**|複数のPayload Setを使い、各ポジションに対応するリストから1つずつ代入する（並行して進む）。リストのうち最も短いものが終了した時点でテスト終了|Credential stuffing（漏洩アカウントで他サービスへ攻撃）など、ユーザ名とパスワードを組み合わせるとき|
|**Cluster Bomb**|複数のPayload Setを使い、全ての組み合わせを試す|何も情報がないときのクレデンシャルブルートフォース（トラフィックが大きくなる）|

![150](https://claude.ai/Images/Pasted%20image%2020230312164020.png)

![150](https://claude.ai/Images/Pasted%20image%2020230312164332.png)

## Payload設定

|設定項目|説明|
|---|---|
|Payload Set|SniperとBattering Ramは1つのみ選択可。Pitch Fork・Cluster Bombは複数選択可|
|Payload Type|種類によってPayload Settingsが変わる|
|Payload Processing|リスト内の各ペイロードに正規表現・大文字変換などのルールを適用できる|
|Payload Encoding|デフォルトのURLエンコードをオーバーライドできる（ペイロードの動作のためにエンコードしたくない場合に使用）|

**結果の見方：** StatusやLengthを見比べて他と異なるレスポンスを返すリクエストを探す。

![](https://claude.ai/Images/Pasted%20image%2020230313134935.png)

## メッセージフィルタリング

Intruder → Settings → Grep - Extract → Add → Fetch Response → エラーメッセージを選択すると、結果テーブルにカラムが追加されてソートできるようになる。

## CSRFトークン・セッションCookieへの対応

ログインフォームをキャプチャすると、ユーザ名とパスワード以外にリクエスト毎に異なるセッションCookieやCSRFトークンが含まれている場合がある。これらは通常のIntruder攻撃では対応できないため、**[Macro](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#Macro)** を使用して各リクエストのたびに自動取得する。

---

# Decoder

キャプチャした情報やペイロードをエンコード・デコードするツール。

- 平文に戻るまで再帰的にデコードし続ける**Smart Decode**機能がある
- デコード・エンコード・ハッシュの各モードを選択して繰り返し適用できる
- 類似機能を持つより便利な外部ツールも多いため、あまり使われない

---

# Sequencer

セッション値などのランダム性（エントロピー）を検証するツール。ランダム化が不十分な場合、壊滅的な脆弱性につながる可能性がある。主にWeb開発のテストで使われる。

## Live Capture

同じ値を何度もリクエストして、レスポンスのランダムな値を集計してランダム性を確認する（大量のメモリを消費）。

1. キャプチャしたリクエストをSend to Sequencerで送る
2. Token location within responseセクションでエントロピーをテストしたい場所を選択する
3. Start live captureで計測を開始する
4. 適当な数だけ値を取得したらPauseし、Analyze nowをクリックする
5. Auto analyzeにチェックを入れると2000リクエストごとにエントロピー検査を自動実行し、徐々に正確な計測結果が出る

## Manual Load

ランダムな値のリストをファイルなどからロードしてランダム性を測る。リソースはほぼ消費しないが、事前にリストを用意する必要がある。

---

# Macro

## 概要

複数のHTTPリクエストを組み合わせて1つのタスクとして自動化する機能。セッションCookieやCSRFトークンの取得など、リクエストごとに変わるパラメータの値を自動更新するのに非常に有用。

- 設定しない限り、Macroに加えたリクエストのパラメータは1リクエストごとに書き換えられる

> [!WARNING] IntruderやTurbo Intruderで使用する場合は**同時リクエスト数を1に設定**すること。Macroはリクエストごとにトークンを取得するため、並列実行に対応していない。

- 参考：[Burp Suite Intruder THM](https://rahulk2903.medium.com/burp-suite-intruder-tryhackme-walkthrough-270489854c9d)
- 参考LAB：[Expert - Broken brute-force protection](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request)

## 設定手順

1. Settings → Sessions → Session handling rulesのAddをクリックする
2. DetailsタブでRule descriptionに名前をつけ、Rule actionsのAddをクリックし、Run a macroを選択する
3. ポップアップのSelect macro欄のAddをクリックする
4. Macro Recorderでリクエストを選択する（Ctrl+Optionで複数選択可）

**選択するリクエストの基準：**

|シナリオ|選択するリクエスト|
|---|---|
|通常のセッション・CSRF取得|セッションやCSRFトークンが生成される**GETリクエスト**|
|2FAのような2段階ログイン|1段階目のGET + 1段階目のPOST（username+password送信） + 2段階目のGETリクエスト|

> [!TIP] 自動化に最低限必要なリクエストのみ選択する。

5. **Macro Editor**で設定する

**Macro items：** 順番は正しい実行順序でないといけない（順番通りにリクエストを送るため）。

**Custom parameter locations in response：**

- パラメータと値のセットを作成する（Parameter nameを決定し、Parameter valueをレスポンス上で指定する）
- 作成したセットを後続リクエストの「Configure item」で「derive from previous response」に設定することで、**リクエストごとに更新されるパラメータの値を自動取得**できる

6. Scopeタブに移動し、Tools scopeで使用するToolを選択する
7. URL scopeを選択する

|URL scope設定|用途|
|---|---|
|Include all URLs|こだわりなければこれ。とりあえずRuleを適用したいとき|
|Use suite scope [defined in Target tab]|ScopeにAddしているURLに適用したいとき|
|Use custom scope|1つのリクエストのみにRuleを適用したいとき|

## トラブルシューティング

**パラメータの値がリクエストごとに変更・消える場合：** Session handling rule editor → Rule actions → Edit → Update all parameters and headers except for: {削除したくないパラメータ名}

**BAppのインストールが進まない場合：**

1. 🔗[BAPP store](https://portswigger.net/bappstore)からBAppをダウンロードする
2. Extensions → BApp Store → Manual installよりインストールする

---

# WebSocket

詳細：🔗[PortSwigger - Manipulating WebSocket messages](https://portswigger.net/web-security/websockets#manipulating-websocket-messages-to-exploit-vulnerabilities)

## メッセージのInterceptと変更

- Proxy → WebSockets historyを確認する
- Proxy → Settingsから WebSocketのInterceptルールを変更できる
- 通常通りInterceptionをONにすれば傍受・変更が可能

## メッセージのリプレイと生成

1. WebSocket historyのエントリをRepeaterへ送る
2. ✏️アイコンで接続中のWSにアタッチ・クローン・切断したWSへの再接続ができる（ハンドシェイクリクエストの詳細も表示される）
3. ✏️アイコン左のスイッチをON（Connect）にするとハンドシェイクリクエストが実行され、レスポンスも表示される
4. 接続成功後はRepeaterで新しいメッセージを送信できる
5. HistoryペインでRepeaterタブで送信した履歴を確認・改変・再送できる

---

# Extender（Extensions）

BApp Storeから拡張機能をダウンロード・管理する機能。

- ページ上部のBurp Extensionsリストは上から順にトラフィックが通過するため、Up/Downボタンで順番を変更できる
- ページ下部のボックスには現在選択しているモジュールの内容が表示される
- インストールすると新しいタブが追加されるものや、見た目に反映されないものがある

**Pythonで書かれた拡張機能を使う場合：**

1. 🔗[Jython website](https://www.jython.org/download)にアクセスし、Jython standaloneを適当な場所に保存する
2. BurpのOptionsサブタブを開き、Python environmentまでスクロールして保存したファイルのパスを指定する

---

# トラブルシューティング

## Repeaterからレスポンスが返ってこない

**原因1：** Proxy Settingsでスコープフィルタを設定しており、かつRepeaterへ送ったエントリがScopeに登録されていない → Scopeに対象サイトを登録する

**原因2：** リクエストのヘッダ行の後に空行がない → HTTPの仕様上、ヘッダとボディの間には空行（CRLF）が1行必要

# ツールの基本

## Proxy

- 通信内容を傍受する

### 不要なリクエストがinterceptされないようにするには？

1. Burpを起動し、Interceptはoffのままで、診断対象のwebサイトにアクセスする
2. ある程度アクセスしたら、TargetのSitemapのエントリから診断したいエントリを右クリックで選択し、Add to scopeを選択する
3. Proxy Settings->Request|Responce Interception rulesの下記項目にチェックを入れる
![ 500](../../Images/Pasted%20image%2020230311192214.png)

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
	![ 150](../../Images/Pasted%20image%2020230312164020.png)

3. Pitch fork
	- sniperの次によく使われるattack type
	- 多数のスナイパーが同時に稼働しているイメージ
	- 複数のpayload setを使用可能
	- credential stuffing（漏洩したアカウント情報で他のサービスへの攻撃をする）でよく使う
	- 理想的にはペイロードの数は同じくらいがいい
	- なぜなら、100行と90行のリストがあった時、intruderは90回しかリクエストしない。何か一つのリストが終了したらintruderはテストをやめてしまう
	- ポジションは上から下、左から右の順番でpayloadsetを選択してpayload settingsを変更する
	 ![ 150](../../Images/Pasted%20image%2020230312164332.png)

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
![](../../Images/Pasted%20image%2020230313134935.png)

- ログインフォームを突破しようとしたとき、ログインフォームに適当な値を入力してキャプチャすると、ユーザ名とパスワードだけでなく、*リクエスト毎に*異なるセッションクッキーやCSRFトークン（正規のページからアクセスが行われてることを証明するための値）がレスポンスに見られることがある。
	- この場合、ユーザ名とパスワードはpitch fork攻撃などで割り出せるが、セッションクッキーとCSRFトークンは割りだせない
	- ログイン試行のたびにセッションクッキーやCSRFトークンのユニークな値が必要
	- そのため、[Burpの基本的な操作やトラブルシューティング](#Macro)を使用する
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

![](../../Images/Burp%20Suite%20Macro%20How-to.mov)

### トラブルシューティング

パラメータの値がRequestごとに変更されたり消えたりする

- Session handling rule editor -> Rule acrionts -> Edit -> Update all parameters and headers except for: {削除したくないパラメータ名}

BAPPのインストールが進まない

- オンラインのBAPP storeから対象のBAPPをダウンロードする（[BAPP store](https://portswigger.net/bappstore)）
- "Extensions > BApp Store > Manual install"よりインストールする

---

# WebSocket関連


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

- Proxy Settingsで[Burpの基本的な操作やトラブルシューティング](#不要なリクエストがinterceptされてしまう)の設定をしており、かつ、Repeaterへ送ったエントリがScopeに登録されていないから
	- →Scopeに対象サイトを登録する

- ヘッダの後に空行がないから
	- →2行の空白が必要
