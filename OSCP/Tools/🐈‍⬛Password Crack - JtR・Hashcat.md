>[!TIP]
>パスワードクラックに使用する wordlsit は fasttrack.txt の後、rockyou.txt。

# ✂️JtR vs 🐈‍⬛Hashcat比較表

それぞれが異なるアルゴリズムをサポートするので、両方に精通する必要がある

| 項目        | John the Ripper（JtR）                    | Hashcat                     |
| --------- | --------------------------------------- | --------------------------- |
| 開発方針      | CPUベース設計（GPU対応も可能）                      | GPUベース設計（CPUも対応）            |
| 性能（対応ハード） | CPU高速化に最適化、GPU対応はやや限定的                  | GPU最適化（CUDA / OpenCL 利用で高速） |
| GPUドライバ依存 | なし                                      | あり（CUDAやOpenCLが必要）          |
| 対応ハッシュ形式数 | 非常に豊富（特にUnix系）                          | 非常に豊富（JtRと同程度、少し多い場合も）      |
| ルールベース攻撃  | 標準対応（`john.conf`などでルール記述）               | 対応（`.rule`ファイルで指定）          |
| モード切替     | 自動（hash形式の自動判別）                         | 明示的に `-m` で指定が必要            |
| ハッシュ変換ツール | `*2john` 系（e.g. `zip2john`, `ssh2john`） | JtRのツールと連携して使用することが多い       |

# ✂️John the Ripper 

## Cracking Basic Hash

### wordlistモード

指定したワードリストでクラッキングするモード。

1. 使用可能なフォーマットの検索
```zsh
john --list=formats | grep -iF <keyword>
```
- `-i`: 大文字小文字を無視
- `-F`: キーワードを固定文字列として扱う（正規表現無効）

2. `--format=<format>` を指定
```zsh
john --format=<format> --wordlist=<wordlist> <hash_file>
```

