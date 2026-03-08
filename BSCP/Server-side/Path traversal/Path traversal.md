## 基本

###### Traversal sequences

- Unix → `../`
- Windows → `../` or `..\`

つまり、==スラッシュ・バックスラッシュのどちらも試すこと。==

#### 標準的なAttack Vector

###### Linuxの場合：/etc/passwd → SSH Private Key窃取

1. `/etc/passwd`の内容を表示することでシステムのユーザーをリストアップ
2. そのユーザーのホーム・ディレクトリーに秘密鍵がないかチェック
3. 秘密鍵を使ってSSH経由でシステムにアクセスする

###### Windowsの場合：決まったなし

機密性の高いファイルは、Windowsではディレクトリの中身をリストすることができなければ、簡単に見つからないことが多い。つまり、機密情報を含むファイルを特定するためには、Web アプリを綿密に調べ、Webサーバー、フ レームワーク、プログラミング言語に関する情報を収集する必要がある。

実行中のアプリケーションやサービスに関する情報を収集したら、機密ファイルにつながるパスを調査することができる。

例えば、ターゲットシステムが*IIS* Webサーバを実行している場合：
	- C:\inetpub\logs\LogFiles\W3SVC1\\.：ログファイルが格納
	- C:\inetpub\wwwroot\web.config：クレデンシャルが格納


#### ⚠️脆弱性のある部分

1. URLパス
2. `multipart/form-data`のファイル名のパラメータ(下記`action`)
```html
<form enctype="multipart/form-data" action="/upload.php" method="POST">
```
3. ファイルをアップロードするときに指定する属性（バイナリデータを送れるようになる）

---
## サニタイジング等の防御を突破する方法

#### 自動

- Intruder-> Payload Settings -> Add from list... -> Fuzzing -path traversal。あらゆるTraversal sequencesを試すので、サニタイジングをバイパスできるかもしれない。
	or
- ProxyのHTTP history -> Requestを選択して右クリック -> Scan -> Active Scan -> Dashboardタブへ移動

#### 手動

- [参考：PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md)

1. `../`が無効化されている場合 👉 絶対パスを入力(`filename=/etc/password`)
2. `../`がサニタイジングされて除去されている場合 👉 `....//` or `..../\`

3. Webサーバが[Path traversal](#⚠️脆弱性のある部分)をアプリケーションに入力を渡す前に[Path traversal](#Traversal%20sequences)を取り除いている場合 
	- 👉 1回もしくは2回のURLエンコーディング(`Ctrl+U`)
	- 👉一般的でない様々なエンコーディング方式(PayloadAllTheThingsの記事Basic exploitation参照)
	- `/`スラッシュはURLの特殊文字なので基本エンコードすること

4. baseとなるフォルダが最初から指定されている場合(例：`image?filename=/var/www/images/`) 👉 baseフォルダに続けてTraversal Sequenceを含める
	- `filename=/var/www/images/../../../etc/passwd`

5. ファイル拡張子を指定されている場合👉Null byteを使用する(`%00`)
	- `filename=../../../etc/passwd%00.png`

💡Tips :  相対パスの場合、rootへ遷移させたい場合は`../`の数は何個でもいい。正確な階層数は把握してなくてもいい。

---
### 🛡️Path traversal 対策

#### 根本的対策
- [ ] 外部からのパラメータでWebサーバ内のファイル名を直接指定する実装を避ける
- [ ] 固定のディレクトリを指定し、basename()関数などのパス名からファイル名のみを取り出すAPIを使用する
- [ ] 入力値をホワイトリストと比較、もしくは許可された文字種か確認

#### 保険的対策
- [ ] ファイル名のチェック
	- Traversal Sequenceのチェック
	- デコードした後にチェックすること。
- [ ] Webサーバのファイルへのアクセス権限の設定



