[⚡️HTTP request smuggling](⚡️HTTP%20request%20smuggling.md)

#### 注目ポイント

- Post Comment機能
- svgなど静的リソース
- `GET /`リクエスト

###### リクエストキャプチャの場合：

- Comment　(任意のデータを長文で保存しやすいので第一優先)
- email　(特定の形式のデータでないとエラーが起きるかも)
- プロフィール説明
- スクリーンネーム(一意性のあるユーザ名)
- Recent searches

---
## Detect

- [🔍HTTP request smuggling](#注目ポイント)にHTTP Request Smugglerの以下の機能を同時に実行する：
	- Smuggle Probe 
	- HTTP/2 probe
	- CL.0

- 結果：複数検知されたら以下の順に優先してエクスプロイトしていく
	- CL.TE multicase, TE.CL multicase, TE.CL dualchunk(TE.TEの事)などHTTP/1.1系 ->[⚡️HTTP request smuggling](⚡️HTTP%20request%20smuggling.md#HTTP%20/1.1%20×%20Smuggler)
	- CL.0 desync -> [⚡️HTTP request smuggling](⚡️HTTP%20request%20smuggling.md#CL.0)
	- HTTP/2 TE desync~などHTTP/2系↓

#### ⚠️ HTTP/2 TE desync~の場合

- HTTP/2 CLの可能性もあるので必ず検証する。[1. HTTP2 request smuggling](5.%20Advanced%20request%20smuggling/1.%20HTTP2%20request%20smuggling.md#1.%20H2.CL検出ステップ)

###### やっても無駄なこと

- いきなりLaunch all scanはダメ。時間15分かかるので
- HTTP/2 TEと検出されたIssueのRequestで400エラーを確認しようにもレスポンスすら返らないときがあるので手動でのHTTP/2 TE検出はあまり意味ない
- HTTP/2 dual :path probeはHTTP/2 TE/CL脆弱性なら反応する。CRLFを見分けるわけではない