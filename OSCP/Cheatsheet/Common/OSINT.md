Passive Reconとも呼ばれる。

# whois

## whoisレコード

| 項目             | 役割                                     | 具体例/観点                                                  |
| -------------- | -------------------------------------- | ------------------------------------------------------- |
| Registry       | TLDを管理する親分的な組織                         | `.com`, `.net` → Verisign / `.jp` → JPRS / `.org` → PIR |
| Registrar      | Registryと契約し、ユーザーにドメイン登録を提供する業者（ブローカー） | GoDaddy, Google Domains, Namecheap, お名前.com             |
| **Registrant** | 実際にドメインを登録する企業や個人（利用者）                 | 個人情報                                                    |
| Name Server    | IPアドレスとホスト名を変換 or 知らないと応答              | `dig`などのコマンドで情報収集につながる。<br>==ゾーン転送に使える情報==。             |

## コマンド

```zsh
whois <domain|IP>
```

指定したWHOISサーバーにリクエスト
```zsh
whois <domain|IP> -h <whois_server_IP>
```

### 補足：dig←Active Recon

ゾーン転送
```zsh
dig @<name_server> <domain> AXFR
```

MX（メールサーバ）列挙
```zsh
dig <domain> MX
```

## IP・ドメインを指定したときの違い

|                 | IP WHOIS | ドメイン WHOIS |
| --------------- | -------- | ---------- |
| ネットワークインフラ調査    | 最強       | 弱め         |
| ドメイン管理者調査       | 微妙       | 最強         |
| メール・フィッシング準備    | △        | ◎          |
| サブドメイン・クラウド資産調査 | △        | ◎          |
| **スキャンの足がかり**   | ◎        | △          |
| **ゾーン転送（AXFR）** | 使えない     | 使える        |

###### 詳細

|              | `whois <IP>`                                                                                                           | `whois <domain>`                                                                                         |
| ------------ | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 主な目的         | ネットワークの割り当て情報から、所属組織・ネットワーク範囲を特定                                                                                       | ドメインの所有者・運用者を特定して、ブランド情報・関連企業を把握                                                                         |
| 得られる情報       | - IPブロック範囲（同じ組織が持つIP）  <br>- 所属組織（OrgName）  <br>- データセンター/ISP情報  <br>- 連絡先（Tech/Abuse Contact）  <br>- ネットワーク名（NetName） | - 登録者（Registrant）情報  <br>- レジストラ名（Registrar）  <br>- 登録日・更新日・有効期限  <br>- ネームサーバー  <br>- DNSSEC設定          |
| 攻撃者の狙い       | - 他の関連IP（隠れた資産）も芋づる式に見つける  <br>- クラウド vs オンプレの区別（クラウドなら攻撃法変更）  <br>- AS番号や逆引きからネットワーク内部のネタ探し                           | - ドメインの登録担当者・組織名を特定  <br>- スピアフィッシング対象を特定 <br>- ブランド名から関連ドメイン・サブドメインを列挙  <br>- 管理が甘いレジストラなら、ドメイン乗っ取りの可能性 |
| ドメインが分からない時  | IPから逆引き（PTR）でドメイン名を推測                                                                                                  | 逆引きがない限り、IPは分からない                                                                                        |
| OSINTツールとの連携 | Shodan / Censys（IP範囲検索）  <br>ASNからインフラ全体を把握                                                                            | crt.sh（証明書からサブドメイン列挙）  <br>DNSDumpster（サブドメイン＆IPセットで発見）                                                  |
| 見える内部構成      | CIDR全体のIP割り当て  <br>過去のIP移転履歴  <br>逆引きホスト名からシステム名が漏れる                                                                   | ネームサーバーからDNS管理元を特定  <br>歴代レジストラを調査（WHOIS履歴ツールと組み合わせ）                                                     |
| クラウドかオンプレか   | IPからクラウド事業者を特定                                                                                                         | ネームサーバーがAWSやCloudflareならクラウドと推測                                                                          |
| 防御の甘さチェック    | オープンポートチェック  <br>古いホスト名が生きているか                                                                                         | ゾーン転送チェック  <br>サブドメインに死にレコードがないか                                                                         |

