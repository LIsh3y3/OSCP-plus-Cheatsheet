# 概要

- ソーシャル・エンジニアリングとは、人間の弱点を突くことで、人を心理的に操り、任意の操作を実行させたり、情報を漏洩させたりすること
- フィッシングは電子メールを通じて行われるソーシャル・エンジニアリングの一つで、個人情報・認証情報の窃取や、悪意のあるコードの実行をターゲットに行わせるために騙す手法
- 個人・企業を問わず、信頼できる送信元に見せかけて振る舞い、フィッシングメールの内容には、ソフトウェアのダウンロード・添付ファイルの開封・偽のWebサイトへのリンクなど、ターゲットを誘導する内容が含まれる

---

# フィッシングの種類

## スピアフィッシング

- 特定の企業・個人に絞った効果的なフィッシング手法
- ターゲットを絞っているため、スパムフィルター・アンチウイルス・FWなどのテクノロジーで検知することが難しい

## フィッシングキャンペーン

- 大規模なばらまきフィッシング
- **clone phishing**という手法が使われることがある
	- 実在する正規メールをほぼそのままコピーし、リンクや添付ファイルのみ悪意あるものに差し替えて送信する手法

## スミッシング

- SMSを使ったフィッシング
- 職場の電話に送信されるものは仕事に関連した文脈をもつことが多く、個人の電話に送信されるものは特定の友人や家族への言及を含むことが多い

### 補足：電話やSMSの発信者番号の偽装

- VoIP（IPネットワークを使って音声通信を行う技術の総称）を使って簡単になりすましが可能
    - 2つ目の電話番号を設定できるサービスがあり、発信者番号を任意の番号に置き換えられる
