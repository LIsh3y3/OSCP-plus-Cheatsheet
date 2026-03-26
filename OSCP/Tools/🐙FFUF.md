- [FFUF - GitHub](http://github.com/ffuf/ffuf)

# 特徴

- Gobusterのようにディレクトリを発見、または脆弱性の検出し、さらにパスワード攻撃などにも使われる汎用ツール
- SOCKSプロキシでのスキャンに便利：参考🔗：[HTTP proxy vs SOCS proxy - Developer.io](http://dev.classmethod.jp/articles/socks-proxy-and-http-proxy/)

---

# ディレクトリ・ファイル探索

隠しディレクトリやファイルを探す基本例
```zsh
ffuf -c -w <wordlist> -u http://<target.com>/FUZZ
```
- `-w`: ワードリストを指定
- `-u`: `FUZZ`の位置にワードが挿入されるURL

拡張子付きで探索
```zsh
ffuf -c -w <wordlist> -u http://<target.com>/FUZZ -e .php,.html,.txt
```
- `-e`: `.php`や`.html`などを付けて試す

再帰探索を有効にする
```zsh
ffuf -c -w <wordlist> -u http://<target.com>/FUZZ -recursion -recursion-depth <num>
```
- `-recursion`: サブディレクトリも対象にする
- `-recursion-depth`: 探索の深さを指定

## ディレクトリ探索の一連のコマンド

SOCKSプロキシ実行
```zsh
ffuf -o ffuf.json -recursion -recursion-depth 2 -x socks5://localhost:<Port> -e .php,.jsp,.txt,.cgi,.asp,.aspx -u http://<target_IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

# pretty print
python -m json.tool < ffuf.json | tee pretty_result.json

# 特定のレスポンスコードで絞る
cat ffuf.json | jq '.results[] | select(.status == 200) | .url'
```
- レスポンスのフィルタリングは、ffuf実行時点でも可能: [🐙FFUF](#レスポンスのフィルタリングとマッチング)

---

# サブドメイン・バーチャルホスト探索

サブドメインの検出
```zsh
ffuf -c -w <subdomains.txt> -u http://<target.com>/ -H "Host: FUZZ.<target.com>"
```

バーチャルホストの検出
```zsh
ffuf -c -w <vhosts.txt> -u http://<target.com>/ -H "Host: FUZZ"
```

---

# パラメータ・値のファジング

脆弱性の検出に使う場合は、狙った脆弱性に適したwordlistを選択すること。
 `/usr/share/seclists/Fuzzing/`をよく使う。

GETパラメータのファジング
```zsh
ffuf -c -w <wordlist> -u http://<target.com>/page.php?FUZZ=test
```
- すべてErrorsの場合は、ワードリストと指定URLでURL形式が崩壊しているか、(例：`http://<target.com>//etc/passwd`)

POSTでの値ファジング
```zsh
ffuf -c -w <wordlist> -X POST -d "username=admin&password=FUZZ" -u http://<target.com>/login.php
```
- `-X`: メソッド指定（POSTなど）
- `-d`: リクエストボディ指定

multipart/form-dataのときのファジング

![](../Images/Pasted%20image%2020260108132852.png)

```zsh
# HTTPリクエストを丸ごとコピーし、FUZZと記入(例としてfuzz.reqと保存)
POST /api/v1/reset/index.php HTTP/1.1
...

------WebKitFormBoundary8a04b3vGdGjphqRa
Content-Disposition: form-data; name="user"

FUZZ
------WebKitFormBoundary8a04b3vGdGjphqRa--
```
```zsh
ffuf -c --request-proto http -w <wordlist> -request fuzz.req
```

---

# レスポンスのフィルタリングとマッチング

ステータスコードでマッチ
```zsh
ffuf -c -w wordlist.txt -u http://<target.com>/FUZZ -mc 200
```

404レスポンスを除外
```zsh
ffuf -c -w wordlist.txt -u http://<target.com>/FUZZ -fc 404
```

特定のワード数を除外
```zsh
ffuf -c -w wordlist.txt -u http://<target.com>/FUZZ -fw <word_num>
```
- `-fw`：複数指定、範囲指定が可能（範囲指定例）`-fw 1000-1200`

## アウトプットの、整形・抽出

デフォルトアウトプットはjson
（`ffuf -o ffuf.json...）

### prettyに整形
 
```zsh
python -m json.tool < ffuf.json | tee pretty_ffuf.json
```

### 抽出

- レスポンスコード 200のurl
```zsh
cat ffuf.json | jq '.results[] | select(.status == 200) | .url'
```

- ワード数が特定値以外のペイロードを抽出
```zsh
cat ffuf.json | jq '.results[] | select(.words != 4632) | .input.FUZZ'
```

---

# その他テクニック

標準入力からワードリストを流す
```zsh
seq 1 1000 | ffuf -c -w - -u http://<target.com>/api/user?id=FUZZ
```
- `-w -`: 標準入力をワードリストとして使う

# 補足

- 似たツールとして[wfuzz](http://wfuzz.readthedocs.io/en/latest/)があるが、fuffの方が良いと考える。
	- [理由](http://jpn.nec.com/cybersecurity/blog/210604/index.html)