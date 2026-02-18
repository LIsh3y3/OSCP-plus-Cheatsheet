# Gobusterとは

- Webサーバに隠された次のよう情報をブルートフォースで列挙するためのツール
	- ディレクトリ：dirモード
	- サブドメイン：dnsモード
	- バーチャルホスト：vhostモード
- [Gobuster - GitHub](https://github.com/OJ/gobuster?tab=readme-ov-file)

---

# ディレクトリ / ファイル列挙：dir モード

## dirモードの基本・目的

- Webサーバ上の隠れたディレクトリ / ファイルを探索する
- 🚨**再帰探索はできない**ため、一階層下のディレクトリしか探索しない
	- 手動で`-u`の値を都度変えてスキャンするか、[[🦝FeroxBuster]]を使う

## ディレクトリ・ファイル列挙コマンド

隠されたディレクトリと特定の拡張子を持つファイルを列挙する
```zsh
# ファイルアクセスエラー回避のため、同時に開けるファイルの最大数を調整
ulimit -n 8192
```
```zsh
# 気になるディレクトリがあれば-uに指定して再度実行すること
gobuster dir -u http://<TargetIP|Domain>:<Port>/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -o gobuster.txt -x '<extensions>'
```
- `-x`にはターゲットのテクノロジースタック等に応じて拡張子を指定する（[[#拡張子リスト]]）

ワードリストとパターンリストを組み合わせて、APIのバージョンパスなどを列挙する
	（ファイル名：pattern.txtとする） 
```pattern.txt
{GOBUSTER}/v1
{GOBUSTER}/v2
```
```zsh
gobuster dir -u http://<TargetIP|Domain>:<Port>/ -w <wordlist> -p pattern.txt
```

## dirモードラブルシューティング / Tips

- 何も見つからない時は、他のwordlistを試す（[[#補足：利用ワードリスト候補]]）
- 404などの出力が多いときは`-s`、`-b`を使いフィルタリング
- エラーが多いときは`--no-error`を使うか、`-t`を減らす

### 拡張子リスト

| スキャンタイプ  | 拡張子リスト                                                    |
| -------- | --------------------------------------------------------- |
| 包括的      | html,htm,txt,sh,php,cgi,asp,aspx,jsp,pl,py,pdf            |
| PHP      | php,html,htm,txt,pdf,bak,old,inc,phtml                    |
| ASP.NET  | aspx,asp,html,htm,txt,pdf,config,cs,ashx,asmx             |
| JSP/Java | jsp,html,htm,txt,pdf,do,action,jspx,jsf                   |
| 静的サイト    | html,htm,txt,pdf,xml,json,css,js                          |
| 設定ミス検出   | bak,old,backup,swp,tmp,zip,tar.gz,sql,log,conf,config,ini |
| API/データ  | json,xml,yaml,yml,csv,txt                                 |
| 軽量/高速    | html,php,txt,pdf                                          |

### 補足：利用ワードリスト候補

↓ワード数で昇順
```zsh
# ワード数：4614
/usr/share/wordlists/dirb/common.txt
# ワード数：63088
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
# ワード数：220560
/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

---

# サブドメイン列挙：dns モード

- サブドメインはメインドメインとは異なるセキュリティ設定になっている可能性があり、 パッチが適用されていない脆弱なサブドメインを探す
- 使用ワードリスト：/usr/share/seclists/Discovery/DNS/

```zsh
gobuster dns -d <Target_domain> -i --wildcard -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```
- `-i`：IPアドレスも表示
- `--wildcard`：存在しないサブドメインが返ってきた場合、ワイルドカードDNSという旨の警告を出す

---

# バーチャルホスト列挙：vhost モード

- バーチャルホスト（1つのサーバー上で複数のドメインを運用）を発見する
- 一部のサーバーでは、異なるバーチャルホストごとに異なるコンテンツが配置されているため、追加の攻撃対象を発見できる
	- たとえば、ホストAは堅牢だが、ホストBは脆弱で Web shell 実行可能など
- 使用ワードリスト：`/usr/share/seclists/Discovery/DNS/`
```zsh
gobuster vhost -u http://<TargetIP|Domain>:<Port>/> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

>[!TIP]
>ディレクトリ構成を閲覧でき、以下のようにiisstart.htmと他のサービスのディレクトリ (umbracoなど) が同じ階層にあれば、vhostの可能性大
```
drwxrwxrwx 1 ftp ftp               0 Nov 30  2022 aspnet_client
-rw-rw-rw- 1 ftp ftp             703 Nov 29  2022 iisstart.htm
-rw-rw-rw- 1 ftp ftp           99710 Nov 29  2022 iisstart.png
-rw-rw-rw- 1 ftp ftp              74 Dec 01  2022 security.txt
drwxrwxrwx 1 ftp ftp               0 Nov 29  2022 umbraco
```

## 補足：バーチャルホストとは

- 1 台のマシン上で (company1.com と company2.com のような) 2つ以上の Web サイトを扱う運用方法のこと
- 以下 Apache 例の場合 (/etc/apache2/sites-available/xxxx.conf)：
	- IPアドレスでアクセス → /var/www/html/umbraco のindexが表示
	- skylark.jpでアクセス →/var/www/html のindexが表示
```xml
<VirtualHost *:80>
    DocumentRoot /var/www/html/umbraco/
</VirtualHost>

<VirtualHost *:80>
    ServerName skylark.jp
    DocumentRoot /var/www/html/
</VirtualHost>
```

- 設定ファイルの基本的な場所：
	- Apache： /etc/apache2/sites-available/xxxx.conf
	- IIS：
		- %SystemRoot%\System32\inetsrv\config\applicationHost.config
		- web.config
		- appsettings.jsonなそ

---

# Error

- 参照：[Gobuster Trouble Shooting - GitHub](https://github.com/OJ/gobuster?tab=readme-ov-file#-troubleshooting)

| エラー                                                                                       | 想定原因                                                                           | 対策                                                                  |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| the server returns a status code that matches the provided options for non existing urls. | ディレクトリ探索のために存在しないはずのディレクトリにリクエストしたが、エラーページ（status 200）が返るため、ディレクトリの存在有無を判断できない | `<url> => 200 (Length: <num>).`と表示されるので、`--exclude-length <num>`とする |
| content deadline exceeded                                                                 | ・URLの末尾にスラッシュがないことで、ディレクトリなのかファイルなのか判断するのに時間がかかる                               | ・/をつける<br>・`-t 64`等に減らすか、`-t`を使わない                                  |
| unable to connect to URL                                                                  | ・スレッド数（`-t`）が大きい                                                               | ・`-t 64`等に減らすか、`-t`を使わない                                            |
| error on running gobuster on http://<IP\>:80/: timeout occurred during the request        | ・IPアドレス誤り<br>・スレッド数（`-t`）が大きい                                                  | ・IPアドレスを再確認<br>・`-t 64`等に減らすか、`-t`を使わない                             |
