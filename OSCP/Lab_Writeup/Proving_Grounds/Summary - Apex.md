Tecrail RESPONSIVE filemanger の Path Traversal の脆弱性をエクスプロイトするPoCに軽微な変更を加え、OpenEMRというOSSからSQLの認証情報を抽出。
リモートでMySQLに接続し、OpenEMRの認証情報ハッシュをダンプし、クラック。
OpenEMRのRCE (Authenticated)の脆弱性を使い、Foothold を獲得。
Kernel Exploit である PwnKit を用いて権限昇格。

---

1. Port Scanをし、HTTPサービスとSamba、MySQLが動作していることを確認

2. Webにアクセスし、サイトを巡回していると、ユーザー名の候補やOpenEMRのログインフォームを発見

3. ディレクトリスキャンをし、Responsive Filemanager 9.13.4を発見し、Exploit-DBで検索したところ、Path Traversal ([49359 - Exploit-DB](https://www.exploit-db.com/exploits/49359))の脆弱性を発見し、これが対象サーバからファイルをコピーし、特定ディレクトリにファイルをペーストするPoCであることを分析
	- **反省**：PoC に Verified マークがない、かつ、CVE が N/A になっていても、利用可能なケースがあるとは思わず、エクスプロイトを無視していた

4. SMB (Samba) を列挙すると、NULL セッションで READ 可能な Shareがあり、中身を確認すると、HTTP ディレクトリスキャンで発見した Documents というディレクトリに保存されているファイルと全く同じであったため、 Documents ディレクトリは HTTP と SMB 両方からアクセスできるものと推察
	- [[Web Shell#Webアプリのソースコードを取得する考え方と手法概要]]

5. OpenEMR には データベースパスワードが保存されいてる PHP ファイルがあるが、RESPONSIVE filemanager の脆弱性は HTTP サービスにPHPファイルをペーストして中身を読み込むものであるため、HTTP サービスからアクセスすると、PHP ファイルが読み込むと実行されてしまうため、