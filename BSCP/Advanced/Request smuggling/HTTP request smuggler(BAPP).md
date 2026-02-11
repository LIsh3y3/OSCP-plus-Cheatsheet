
- [[1. Request smugglingの基本]]
- [公式ドキュメント：使い方](https://github.com/PortSwigger/http-request-smuggler?tab=readme-ov-file#practice)
- 参考記事：[How to use HTTP Request Smuggler](https://peter-de-witte.medium.com/how-to-use-the-http-request-smuggler-extension-to-perform-an-attack-8a09c1a6801b)


###### 結果の確認方法

- Pro: DashboardのIssuesタブから。Issuesなら[[2. 基本的なHRSの実行]]の種類がわかる
- Community: Extensions -> HTTP request smugglerのOutputタブから

###### 実行中の確認

- BAPP: Logger++を使用

---
## HRSの検出

1. HTTP Request Smuggler -> smuggle probe
2. ExtensionsのOutputを確認する
	- Pro Editionの場合、検出されたsmuggle脆弱性のタイプは==Issuesタブに==表示される

💡
- Outputを"Show in UI"で表示するなら、左下のClearでOutputログを消しておく
- Outputが古いものから上書きされていくので、"Save to file"が良い。自動でファイルに上書き保存される

###### Found issueの例

- 悪用した場合 -> `HTTP Request Smuggling Confirmed: `
	- `bodyConcat -multiCase`：　body部を連結して成功した。multiCaseは大文字小文字変更してもOK
	- `G -multiCase` : `GPOST`(存在しないメソッド)が成功した

- ⭐️検出した場合 -> `Possible HTTP Request Smuggling: TE.CL multiCase (delayed response)`
	- TE.CLを検出したという意味

##### 注意：
- Repeater上でChange Request attributeからHTTP/1.1へ変更しても、検出オプションによっては必ずHTTP/2で検出する
	- smuggle probe -> HTTP/1.1
	- CL.0 -> HTTP/2 など


---
## HRSの実行

- ⭐️`prefix`の値を目的に合わせて変更すれば、HRS関連は(現実のアプリでも)原則すべて悪用可能
- ==⚠️==: 多数のリクエストにより対象サーバが反応しなくなることがある。攻撃を停止し時間が経てば元に戻る
	- 維持系の攻撃には使用しない[[4. HRS脆弱性の悪用]]

#### Pro Edition

1. DashboardのIssuesで報告された脆弱性を見つけ、最初に添付されたリクエストを開く
2. HTTP Request Smuggler -> Smuggle attackで報告された脆弱性を選択する
	- (CL.TE)等

3. 開かれたTurbo Intruderの`prefix`を目的のアクションに設定してAttack

#### Community Edition

1. Extension -> HTTP Request Smuggler -> Outputを開く
2. Found issueを確認する。
	- 脆弱性の検出では`Possible HTTP Request Smuggling`と表示
	- 悪用できた場合は`HTTP Request Smuggling Confirmed`と表示

3. `Evidence:`以降の `====`で囲まれたリクエスト全体をコピーする。⚠️：末尾の空白行まで全てコピー
![[Pasted image 20240127113243.png | 400]]

4. Repeater -> 上記タブの`+`を選択 -> HTTPを選択 -> ステップ3でコピーしたリクエストをペーストする
5. 右上の`Target`✏️からターゲットサイトのURL、ポート番号を入力する
6. HTTP Request Smuggler -> smuggle attackで"Found issue"にて報告された脆弱性の種類を選択する
	- CL.TE等

7. 開かれたTurbo Intruderの`prefix`を目的のアクションに設定してAttack
	-  Collaboratorを使用しないときはHostヘッダごと削除する
 ![[Pasted image 20240126192117.png | 400]]

###### smuggle attackが実行されると...

- CLがリクエストごとに自動的にアップデートされる
- `Connection: keep-alive`に自動的に変更されている

