Passive Reconとも呼ばれる。ターゲットと直接接触せず、公開情報から情報収集する手法。

---

# whois

ドメインレジストラはTCP 43番ポートでwhoisサーバーを運用しており、ドメイン名だけで多くの貴重な情報を得ることができる。運が良ければ名前・メールアドレス・郵便番号・電話番号などの個人情報に加え、技術的情報も入手できる。

## コマンド

```zsh
# 基本
whois <domain|IP>

# 指定したWHOISサーバーにリクエスト
whois <domain|IP> -h <whois_server_IP>
```

## whoisレコード

|項目|役割|具体例/観点|
|---|---|---|
|Registry|TLDを管理する親分的な組織|`.com`, `.net` → Verisign / `.jp` → JPRS / `.org` → PIR|
|Registrar|Registryと契約し、ユーザーにドメイン登録を提供する業者（ブローカー）|GoDaddy, Google Domains, Namecheap, お名前.com|
|**Registrant**|実際にドメインを登録する企業や個人（利用者）|個人情報（プライバシー保護で非公開の場合あり）|
|Name Server|IPアドレスとホスト名を変換 or 知らないと応答|ゾーン転送に使える情報|

その他、レコード作成日・更新日・管理者/技術者の連絡先・レジストラのURL・WHOISサーバーURLなども含まれる。

## IP WHOISとドメインWHOISの違い

|              | `whois <IP>`                                     | `whois <domain>`                                |
| ------------ | ------------------------------------------------ | ----------------------------------------------- |
| 主な目的         | ネットワークの割り当て情報から所属組織・ネットワーク範囲を特定                  | ドメインの所有者・運用者を特定してブランド情報・関連企業を把握                 |
| 得られる情報       | IPブロック範囲・所属組織（OrgName）・データセンター/ISP情報・連絡先・ネットワーク名 | 登録者（Registrant）情報・レジストラ名・登録日・ネームサーバー・DNSSEC設定   |
| スキャンの足がかり    | ◎                                                | △                                               |
| ゾーン転送（AXFR）  | 使えない                                             | 使える                                             |
| ドメイン管理者調査    | △                                                | ◎                                               |
| OSINTツールとの連携 | Shodan / Censys（IP範囲検索）・ASNからインフラ全体を把握           | crt.sh（証明書からサブドメイン列挙）・DNSDumpster（サブドメイン&IPセット） |

---

# DNS関連ツール

## nslookup

- ドメインに関連するAレコードとAAAAレコードを取得する
- 古くからあるツールで、オプションが少ない
- dig登場以降はdigが推奨される場面が多い

## dig

- Domain Information Groper（DIG）の略
- Unix系システムでよく使われるDNS問い合わせツール
- 多くの問い合わせオプションがあり、別のDNSサーバーを指定することも可能

```zsh
# Aレコード問い合わせ
dig <domain>

# MXレコード（メールサーバ）列挙
dig <domain> MX

# ゾーン転送（※Active Recon）
dig @<name_server> <domain> AXFR
```

> [!WARNING]
> ゾーン転送はActive Reconに分類される

## host

- digを簡潔にして見やすくしたもの

---

# traceroute / tracert

- システムからターゲットホストまでのパケットが通ったルートをトレースする
- 経由するルーター（ホップ）のIPアドレスが表示される
- ただし、tracerouteが送信したパケットに応答しないルーターもあり、IPアドレスが表示されないことがある

|OS|コマンド|
|---|---|
|Unix系|`traceroute`|
|Windows|`tracert`|

---

# Exiftool

```zsh
exiftool -a -u <file>
```
- `-a`：重複したタグも表示する
- `-u`：well-knownではないタグも表示する

## Exiftoolの基本機能

- メタデータの表示：画像ファイルからExif情報を読み取り、カメラのメーカーやモデル、撮影日時、露出設定、焦点距離などの情報を表示
- メタデータの編集：画像ファイルのExif情報を変更したり、新しいメタデータを追加したりできる。例えば、撮影日時の修正や作者名の追加などが可能
- メタデータの削除：不要なExif情報を削除することができる。個人情報や位置情報などのプライバシーに関わるデータを削除する場合に使用されることがある

##  出力の注目ポイント

- Creator：ファイルを最初に作成したユーザー名
- Author：ドキュメントの作成者として設定された名前
- Producer：PDFを生成・変換したソフトウェア
	- 使用しているソフトウェアとバージョンから、既知の脆弱性はないか

---

# Google Hacking（Dorking）

