# 現実世界の攻撃の傾向

内部NWへのイニシャルアクセスへのアタックベクターの上位ランキングは以下の通り：
- １位：Credential attack（認証情報の盗難）
- ２位：<u>Phishing</u>

→人的な脆弱性が侵害の原因の**82%** を占める。

また、外部との境界に設置しているWebサーバ（またはそのアプリ）等の<u>技術的な脆弱性を使用した侵入の割合は非常に低い。</u>

ソース:
🔗[2022-data-breach-investigations-report-dbir.pdf - Verizon（米大手通信会社）](https://www.verizon.com/business/resources/reports/2022/dbir/2022-data-breach-investigations-report-dbir.pdf)

よって、**攻撃者にとって、Client-side Attacksの重要性が高まっている**。

---

# Client-side Attacksとは

- 悪意あるファイルをターゲットに直接配信し、ターゲットの環境でファイルを実行させる攻撃のこと
- 具体的な配信方法としては以下が挙げられる：
	- Phishing
	- USB Dropping
	- 水飲み場攻撃

- 具体的な悪意あるファイルとしては以下が挙げられる：
	- Office マクロ
	- Jscriptコード
	- .lnkショートカットファイル

- 攻撃を成功させるためには、ターゲットのOSとインストールされているアプリケーションを特定する必要がある

---

# Client Fingerprinting（識別）

## Clinet Fingerprintingとは

- 直接到達できない内部NWにいるターゲットの使用OS・ブラウザを明らかにすること
- 目的は侵害前のターゲットの偵察
- 侵害時にターゲットがMacなのに、WindowsのEXEファイルを送っても無意味

## Canarytokens

- Canarytokensとは、ターゲットがブラウザで開くと、ブラウザ、IPアドレス、OSの情報が入手できるリンク（トークン含む）を生成できる
	- →生成したリンクをクリックすると空白のページが表示される

1. 🔗[Create a Canarytoken.](https://canarytokens.org/nest/generate)にアクセスする
2. 任意の種類のトークンを選択する（ここでは例としてWeb bugを選択）
	- 以下キャプチャ以外にも、下にスクロールすると、WordやExcel、PDFなども選択できる

![](../../画像ファイル/Pasted%20image%2020250513065718.png)

3. メールアドレス、コメントを入力し、任意で通知先のWebサイトも入力した上でCreate Canarytokenをクリック

![](../../画像ファイル/Pasted%20image%2020250513065904.png)

4. トークンURLが生成されたことを確認
	- How to use ：概要レベルの使い方が記載
	- Manage Canarytoken：トークンのトリガー有無、Emailアラート通知のon / offが切り替えられる

![](../../画像ファイル/Pasted%20image%2020250513070140.png)

5. Manage Canarytokenをクリック後、画面右上のALERTS HISTORYをクリック

![](../../画像ファイル/Pasted%20image%2020250513070243.png)

6. Alert Historyには、トークンをクリックしたユーザーのシステム情報が表示される。

↓誰もクリックしていない場合

![](../../画像ファイル/Pasted%20image%2020250513070649.png)

↓クリックされた後（自分でクリックした）：CSVやJSONでダウンロード可能

![](../../画像ファイル/Pasted%20image%2020250513071026.png)

7. Alerts listに記載のエントリをクリックすると、User-Agent、位置情報など、より詳細な情報を閲覧できる
	- User-Agentから、ターゲットのOS・ブラウザを推測できる
	- CanarytokenはWebページに埋め込まれたJavaScriptの🔗[Fingerprintingのコード](https://github.com/fingerprintjs/fingerprintjs)でUser-Agentを調査しているので、信頼性は高い（単純にHTTPヘッダから入手しているのではない）
	- 🔗[User-Agent parser](https://explore.whatismybrowser.com/useragents/parse/)

>[!Warning]
> - User-Agentの値は、プロキシなどによって書き換えられている場合があるので、Canarytokenの結果を完全には信頼しないこと
> - ターゲットがAd-blockerのブラウザを使っているかそうでないかで、結果が異なることがある
