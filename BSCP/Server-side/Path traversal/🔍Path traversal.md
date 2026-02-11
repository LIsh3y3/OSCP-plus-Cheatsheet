[[⚡️Path traversal]]

#### 注目ポイント

- `/image?filename=`リクエスト
- `multipart/form-data`のファイル名のパラメータ(下記`action`)
```html
<form enctype="multipart/form-data" action="/upload.php" method="POST">
```

---
## Detect

- `filename`パラメタの値など、脆弱かと思われる箇所をIntruderで選択してからScanする
	- 💡File Path Manipulation(severity: Medium)と表示されたら[[⚡️Command injection#(blind)Path Traversalと組み合わせ、コマンド出力を保存&取得]]を考慮する

---
## Fuzz

1. 脆弱かと思われる箇所をIntruder -> Add from list -> Fuzzing: Path traversal(single file)
2. 200を探し、そのtraversal sequenceで目的のファイルを取得する