- 検索エンジンは新しいWebページやファイルをインデックスするためにwwwをクロールしており、機密情報がインデックス化されることがある
- Googleの高度な検索と特定の用語を組み合わせることで、機密情報を含む文書や脆弱なWebサーバを発見できる

関連ツール：
- 🔗[DorkSearch](https://dorksearch.com/)：用意済みのdorkingがある（※AIは廃止されている）
- 🔗[GHDB（Google Hacking Database）](https://www.exploit-db.com/google-hacking-database)：「Quick Search」よりも「Category」でフィルタリングする方が目的のDorkが見つかりやすい
- 各検索エンジンの詳細文法：[Google Advanced Search](https://www.google.com/advanced_search) / [Google Refine Web Searches](https://support.google.com/websearch/answer/2466433) / [DuckDuckGo Search Syntax](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/) / [Bing Advanced Search Options](https://help.bing.microsoft.com/apex/index/18/en-US/10002)

> [!WARNING]
> Dorkingで発見したサイトに不正にアクセスしたり、データを取得・改ざんしたりする行為は 不正アクセス禁止法等に抵触する可能性がある。検索自体は合法。

## Dorking syntax

### フィルタ

|フィルタ名|説明|例|
|---|---|---|
|`allintext`|指定したすべてのキーワードがページ本文に含まれているものを検索|`allintext:"keyword1" "keyword2"`|
|`intext`|指定したキーワードがページ本文に含まれているものを検索|`intext:"keyword"`|
|`inurl`|URLに指定したキーワードが含まれているものを検索|`inurl:"keyword"`|
|`allinurl`|URLに指定したすべてのキーワードが含まれているものを検索|`allinurl:"hoge foo"`|
|`intitle`|タイトルに指定したキーワードが含まれているものを検索|`intitle:"keyword"`|
|`allintitle`|タイトルに指定したすべてのキーワードが含まれているものを検索|`allintitle:"hoge foo"`|
|`site`|指定したサイトに限定して検索|`site:"www.google.com"`|
|`filetype`|指定したファイル形式のものを検索|`filetype:"pdf"`|
|`inanchor`|リンクのアンカーテキストに指定キーワードが含まれるものを検索|`inanchor:rat`|
|`related`|指定したページと似ているページを検索|`related:www.google.com`|
|`before` / `after`|特定の日付範囲を検索（⚠️非公式オペレータのため動作不安定）|`filetype:pdf & (before:2000-01-01 after:2001-01-01)`|

### 演算子

|演算子|説明|例|
|---|---|---|
|`"keyword"`|完全一致|`"tomato" "pizza"`|
|`OR`|OR|`site:facebook.com OR site:twitter.com`|
|`AND`|AND|`site:facebook.com AND site:twitter.com`|
|`-`|除外|`-filetype:txt`|
|`()`|演算子の組み合わせ|`(site:facebook.com \| site:twitter.com) & intext:"login"`|
|`*`|0文字以上全指定|`site:*.com`|

- 参考：🔗[Google Dorking cheat sheet](https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06)

## Dorking実例

**GHDB例**

| カテゴリ         | GHDB-ID                                      | 内容                                  |
| ------------ | -------------------------------------------- | ----------------------------------- |
| Footholds    | [6364](https://www.exploit-db.com/ghdb/6364) | Nginxのログを発見し、悪用可能なサーバーの誤設定を明らかにするかも |
| 機密ディレクトリ     | [6768](https://www.exploit-db.com/ghdb/6768) | RSA秘密鍵が公開されているかどうかを調べる              |
| Web Server検出 | [6876](https://www.exploit-db.com/ghdb/6876) | GlassFish Serverを探す                 |
| 脆弱なサーバ       | [6728](https://www.exploit-db.com/ghdb/6728) | SolarWinds Orionウェブコンソールを発見         |
| エラーメッセージ     | [5963](https://www.exploit-db.com/ghdb/5963) | エラーに関連するログファイルを検索                   |

管理画面探し（Adminページ探し）

|目的|Dork例|備考|
|---|---|---|
|管理画面URL探索|inurl:admin|超基本・よくヒット|
|管理画面ログインページ|intitle:"管理画面" inurl:login|日本サイト向け|
|WordPress管理画面|inurl:wp-admin|WPサイト特定に|
|CMS管理画面|inurl:typo3  <br>inurl:joomla  <br>inurl:drupal|CMS狙い撃ち|
|VPN/ネットワーク機器のログイン画面|intitle:"VPN login"  <br>intitle:"Firewall Login"|重要インフラ系|

ログインページ探し

|目的|Dork例|備考|
|---|---|---|
|一般的なログインページ|intitle:"login" inurl:login|汎用Dork|
|社内システムログイン|site:example.com intitle:login|特定ドメイン向け|
|基幹システムログイン|intitle:"ERP Login"  <br>intitle:"SAP Login"|大企業狙い撃ち|
|クラウドサービスログイン|inurl:auth  <br>inurl:signin|SaaS系も対応|

機密情報探し

| 目的         | Dork例                                            | 備考           |
| ---------- | ------------------------------------------------ | ------------ |
| 環境設定ファイル探し | filetype:env OR filetype:conf intext:DB_PASSWORD | クラウド・アプリ設定狙い |
| SQLダンプ探し   | filetype:sql "INSERT INTO"                       | データベース漏洩チェック |
| 社内文書探し     | filetype:pdf site:example.com                    | 特定企業の内部文書    |
| 機密ログ探し     | filetype:log "password" OR "API key"             | 誤公開ログ探し      |

意図せず公開されたドキュメント探し

|目的|Dork例|備考|
|---|---|---|
|社内資料のPDF|site:example.com filetype:pdf|汎用Dork|
|Excelに埋め込まれたパスワード|filetype:xls intext:password|よくあるミス|
|Google Drive公開ファイル|site:docs.google.com|意図せず共有されがち|

コード・設定ファイル探し

|目的|Dork例|備考|
|---|---|---|
|Gitリポジトリの誤公開|inurl:.git|.gitディレクトリ直叩き|
|設定ファイル探し|filetype:conf OR filetype:ini|サーバー設定漏洩|
|APIキー漏洩|intext:"API_KEY="|直書き事故チェック|

脆弱なページ・古いページ探し

| 目的           | Dork例                                 | 備考                                       |
| ------------ | ------------------------------------- | ---------------------------------------- |
| 古いページ・アーカイブ  | site:example.com before:2010          | ⚠️ `before:` は非公式オペレータのため動作が不安定          |
| CGIスクリプト狙い   | inurl:.cgi                            | 古いスクリプト発掘                                |
| PHPエラー画面探し   | intext:"Fatal error" inurl:.php       | ~~filetype:php~~ → Googleはインデックスしないため非推奨 |
| ディレクトリリスティング | intitle:"index of" "parent directory" | ディレクトリ一覧が公開されているページ                      |

監視カメラや機器のWebUI探し

| 目的           | Dork例                                               | 備考               |
| ------------ | --------------------------------------------------- | ---------------- |
| 監視カメラ映像      | inurl:/view.shtml  <br>intitle:"Live View / - AXIS" | AXISカメラ          |
| NAS管理画面      | intitle:"NAS Login"                                 | Synology, QNAPなど |
| ルーター/モデム管理画面 | intitle:"Router Login"                              | SOHOルーターなど       |

サブドメイン・関連サイト探し

|目的|Dork例|備考|
|---|---|---|
|サブドメイン洗い出し|site:example.com -www|サブドメイン探索|
|関連サイト洗い出し|related:example.com|公式系列チェック|
|サイト内検索|site:example.com "confidential"|キーワードと組み合わせ|

社員リスト・名簿探し

| 目的     | Dork例                                       | 備考        |
| ------ | ------------------------------------------- | --------- |
| 社員名簿探し | filetype:xls intext:"社員名簿" site:example.com | 内部情報流出系   |
| 連絡先リスト | intext:@example.com filetype:csv            | メールアドレス一覧 |

---

# Open-Source Code

公開されているコードリポジトリは、APIキー・パスワード・内部構造などの機密情報が誤ってコミットされていることがある。

公開場所：
- 🔗[GitHub](https://github.com/)
- 🔗[GitHub Gist](https://gist.github.com/)：コードスニペットのシェア
- 🔗[GitLab](https://about.gitlab.com/)：DevSecOpsに特化
- 🔗[SourceForge](https://sourceforge.net/)：GitHubやGitLabの劣化版で現在はほぼ使われない

## GitHub Search

GitHubの右上検索バーで使えるGoogle Dorkingのようなもの。

### フィルタ

|フィルタ|説明|例|
|---|---|---|
|`owner`|レポジトリ所有者指定|`owner:megacorpone`|
|`org`|レポジトリ所有企業指定|`org:megacorpone`|
|`path`|指定した値がパスに含まれるファイルを表示|`path:user` / `path:*.py`|
|`language`|指定した言語のファイル|`language:php`|
|`in:file`|ファイルの中身にキーワードがある|`keyword in:file`|
|`in:path`|パスにキーワードがある|`keyword in:path`|

### 演算子

|演算子|説明|例|
|---|---|---|
|`OR`|OR|`language:php OR language:csharp`|
|`NOT`|除外|`"fatal error" NOT path:__testing__`|
|`()`|組み合わせ|`(language:ruby OR language:python) AND NOT path:"/tests/"`|

- 参考：[GitHub code search syntax](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax)

## gitleaks（自動化）

🔗[gitleaks](https://github.com/gitleaks/gitleaks)：大規模リポジトリ向け。エントロピー（複雑性）で機密情報と思われるハードコードを検出する。

> [!NOTE] 
> 手動での列挙後に実行すること。検知できないケースもある。

```zsh
sudo apt install gitleaks

# 対象リポジトリをローカルにクローン
git clone <https://xxx.git>

# クローン先にcdしてから探索実行
gitleaks detect -v
```

---

# 特化型OSINTツール

## Netcraft

🔗[Tools Netcraft](https://www.netcraft.com/tools/)：Webページの構成技術を表示するツール（Wappalyzerより情報量が多い。2024年に一部機能が有料化・制限された）。

> [!TIP]
> Active ReconでBurp Suiteでレスポンスの`Server`や`X-Powered-By`ヘッダーを確認するほうが確実性が高い。

使い方：「Search DNS」を開いてターゲットのhostnameを入力 → Site Reportに構成技術が表示される

![](../../Images/Pasted%20image%2020250302145854.png)

## Shodan

🔗[Shodan](https://www.shodan.io/)：NmapのOSINT版のようなもので、**ターゲットとの接触なしにサービス情報や脆弱性を確認**できる。Active列挙に入る前の当たりをつけるために使う。

**Web UI の使い方：**

- `hostname:<target_domain>` を検索バーに入力
- 結果をクリックしてレポートを確認（open portで絞り込みも可能）

![](../../Images/Pasted%20image%2020250302172845.png)

![](../../Images/Pasted%20image%2020250302173035.png)

**CLI の使い方：**

```zsh
# API keyの設定
shodan init <api_key>

# 基本的な使い方
shodan host <target_IP>
```
- shodanにログイン → Account > Account Overview > API keyをコピーしてから `shodan init` を実行
- 詳細は🔗[Shodan CLI](https://cli.shodan.io/)

## ViewDNS.info

🔗[ViewDNS.info](https://viewdns.info/)：リバースIPルックアップを提供する。

共有ホスティングでは1つのIPアドレスが複数のWebサーバーで共有されており、IPを逆引きすることで同じIPアドレスを使っている他のドメインを発見できる。

![](../../Images/Pasted%20image%2020260319124824.png)

$$ViewDNS.infoの結果例$$

## Threat Intelligence Platform

🔗[Threat Intelligence Platform](https://threatintelligenceplatform.com/)：ドメイン名またはIPアドレスを入力すると、マルウェアチェックからWHOISおよびDNSクエリまで一連のテストが実行される。`whois`と`dig`を視覚的にわかりやすく表示してくれる。

## Censys

🔗[Censys Search](https://search.censys.io/)：IPアドレスとドメインに関する多くの情報を提供する。ポート80・443などに関連する情報や、同じIPを使っている他のWebサイトも確認できる。

![](../../Images/Pasted%20image%2020260319124846.png)

$$Censys検索例$$

## Security Headers

🔗[Security Headers](https://securityheaders.com/)：ターゲットのHTTPレスポンスヘッダーで欠けているものを列挙する。

> [!NOTE] 
> ヘッダーの欠落はそれ自体が脆弱性とは限らないが、Web開発者やサーバー管理者がハードニングに精通していない可能性を示す。

![](../../Images/Pasted%20image%2020260319124859.png)

$$SecurityHeaders出力例$$

## Qualys SSL Labs

🔗[Qualys SSL Labs](https://www.ssllabs.com/ssltest/)：暗号スイートのベストプラクティスと比較してスコアリングする。

> [!NOTE] 
> 暗号スイートのベストプラクティスは頻繁に変わるものではないため、未対応の場合はセキュリティへの意識が低いと考えることができる。

## Wayback Machine

🔗[Wayback Machine](https://archive.org/web/)：ドメイン名を検索すると過去のスナップショットを確認できる。現在のWebサイトでまだアクティブな可能性がある古いページや、過去に読み込まれたURLなどを発見できる。

## S3 Bucket

Amazonのストレージサービス。設定ミスで機密情報を公開しているかもしれない。以下のような名前を試してみる：

- `<企業名>-assets`
- `<企業名>-www`
- `<企業名>-public`
- `<企業名>-private`

## crt.sh（サブドメイン列挙）

🔗[crt.sh](https://crt.sh/)：SSL/TLS証明書の透明性ログからサブドメインを発見できる。攻撃可能な範囲を広げるために使う。

## Sublist3r

🔗[Sublist3r](https://www.kali.org/tools/sublist3r/)：複数の検索エンジンを使ってサブドメインを自動列挙する。

```zsh
sublist3r -d <domain> -t <thread> -e <search_engine>
```

---

# Recon-ng

🔗[Recon-ng](https://github.com/lanmaster53/recon-ng)：OSINTタスクに役立つ断片的な情報を見つけるためのフレームワーク。収集されたデータはすべてワークスペースに関連するDBに自動保存される。

**用途例：** ポートスキャンのためのホストアドレスの発見・フィッシング攻撃用の連絡先メールアドレスの収集

```zsh
recon-ng
# プロンプトが [recon-ng][default] > になれば成功
```

## 基本的な流れ

1. ワークスペースの作成
2. データベースに開始情報を挿入
3. マーケットプレイスでモジュールを検索・インストール
4. モジュールをロード・実行

## ① ワークスペースの作成

以下、recon-ngのプロンプトで実行

```sh
workspaces create <workspace_name>

# 起動時に作成済みワークスペースを使用する
recon-ng -w <workspace_name>
```

## ② DBにデータを挿入

```sh
# DBのテーブル名を確認
db schema

# テーブルの中身を閲覧
db query select * from domains

# ドメインを挿入
db insert domains
```

## ③ マーケットプレイスでモジュールを検索

|コマンド|説明|
|---|---|
|`marketplace search`|すべての利用可能なモジュールを表示|
|`marketplace search <keyword>`|キーワードでモジュールを検索|
|`marketplace info <module>`|モジュールの詳細情報を確認|
|`marketplace install <module>`|モジュールをインストール|
|`marketplace remove <module>`|モジュールを削除|

> [!NOTE]
> 
> - "K"列に`*`があるモジュールはAPIキーが必要（→[Keys](#Keys)）
> - "D"列に`*`があるモジュールはサードパーティのPythonライブラリが必要

![](../../Images/Pasted%20image%2020260319124934.png)

## ④ モジュールをロードし実行

```sh
# インストール済みモジュールの一覧
modules search

# モジュールをロード
modules load <module>

# ロードしたモジュールの詳細を表示
list

# オプションをリストアップ
options list

# オプションの値を設定
options set <option> <value>

# 実行
run
```

## Keys

"K"列に`*`がある一部のモジュールは、各サービスAPIのキーがないと使用できない。

```sh
keys list                     # keyをリストアップ
keys add <key_name> <key_value>  # keyの追加
keys remove <key_name>           # keyの削除
```

---

# Maltego

🔗[Maltego](https://www.maltego.com/)：Recon-ngと用途は同じだがより視覚的。マインドマップとOSINTを融合させたアプリケーション。

ドメイン名・会社名・人名・メールアドレスなどを**エンティティ**として入力し、**トランスフォーム**（APIに問い合わせるコード）を使って関連情報を芋づる式に発見していく。収集した情報はフィッシングメール作成などの後のステージで利用できる。

> [!WARNING] 
> Maltegoで利用可能なトランスフォームの中には、ターゲットシステムに積極的に接触するものもある。Passive Reconにとどめたい場合は、そのトランスフォームがどのように機能するかを事前に確認すること。

## 手順例

1. New > Entity Paletteに「DNS name」と入力し、エンティティをダブルクリックしてドメイン名を入力

![](../../Images/Pasted%20image%2020260319125327.png)

2. エンティティを右クリック > All Transforms > 検索タブに「IP」と入力 > 「To IP Address `[DNS]`」をクリック

![](../../Images/Pasted%20image%2020260319125208.png)

3. IPアドレスが表示される

![](../../Images/Pasted%20image%2020260319125933.png)

4. IPアドレスの一つを右クリックして「To DNS Name from passive DNS (Robtex)」などのトランスフォームを選択し、さらに情報を展開する

![](../../Images/Pasted%20image%2020260319125949.png)