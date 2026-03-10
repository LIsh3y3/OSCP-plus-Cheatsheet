- 関連ノート:
	- [🐶Bloodhound](../Tools/🐶Bloodhound.md)

- 参考リンク🔗
	- [Cypherクエリの基礎 2020 #neo4j](https://www.creationline.com/tech-blog/data-management/neo4j/33668)

---

# 基本概念

## グラフモデルの構成要素

Cypherはグラフ構造のデータ処理を行うために開発されたクエリ言語で、簡略な構文で複雑な論理構成が可能。

|要素|説明|表記例|
|---|---|---|
|ノード(頂点)|データの実体を表す|`(:Person {name: 'Dan'})`|
|リレーションシップ(辺)|ノード間の関係を表す|`-[:MARRIED]->`|
|プロパティ|ノードやリレーションシップが持つキーバリュー|`{key1:value1, key2:value2}`|
|ラベル|ノードのグループ化|`:Person`, `:Movie`|

## ノードの構文

### アスキーアート表現

|表記|説明|
|---|---|
|`()`|任意のノード|
|`(p)`|変数pに格納された任意のノード|
|`(:Person)`|匿名のPersonラベルノード(クエリから参照不可)|
|`(p:Person)`|変数pに格納されたPersonノード(クエリから参照可能)|
|`(p:Person:Actor)`|複数ラベルの階層化|
|`(p:Person {born: 1970})`|プロパティによるフィルター|

### ノードフィルターの例

```cypher
// 特定のプロパティでフィルター
MATCH (p:Person {born: 1970}) RETURN p
```

## リレーションシップの構文

### アスキーアート表現

|表記|説明|
|---|---|
|`()--()`|双方向の関係(方向なし)|
|`()-->()`|左から右への方向|
|`()<--()`|右から左への方向|
|`()-[*]-()`|任意の深さの関係|
|`()-[*0..10]-()`|0～10階層の関係|
|`()-[*0..]-()`|無制限の深さ|
|`()-[r]-()`|変数rに格納された関係|
|`()-[r:MARRIED]-()`|特定タイプの関係|
|`()-[r:MARRIED\|LIVES_AT]-()`|複数タイプ(OR条件)|

### リレーションシップの操作

```cypher
// 特定のリレーションシップをフィルター
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie {title: 'The Matrix'}) 
RETURN p, rel, m

// リレーションシップのタイプを取得
MATCH (p:Person)-[rel]->(m:Movie {title:'The Matrix'}) 
RETURN p.name, type(rel), m.title

// リレーションシップのプロパティを取得
MATCH (p:Person)-[r:REVIEWED {rating: 65}]->(m:Movie) 
RETURN p.name, r.rating, m.title
```

---

# 命名規則とスタイル

|要素|推奨スタイル|例|
|---|---|---|
|ノードラベル|キャメルケース、大文字開始|`Person`, `NetworkAddress`|
|プロパティキー|キャメルケース、小文字開始|`businessAddress`, `userName`|
|リレーションシップタイプ|大文字、アンダースコア区切り|`ACTED_IN`, `FOLLOWS`|
|Cypherキーワード|大文字|`MATCH`, `RETURN`, `WHERE`|
|文字列|シングル/ダブルクォート|`'The Matrix'`, `"Something's Gotta Give"`|

## WHERE句によるフィルタリング

基本演算子

|演算子|説明|例|
|---|---|---|
|`=`|等価|`WHERE u.enabled = true`|
|`<>`|不等価|`WHERE u.admincount <> 0`|
|`<`, `>`, `<=`, `>=`|比較|`WHERE m.released >= 2003`|
|`IS NULL`, `IS NOT NULL`|NULL判定|`WHERE u.description IS NOT NULL`|
|`AND`, `OR`|論理演算|`WHERE u.enabled = true AND u.hasspn = true`|

範囲フィルター
```cypher
// 標準的な書き方
WHERE m.released >= 2003 AND m.released <= 2004

// Cypher特有の簡潔な書き方
WHERE 2003 <= m.released <= 2004
```

exists()関数
```cypher
// プロパティの存在確認
MATCH (p:Person)-[:ACTED_IN]->(m:Movie) 
WHERE p.name='Jack Nicholson' AND exists(m.tagline) 
RETURN m.title, m.tagline
```

正規表現
```cypher
// =~ 演算子を使用
MATCH (p:Person) 
WHERE p.name =~ 'Tom.*' 
RETURN p.name
```

IN演算子
```cypher
// リストを使ったフィルター
MATCH (p:Person) 
WHERE p.born IN [1965, 1970] 
RETURN p.name, p.born

// 変数として使用
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie) 
WHERE 'Neo' IN r.roles AND m.title='The Matrix' 
RETURN p.name
```

リスト評価関数

|関数|説明|条件|
|---|---|---|
|`ALL()`|すべての要素が条件を満たす|list1がlist2に完全に含まれる|
|`NONE()`|すべての要素が条件を満たさない|list1とlist2に共通要素がない|
|`ANY()`|1つでも条件を満たす|list1とlist2に1つ以上の共通要素|
|`SINGLE()`|条件を満たす要素が1つのみ|list1とlist2の共通要素が1つだけ|

```cypher
// ALLの例
WITH ["A","B","C"] AS list1, ["A","B","C","D"] AS list2 
WHERE ALL(x IN list1 WHERE x IN list2) 
RETURN list1

// ANYの例
WITH ["A","B","C"] AS list1, ["A","F"] AS list2 
WHERE ANY(x IN list1 WHERE x IN list2) 
RETURN list1
```

## 結果値の制御

DISTINCT - 重複排除
```cypher
// collect関数内で使用
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie) 
WHERE p.name = 'Tom Hanks' 
RETURN m.released, collect(DISTINCT m.title) AS movies

// WITH句で使用
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie) 
WHERE p.name = 'Tom Hanks' 
WITH DISTINCT m 
RETURN m.released, m.title
```

ORDER BY - ソート
```cypher
// 降順ソート
ORDER BY m.released DESC

// 昇順ソート
ORDER BY m.released ASC  # ASCは省略可能
```

LIMIT - 結果数制限
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie) 
RETURN p, m 
LIMIT 10
```

## 集約と変換

collect() - リスト化
	集合関数の前の列は暗黙的にグループ化される(GROUP BY句は存在しない)
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie) 
WHERE p.name ='Tom Cruise' 
RETURN p.name, collect(m.title)
```

