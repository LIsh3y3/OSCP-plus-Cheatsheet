（PEN-200のModule 11: Phishing BasicsとTHMのノートを融合）

# 基本情報

## Intro

- ソーシャル・エンジニアリングとは、人間の弱点を突くことで、人を心理的に操り、情報を実行させたり、漏らしたりすること。
- フィッシングは、電子メールを通じて行われるソーシャル・エンジニアリングの一つで、個人情報、認証情報、あるいは悪意のあるコードをコンピュータ上で実行させるために、誰かを騙すもの
- 個人または企業を問わず、信頼できる送信元かのように振る舞う。これらのメールには、ソフトウェアのダウンロード、添付ファイルの開封、偽のWebサイトへのリンクなど、人々を誘惑したり騙そうとする内容が含まれている。  

### フィッシングの種類

#### スピアフィッシング

- 特定の企業・個人に絞った効果的なフィッシング手法
- ターゲットに狙いを定めているので、スパムフィルターやアンチウイルス、FWなどのテクノロジーで検知することが難しい

#### フィッシングキャンペーン

- 大きなスケールのばらまきフィッシング
- clone phishingという、特定サイトをクローンしたものが使われる’

#### スミッシング

- SMSをつかったフィッシング
- 職場の電話に送信されるフィッシングメールは仕事に関連した文脈をもつべきで、個人の電話に送信さえるものは特定の友人や家族への言及を含むべき

###### 補足：電話やSMSの発信者番号の偽装

- VoIP（IPアドレスを使って電話するプロトコル）を使って簡単になりすましが可能
	- 2つ目の電話番号を設定できるサービスがあり、例えば固定電話や2つめの携帯電話番号を設定できる
	- 発信者番号を好きな別の番号に置き換えられる
- なりすまし用の電話サービスがある

