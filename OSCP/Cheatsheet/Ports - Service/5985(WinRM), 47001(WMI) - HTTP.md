- 関連ノート：
	- [💥Lateral Movement & Persistance in AD](../🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md)

>[!NOTE]
>5985/tcp と 47001/tcp が `http` と表示されていても、ブラウザでアクセスして意味のあるページは出てこない。

---

# ログイン情報の有効性検証

```zsh
# winrm/wmi
netexec <winrm|wmi> <target_subnet(192.168.xx.0/24)> -u <username> -p '<password>' --continue-on-success
```

---

# WinRMポートの活用

- evil-winrmでの接続
```zsh
evil-winrm -i <target_IP> -u '[domain\]<username>' -p '<password>'
```

- インタラクティブなシェル（リバースシェル）への切り替え
	- →[Windows PowerShellのリバースシェル w/ Nishang](../Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Windows%20PowerShellのリバースシェル%20w/%20Nishang)
	- →[3389 - RDP](3389%20-%20RDP.md)
	- WinRM接続状態では、非インタラクティブであるため、切り替えがベター（※）
		- （※）`powershell -ep bypass -c <cmd>`と実行ポリシー回避を都度都度実行する必要がある

- SeImpersonateもしくはSeAssignPrimaryTokenが付与されている場合は、[Token Impersonationによる権限昇格](../🪟Windows/💥Windows%20Privilege%20Escalation.md#Token%20Impersonationによる権限昇格)

- 横展開のための[WinRM & WMI](../🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md#WinRM%20&%20WMI)

---

# 各ポートの正体と挙動

## `5985/tcp` - WinRM (Windows Remote Management)

- Windowsのリモート管理用ポート
- HTTPベースのSOAP通信を使っており、ブラウザからアクセスしても何も役に立たない
- 通信の実体はPowerShell Remotingなどの管理系機能
- 通常、認証なしで操作するのは不可能

- evil-winrmで接続できる可能性がある
>[!NOTE]
>攻撃者のペイロードなどの操作によって追加されたAdminユーザーに接続できない場合は、winrmがドメイン認証のみを許可している可能性が高い。

##  `47001/tcp` - WinRM + WMI 通信用

- WinRMとWMIを連携させた管理通信で使われる
- ブラウザで叩くと `Not Found` が返るのは正常
- SOAP通信専用で、ユーザー向けのサービスではない
- 認証情報がなければ何もできない。こちらも攻撃には使いづらい

### 補足： なぜNmapが `http` と表示するか

- サービスのヘッダーや応答がHTTPプロトコルの形式（例：`Microsoft-HTTPAPI/2.0`）になっている
- Nmapはこれを見て `http` と判定するが、単に通信形式がHTTPというだけで、Webアプリが動いているわけではない
