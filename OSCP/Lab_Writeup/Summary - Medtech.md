1. 外部公開WebサイトからGobusterでlogin.aspxページを明らかにする

2. ログインページにSQLiを試行し、MSSQLのxp_cmdshellでWEB02に初期侵入する
	- 正常なクエリ、異常なクエリでもデータの抽出には至らなかったため、詰まった

3. WEB02のmssqlユーザー（サービスユーザー）は"SeImpersonatePrivilege"であったこと、Print Spoolerのサービスが動作していることから、PrintSpooferで権限昇格した
	- SigmaPotatoは動作が不安定だった

4. 権限昇格したWEB02上でpowerview.ps1を使い、ADユーザーを明らかにした

5. 管理者権限をもったWEB02錠でmimikatzを実施し、joeのNT hashを抽出し、解析した

6. joeの認証情報でNetExecでpassword sprayをしたところ、FILES02で管理者権限を持つことを確認し、winrmでログインした

7. FILES02で興味深いファイル（バックアップログ）があり、その中にwario,toad, daisy,goombaのNTハッシュがあった
	- warioのハッシュは解析できたが、toad, daisy, goombaのNTハッシュ解析はできなかった
	- 上記３ユーザーは==ADのユーザーではないことはわかっていたので、優先度を下げても良かった==かもしれない

8. warioでスプレーし、CLIENT02にログインできることを確認した

9. CLIENT02でwinPEASを実行したところ、サービスの実行ファイルが書き換え可能であったため、ペイロードに書き換え、高権限のリバースシェルを獲得した

10. これまで発見したユーザーリストとパスワードリストでNetExecのパスワードスプレーを実行したところ、yoshiがwarioのパスワードでCLIENT01で管理者権限をもつことがわかった
	- ==異なるユーザーが同じパスワードを使い回している可能性を考慮すること==
	- CLIENT02上で特に有益な情報がなく、ここで時間を要した
	- これまでアクセスできなかった`Administrator.Medtech`ディレクトリにアクセスでき、その中に`hole.txt`というファイルがあった
		- →これはラビットホールで、中に`leon:rabbit!:)` と記載があった

11. yoshiの認証情報でrdpでNetExecのパスワードスプレーを実行したところ、DEV04でRDP接続可能なことがわかった
	- NetExecはSMBしかできないと思い込んでいたことで時間がかかった
		- →展開先のopenポートを確認し、それにあわせたネットワークプロトコルでパスワードスプレーすべき

12. DEV04で`C:\TEMP\`ディレクトリにあるbackup.exeがyoshiによって実行可能であることがわかった（`Get-ACL`でOwnerはAdministratorのファイル）

13. backup.exeがサービスやスケジュールタスクで実行されていなかったので、プロセスを`Watch-Command`でリアルタイムに監視したところ、定期的に一瞬だけbackup.exeがSI(Session ID) 0(高権限の可能性有)で実行されていることがわかった
	- `Watch-Command`の存在を知らず、時間がかかった

14. backup.exeは`icacls`で確認したところ、yoshiによって編集可能だったため、リバースシェルペイロードに書き換えたところ、DEV04の管理者権限のリバースシェルを獲得した

15. DEV04上でmimikatzで認証情報をダンプしたところ、leonの認証情報(`rabbit:)`)を発見し、leonがDomain Adminsグループに所属することはBloodHoundの結果でわかっていたので、この認証情報でDC01にアクセスした
	- CLIENT01で発見した`hole.txt`に記載されていた`rabbit!:)`とは微妙に違う認証情報であり、ここでrabbit holeであったことを確信

16. DC01で列挙していくと、`credentials.txt`というファイルを発見し、そこでWEB01のoffsecの認証情報を発見した

17. Domain Adminsの権限をもっていたので、DCsync攻撃で全てのユーザーのNTハッシュをダンプした
	- mario、peachのNTハッシュをクラックしたが、netexec sshでパスワードスプレーしたところ、これは使えないものだった

18. NetExec sshでまだ侵害できていないマシンにパスワードスプレーしたところ、WEB01でoffsecはローカル管理者権限をもっていたため、WEB01を侵害完了

19. VM5とVM8（両方sshのみopen）がクラックできておらず、詰まったが、VM5のsshに対しoffsecユーザーで`hydra`で単純なパスワードワードリストを用いて辞書攻撃したところ、ログイン成功

20. VM5のoffsecユーザーはsudoでopenvpnが実行可能だったため、GTFOBinsで手法を検索し、権限昇格した

21. VM5でmarioの`id_rsa`があったため、`ssh -i id_rsa mario@<VM8IP>`でログインでき、すべてのマシンを侵害完了