- 参考記事🔗：[Everything to Know About Phone Number Spoofing- カスペルスキー](https://www.kaspersky.com/resource-center/preemptive-safety/phone-number-spoofing)

---

# 説得力のあるフィッシングメールの用意

### 概要

- 送信者のメールアドレス、件名、内容の3つが大事
- **Pretext**が重要
	- 攻撃者がターゲットを誘導するための偽のストーリーのこと
	- 違和感があってはいけない
- 誤字脱字、間違った文法、書式の間違いは攻撃がバレる原因となる
	- 生成AIがこの問題を解消している
- 恐怖と緊急性を訴える

### 送信者のメールアドレスの入手先

- 送信者のアドレスは、重要なブランド、既知の連絡先、または同僚を偽装したドメイン名であることが望ましい。
	- [Module 11：Phishing](#フィッシングドメインの選択)
- ターゲットがどのようなブランドや人物と交流しているかを調べるには、OSINT戦術を活用：[1. Recon](../TryHackME/Red%20Teaming/2.%20Initial%20Access/1.%20Recon.md)
- SNSアカウントを観察し、ターゲットが興味をもっているブランドや友人について調べる。  
- Googleでターゲットの名前と大まかな場所を検索し、ターゲットが地元の企業やブランドについて残したレビューがないか確認する。  
- ターゲットのビジネスWebサイトを見て、取引先を見つける。  

###  件名

- ターゲットが無視せずすぐに行動に移せるよう、緊急性の高いもの、心配になるもの、好奇心をそそるものなどをテーマに設定する
- 例：
	- アカウントがハッキングされました
	- お客様の荷物が発送されました  
	- スタッフの給与情報（転送厳禁）
	- あなたの写真が公開されました　...etc

### 内容

- ブランドや取引先になりすます場合、彼らの標準的なメールテンプレートやブランディング（スタイル、ロゴの画像、signなど）を調査し、内容をマネすることで、ターゲットに怪しまれないようにする。
- 連絡先の相手や同僚になりすます場合は、ターゲットに連絡する前になりすます対象に連絡してみるのが効果的。なりすます対象のテンプレートにブランディングが施されていたり、特定のメールの署名があったり、あるいは自分の呼び方など小さなことでも重要。
	- 例えば、Dorothyという名前の人がいて、メールは `dorothy@company.thm` であったりするかもしれない。署名には「Best Regards, Dot」と書かれているかもしれない。このような些細な習慣を知り、使うことで、ターゲットを安心させられる
- リンクは`<a href="http://spoofsite.thm">Click Here</a>`のようにする。

---

# 悪意あるファイルの形式とフィッシングの障害

- すぐにパッチが当てられるため、攻撃者はゼロデイの脆弱性を探してエクスプロイトすることが主流となっている
- ゼロデイを脆弱性を発見するために、セキュリティパッチをリバースエンジニアリングし、どの部分が脆弱かを検証している

## フィッシングに利用されるファイル形式一覧

| 形式              | 説明                                                                                                                                                                                             |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EXE             | ・Windowsの実行可能ファイル  <br>・ダブルクリックで即実行  <br>・多くのメールサービスやOSがブロック対象                                                                                                                                 |
| SCR             | ・スクリーンセーバーファイル  <br>・実体はEXEと同じく実行可能  <br>・見た目の誤認を狙った手口で多用される                                                                                                                                   |
| ISO             | ・仮想ディスクイメージ（CD/DVDの内容を1ファイルにまとめたもの）  <br>・展開せず直接開けるため、内部にEXEやLNKなどのマルウェアが含まれていることがある  <br>・Windows 10以降はダブルクリックでマウントされるため危険性が増している                                                             |
| ZIP             | ・圧縮ファイル形式  <br>・マルウェア本体を隠蔽したり、複数ファイルに分割して送信されることがある  <br>・パスワード付きZIPでウイルススキャンを回避する「PPAP攻撃」などが有名                                                                                                |
| HTA             | ・HTML Application の略（`.hta`）  <br>・IEエンジン上でHTML + JavaScriptやVBScriptを**Windowsアプリのように実行可能**  <br>・拡張子を偽装して送ることで実行を誘導されやすい                                                                     |
| JS / JSE        | ・JScript または JavaScriptファイル（`.js`, `.jse`）  <br>・Windows上では WScript で実行され、ファイル操作やネットワーク通信が可能  <br>・`jse` は暗号化されたスクリプト                                                                          |
| VBS / VBE       | ・VBScriptファイル（`.vbs`）とその暗号化版（`.vbe`）  <br>・ユーザー操作や自動実行でコードを実行可能                                                                                                                                |
| DOC / XLS / PPT | ・Office文書形式  <br>・マクロを含むことが可能（`.docm`, `.xlsm` など）  <br>・MotWがあるとマクロが自動実行されないように設計されている：[Module 12：Client-side Attacks](Module%2012：Client-side%20Attacks.md#MotW（Mark%20of%20the%20Web）)                                                |
| RTF             | ・Rich Text Format（書式付きテキスト）  <br>・テキストベースであるが、埋め込みオブジェクトや脆弱性（例：CVE-2023-21716）を利用してコード実行が可能  <br>・マクロ不要で実行される場合もあり危険性が高い<br>・[CVE-2023-21716](https://nvd.nist.gov/vuln/detail/CVE-2023-21716) |
| LNK             | ・Windowsショートカットファイル  <br>・見た目は普通のファイルだが、内部で任意のコマンド（EXEやスクリプト）を実行できる                                                                                                                            |
| PDF             | ・use-after-free脆弱性によるRCE：[CVE-2023-21608](https://nvd.nist.gov/vuln/detail/CVE-2023-21608)                                                                                                     |

---
---

# MFAのバイパス

- [MFA fatigue](https://duo.com/blog/mfa-fatigue-what-is-it-how-to-respond)攻撃
- フィッシングサイトにMFA入力欄を設ける
- ブラウザ・イン・ザ・ミドル攻撃
	- 攻撃者が被害者と正規サイトとの通信の間に自分のプロキシ（中間者）を挟むことで、被害者にまるで正規のWebサイトにアクセスしているかのように見せかけながら、認証情報（ID・パスワード・MFAトークン）やセッションを奪取する手法
- （SIMスワッピング）


---

# Phishing Infrastructure

- フィッシングキャンペーンを成功させるためには、ある程度のインフラを整える必要がある

✅Domain Name:
	本物そっくりのドメイン名か、他のドメインを模倣したドメイン名を登録する。完璧なドメイン名を作成する方法の詳細は[Module 11：Phishing](#Using%20GoPhish)

✅ SSL/TLS Certificates:
	選択したドメイン名のSSL/TLS証明書を作成することで、攻撃に信憑性を持たせる
	安全でないHTTP接続は疑念を抱かせる

✅Email Server/Account:
	メールサーバーを立ち上げるか、SMTPメールプロバイダーに登録する

✅DNS Records:
	SPF、DKIM、DMARCなどのDNSレコードを設定することで、メールの配信性が向上し、迷惑メールフォルダではなく、受信トレイに届くようになる

✅Web Server: 
	フィッシング用のWebサイトをホストするために、Webサーバーをセットアップするか、会社からWebホスティングを購入する。WebサイトにSSL/TLSを追加することで、信憑性がさらに高まる

✅Analytics:
	送信された電子メール、開封された電子メール、クリックされた電子メールを記録する。また、フィンシング用Webサイトで、どのユーザが個人情報を提供したり、ソフトウェアをダウンロードしたのかの情報と組み合わせる。

## インフラ整備自動化ソフト

上記のインフラの一部は、以下のツールを使用することで迅速に自動化できる。

- GoPhish - (Open-Source Phishing Framework) ： [getgophish.com](https://getgophish.com/)
- SET - (Social Engineering Toolkit)：[social-engineer-toolkit](https://github.com/trustedsec/social-engineer-toolkit)
	- 数多くのツールが含まれている
	- クローンサイトの作成が容易

---

## GoPhishの使い方

### Sending Profiles:

- 実際にフィッシングメールを送信するために必要な接続の詳細で、単にアクセス可能なSMTPサーバーのこと。
- 左メニューのSending Profiles -> +New Profile
- Name, From, Hostに適切な情報を入力し、Save Profileする
 ![](../画像ファイル/Pasted%20image%2020230613114928.png)


### Landing Pages:

- これはフィッシングメールがターゲットを誘導する先のWebサイト
- 企業のWebサイトをクローンするなどの方法もとられる
	- →[Module 11：Phishing](#Credential%20Phishing実践)

- Landing Pages -> New Page
- Name：欄に適切な名前をつけ、次にHTMLボックスで「Source」ボタンを押し、以下のようにHTMLコードを入力する
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>ACME IT SUPPORT - Admin Panel</title>  
    <style>        body { font-family: "Ubuntu", monospace; text-align: center }  
        div.login-form { margin:auto; width:300px; border:1px solid #ececec; padding:10px;text-align: left;font-size:13px;}  
        div.login-form div input { margin-bottom:7px;}  
        div.login-form input { width:280px;}  
        div.login-form div:last-child { text-align: center; }  
        div.login-form div:last-child input { width:100px;}  
    </style>  
</head>  
<body>  
    <h2>ACME IT SUPPORT</h2>  
    <h3>Admin Panel</h3>  
    <form method="post">  
        <div class="login-form">  
            <div>Username:</div>  
            <div><input name="username"></div>  
            <div>Password:</div>  
            <div><input type="password" name="password"></div>  
            <div><input type="submit" value="Login"></div>  
        </div>    </form></body>  
</html>
```
- Sourceボタンを再度クリックすると、下の画像のようにコードをレンダリングした結果を表示する。Capture Submitted DataボックスとCapture Passwordsボックスもクリックし、Save Pageボタンをクリック
![](../画像ファイル/Pasted%20image%2020230613115508.png)

### Email Templates:

- 実際に被害者に送るメールのデザインと内容のこと。

1. Email Templates -> New Template
2. Template Nameに"Email 1"などの名前を入力 -> Subjectに"New Message Received"など件名を入力 -> "HTML"タブをクリックし、"Source"ボタンをクリックしてHTMLエディターモードを有効にする。
3. コンテンツに、ユーザにリンクをクリックさせるようなメールを書く。
4. リンクテキストは、Linkボタンをクリックで編集できる
![](../画像ファイル/Pasted%20image%2020230613120312.png)
5. Display Textには`https://admin.acmeitsupport.thm`と入力
6. 実際のリンクは`{{.URL}}`に設定する。これは、メール送信時に偽装したランディングページに変更される。
7. Protocolを`<other>`に変更する。
![](../画像ファイル/Pasted%20image%2020230613120629.png)

8. Save Template
![ 400](../画像ファイル/Pasted%20image%2020230613120936.png)

### Users & Groups

- ここに、ターゲットのメールアドレスを保存する。
- Users & Groupsをクリック -> New Group -> Group Nameを"Targets"など名前付けし、メールアドレスを追加していく。
- Save Changesボタンを押す
![](../画像ファイル/Pasted%20image%2020230613132642.png)

### Campaigns

- 左側のCampaigns -> +New Campaign
- 必要情報を入力していく
- Launch CampaignをするとResultページに飛ぶ
![](../画像ファイル/Pasted%20image%2020230613132948.png)

### Results

- Resultsページでは、メールの配信数、開封数、クリック数、偽装サイトにデータを送信したユーザ数などを知ることができ、フィッシングキャンペーンのパフォーマンスを知ることができる

- 画面下部には、各メールアドレスの内訳が表示されている。MartinとBrianのメールは正常に送信されていて、このaccountsのメールはエラーになっていることがわかる
- 左側の▶️ボタンから、エラーの詳細をみることができる。
- "Submitted Data"のアカウントの▶️ボタンから詳細をみるとクレデンシャルなどのデータが入力・送信されたことがわかる。
- ![](../画像ファイル/Pasted%20image%2020230613133226.png)

---
---

## Droppers

- ドロッパーとは、フィッシングの被害者が騙されてダウンロードし、システム上で実行させるソフトウェアのこと。
- ドロッパーは、特定のビデオを見るためのコーデックや特定のファイルを開くためのソフトウェアなど、便利で合法的なものを自称している場合がある。
  
- ドロッパーは通常、それ自体には悪意がないため、アンチウイルスチェックを通過する傾向がある。インストールされると、目的のマルウェアは解凍されるか、サーバからダウンロードされ、ターゲットのコンピュータにインストールされる。
- 悪意のあるソフトウェアは、通常、攻撃者のインフラストラクチャに接続する。攻撃者はターゲットのコンピュータを制御することができ、さらにローカルネットワークを探索し悪用することができる

---
---

## フィッシングドメインの選択

[Module 11：Phishing](#送信者のメールアドレス)

- ターゲットに対して心理的優位性を確保するためには、攻撃を開始するフィッシング・ドメインを適切に選択することが不可欠。攻撃者は以下のような方法を用いて最適なドメイン名を選択する

###### Expired Domains:

- 必須ではないが、ある程度の歴史を持つドメイン名を購入することで、スパムフィルターの判定が良くなる可能性がある。
- スパムフィルターは、ある程度歴史のあるドメイン名と比較して、真新しいドメイン名を信用しない傾向がある

###### Typosquatting:

- タイポスクワッティングとは、登録されたドメインが、なりすまそうとする対象ドメインに酷似していることで
- ぱっと見では、人間の脳は空白を埋めて見てしまう。つまり正しいドメイン名に解釈して見る傾向がある

- スペルミス：gogle.com Vs google.com  
- ピリオドの追加：go.ogle.com Vs google.com  
- 数字と文字の入れ替え：g00gle.com Vs google.com  
- フレーズ化：googles.com Vs google.com  
- 追加ワード：googleresults.com Vs google.com

###### TLD Alternatives:

- TLD（Top Level Domain）とは、ドメイン名の.com .net .co.uk .org .gov などの部分のことで、現在では100種類以上のTLDがある。
- ドメインを選ぶ際の一般的なコツは、同じ名前で異なるTLDを使用すること。例えば、tryhackme.co.ukを登録して、tryhackme.comになりすます。

###### IDN Homograph(同形異義語) Attack/Script Spoofing:

- もともとドメイン名はラテン文字のa-zと0-9で構成されていたが、1998年にIDN（International Domain Name）が実装され、アラビア語、中国語、キリル文字、ヘブライ語など、他の言語の文字やアルファベットに対応した。
- IDNの実装によって生じる問題は、異なる言語の異なる文字が同一に見えるということ。
- 例えば、Unicode文字U+0430（キリル文字の小文字a）は、英語で使用されるUnicode文字U+0061（ラテン文字の小文字a）と同じに見えるため、攻撃者は別のドメイン名とほとんど同じに見えるドメイン名を登録することができる
 ![ 300](../画像ファイル/Pasted%20image%2020230613134518.png)

---
---

# フィッシングでのMS Officeの利用

- フィッシングキャンペーンでは、Microsoft Office文書（通常、Word、Excel、PowerPoint）が添付ファイルとして添付されることがよくある。
- Office文書にはマクロが含まれていることがある。マクロはコンピュータのコマンドを実行するために使用され、被害者のコンピュータにマルウェアをインストールさせたり、攻撃者のネットワークに接続して被害者のコンピュータを制御させたりする可能性がある。[2. WeaponizationとDelivery](../TryHackME/Red%20Teaming/2.%20Initial%20Access/2.%20WeaponizationとDelivery.md#Visual%20Basic%20for%20Application%20-%20VBA)

- 例えば、会社で働くスタッフが、人事部から「Staff_Salaries.xlsx」というExcelスプレッドシートを含むメールを受け取った。 これは実は、攻撃者が人事部のメールアドレスを詐称し、スタッフを誘惑して添付ファイルを開かせようとしている。  職員が添付ファイルを開き、マクロを有効にすると、その職員のコンピュータが危険にさらされた

---

# ブラウザのエクスプロイトを利用する

- 攻撃者がターゲットのコンピュータ上でリモートコマンドを実行できるようなブラウザ自体に脆弱性があることで、ターゲットのコンピュータを制御する
- ブラウザの脆弱性は、現場で古い技術が使用されているなどの予備知識がない限り、攻撃者にとっては一般的なベクトルではない。
- 多くのブラウザは最新の状態に保たれており、ブラウザの開発方法によって悪用されにくく、バグバウンティングも積極的
- しかし、商用ソフトウェア/ハードウェアとの互換性がないためにブラウザソフトウェアを更新できない可能性があるため、現場で古い技術を使っていることもある
- 通常、被害者は電子メールを受け取り、攻撃者が設定した特定のWebサイトを訪問するよう誘導される。被害者がそのサイトにアクセスすると、エクスプロイトがブラウザに作用し、攻撃者は被害者のコンピュータ上で好きなコマンドを実行できるようになる
- この例として、[CVE-2021-40444](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-40444)があり、これはマイクロソフトのシステムに見つかった脆弱性で、Webサイトを訪問しただけでコードを実行される可能性がある

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


- 上記の2〜4はSETで自動化できる：[Module 11：Phishing](#フィッシングサイト作成の自動化%20w/%20SET(Social-%20Engineer-Toolkit))
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

3. LLMに正規のSiginページに存在する画像（ステップ２でコピーしたURL）と、HTMLフォームを挿入させるコードを書かせる（POSTデータの送信先を*custom_login.php*とする：[Module 11：Phishing](#4.4%20LLMにPOSTデータ送信先を用意させる)）
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

- [Module 11：Phishing](#1.%20Credential%20Phishing%20Pretext（口実）の作成)で作成されたメールに、フィッシングサイトへのリンクを追加し、全返信する。
- ドメイン名は、正規のFQDNを真似ること（[Module 11：Phishing](#フィッシングドメインの選択)）
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

- [SET - github](https://github.com/trustedsec/social-engineer-toolkit)

## 概要：

- 特定のWebサイト上記で説明した以下の作業は自動化できる：
	- 正規のWebサイトのクローン
	- クローンのクリーンアップ（整頓）
	- クローンに悪意のある要素を注入する

- SETのプロンプトに答えていくだけでクローンを作成し、リクエストのキャプチャ・レポート出力も可能

## 🚨留意点

- 指定したURLのページのみ、HTMLを==静的==に取得してクローンする

- 異なるエンドポイントへPOSTリクエストしているWebサイトは、<u>入力情報が正常にキャプチャできないため使用不可</u>（[SPA - single page application](https://qiita.com/shinkai_/items/79e539b614ac52e48ca4)など)

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

→[25,465,587 - SMTP](../Cheatsheet/Ports%20-%20Service/25,465,587%20-%20SMTP.md#Phishingの送信)

---
---

# 参考リンク集🔗

- [How I made LastPass give me all your passwords](https://labs.detectify.com/writeups/how-i-made-lastpass-give-me-all-your-passwords/)
- [LastPass: websiteConnector.js content script allows proxying internal RPC commands](https://project-zero.issues.chromium.org/issues/42450228)
- [AutoSpill](https://i.blackhat.com/EU-23/Presentations/EU-23-Gangwal-AutoSpill-Zero-Effort-Credential-Stealing.pdf)
- [TinyURL](https://tinyurl.com/) | [Bitly](https://bitly.com/) : URL短縮サービス
