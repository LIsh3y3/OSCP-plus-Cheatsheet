# AD

## MS01

1. 提供された認証情報でSSH接続し、カレントユーザー情報を検索すると、SeImpersonatePrivilegeが有効、かつ、PrintSpoolerもRunningになっていたため、PrintSpooferを使い権限昇格
	- 試験ではinitial compromiseとして認証情報が１つ提供されるが、それでも内部でどのようなサービスが動作しているか"あたり"をつけるためにも、Nmapは実施すること
	- ==権限昇格ベクターに飛びつきすぎている==→満遍なく浅く

2. Mimikatzで認証情報をダンプし、横展開につかえるNTLMハッシュを入手（クラックはできず）
	- 権限昇格後の列挙で必要以上に時間をかけてしまったので、権限昇格後に横展開のための認証情報を１つ以上入手している状態では、==**権限昇格後の列挙は30分に限定して、ダメなら次のマシンに移動**==

## MS02

1. MS01で入手した認証情報でwinrmのPtHで接続

2. 非常に時間をかけて列挙したが、WinPEASでも手動でも権限昇格ベクターはみつからなかったが、`C:\`を列挙すると、`C:\windows.old`というディレクトリを発見

3. `C:\windows.old`には古いシステムのバックアップが保存されており、`C:\Windows\System32\SAM`及び`C:\Windows\System32\SYSTEM`が読み取り可能であったため、ローカルにコピーし、`impacket-secretsdump`でパスワードハッシュを抽出（クラックはできず）
	- 💡ADにも存在しない、ローカルのC:\Usersディレクトリにも存在しないユーザーの認証情報はメモはするがほぼ使わない

## DC01

- MS02で入手したNTLMハッシュでwinrm接続し、終了

---

# Aero

1. Nmapで見慣れない3000(ppp?), 3001(nessus?), 3003(cgms?)ポートがopenになっていることを確認
	- ?とある場合は、サービスが全然違う可能性も大

2. http 80に対しGobusterで/apiディレクトリを列挙すると3000, 3001, 3003で何が動作しているかが記載されているファイルを発見
	- ==気になるディレクトリは再帰探索すること==
	- wordlistはcommon.txtではなくraft-medium-words.txtを使用すること（==ワードリストの選択で詰まった==）
	- 自分はCMSに飛びかかって必要以上に列挙してしまうので、満遍なく浅く注意（**深すぎる列挙をするようには求められていない**）
	- CMSでエクスプロイトらしきものをよくわからないソースで見つけたのでそれで実行しようとし、ハマる。深く難しいエクスプロイトはないので、まずは列挙する。**GitHubやExploit-DBにないPoCを使うことはない**と考えて良い

3. Aerospikeというサービスが上記のポートで動作しており、公式ドキュメントを読むと、telnetで3003に接続でできるとあるので、接続し、バージョン情報を確認

4. バージョン情報をExploit-DBで検索すると、RCEの脆弱性がヒットするが、Verifiedではなく、実行してもうまくいかなかったので、GitHubで検索してヒットしたPoCを使用すると成功
	- ==Exploit-DBでVerifiedとなっていないPoCでうまくいかなかったら、必ずGitHubで同じCVEを検索する==

5. LinPEASを実行し、'Unknown SUID binary!'と表示されたバイナリをネットで検索すると、Local Privilege EscalationのPoCが出てくるので、それで権限昇格
	- **SUIDは、GTFObinsでヒットしなくても、LinPEASの結果でUnknow SUID binary!とあれば注目すること**

---

# Crystal

1. Nmapを実行すると、http-80で " Git repository found! " と表示されたので、`git-dumper`でリモートリポジトリをローカルにコピーし、列挙すると認証情報を発見

2. 入手した認証情報でターゲットに接続し、LinPEASを実行すると、/optディレクトリに"Unexpected"と書かれたbackupというツールを発見
	- LinPEASで"Linux Exploit Suggester"でよく検出する脆弱性のエクスプロイトに時間をかけすぎた
	- ==LinPEASの"Unexpected"や"Unknown"というキーワードには着目すること==

3. カレントユーザーで`id`とすると、`adm`という特殊グループに所属していることがわかり、backupファイルの読み取り権限があることがわかったので、バックアップファイル(zip)をローカルにダウンロードし、暗号化された`zip2john`でzipファイルを解凍
	- zipファイルはCLI(`unzip`)ではなくGUIで解凍すること（パスワード暗号化zipファイルであるかどうかが、CLIではわからない）

4. Joomla CMSを使っており、"configuration.php"にパスワードがあることが多いことをネットで調べ、解凍したzipファイルにあったconfiguration.phpを探索すると、高権限ユーザーの認証情報を発見
	- ==ドキュメントでは本来あるべきファイルがなければ、==名前の変更/削除/==バックアップ==を疑う==

---

# Hermes

1. NmapでTCPスキャンをするだけでは特に表示されなかったSNMPポートが、UDPスキャンでSNMPポートがopenであると検知
	- **UDPスキャンも必ず実行**

2. SNMPで`snmpwalk`を使い、通常のNmapでは検出されなかったポート(1978)を検出し、ポート番号をネットで検索すると、「WiFi Mouse 1.7.8.5 - Remote Code Execution　References: [EDB-49601]」と表示された（[source](https://www.speedguide.net/port.php?port=1978))

3. PoCを使い、リバースシェルを獲得した
	- pythonのPoCでうまくいかないときは、python2、3とバージョンを指定してみる
	- シェルのペイロードがうまくいかないときは、[[What is the shell]]もしくは[revshells.com](https://www.revshells.com/)で探す

4. WinPEASを実行し、PuTTY sessionで高権限ユーザーの認証情報を入手

---
