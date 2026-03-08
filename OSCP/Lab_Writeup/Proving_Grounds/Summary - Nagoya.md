- Webサービスから漏洩したユーザー名をもとに、季節と年号の簡易的なパスワードをスプレーし、有効なアカウントを入手
- SMBで入手したバイナリからパスワードを抽出し、ACL(GenericAll)でRemote Management Usersグループに所属するユーザープリンシパルのパスワードを変更し、ログイン
- Kerberoastで抽出したMSSQLサービスアカウントのNTハッシュでSilver Ticketを作成し、xp_cmdshellでアカウント侵害→SeImpersonateで権限昇格

---

1. ポートスキャンでSMB、kerberos、ldap、httpなどがopenであることを確認

2. httpサービスでユーザー名の一覧を入手
	- 入手したユーザー名の有効性の確認としてkerbruteが有効[💥AD認証システムの攻撃](../../Cheatsheet/🪟Windows/Active%20Directory/💥AD認証システムの攻撃.md#(b)%20Kerbruteによるパスワードスプレー)
	- ASPNETCORE_ENVIRONMENTのErrorページが怪しく見えて時間を浪費
	- Development Modeに切り替えればエラーの詳細情報が表示→Development Modeではないので、無意味
![](../../画像ファイル/Pasted%20image%2020260208172319.png)

3. 季節と年号の簡易的なパスワード(summer2023など）でスプレーし、有効な認証情報を入手
	- **反省**：詰まったときは、OSINTで見つけた情報からパスワードリストを作成し、スプレーする（今回は西暦の情報があった）

4. 入手した認証情報でSMBのSYSVOLにアクセスし、ResetPasswordというバイナリを入手したので、機密情報がないかを探索し、パスワードを入手
```sh
strings <binary>
strings -e l <binary> # windowsのバイナリはUTF-16LEでエンコードされていることがある
```

5. リモートでBloodHoundの情報を入手
```sh
netexec ldap <LDAPserverIP> -u <username> -p '<pw>' --bloodhound --collection All --dns-server <DNSserverIP>
```

6. 入手したユーザーはGenericAllをもつユーザーであったため、Remote Management Usersグループに所属するユーザーのパスワードをリセット
	- BloodHound の Cypher で "Shortest paths from Owned objects" としたら、次の攻撃につながりそうなパスを明示できる
	- [💥AD Exploit](../../Cheatsheet/🪟Windows/Active%20Directory/💥AD%20Exploit.md#ForceChangePassword)

7. リセットしたパスワードでwinRM接続し、`netstat`とすると、MSSQLが動作していることを確認
	- **反省**：
		- ポートスキャンの結果になかったMSSQLが127.0.0.1で動作しているものと思い込み、FWで遮断されている可能性に思考が及ばなかった
		- `netstat -ano | Select-String 127.0.0.1`とし、MSSQL(1433)が動作していることに気づかず、時間を浪費したので、==FWで遮断されている可能性も考慮すること==

8. Kerberoastをし、MSSQLユーザーのパスワードハッシュを入手できたので、リモートからSilver Ticketを発行
	- [🥝Mimikatz](../../Tools/🥝Mimikatz.md#Silver%20Ticket)

9. リモートからMSSQLで接続
```sh
impacket-mssqlclient -k nagoya.nagoya-industries.com
```

10. xp_cmdshellでリバースシェルを獲得
	- [1433 - MSSQL](../../Cheatsheet/Ports%20-%20Service/1433%20-%20MSSQL.md#対話セッション経由のコマンド実行（xp_cmdshell）)

11. MSSQLがSeImpersonatePrivilegeが有効であったため、SigmaPotatoで権限昇格