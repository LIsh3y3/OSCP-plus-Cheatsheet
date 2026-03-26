# ✋️Hydra実行前に

- hydraによるパスワード攻撃は、アカウントロックのリスク、ターゲットのダウンなど、非常にハイリスクであり、やみくもに実行するものではない
- すべての列挙、攻撃ベクトルは試したか？
- 以下は実行したか？
```
admin:admin
Admin:Admin
administrator:administrator
Administrator:Administrator
デフォルト認証情報(※)
ユーザー名：ユーザー名
ユーザー名小文字：ユーザー名小文字
```
- （※）list of xxxx username password　と検索
- そのほか、簡易的なパスワード（年号＋季節など）を試す

---

# Summery

Webログインフォームへのパスワードアタック(POST)
	（[🐉THC-Hydra](#構成要素)を確認（特にFailure Message））
```zsh
hydra -l <username> -P <wordlist> <target_ip> http-post-form "/<url残り>:<クエリ文字列>:F=<Failure_Message>" 
```
```sh
# 具体例
hydra -l admin -P /usr/share/wordlists/fasttrack.txt 10.10.10.43 http-post-form "/user/login.php:username=^USER^&password=^PASS^:F=Invalid Password!"
```

Basic authenticationへのパスワードアタック
```zsh
hydra -l <username> -P <wordlist> <target_ip> http-get /<URL_path>
```

> [!INFO]
> - ユーザー名とパスワードのプレースホルダーはそれぞれ^USER^, ^PASS^
> - HTTPSの場合はhttp*s*-get
> - JSONとの相性が悪いため、ffufかBurp Suiteを使用
> - Errorメッセージのないログインフォームをパスワードアタックするようなことは少ない

特定サービスへのパスワードアタック
```zsh
hydra -l <username> -P <wordlist> <service>://<target_ip>
```

ユーザー名とパスワードの同時辞書攻撃(時間かかる)
```zsh
hydra -L <username_wordlist> -P <pw_wordlist> -u <service>://<target_ip>
```

デフォルトクレデンシャルの組み合わせ(ftpの例)
```zsh
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://<target_ip>
```

単純なブルートフォース
```zsh
hydra -l <username> -x 1:6:1A <service>://<target_ip>
```

プロキシ経由で攻撃（通信内容をBurp Suiteで確認したいときなどに使う）
```zsh
export HYDRA_PROXY_HTTP=http://127.0.0.1:8080
hydra ...
unset HYDRA_PROXY_HTTP
```

その他よく使うオプション：

| オプション       | 説明                                  |
| ----------- | ----------------------------------- |
| `-v`        | 詳細モード                               |
| `-s <port>` | ポート番号を指定（デフォルト以外を使う場合）              |
| `-u`        | 各ユーザー名に対してすべてのパスワードを試す（ユーザー名列挙時に使用） |
| `-f`        | 最初に成功したらそこで試行停止                     |
| `-t <num>`  | 同時接続数(並行実行数)。デフォルト16。               |

よく使うパスワードワードリスト

| ワードリスト                                                        | 説明                                                           |
| ------------------------------------------------------------- | ------------------------------------------------------------ |
| secLists/Passwords                                            | デフォルトクレデンシャルや、最も使われるパスワード100選など豊富なワードリスト                     |
| dirb/others/names.txt                                         | 8000以上のユーザー名エントリ                                             |
| secLists/Passwords/Common-Credentials/500-worst-passwords.txt | 非常に脆弱（単純）なパスワードを集めたもので、hydraでデフォルト認証情報を使っているサービスをクラックするときに使う |

---
---

# 詳細

## 概要

- [THC-Hydra - Github](https://github.com/vanhauser-thc/thc-hydra)
- 多くのプロトコルやサービス（SSH、FTP、Telnet、HTTP、SMTPなど）に対して、自動的に複数の認証情報を試行し、正しい認証情報が見つかるまで繰り返す
- ブルートフォース攻撃、辞書攻撃、パスワードスプレーなどが可能

---

## 使用方法

### サービスに対する辞書攻撃

- 指定したサービスに対して、ユーザー名とパスワードでログインを試行する
```zsh
hydra -l <username> -P <wordlist>　<service>://<target_ip>
```

### POSTフォームに対する辞書攻撃

- 使い方の表示
```zsh
hydra -U http-post-form
```

- リクエスト内容を把握する必要があり、そのためにBurp Suiteなどでフォームの構造を確認する。
```zsh
hydra -l <username> -P <wordlist> <target_ip> http-post-form "/<URL_path>:<クエリ文字列>:F=<Failure_Messeage>"
```

#### 構成要素

- `<URL_path>`: フォームのあるパス（例 `/login.php` など）
- `<クエリ文字列>`: Burpなどで取得した実際のフォーム内容を使用し、ユーザー名とパスワードはそれぞれ `^USER^` と `^PASS^` プレースホルダーに置換する

![](../Images/Pasted%20image%2020230519113243.png)

$$POSTフォームのクエリ文字列$$

- `F=[Failure Message]`: ログイン失敗を判断する識別子。ステータスコードも使用可能。<u>失敗のときのみ表示される内容でないといけない。</u>
	- 💡誤検知を減らすために：
		- 一意に
		- 短く
		- ==`username`や`password` というキーワードは省く==(ログイン成功にも含まれる可能性大)
- `S=[Success Message]`: ログイン成功を判断する識別子。Fでうまくいかない場合に使う。

### BASIC認証における辞書攻撃

```zsh
hydra -l <username> -P <wordlist> <target_ip> http-get /<URL_path>
```

### ブルートフォース

```zsh
hydra -l <username> -x <最小長>:<最大長>:<文字セット> <service>://<target_ip>
```

|記号|意味|
|---|---|
|`1`|数字（`0-9`）|
|`a`|小文字（`a-z`）|
|`A`|大文字（`A-Z`）|
|その他の文字|その文字自体が候補になる|

#### 文字セット例

-  例1：`-x 1:2:a1`
	- パスワード長は1文字〜2文字
	- 使う文字は 小文字アルファベット + 数字

- 例 2：`-x 3:4:aA1!`
	- パスワード長：3〜4文字
	- 使用文字：
	    - 小文字（`a-z`）
	    - 大文字（`A-Z`）
	    - 数字（`0-9`）
	    - 感嘆符（`!`）
	→`aA1 AA! zZ9 123! a1B! ...`