- なりすまし用の電話サービスが存在する
- 参考記事🔗：[Everything to Know About Phone Number Spoofing - カスペルスキー](https://www.kaspersky.com/resource-center/preemptive-safety/phone-number-spoofing)

---

# 説得力のあるフィッシングメールの用意

## 重要な3要素

メールの成否は以下の3つで決まる：
- 送信者のメールアドレス
- 件名
- 本文

加えて以下を意識する：
- **Pretext（口実）**：攻撃者がターゲットを誘導するための偽のストーリーで、違和感があってはならない
- 誤字脱字・文法ミス・書式の崩れは発覚の原因になる（生成AIがこの問題を大幅に解消している）
- **恐怖・緊急性**を訴えることでターゲットの冷静な判断を奪う

## 送信者のメールアドレス

信頼できるブランド・既知の連絡先・同僚を偽装したドメイン名を使用する（[[#フィッシングドメインの選択]]）。

ターゲットが交流しているブランドや人物を調べる手段：
- SNSアカウントを観察してブランドや友人を把握する
- Googleでターゲットの名前と場所を検索し、地元企業への口コミがないか調べる
- ターゲットのビジネスWebサイトで取引先を確認する
- OSINT戦術を活用する：[[OSINT]]

## 件名

緊急性・不安・好奇心を喚起するテーマを設定し、ターゲットがすぐに行動するよう誘導する。

例：
- 「アカウントがハッキングされました」
- 「お客様の荷物が発送されました」
- 「スタッフの給与情報（転送厳禁）」
- 「あなたの写真が公開されました」…etc

## 本文・なりすまし

- ブランド・取引先になりすます場合
	- 正規のメールテンプレートやブランディング（スタイル・ロゴ・署名など）を調査し、内容を模倣することで怪しまれないようにする

- 連絡先・同僚になりすます場合
	- ターゲットに連絡する前に、なりすます対象と実際にやり取りしてみるのが効果的
	- テンプレート・署名・自分の呼び方など細かい習慣を把握して使うことで、ターゲットを安心させられる
	    - 例：Mikeという人物で、アドレスが `mike@company.com`、署名が「Best Regards, M」など

- リンクの偽装
	- 表示テキストに正規のURLを見せかけ、実際のリンク先に悪意あるURLを設定する
```html
<!-- 表示：給与明細　実際のリンク先：malicious.com（架空のフィッシングドメイン） -->
<a href="http://malicious.com">給与明細</a>
```

---

# 攻撃の手口（ペイロード配送方法）

既知の脆弱性はすぐにパッチが当てられるため、攻撃者はゼロデイ脆弱性を探してエクスプロイトすることが主流となっている。ゼロデイ脆弱性を発見するために、セキュリティパッチをリバースエンジニアリングして脆弱な箇所を検証している。

## ドロッパー（Dropper）

- フィッシングの被害者が騙されてダウンロード・実行するソフトウェア
- 特定の動画を見るためのコーデックや特定のファイルを開くためのソフトウェアなど、便利で合法的なものを自称している場合がある
- ドロッパー自体には悪意がないためアンチウイルスチェックを通過する傾向がある
- インストールされると、目的のマルウェアが解凍またはサーバからダウンロードされてインストールされる
- マルウェアは攻撃者のインフラに接続し、ターゲットのマシンの制御・内部NWの探索・悪用を可能にする

## 悪意あるファイル形式一覧

| 形式              | 説明                                                                                                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| EXE             | ・Windowsの実行可能ファイル<br>・ダブルクリックで即実行<br>・多くのメールサービスやOSがブロック対象                                                                                                                     |
| SCR             | ・スクリーンセーバーファイル<br>・実体はEXEと同じく実行可能<br>・見た目の誤認を狙った手口で多用される                                                                                                                       |
| ISO             | ・仮想ディスクイメージ（CD/DVDの内容を1ファイルにまとめたもの）<br>・展開せず直接開けるため、内部にEXEやLNKなどのマルウェアが含まれていることがある<br>・Windows 10以降はダブルクリックでマウントされるため危険性が増している                                                 |
| ZIP             | ・圧縮ファイル形式<br>・マルウェア本体を隠蔽したり、複数ファイルに分割して送信されることがある<br>・パスワード付きZIPでウイルススキャンを回避する「PPAP攻撃」などが有名                                                                                    |
| HTA             | ・HTML Application の略（`.hta`）<br>・IEエンジン上でHTML + JavaScriptやVBScriptを**Windowsアプリのように実行可能**<br>・拡張子を偽装して送ることで実行を誘導されやすい                                                         |
| JS / JSE        | ・JScript または JavaScriptファイル（`.js`, `.jse`）<br>・Windows上ではWScriptで実行され、ファイル操作やネットワーク通信が可能<br>・`.jse` は暗号化されたスクリプト                                                               |
| VBS / VBE       | ・VBScriptファイル（`.vbs`）とその暗号化版（`.vbe`）<br>・ユーザー操作や自動実行でコードを実行可能                                                                                                                  |
| DOC / XLS / PPT | ・Office文書形式<br>・マクロを含むことが可能（`.docm`, `.xlsm` など）<br>・MotWがあるとマクロが自動実行されないように設計されている：[[Module 12：Client-side Attacks]]                                                          |
| RTF             | ・Rich Text Format（書式付きテキスト）<br>・埋め込みオブジェクトや脆弱性（例：CVE-2023-21716）を利用してコード実行が可能<br>・マクロ不要で実行される場合もあり危険性が高い<br>・[CVE-2023-21716](https://nvd.nist.gov/vuln/detail/CVE-2023-21716) |
| LNK             | ・Windowsショートカットファイル<br>・見た目は普通のファイルだが、内部で任意のコマンド（EXEやスクリプト）を実行できる                                                                                                              |
| PDF             | ・use-after-free脆弱性によるRCE：[CVE-2023-21608](https://nvd.nist.gov/vuln/detail/CVE-2023-21608)                                                                                     |

## MS Officeマクロの悪用

フィッシングキャンペーンでは、Word・Excel・PowerPointなどのOffice文書が添付されることが多い。Office文書に埋め込まれたマクロが実行されると、マルウェアのインストールや攻撃者への接続が行われる。

例：人事部からの「Staff_Salaries.xlsx」に見せかけたメールをスタッフが受信し、添付ファイルを開いてマクロを有効にすると、そのスタッフのPCが侵害される。

## ブラウザエクスプロイトの悪用

ブラウザ自体の脆弱性を悪用して、ターゲットのコンピュータ上でリモートコマンドを実行する手法。

- 多くのブラウザは最新の状態に保たれており、積極的なバグバウンティプログラムもあるため、一般的な攻撃ベクターではない
- ただし、商用ソフトウェア/ハードウェアとの互換性の問題でブラウザを更新できない現場では有効な手法となる
- 通常の流れ：ターゲットがメールで誘導され攻撃者の設定したサイトにアクセス → ブラウザの脆弱性が発動 → 攻撃者がリモートコマンドを実行可能になる
- 例：🔗[CVE-2021-40444](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-40444) — Microsoft製品の脆弱性で、Webサイトを訪問しただけでコードが実行される可能性があった

## MFAのバイパス

MFAがあってもフィッシングで突破する手法として以下が挙げられる

- MFA Fatigue攻撃（🔗[詳細](https://duo.com/blog/mfa-fatigue-what-is-it-how-to-respond)）：プッシュ通知型MFAを連続送信してターゲットを疲弊させ、誤って承認させる
- フィッシングサイトにMFA入力欄を設ける：ターゲットがMFAコードを入力すると攻撃者がリアルタイムで取得する
- ブラウザ・イン・ザ・ミドル攻撃：攻撃者が被害者と正規サイトの通信の間にプロキシを挟み、認証情報（ID・パスワード・MFAトークン）とセッションを奪取する
- SIMスワッピング：電話会社を騙してターゲットの電話番号を攻撃者のSIMカードに移管し、SMS認証を乗っ取る

---

# フィッシングインフラの整備

フィッシングキャンペーンを成功させるためには、以下のインフラを整える必要がある。

| 項目                          | 内容                                                         |
| --------------------------- | ---------------------------------------------------------- |
| ドメイン名                       | 本物そっくりか、なりすます対象を模倣したドメイン名を登録する（→[[#フィッシングドメインの選択]]）。       |
| SSL/TLS証明書                  | 選択したドメインに証明書を付与することで信憑性を高める。HTTP接続は疑念を抱かせる。                |
| メールサーバー / SMTPアカウント         | メールサーバーを立ち上げるか、SMTPメールプロバイダーに登録する。                         |
| DNSレコード（SPF / DKIM / DMARC） | 適切に設定することでメールの配信性が向上し、迷惑メールフォルダではなく受信トレイに届くようになる。          |
| Webサーバー                     | フィッシング用Webサイトをホストするためのサーバーを用意する。SSL/TLSを追加することでさらに信憑性が高まる。 |
| アナリティクス                     | 送信・開封・クリック・情報送信・ダウンロードを記録し、キャンペーンのパフォーマンスを把握する。            |

## フィッシングドメインの選択

ターゲットに対して心理的優位性を確保するためには、適切なフィッシングドメインを選択することが重要。

### Expired Domains（期限切れドメインの活用）

必須ではないが、ある程度の歴史を持つドメインを購入するとスパムフィルターの判定が良くなる可能性がある（スパムフィルターは新しいドメインを信用しない傾向がある）。

### Typosquatting（タイポスクワッティング）

なりすます対象ドメインに酷似したドメインを登録する手法。
人間の脳はぱっと見で文字列を補完して読む傾向がある。

- スペルミス：`gogle.com` vs `google.com`
- ピリオドの追加：`go.ogle.com` vs `google.com`
- 数字と文字の入れ替え：`g00gle.com` vs `google.com`
- フレーズ化：`googles.com` vs `google.com`
- 追加ワード：`googleresults.com` vs `google.com`

### TLD Alternatives（TLDの変更）

TLD（`.com`, `.net`, `.co.uk`, `.org` など）を変えて同じ名前のドメインを登録する。現在100種類以上のTLDが存在する。
例：`legit.co.jp` を `legit.co.uk` でなりすます。

### IDN Homograph攻撃（同形異義語攻撃）

1998年にIDN（International Domain Name）が実装されたことで、アラビア語・中国語・キリル文字など他言語の文字がドメイン名に使えるようになった。異なる言語の文字が同一に見えることを悪用する。

例：キリル文字の小文字a（U+0430）はラテン文字の小文字a（U+0061）と見た目が同じため、ほぼ同じに見える別のドメインを登録できる。

## 自動化ツール

インフラの一部は以下のツールで迅速に自動化できる：

- **GoPhish**（Open-Source Phishing Framework）：🔗[getgophish.com](https://getgophish.com/)
- **SET**（Social Engineering Toolkit）：🔗[social-engineer-toolkit - GitHub](https://github.com/trustedsec/social-engineer-toolkit)
    - 多数のツールが含まれており、クローンサイトの作成が容易

---
---

# Credential Phishing実践

## 概要

1. Credential Phishing Pretext（口実）の作成
2. 正規のWebサイトのクローン
3. クローンのクリーンアップ（整頓）
4. クローンに悪意のある要素を注入する
5. フィッシングメールの作成

※以下、Zoomをクローン対象とする

- 上記の2〜4はSETで自動化できる：[フィッシングサイト作成の自動化 w/ SET(Social- Engineer-Toolkit)](Phishing.md#フィッシングサイト作成の自動化%20w/%20SET(Social-%20Engineer-Toolkit))
	- しかし、Zoomのログインページのように、POSTリクエストを他のエンドポイントに送信して処理する場合、SETは指定したURLの静的なページしか入力情報をキャプチャできないので、うまくいかない
	- 💡まず、SETでクローンしてみて、うまくキャプチャできないのであれば、下記の手法でクローンする。

---

## 1. Credential Phishing Pretext（口実）の作成

### 前提条件

- ターゲットのEmailアカウントの１つを侵害しているとする
- Emailアカウントでメールが送受信されている
- 送受信されているメールの内容と関連するWebサイトがある

### 具体的なPretextの作成 w/ ChatGPT

- 侵害したEmailアカウント上で、Zoomのライセンスについて従業員にアクティベートを要求するメールの送受信が行われていることが確認できた

![](../画像ファイル/Pasted%20image%2020250509074001.png)

- ChatGPTなどの生成AIを用いて、正規のメールの特徴を"模倣"したメールを作成させる(↓プロンプトプロンプト)
```txt
以下のメールを閲覧してください： 

"Hello Sales department, Hope you're knocking it out of the park this week! We're trying to redo our inventory of Zoom licenses as we seem to have a large number which aren't being used at the moment. Rather than having everyone reply to the email, in order to keep your Zoom license, please just ensure that you login to your account and schedule a meeting within the next two weeks. Any accounts which don't do this within the time frame will be transitioned to a free license. Thank you very much for your cooperation and apologies for the hassle!" 

このメールのスタイルを模倣し、従業員にzoomにログインするようなメールを書いてください。 また、メールには適切なページへ遷移するハイパーリンクも記載してください。
```
↓結果
```txt
Subject: Reminder: Please Log In to Keep Your Zoom License!

Hello Sales department,

Just a quick reminder—hope everything’s going smoothly on your end! We’re still working on updating our Zoom license inventory and noticed that some accounts haven’t yet logged in to schedule a meeting. To make sure your account remains on a full license, please click here to log in and schedule a meeting within the next week.

If no meeting is scheduled by the deadline, any inactive accounts will be moved to a free license.

Thanks again for your cooperation, and sorry for the added task! Let us know if you have any questions.

Best regards,
[Your Company] Helpdesk Team
```

---

## 2. 正規のWebサイトのクローン

- Zoomのサインイン・ページをできるだけ忠実に、説得力のある形で複製
	- ニーズに合わせて細部を変更することもある

1. 正規のサインインページのURLを探す
	Zoom signin：をインターネットで検索すると、このWebサイトがヒットする
	https://zoom.us/ja/signin#/login

![](../画像ファイル/Pasted%20image%2020250509074928.png)

2. クローンするwebサイトの構成情報を保持するディレクトリを作成し、cdする
```zsh
mkdir ZoomSignin
cd ZoomSignin
```

3. wgetでクローンする
```zsh
wget -E -k -K -p -e robots=off -H -Dzoom.us -nd "https://zoom.us/signin#/login"
```
- `-E`: 拡張子を自動変更（例：HTMLと判別された場合は `.html` 拡張子を追加）
- `-k`: ローカルでの閲覧用にリンクを書き換える（絶対パス → 相対パス）
- `-K`: オリジナルファイルを `.orig` 拡張子で保存（リンク書き換え前のバックアップ）
- `-p`: ページ表示に必要なすべての要素（画像、CSSなど）を取得
- `-e robots=off`: robots.txt によるアクセス制限を無視
- `-H`: 外部ホストのファイルもダウンロード対象に含める（ただし `-D` により制限可能）
- `-Dzoom.us`: `zoom.us` ドメインのファイルのみに限定（外部ホストの中でもこのドメインだけ許可）
- `-nd`: ディレクトリ構造を作らず、すべてのファイルをカレントディレクトリに保存

4. クローンの成否をローカルアドレス上で確認する
```zsh
sudo python -m http.server 80
```
```
http://localhost/signin.html
```
以下のエラーはOWASP CSRF Guardによるもので、外部にホストされていたJSを読み込むと表示される。無視してOKでよい。

![](../画像ファイル/Pasted%20image%2020250510094458.png)

5. クローンしたサイトのログインフォームなどの入力フィールド（※）に適当な値を入力し、python webサーバーのログを確認
	（※：ターゲットが機密情報を入力するであろうフィールドをテストする）
```
...
127.0.0.1 - - [09/May/2025 12:41:51] "GET /AlmadenSans-SemiBold-WebXL.woff2 HTTP/1.1" 200 -
127.0.0.1 - - [09/May/2025 12:45:01] code 501, message Unsupported method ('POST')
127.0.0.1 - - [09/May/2025 12:45:01] "POST /sendUserBehavior HTTP/1.1" 501 -
127.0.0.1 - - [09/May/2025 12:50:20] code 501, message Unsupported method ('POST')
127.0.0.1 - - [09/May/2025 12:50:20] "POST /sendUserBehavior HTTP/1.1" 501 -
```

- 出力
	- `/sendUserBehavior`へのPOSTリクエスト
	- http.serverの501エラーで拒否

### クローンしたサイトの挙動からわかること

- `/sendUserBehavior`にエンドポイントがあることから、POSTリクエストを受信可能
- 認証情報を含むリクエストをログに記録することが可能

---

## 3.クローンのクリーンアップ（整頓）

### OWASP CSRFGuardの削除

1. 作成したディレクトリ内のすべてのファイル(ワイルドカード：\*)に対しgrepを実行し、OWASPアラートのテキストが含まれているファイルを探す
```zsh
kali@kali:~/ZoomSignin$ grep "OWASP" *
csrf_js: * The OWASP CSRFGuard Project, BSD License
csrf_js: *    3. Neither the name of OWASP nor the names of its contributors may be used
csrf_js:                    this.setRequestHeader("X-Requested-With", "OWASP CSRFGuard Project");
csrf_js:        alert("OWASP CSRFGuard JavaScript was included from within an unauthorized domain!");
```
→`csrf_js`ファイルに記載されていることがわかった

2. ステップ１で判明したファイルを読み込んでいるページを検索する
```zsh
kali@kali:~/ZoomSignin$ grep "csrf_js" *
csrf_js:        xhr.open("POST", "/csrf_js"+(typeof(resourceAccountIdRoutingURl)!="undefined"?"?"+resourceAccountIdRoutingURl:""), false);
csrf_js:            xhr.open("POST", "/csrf_js"+(typeof(resourceAccountIdRoutingURl)!="undefined"?"?"+resourceAccountIdRoutingURl:""), false);
signin.html:<script nonce="Sxwi1J4PRJSzTs4fu1bzvQ" src="csrf_js"></script>
signin.orig:<script nonce="Sxwi1J4PRJSzTs4fu1bzvQ" src="/csrf_js"></script>
```
→`signin.html`でOWASP CSRFGuard(`csrf_js`)が読み込まれていることがわかる

3. `signin.html`でOWASP CSRFGuardを読みこんでいるラインを削除する
	(`signin.html:<script nonce="Sxwi1J4PRJSzTs4fu1bzvQ" src="csrf_js"></script>`)

4. クローンしたWebサイトをリロードし、OWASP CSRFGuardのポップアップが出ないことを確認する

#### 正規のサイトへのリクエストがないことを確認する

- `signin.html` を確認して、`https://zoom.us/...` からの読み込みが他にもないか探す
- なぜなら、それらの読み込みが Zoom 側に不審なアクセスとして検知される可能性があるから
- Zoom側へのリクエストの発生そのものがステルス性を損なう

---

## 4. クローンに悪意のある要素を注入する

### 4.1. Apacheサーバーにファイルをホストする

1. python webサーバーが稼働しているのであれば、停止する
2. 現在のディレクトリ配下にあるファイルをすべてApacheのrootディレクトリ移動させる
```zsh
kali@kali:~/ZoomSignin$ sudo mv -f * /var/www/html
```
- `-f`：移動先に同名のファイルがあっても確認せずに上書き
- `*`：現在のディレクトリにあるすべてのファイルとディレクトリを指す

3. Apacheをスタートし、cdする
```zsh
sudo systemctl start apache2
cd /var/www/html
```

### 4.2.HTMLフォームの分析

1. ローカルのApacheサーバにホストしているページにアクセスする
```
http://127.0.0.1/signin.html
```

#### 4.2.3. HTMLフォームが動的か静的かを判断する：

1.  どの`<div>`タグ内にHTMLフォームがあるかを確認するため、開発者ツールのElementsで分析する

- 上の要素から順にマウスカーソルをあわせ、選択した要素がHTMLフォームをハイライトしているなら、その`<div>`タグ内にHTMLフォームがあると判断する

![](../画像ファイル/Pasted%20image%2020250511154425.png)

- HTMLフォームは以下のdivタグ内（`<div id="app" class="login-page">`）にあることがわかる
```html
<div id="nested" class="zoom-newpd">
	<div class="mini-layout" role="main" aria-label="main content">
	<div class="HiddenText">
		<a id="the-main-content" tabindex="-1"></a>
	</div>
	<div id="app" class="login-page">
		<header class="layout-header">...</header>
		...
	</div>
</div>
```

3. 「テキストエディタ」でHTMLフォームが存在するdivタグを閲覧する

- `id="app"`の中身(子要素)が、開発者ツールで表示されている内容と異なり、空である
```html
<div id="nested" class="zoom-newpd">
	<div class="mini-layout" role="main" aria-label="main content">
	<div class="HiddenText"><a id="the-main-content" tabindex="-1"></a></div>
	<div id="app"></div>
</div>
```

- つまり：
	- HTMLファイル上（`signin.html`）では `<div id="app"></div>` のみで、内部に何もない
	- しかし、ブラウザで表示されている内容では `<div id="app" class="login-page">` の中にログインフォーム等が存在する
	- したがって、それらの要素は HTMLファイルから来たのではなく、**JavaScriptで後から追加された**と推測できる

##### 💡Tips：HTMLフォームが動的に生成されているかどうかを素早く判別する方法

1. 開発者ツールのSourcesタブで、`div id="app"`と検索し、存在することを確認
2. テキストエディタで同じファイルを開き、`div id="app"`と検索し、開発者ツールと出力が異なることを確認できれば、そのHTMLフォームは動的に生成されるものだとわかる

- Vue / React / Angularなどのフレームワークでは、`<div id="app">`をマウントポイントにしてJavaScriptで内容を挿入・更新する

### 4.3 LLMに任意の静的なHTMLフォームを作成させる

（動的に生成されるJavaScriptコードを分析することも技術的には可能だが、時間がかかるので、LLMに静的なページを作成させる方が早い）

1. 元ページのバックアップを作成する
```zsh
sudo cp -f signin.html signin_orig.html
```
`-f`：コピー先に同名のファイルがあっても確認せず上書き

2. 開発者ツールのElementsタブから、singin.html上で動的に生成されているHTMLフォーム部分をコピーする（ここでは、`<header class...`を右クリック → Copy → *Copy OuterHTML*）とする
```html
<div id="app" class="login-page">
	<header class="layout-header">...</header> <!-- ⇦この部分 -->
	...　　<!-- ⇦この部分 -->
</div>
```
	(Copy OuterHTMLは、選択した要素と、その子要素全てをコピーする)

3. テキストエディタで該当の箇所をステップ２でコピーしたHTMLに書き換えて保存する
```zsh
subl signin.html
```
- 変更前：

![](../画像ファイル/Pasted%20image%2020250511155905.png)

- 変更後：

![](../画像ファイル/Pasted%20image%2020250511160432.png)

4. 保存後、ページをリロードして中身を表示し、本来のページと異なる箇所を把握する
	- zoomのアイコンがでかい
	- 本来であれば画面の左側には画像が、右側にはフォームがあったのに消えている

![](../画像ファイル/Pasted%20image%2020250511160755.png)

#### 4.3.5 LLMを用いたページの調整

>求める回答がくるようにプロンプトを微調整する必要がある可能性あり

1. Zoomのアイコンを調整する
```txt
以下のコードについて、画像を縮小し、少し右下に移動させてください：

<header class="layout-header">..{中略}..</header>
```

![](../画像ファイル/Pasted%20image%2020250511162117.png)

2. 正規のSigninページに存在する画像のURLをコピーする

![](../画像ファイル/Pasted%20image%2020250511162853.png)

3. LLMに正規のSiginページに存在する画像（ステップ２でコピーしたURL）と、HTMLフォームを挿入させるコードを書かせる（POSTデータの送信先を*custom_login.php*とする：[4.4 LLMにPOSTデータ送信先を用意させる](Phishing.md#4.4%20LLMにPOSTデータ送信先を用意させる)）
```txt
divタグを2つ書き、1つは左側に配置し、次の画像を入れてください：
https://file-paa.zoom.us/1xJu7pL8RIWc9lsGFcnRcQ/MS4yLqsI3_R6GW921i53kTJpxI85yLrAIDXer91U-i7ukgBK/12b43c75-e1f7-4d9c-93ee-aaa7f83fdd1a.png

2つめのdivタグには次のZoomのSign inページに似たログインフォームを追加してください：
https://zoom.us/ja/signin#/login

最後に、誰かが "Sign In "ボタンをクリックすると、emailとpasswordが "custom_login.php "ページに送られるようにしてください。
```

4. ステップ３でLLMに生成させたコードを`</header>`タグの後に注入し保存する
```html
</header>

{この部分}

</div> <!-- ← <div id="custom_login">の閉じタグ-->

</div>
<input type="hidden" id="gtm_pageName" value=""/>
...
```

5. 保存後、再度ページをリロードして正規とのページの差異を確かめる
	- 画像の位置

![](../画像ファイル/Pasted%20image%2020250511163847.png)

6. LLMに微調整させる
```txt
上記のコードを以下のように編集してください：
- 画像をもっと上と右に動かす。
- フォームが画像と重ならないように調整する。
- Zoomの公式サインインページには、"パスワードをお忘れですか？"の隣に "ヘルプ "のリンクがあるので、それも追加してください。
- サインインボタンの下に、「サインインにより、私はZoom のプライバシーステートメントとサービス利用規約に同意します。」というテキストを追加し、「Zoomのプライバシーステートメント」と「サービス利用規約」に公式ウェブページへのリンクを設定します。
- この下にチェックボックスを作り、その横に「サインインしたままにする」と書いてください。
- この下に「Zoom は reCAPTCHA で保護されています。Zoom には、Google のプライバシー ポリシーと利用規約が適用されます。」というテキストを追加し、「プライバシーポリシー」と「利用規約」が公式リソースを指すハイパーリンクになっていることを確認してください。
```

![](../画像ファイル/Pasted%20image%2020250511191303.png)

### 4.4 LLMにPOSTデータ送信先を用意させる

- POSTデータ送信先のページを`custom_login.php`とする

1. LLMにPHPファイル生成を依頼する
```txt
custom_login.phpファイルのコードを作成するのを手伝ってください。
まず、メールアドレスとパスワードを取得し、既存のファイルやその内容を上書きすることなく、それらを「test_data.txt」というファイルに書き込む必要があります。
その後、ユーザーを以下のページにリダイレクトする：
https://zoom.us/ja/signin#/login
```
>`test_data.txt`を`credential.txt`と依頼すると、生成AIのポリシーにより手伝ってもらえない可能性がある

2. ステップ１の結果作成されたPHPコードをファイル(`custom_login.php`)に保存し、抽出したデータを書き込むファイルも作成し、権限を付与する
```zsh
sudo echo "" > test_data.txt
sudo chmod 777 custom_login.php
sudo chmod 777 test_data.txt
```

- 実際にメールアドレス、パスワードを入力し、サインインボタンを押すと、以下のようにデータを抽出できる
```zsh
cat test_data.txt      

メールアドレス: hoge@gmail.com
パスワード: foobaradmin
```

---

## 5. フィッシングメールの作成

- [1. Credential Phishing Pretext（口実）の作成](Phishing.md#1.%20Credential%20Phishing%20Pretext（口実）の作成)で作成されたメールに、フィッシングサイトへのリンクを追加し、全返信する。
- ドメイン名は、正規のFQDNを真似ること（[フィッシングドメインの選択](Phishing.md#フィッシングドメインの選択)）
```txt
Hello Sales department,

Just a quick reminder—hope everything’s going smoothly on your end! We’re still working on updating our Zoom license inventory and noticed that some accounts haven’t yet logged in to schedule a meeting. To make sure your account remains on a full license, please click [here](http://フィッシングサイト) to log in and schedule a meeting within the next week.

If no meeting is scheduled by the deadline, any inactive accounts will be moved to a free license.

Thanks again for your cooperation, and sorry for the added task! Let us know if you have any questions.

Best regards,
[Example Company] Helpdesk Team
```

---
---

# フィッシングサイト作成の自動化 w/ SET(Social- Engineer-Toolkit)

🔗[SET - github](https://github.com/trustedsec/social-engineer-toolkit)

## 概要

- 特定のWebサイト上記で説明した以下の作業は自動化できる：
	- 正規のWebサイトのクローン
	- クローンのクリーンアップ（整頓）
	- クローンに悪意のある要素を注入する

- SETのプロンプトに答えていくだけでクローンを作成し、リクエストのキャプチャ・レポート出力も可能

## 🚨留意点

- 指定したURLのページのみ、HTMLを==静的==に取得してクローンする

- 異なるエンドポイントへPOSTリクエストしているWebサイトは、<u>入力情報が正常にキャプチャできないため使用不可</u>（🔗[SPA - single page application](https://qiita.com/shinkai_/items/79e539b614ac52e48ca4)など)

- SETでうまく動作するのは、例えば以下のようなサイト：
	- 古いWordPressのログインページ
	- 静的HTMLベースのフォームページ
		- Facebook, Twitterなど
	- PHPやASPのサーバーサイドフォーム付きページ


## SETによる認証情報窃取手順

1. SETプロンプトの起動
```zsh
sudo setoolkit
```

2. 「1) Social-Engineering Attacks」→「2) Website Attack Vectors」→「3) Credential Harvester Attack Method」→「2) Site Cloner」と遷移する
	- 💡2)Site Clonerではなく、1)Web Templateを選択すると、GoogleやTwitterなどのフィッシングサイトテンプレートを生成できる。

3. クローンしたWebサイトをホストするIPアドレスを入力する
```zsh
set:webattack> IP address for the POST back in Harvester/Tabnabbing [172.16.1.136]: [IPアドレス]
```
- 🚨注意
	- ドメイン名は不可
	- グローバルIPアドレスを指定（`curl https://ipinfo.io/ip`の出力）
	- ネットワークアダプタ設定はNATではなくブリッジで利用する
		- （VM上ではデフォルトでネットワーク設定がNATとなっていることが多い。NATはゲストマシンのプライベートIPアドレスをルーターでグローバルIPアドレスに変換しているため、NAT設定のVM上で作成したクローンサイトをグローバルIPアドレスでホストしても使えない）

4. クローン対象のURLを指定する(ユーザー名・パスワードを入力するサイトが望ましい)
```zsh
set:webattack> Enter the url to clone: [URL]
```

5. サイトがクローンされ、リクエストをキャプチャする状態となる

![](../画像ファイル/Pasted%20image%2020250510160554.png)

6. Ctrl + Cでキャンセルするまでクローンしたサイトをホストし続ける

7. キャンセルするとレポートが生成され、`/root/.set/reports`に格納される（root権限で閲覧可）

- SET実行イメージ参考ブログ🔗：[https://yamashiro.blog/it/phishing/](https://yamashiro.blog/it/phishing/)

---
---

# 補足：メールの送信方法

[💥Phishingの送信](../Cheatsheet/Ports%20-%20Service/25,465,587%20-%20SMTP.md#💥Phishingの送信)

---

# 参考リンク集🔗

- [How I made LastPass give me all your passwords](https://labs.detectify.com/writeups/how-i-made-lastpass-give-me-all-your-passwords/)
- [LastPass: websiteConnector.js content script allows proxying internal RPC commands](https://project-zero.issues.chromium.org/issues/42450228)
- [AutoSpill](https://i.blackhat.com/EU-23/Presentations/EU-23-Gangwal-AutoSpill-Zero-Effort-Credential-Stealing.pdf)
- [TinyURL](https://tinyurl.com/) | [Bitly](https://bitly.com/) : URL短縮サービス
