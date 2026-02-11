[[⚡️DOM-based Vuln]]
#### 注目ポイント

- Post Comment機能
- Search機能
- 怪しげなJavaScriptファイル
- Check Stock機能
- Back to Blog機能 -> Open redirection
- Last viewed product cookie

---
## Detect

- DOM Invaderを使用する

- DOM Invaderのcanaryを怪しい箇所に注入し、DOM Viewでsinkが表示されるか？
	or
- DOM InvaderのWeb message viewでSeverityがHighのものはあるか？
	or
- `<script>`タグ周辺を確認する：Check Stock機能で登場：詳細＝[[5. DOM based XSS#DOM based XSSの検出とテスト方法]]：

##### SourceとSinkのリスト

- [[0. sinkとsourceのリスト]]

---
## Fuzz


```
<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
```
