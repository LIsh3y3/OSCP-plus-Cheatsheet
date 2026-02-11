### Projectスタート時に変更する設定

- 参考：[Tib3rious](https://youtu.be/wdoMaQeMpy8)の5:50~
- User/Project設定ではないので、毎度設定する必要があること
- Live passive crawlとLive audit from Proxyに対して
	- Tools ScopeをProxy, Repeater, Intruderにチェック
	- URL ScoopeをSuite Scopeに

## Scan

- Crawl + Auditのこと
- Crawl ＝　サイト巡回
- Audit ＝　脆弱性テスト

- 各リクエストを選択しDo Active ScanでScan(Audit)のみ
- もしくは、New Scanで全体にActive Scanする
	- Crawl & Auditで、探索とscanをどちらも行う

- ハイライトした箇所にscanができる: [Scan selected insertion point](https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing#scanning-custom-insertion-points)
- 💡Intruderで指定したpositionをscanできる
![[Pasted image 20240208141001.png | 400]]

- 該当Scanの進捗はDashboardのLogger++で確認できる。

## Crawl

- 詳細：[Crawling](https://portswigger.net/burp/documentation/scanner/crawling)
- 目的：Target mapにサイトを探索して追加していく
- Scan configurationからいろいろいじれるが、そのままで良い。

###### "Application login"設定

- 複数ユーザのクレデンシャルを設定し、複数ログインでクローリングできる
- "Recorded login sequences"：SSOなど複雑なログイン機構を使用するサイト向け
- 自動セッションハンドリング：直前のリクエストに基づいて次のリクエストを生成する。CSRFトークンを自動で追従する

---
## Audit

- 詳細：[Auditing](https://portswigger.net/burp/documentation/scanner/auditing)
- 目的：Scan。Issueを見つける
- 

---
## live task

- これをonにして、ブラウザを探索すると自動的に興味ぶかいところやissueを取り上げる

- suite scope = Target scopeに追加したもの
- custom scope = crawlやlive taskにカステムで設定する


- live passive crawlがないと、Targetに追加されない
- アクセスしたページのリンクをsitemapに追加するだけ
- ==Live audit, Live passive crawlはマシンごとにnewして必ず使用すること==
- Live auditはアクセス先のissueを探してくれる。
- 🚨Capturingをoffにしてもscanは走っているので、scanを止めるには⏸️ボタンを押すこと

###### 参考記事

- https://www.docswell.com/s/wild0ni0n/KLL17Q-2023-05-08-114749#p15

---
## Collaborator

- Insertion Collaborator Payloadは実際のCollaboratorドメインではないので、Pollしてもlookupはない。あくまで検証用

---
## Engagement tools

- [詳細](https://portswigger.net/burp/documentation/desktop/tools/engagement-tools)

###### Find references

- 現在表示されているアイテムにリンクしているHTTPレスポンスをBurpのツールから検索する

###### Discover content

- "Session is not running"をクリックしてスタート
	- 名前の予測
	- Web crawling
	- アプリが使用する命名規則からの予測

- Sitemapに追加されていく
- 閲覧可能なコンテンツやBurp Scannerがクロールできる可視コンテンツからリンクされていないコンテンツや機能を発見。

###### Target analyzer

- どこに注意を向けるべきか決定する

###### Find comment

- コメントを検索する。
- そのためにはある程度探索しておく必要がある
- たまにコメントに重要なエンドポイントが書いてあることも

---
### Misc: Active / Passive scan, Crawl , Auditの違い

- Crawl: サイトを満遍なく探索していく。
- Audit: Active + Passive Scan
- Crawl & Audit = 探索したサイトすべてにAuditする
- Active / Passive Scan = 指定したリクエストのみにScanする
- Active Scan = ペイロード候補を送りながらAudit
- Passive Scan =　脆弱性を解析するだけで、何もアクションはしない