UNWIND - リストを行に展開
```cypher
// リストの要素を取り出す
WITH [1, 2, 3] AS list 
UNWIND list AS row 
RETURN list, row

// 実践例
MATCH (:Person {name:'Tom Hanks'})-->(movie) 
WITH collect(movie.title) AS list 
UNWIND list AS titles 
MATCH (m:Movie {title: titles}) 
RETURN m.title, m.released
```

集計関数

|関数|説明|
|---|---|
|`count()`|カウント|
|`sum()`|合計|
|`max()`|最大値|
|`min()`|最小値|
|`avg()`|平均値|
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie) 
WHERE p.name ='Tom Cruise' 
RETURN p.name, count(m.title), collect(m.title)
```

## WITH句 - クエリの連鎖

WITH句はクエリブロックの実行結果をテンプレート化し、次のクエリブロックへ受け渡しを行う
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie) 
WITH a, count(a) AS numMovies, collect(m.title) as movies 
WHERE numMovies > 1 AND numMovies < 4 
RETURN a.name, numMovies, movies
```

**WITH句の特徴:**
- RETURNをWITHに置き換えるだけで使用可能
- 次のクエリブロック前でソート、一意化、フィルターを実行可能
- クエリを段階的に構築できる

## OPTIONAL MATCH - 外部結合

繋がりを持たないパターンを漏れなく探索するときに使用
```cypher
MATCH (p:Person) 
WHERE p.name STARTS WITH 'James' 
OPTIONAL MATCH (p)-[r:REVIEWED]->(m:Movie) 
RETURN p.name, type(r), m.title
```

## UNION - クエリ結果の結合

```cypher
// UNION ALL - 重複を含めて結合
WITH [1,2,3] AS list UNWIND list AS n RETURN n 
UNION ALL 
WITH [1,2,3] AS list UNWIND list AS n RETURN n

// UNION - 重複を排除して結合
WITH [1,2,3] AS list UNWIND list AS n RETURN n 
UNION 
WITH [1,2,3] AS list UNWIND list AS n RETURN n
```

## サブクエリ

```cypher
// CALL {}でネストしたクエリを実行
CALL { 
  MATCH (p:Person) 
  WHERE p.name = 'Tom Hanks' 
  RETURN p 
} 
RETURN p.name
```

## 実践的なクエリパターン

### パターンマッチング

```cypher
// 複雑なパターン
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person) 
RETURN a.name, m.title, d.name

// 可変長パス
MATCH (a:Person)-[:ACTED_IN*1..3]->(m:Movie) 
RETURN a, m
```

### 条件分岐

```cypher
// CASE式
MATCH (p:Person) 
RETURN p.name, 
  CASE 
    WHEN p.born < 1950 THEN 'Old' 
    WHEN p.born < 1980 THEN 'Middle' 
    ELSE 'Young' 
  END AS generation
```

---

# BCEで使えるカスタムクエリ

| 説明                | クエリ                                                       | 備考                        |
| ----------------- | --------------------------------------------------------- | ------------------------- |
| すべての指定したオブジェクトを表示 | `MATCH (m:Computer) RETURN m`                             |                           |
| アクティブなセッションを確認する  | `MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p` | owened userでどのマシンに横展開できるか |
