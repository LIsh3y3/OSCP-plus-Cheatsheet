
- Webアプリのパスワードリセット機能を用いてログイン後、MariaDBのSQLi脆弱性をもとにUNION SELECTでコマンド実行し、BarracudaDrive 6.5のLocal Privilege Escalationの脆弱性（Service Binary Hijacking）を用いて権限昇格する

1. Port Scanの結果、anonymousログイン可能なFTP、複数のWebアプリが検知

2. 一見静的なWebアプリにGobusterを実行したところ、phpinfoが表示できたため、DOCUMENT_ROOTやdisable_functionsがないことなどを確認
	- 静的だからと列挙をやめなかったのが良かった点

3. 他のWebアプリを列挙すると、メールアドレスやユーザー名が記載されたページを発見したため、cewlで抽出
```sh
cewl -e --email_file WebEnum/emails.txt -m 5 -w WebEnum/cewl.txt http://$TargetIP:33033
```

4. ログインページに"Forgot Password?"機能があり、ユーザー名、秘密のキーワード、新しいパスワードの３つのパラメタを入力するようにあった

5. 秘密のキーワードとして、メールアドレスやユーザー名が記載されたページに記載されている自己紹介文の単語を入力し、パスワードリセットに成功

6. リセットしたパスワードでWebアプリにログインしたところ、`#<Mysql2::Result:0x...`という記載があったため、シングルクォーテーションを入力したところ、SQLのエラーが表示→SQLi脆弱性あることが確実（そのほか、500エラーでも脆弱な可能性あり）

7. [SQL Injection](../Cheatsheet/Common/SQL%20Injection.md#★Webシェルの書き込み（MySQL）)でリバースシェルにつなげ、内部に侵入

8. `C:\`ディレクトリを列挙し、README.txtからBarracudaDrive 6.5が動作していることがわかったので、[CVE-2020-23834](https://www.exploit-db.com/exploits/48789)（Service Binary Hijacking）を用いて権限昇格