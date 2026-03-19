Passive Reconとも呼ばれる。

## IP・ドメインを指定したときの違い

|                 | IP WHOIS | ドメイン WHOIS |
| --------------- | -------- | ---------- |
| ネットワークインフラ調査    | 最強       | 弱め         |
| ドメイン管理者調査       | 微妙       | 最強         |
| メール・フィッシング準備    | △        | ◎          |
| サブドメイン・クラウド資産調査 | △        | ◎          |



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

### 補足：dig

>[!Warning]
>ゾーン転送はActive Reconに分類される

ゾーン転送
```zsh
dig @<name_server> <domain> AXFR
```

MX（メールサーバ）列挙
```zsh
dig <domain> MX
```


---

# Open-Source Code

- 以下の公開場所がある
	- 🔗[GitHub](https://github.com/)
	- 🔗[GitHub Gist](https://gist.github.com/)：コードスニペットのシェア
	- 🔗[GitLab](https://about.gitlab.com/)：DevSecOpsに特化
	- 🔗[SourceForge](https://sourceforge.net/)（GitHubやGitLabの劣化版。もはや使われない）

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

### 自動 w/🔗[gitleaks](https://github.com/gitleaks/gitleaks)

- 大規模レポ向け
- 手動での列挙後に実行すること
	- エントロピー(複雑性)で機密情報と思われるハードコードを検出する。検知できないときもあるので注意

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

# 無料ツール：Netcraft、Shodan

## Netcraft：Search DNS

**Webページの構成技術を表示**することができるWappalyzerのようなもので、Wappalyzerよりも情報量が多い。（2024年に一部機能が有料化・制限された。）
ただし、個人的にはBurp Suiteを使用してレスポンスのServerやx-powered-byヘッダーの値を見るのが確実性高いと思う。

### 使い方

-  [Tools Netcraft](https://www.netcraft.com/tools/) にアクセスし、"Search DNS"を開く
- ターゲットのhostnameを入力
- Site Reportに構成技術が書いてある

![](../../画像ファイル/Pasted%20image%2020250302145854.png)

## Shodan

- NmapのOSINT版のようなもので、ターゲットとの接触なしで、**サービス情報や脆弱性を確認**できる
- 用途は、Activeな列挙に入る前に、どこから列挙していくか当たりをつけるために使う

### 使い方

- `hostname:<target_domain>`を検索バーに入力する

![](../../画像ファイル/Pasted%20image%2020250302172845.png)

- 結果をクリックしてレポートを見る
- open portでさらに結果を絞ることができる

![](../../画像ファイル/Pasted%20image%2020250302173035.png)

- その他、"Advanced Search"よりポート番号などを指定可能

## Security Headers & Qualys SSL Labs

ターゲットのセキュリティへの意識に関する洞察を得る。

### Security Headers

- 🔗[Security Headers](https://securityheaders.com/)
- ターゲットのヘッダー情報で欠けているものを列挙する
- ヘッダーの欠落は、それ自体が脆弱性であるとは限らないが、Web開発者やサーバー管理者がサーバーのハードニングに精通していないことを示している可能性がある

![](../../画像ファイル/Pasted%20image%2020250302173940.png)

### Qualys SSL Labs

- 🔗[Qualys SSL Labs](https://www.ssllabs.com/ssltest/)
- 暗号スイートのベストプラクティスと比較してスコアリングする
- 暗号スイートのベストプラクティスは頻繁に変わるものではない
	- →それにもかかわらず、まだ対応していないなら、セキュリティへの意識が低いと考える

---

# その他OSINTツール

- [Wayback Machine](https://archive.org/web/)
	ドメイン名を検索すると、サービスが Web ページをスクレイピングしてコンテンツを保存した回数が常に表示される。このサービスは、現在の Web サイトでまだアクティブな可能性がある古いページを発見するのに役立つ。読み込んだurlなどもわかる

- S3 bucket 
	amazonのストレージサービス。設定ミスで機密情報を公開しているかも。nameのとこに会社名とかいれて　**{name}** -assets、  **{name}** -www、  **{name}** -public、  **{name}** -private など。

- [SSL/TLS 証明書 - crt.sh](https://ui.ctsearch.entrust.com/ui/ctsearchui)
	ドメインに属するサブドメインを発見することができる。→攻撃可能な範囲を広げる

- [Sublist3r](https://www.kali.org/tools/sublist3r/)
	サブドメインを列挙する
```zsh
sublist3r -d <domain> -t <thread> -e <search_engine>
```
