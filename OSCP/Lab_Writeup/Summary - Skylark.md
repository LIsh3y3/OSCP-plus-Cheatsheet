途中までしか終わらなかった。

# VM15

1. osCommerce Online Merchant v2.3.4.1を使っていることから、[44374 - Exploit-DB](https://www.exploit-db.com/exploits/44374)を使ってRCEし、内部に親友

2. 0.0.0.0で動作していたFroxlorがあったので、`/froxlor/lib/userdata.inc.php`に保存されているSQLの認証情報を抽出
	- 127.0.0.1で動作していなくても、FWによって遮断されていて内部からならアクセスできるサービスは要注目

3. Froxlorのデータベースにアクセスしたところ、\Froloxlor\CronSyste