---

# Google hacking

- [DorkSearch](https://dorksearch.com/): 用意済みのdorkingがある（※AIは廃止されている）
- [GHDB](https://www.exploit-db.com/google-hacking-database)：💡"Quick Search"よりも"Category"でフィルタリングする方が、目的のDorkが見つかる

![](../../画像ファイル/Pasted%20image%2020250302124658.png)

## Dorking syntax

### フィルタ

| フィルタ名                          | 説明                                              | 例                                                     |
| ------------------------------ | ----------------------------------------------- | ----------------------------------------------------- |
| allintext                      | 指定した**すべて**のキーワードがページ本文に含まれているものを検索             | allintext:"keyword"                                   |
| intext                         | 指定したキーワードが**すべて**または**どれか1つ**がページ本文に含まれているものを検索 | intext:"keyword"                                      |
| inurl                          | URLに指定したキーワードが含まれているものを検索                       | inurl:"keyword"                                       |
| allinurl                       | URLに指定した**すべて**のキーワードが含まれているものを検索               | allinurl:"hoge foo"                                   |
| intitle                        | タイトルに指定したキーワードが含まれているものを検索（すべて or どれか1つ）        | intitle:"keyword"                                     |
| allintitle                     | タイトルに指定した**すべて**のキーワードが含まれているものを検索              | allintitle:"hoge foo"                                 |
| site                           | 指定したサイトに限定して検索し、そのサイト内のすべての結果を表示                | site:"[www.google.com](http://www.google.com)"        |
| filetype                       | 指定したファイル形式のものを検索                                | filetype:"pdf"                                        |
| link                           | 指定したキーワードがリンクとして貼られているページを検索                    | link:"keyword"                                        |
| allinanchor / inanchor         | 他のページから貼られたリンクの**アンカーテキスト**に指定キーワードが含まれるものを検索   | inanchor:rat                                          |
| allinpostauthor / inpostauthor | ブログ検索限定。指定した著者が書いたブログ記事を検索                      | allinpostauthor:"keyword"                             |
| related                        | 指定したページと**似ているページ**を検索                          | related:www.google.com                                |
| before / after                 | 特定の日付範囲を検索                                      | `filetype:pdf & (before:2000-01-01 after:2001-01-01)` |

### 演算子

| 演算子       | 説明                  | 例                                                            |
| --------- | ------------------- | ------------------------------------------------------------ |
| "keyword" | 指定したキーワードと完全に一致するもの | "tomato" "pizza"                                             |
| OR        | OR                  | site:facebook.com OR site:twitter.com                        |
| AND       | AND                 | site:facebook.com AND site:twitter.com                       |
| -         | 除外                  | -filetype:txt                                                |
| ()        | 演算子のコンビネーション        | (site:facebook.com \| site:twitter.com) & intext:"login"<br> |
| *         | 0文字以上全指定            | `site:*.com`                                                 |

- 参考：[Google Dorking cheat sheet](https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06)

### Dorking実例

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

|目的|Dork例|備考|
|---|---|---|
|環境設定ファイル探し|ext:env OR ext:conf intext:DB_PASSWORD|クラウド・アプリ設定狙い|
|SQLダンプ探し|filetype:sql "INSERT INTO"|データベース漏洩チェック|
|社内文書探し|filetype:pdf site:example.com|特定企業の内部文書|
|機密ログ探し|filetype:log "password" OR "API key"|誤公開ログ探し|

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

- 以下の公開場所がある
	- [GitHub](https://github.com/)
	- [GitHub Gist](https://gist.github.com/)：コードスニペットのシェア
	- [GitLab](https://about.gitlab.com/)：DevSecOpsに特化
	- [SourceForge](https://sourceforge.net/)（GitHubやGitLabの劣化版。もはや使われない）

- [OSINT](#Google%20hacking)のオペレーターをサポートしていることがある

## GitHub Search

githubの右上検索バーで使えるGoogle dorkingのようなもの。

### 手動

フィルタリング

| フィルタ         | 説明                             | 例                       |
| ------------ | ------------------------------ | ----------------------- |
| owner        | レポジトリ所有者指定                     | owner:megacorpone       |
| org          | レポジトリ所有企業指定                    | org:megacorpone         |
| path         | 指定した値がパスに含まれるファイルを表示           | path:user<br>path:\*.py |
| language     | 指定した言語のファイル                    | language:php            |
| in:file      | ファイルの中身にkeywordがある             | keyword in:file         |
| in:path      | パスにkeywordがある                  | keyword in:path         |
| in:file,path | ファイルパスか、もしくはファイルの中身にkeywordがある | keyword in:file,path    |

演算子

| 演算子 | 説明              | 例                                                           |
| --- | --------------- | ----------------------------------------------------------- |
| OR  | OR              | language:php OR language:csharp                             |
| NOT | 指定したクエリの結果を除外する | `"fatal error" NOT path:__testing__`                        |
| ()  | 演算子のコンビネーション    | `(language:ruby OR language:python) AND NOT path:"/tests/"` |
| *   | 任意の文字(0文字以上)    |                                                             |
- 参考：[github code search syntax](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax)

### 自動 w/[gitleaks](https://github.com/gitleaks/gitleaks)

- 大規模レポ向け
- 手動での列挙後に実行すること
	- エントロピー(複雑性)で機密情報と思われるハードコードを検出する。検知できないときもあるので注意（[マインド](../../Misc/マインド.md#🚨ツールに頼りすぎない)）

インストール
```zsh
sudo apt install gitleaks
```

列挙対象のレポジトリをローカルにクローンする
```zsh
git clone <https://xxx.git>
```

クローン先にcdしてから、探索実行
```zsh
gitleaks detect -v
```

---

# ツール(無料)：Netcraft、Shodan

- [ ] Todo:以降編集

## Netcraft：Search DNS

### 基本情報

- **Webページの構成技術を表示**することができるWappalyzerのようなもので、Wappalyzerよりも情報量が多い。
	- 💥2024年にサ終とあるが2025年3月現在使用可能

### 使い方

-  [Tools Netcraft](https://www.netcraft.com/tools/) にアクセスし、"Search DNS"を開く
- ターゲットのhostnameを入力
- Site Reportに構成技術が書いてある

![](../../画像ファイル/Pasted%20image%2020250302145854.png)

- （ただし、個人的にはBurp suiteを使用してレスポンスのServerやx-powered-byヘッダーの値を見るのが確実性高いと思う。）

## Shodan

### 基本情報

- NmapのOSINT版
	- 接触なしで、ターゲットの**サービス情報や脆弱性を確認**可能
- 用途は、Activeな列挙に入る前に、どこから列挙していくか当たりをつけるために使う

### 使い方

- ログインする
- `hostname:<ターゲットドメイン>`を検索バーに入力する
![](../../画像ファイル/Pasted%20image%2020250302172845.png)

- 結果をクリックしてレポートを見る
- open portでさらに結果を絞ることができる

![](../../画像ファイル/Pasted%20image%2020250302173035.png)


- その他、"Advanced Search"よりポート番号などを指定可能

## Security Headers & Qualys SSL Labs

**ターゲットのセキュリティへの意識に関する洞察を得る**。

### Security Headers

- [Security Headers](https://securityheaders.com/)
- ターゲットのヘッダー情報で欠けているものを列挙する
- ヘッダーの欠落は、それ自体が脆弱性であるとは限らない。しかし、ウェブ開発者やサーバー管理者がサーバーのハードニングに精通していないことを示している可能性がある

![](../../画像ファイル/Pasted%20image%2020250302173940.png)

### Qualys SSL Labs

- [Qualys SSL Labs](https://www.ssllabs.com/ssltest/)
- 暗号スイートのベストプラクティスと比較してスコアリングする
- 暗号スイートのベストプラクティスは頻繁に変わるものではない
	- →それにもかかわらず、まだ対応していないなら、セキュリティへの意識が低いと考える