- 補足：hashアルゴリズム指定なしでも、自動でハッシュタイプを識別してくれるが、不正確な結果につながる可能性があるため、[🐈‍⬛Password Crack - JtR・Hashcat](#Hashの識別)を実施する方がよい
```zsh
john --wordlist=<wordlist> <hash_file>
```

### Single Crack モード

- Single Crackモード：ユーザー名から得られる情報を使って、ユーザー名に含まれる文字や数字を少し変えたもの(※)を元値としてクラッキングを試行する
	- (※) *Word mangling*という

GECOSフィールドを利用し候補を生成のうえクラック
```zsh
# 例：john --single --format=sha512crypt shadow_has_homedir
john --single --format=<format> <file>
```
- ファイルの中身は `username:hash`に整形する（余計なフィールドは削除する）
-  [GECOSフィールドとは](https://docs.oracle.com/cd/E19683-01/817-4909/userconcept-74705/index.html)

---

## ワードリストの改変 w/ JtR

### カスタムルール

- [🐈‍⬛Password Crack - JtR・Hashcat](#Single%20Crack%20モード)では、word manglingといい、任意の値をパスワードとして"ありえそうな"値に変換していた
- カスタムルールを作成することで、ワードリストを任意の方式に変更することができる
	- 例えば、ターゲットのパスワードポリシーなどにあわせて、先頭文字を大文字にする、記号を使う、などが考えられる。
	- ==パスワードポリシーに沿ったワードリストは所要時間削減の効果大！==

### カスタムルールの作成 w/ JtR

1. `/usr/share/john`にcd

2. 管理者権限で、`john-local.conf`を作成
```zsh
sudo touch john-local.conf
```
- 設定ファイルである`john.conf`が読み込まれた後、`john-local.conf`が読み込まれ、内容を上書き・追加する(競合した場合は`john-local.conf`が優先される)
	- ⚠️`john.conf`はアップデートで自動更新されるので、直接編集は非推奨

3. `john-local.conf`にルールを定義し保存
```
[List.Rules:<RuleName>]
ここにルール

```
- ⚠️configを読み込むために最後に`\n`が必要なので、改行を入れること

4. ルールが作成されたことを確認
```zsh
john --list=rules | grep -i <RuleName>
```

#### ルールの文法

- 文法の詳細は[Wordlist rules syntax](https://www.openwall.com/john/doc/RULES.shtml)
	- 💡Hashcatの文法も共通で使用可能
- season+ year(2020 | 2021) + special charとしたいとき：
```
Az"202[0-1][!@#$]"
```

- `Az`: A+Numで以降`""`で囲まれた文字列をどの位置に挿入するか決める。zの場合は末尾。
- `[⚪︎ ~ △]`：⚪︎~△の範囲で一つ選択する
- `[abcd]`: abcdの中で一つを選択する

---

## ルールの使用 w/ JtR

- ルールを指定する
```zsh
john --format=<format> --wordlist=<wordlist> --rule=<rule> <hash_file>
```

- 自作のカスタムルール以外にも、標準で作成されたルールを使うことができる。使えるルールのリストアップ
```zsh
john --list=rules
```
- 標準作成のルールの中身は`john.conf`に記載

参考：[🐈‍⬛Password Crack - JtR・Hashcat](#補足：不要な文字行の削除(`sed`))

---

## ハッシュ値の整形

### 前提事項・注意事項

- ハッシュ値をJtRのサポートする形式に整理整頓しないと、クラックできないので、サポートする形式に整形するためのツールがある
- 以下に紹介しているツール以外にもまだまだある：[Tools : John - Kali.org](https://www.kali.org/tools/john/)

- 🚨整形ツールを使用した場合で、JtRではなくHashcatでクラックするときは、Hashcatのハッシュフォーマットを確認の上、必要であればユーザー名を削除する必要がある：
	- 例：keepass2johnで出力されたハッシュの先頭にユーザー名が付与されているが、[Hashcat wiki](https://hashcat.net/wiki/doku.php?id=example_hashes)を確認するとユーザー名が不要であることがわかる（モード13400）
```
Database:$keepass$*2*60*0*...
```
	↓
```
$keepass$*2*60*0*...
```

### ツール使用例

Zipファイル：*zip2john*
```zsh
zip2john <zip_file> > <output_file>
# 単一ファイルを選択する場合（例：zip2john -o backup/web.config backup.zip >...)
zip2john -o <directory>/<file> <zip_file> > <output_file>
```
- 💡CLIで`unzip`すると、パスワード付きzipファイルは解凍エラーが出るだけで原因がわからないため、GUIで解凍すること

SSH Private Keys：*ssh2john*
```zsh
ssh2john <PrivateKey> > <output_file>
```

`/etc/shadow`と`/etc/passwd`（Linux）：*unshadow*
```zsh
unshadow <path_to_/etc/passwd> <path_to_/etc/shadow> > <output_file>
```
- 💡`/etc/passwd`のユーザーのすべてクラックしようとすると時間がかかるため、特定ユーザーに絞ってJtRを実行

kdbxファイル：*keepass2john*
```zsh
# 例：keepass2john Database.kdbx > crackable.txt
keepass2john <file.kdbx> > <output_file>
```
- `Database.kdbx`をクラックするとmasterパスワードを入手できるので、 `keepass2` で`Database.kdbx`ファイルを開き、master パスワードで中身のパスワードを確認する

![](../Images/Pasted%20image%2020260220163242.png)
$$keepass2でパスワードを確認$$

#### 補足：/etc/shadowについて

- /etc/shadowファイルは、Linuxマシンにおいて、パスワードハッシュが保存されるファイル
- 最終パスワードの変更日やパスワードの有効期限情報などが保存される　
- 通常はroot権限でしかアクセスできない
- 形式
```zsh
<username>:<password_hash>:<UID>:<GID>:<追加情報>:<ホームディレクトリ>:<ログイン時に使用するシェルのパス>
```

---

## クラック済みのパスワードを表示

```zsh
john --show --format=<format> <hashファイル>
```
🚨すでにクラック済みのパスワードは、同じコマンドを実行してもパスワードが表示されなくなるため、上記のコマンドを使用

---
---

# 🐈‍⬛Hashcat

## 必要な情報

1. 対象ハッシュのタイプ（例：MD5、SHA1、bcryptなど）: [🐈‍⬛Password Crack - JtR・Hashcat](#Hashの識別)、[Hashcat example](https://hashcat.net/wiki/doku.php?id=example_hashes)
2. 使用するワードリスト（例：rockyou.txt）
3. 使用する攻撃モード（例：辞書攻撃、ブルートフォースなど）

---

## 辞書攻撃 w/ Hashcat

基本構文
```zsh
# 具体例: hashcat hash.txt -m 400 -a 0 /usr/share/wordlists/rockyou.txt --force
hashcat <hash_file> -m <hash_mode> -a 0 <wordlist> --force
```
- `-a` : 攻撃モードで、0は辞書攻撃
- `-m` : 解析するハッシュのモード（[Hashcat example](https://hashcat.net/wiki/doku.php?id=example_hashes)）

---

## ブルートフォース攻撃 w/ Hashcat

基本文法
```zsh
hashcat <hash_file> -a 3 -m <hash_mode> <charset>
```
- `-a 3` はbrute force攻撃

1. charsetの確認
```zsh
hashcat --help
```
↓出力抜粋
```
- [ Built-in Charsets ] -

  ? | Charset
 ===+=========
  l | abcdefghijklmnopqrstuvwxyz [a-z]
  u | ABCDEFGHIJKLMNOPQRSTUVWXYZ [A-Z]
  d | 0123456789                 [0-9]
  h | 0123456789abcdef           [0-9a-f]
  H | 0123456789ABCDEF           [0-9A-F]
  s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
  a | ?l?u?d?s
  b | 0x00 - 0xff
```

2. charsetの使用
例：未知の数字の集合で構成されるパスワード
```zsh
hashcat <hash_file> -a 3 -m <hash_mode> -i ?d?d?d?d?d?d?d
```
- `-i`：文字数を順に増やしていく（このオプションがないと、固定長でのブルートフォースになる）

例：攻撃者が既知の文字列を組み合わせたパスワード
```zsh
hashcat <hash_file> -a 3 -m <hash_mode> -i ?u?l?l?l<word>
```

### 参考： Keyspaceテクニック w/ crunch

- Keyspace: 文字、数字、記号など、パスワードや秘密情報に使用される可能性のあるすべての要素を含む可能性のある組み合わせ
- *crunch*：ワードリストを作成するためのツール

基本構文
```zsh
crunch <min_char_length> <max_char_length> -t <文字セット> -o <output>
```

具体例：
```zsh
crunch 6 6 -t pass%%
```
	↓
```
pass00
pass01
pass02
pass03
```

| 記号     | 意味           |
| ------ | ------------ |
| `%`    | 数字（`0-9`）    |
| `@`    | 小文字（`a-z`）   |
| `,`    | 大文字（`A-Z`）   |
| その他の文字 | その文字自体が候補になる |

---

## クラック済みのパスワードを表示

- 一度クラック済みのパスワードを表示する
```zsh
hashcat <hash_file> -a <attacke_mode> -m <hash_mode> <wordlist> --show
```

---

## ワードリストの改変 w/ Hashcat

- 手段：カスタムルールを作成してワードリストを改変する
	- ==パスワードポリシーに沿ったルールは所要時間削減の効果大！==

- 💡**作成済みのルール**を使うこともできる（`best64`、`rockyou-30000.rule`など）
```zsh
# ルール一覧の表示
ls -la /usr/share/hashcat/rules/
```

### 基本的な流れ

1. ワードリストを用意する（例）
```
passsword
verysecure
```

2. 使用可能なルール関数詳細は[Hashcat Wiki](https://hashcat.net/wiki/doku.php?id=rule_based_attack)を参照し、ルールを作成
	- 必要に応じて改行：[🐈‍⬛Password Crack - JtR・Hashcat](#ルールの改行の解釈について)
```
c $1 $#
```
- ルールは1つの関数にまとめて書けない
	- ✖ NG例：`$1234$`
	- ⚪︎ OK例：`$1 $2 $3 $4`

3. ルール適用後のワードリストの出力（クラックはしない）
```zsh
hashcat -r <example.rule> --stdout <wordlist>
```
	↓
```
Password1#
Verysecure1#
```

4. クラックする
```zsh
hashcat <hash_file> -r <example.rule> -m <mode> -a <attack_mode> <wordlist> --force
```

### ルールの改行の解釈について

1行でルールを記載したときと、改行したときでは、パスワードへのルールの適用方法が異なる

- 1行：1つのパスワードに、同じ行のすべてのルールを連続して適用する
```
$1 c
```

- 改行：1つのパスワードに、各ルールをそれぞれ別々に適用する（バリエーションが増える）
```
$1 c $!
$2 c $!
$1 $2 $3 c $!
```

- 例：`$1`、`c`のとき
```zsh
# 1行に記載した場合
Password1
Iloveyou1

# 改行した場合
password1
Password
iloveyou1
Iloveyou
```

### 補足：不要な文字行の削除(`sed`)

```zsh
# 数字からはじまる文字列の行を削除
sed -i '/^1/d' <wordlist>

# すべて数字のみの行を削除
sed -i '/^[0-9]\+$/d' <wordlist>

# 8文字未満のパスワードを削除
sed -i '/^.\{1,7\}$/d' <wordlist>

# 英字だけの行を削除（数字・特殊文字なし）
sed -i '/^[a-zA-Z]\+$/d' <wordlist>

# 特殊文字を含まない行を削除
sed -i '/^[a-zA-Z0-9]*$/d' <wordlist>
```
- `-i`：上書き保存する

---

## Hashcatのトラブルシューティング

- すぐに辞書攻撃が終わり、StatusがExhaustedになる
	- →ワードリストの候補見直し
	- →クラックできないハッシュ
- Not enough allocatable device memory or free host memory for mapping.
	- →メモリ不足のため、他のプロセスを一旦終了する
- Token length exception
	- →`-m`で指定したハッシュタイプが違う可能性があるため、[🐈‍⬛Password Crack - JtR・Hashcat](#Hashの識別)を実施 
	- →HashcatがサポートしないHashアルゴリズムである可能性があるため、JtRを使用

---
---

# Hashの識別

## 前提事項・注意事項

- JtRでも自動でHashタイプを認識しないときがあるし、HashcatではHashアルゴリズムの指定が必須

- Hashの識別には[hash-identifier](https://www.kali.org/tools/hash-identifier/) もしくは [hashid](https://www.kali.org/tools/hashid/)というツールを使う
	- ==2つのツールで確認すること==（異なる結果を返す場合は、信頼性が低い）
	
- ツールでもわからない場合は、[Hashcat example](https://hashcat.net/wiki/doku.php?id=example_hashes)で確認

- 💡識別結果のHashアルゴリズムでクラックできないときは？
	- 誤ったアルゴリズムである可能性がある
	- インターネットで検索し、クラック対象のサービスが、どのアルゴリズムを使ってHash化しているかを確認する
	- ==Hashcatではサポートしていない==場合はJtRを使う（逆もまた然り）

## Hashの識別方法

Hashid
```zsh
hashid <hash_file>
```

hash-identifier
```zsh
hash-identifier
```

その他、単語ベース（Keepass、ssh等）で検索するときはhashcatを使う
```zsh
hashcat --help | grep -i "<keyword>"
```
- 複数結果が出た場合は、ハッシュ前半の値が一致するものを選ぶ
- カッコ内に記載(`$0$`など)
```
22911 | RSA/DSA/EC/OpenSSH Private Keys ($0$)                      | Private Key
22921 | RSA/DSA/EC/OpenSSH Private Keys ($6$)  
```

### 補足：GUI・参考サイト

- hash-identifierのGUI版：[GUI hash-identifier](https://hashes.com/en/tools/hash_identifier)
- crack station (https://crackstation.net/)
	- ハッシュ値を入力することで元の値と使用されているハッシュ計算アルゴリズムが閲覧可