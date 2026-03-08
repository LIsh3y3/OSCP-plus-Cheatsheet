1. port 80がopenであり、アクセスすると、Wordpressのsetup-config.phpが表示

2. WordPressのsetup-config.phpインストールページでは、攻撃者の用意したMySQLデータベースの認証情報を使って、ターゲットの有効な認証情報がなくともインストールプロセスを完了させることができる脆弱性があった。[Exploit-DB - 18417](https://www.exploit-db.com/exploits/18417)

3. wordpressのセットアッププロセスのリクエストをBurp SuiteでInterceptし、[3306 - MySQL](../Cheatsheet/Ports%20-%20Service/3306%20-%20MySQL.md#MariaDB%20を%200.0.0.0%203306%20でホストする手順)に従いホストしたDBの認証情報を使ってインストールプロセスを完了し、adminパネルにアクセスできるようにした

4. wordpressのadmin panelで、Tools -> Theme File Editor -> patterns/footer-default.phpをPentest MonkeyのPHP reverse shellのペイロードに変更し、リクエストしてシェルを獲得
```
http://192.168.224.16/wp-content/themes/twentytwentythree/patterns/footer-default.php
```

5. `www-data`としてアクセスし、内部を列挙したところ、`/var/www/html/filemanager/`があり、列挙すると、Extplorer (PHPとJSのWebファイルマネージャー)をホストしており、その中のconfig（`.htusers.php`)に認証情報を発見
	- **反省**：認証情報を探すのは、まずはconfigやiniファイル。やみくもにEvilTreeを使わない。

6. 入手した認証情報で`su`し、ユーザーが`disk`グループに所属していたため、以下を参考にroot権限で読み取り可能なファイルを探索
	- https://www.hackingarticles.in/disk-group-privilege-escalation/

7. `/etc/shadow`と`/etc/passwd`からrootのものを抽出し、johnの`unshadow`コマンド→クラックでrootの認証情報を入手