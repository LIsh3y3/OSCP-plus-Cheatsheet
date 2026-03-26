画像引用元（credit）：🔗[Active Directory Basics - TryHackMe](https://tryhackme.com/room/winadbasics)

# Active Directoryとは

- Windowsドメインは、ネットワーク上の複数のコンピュータを一元管理する仕組み
- Active Directoryがユーザー、コンピュータ、サービス、リソースの情報を中央管理する
- Active Directoryを運用するサーバをDomain Controller (DC) と呼ぶ

主なメリット：
1. ID管理の一元化
2. セキュリティポリシーの集中管理

デメリット：
1. Attack Surfaceが増える
	- SMB、RPC、LDAP、Kerberosなどのポートが、内部向けにデフォルトopenになる
		- SMBで横展開をしやすい
		- ※closedにすることも可能

具体例：
- 学校などで共通の認証情報を使ってログインできるのは、各PCではなくADで一元管理しているため
- ADはControl Panelへのアクセス制御なども行える

### 補足：ワークグループとは（ADではない）

- Active Directory などの中央認証基盤を使わず、各コンピュータが独立して認証・管理を行う Windows のネットワーク構成のこと
- ワークグループ環境でも、SMB によるファイル共有やプリンタ共有を構成できる
- リソースが完全に独立しており、認証情報はSAMに保存される

---

# AD用語一覧表

| 用語名                                             |                                                                                                 内容（短説明） | 具体例（値・振る舞い・コマンド出力例など）                                                                                                                                                   |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ドメイン (Domain)                                   |                                                                                 管理単位としての名前空間。ADの最小管理単位。 | `corp.com`（社内ドメイン名）                                                                                                                                                     |
| ドメインコントローラー（DC）                                 |                                                                       AD情報を保持し認証/LDAP/DNS等を提供するサーバ。<br> | `DC01.corp.com`（Active Directoryをホスト）                                                                                                                                   |
| **PDC**<br>(Primary DC)                         |                                                               複数あるDCの中で==最も最新の情報を反映==している。1つのドメインに１つのみ。 | `PdcRoleOwner`プロパティを持つ                                                                                                                                                  |
| DNS（AD依存）                                       |                                                                   ADでは名前解決とドメイン探索に必須。DCがDNSをホストすることが多い。 | `_ldap._tcp.dc._msdcs.corp.com` SRVレコード                                                                                                                                 |
| オブジェクト (Object)                                 |                                                                     AD内で管理されるエンティティ（ユーザー、コンピュータ、グループ等）。 | `CN=Taro Yamada,OU=Users,DC=corp,DC=com`                                                                                                                                |
| 属性（Attribute）                                   |                                                                                オブジェクトが持つフィールド（名前やメール等）。 | `mail: taro.yamada@corp.com`、`sAMAccountName: tyamada`                                                                                                                  |
| 組織単位（OU）                                        |                                                                          オブジェクトを整理するコンテナ。委任やGPOの適用先に使う。 | `OU=Sales,DC=corp,DC=com`                                                                                                                                               |
| ユーザーオブジェクト                                      |                                                                          人がログオンするためのアカウント情報を保持するオブジェクト。 | `sAMAccountName: jsmith`, `userPrincipalName: jsmith@corp.com`                                                                                                          |
| コンピュータオブジェクト（workstation）                       |                                                                                ドメイン参加したPC/サーバを表すオブジェクト。 | `CN=WS-01, OU=Workstations`（ドメイン参加済みワークステーション）                                                                                                                          |
| グループ (Group)                                    |                               複数オブジェクトをまとめて権限付与やポリシー管理に使う。<br>グループの中にグループを作成（ネスト）できる→==意図しない高権限付与の可能性== | `Domain Admins`, `HR-FileShare-Read`                                                                                                                                    |
| samAccountType                                  |                                                                             属性の１つ。オブジェクトの種類を識別するために使われる |                                                                                                                                                                         |
| sAMAccountName                                  |                                                                           旧来のログオン名（Windows互換の短いアカウント名）。 | `jsmith`                                                                                                                                                                |
| userPrincipalName（UPN）                          |                                                                                          電子メール形式のログオン名。 | `jsmith@corp.com`                                                                                                                                                       |
| ADSI<br>(Active Directory Service Interface)    |                                                 ADと通信するためのCOMコンポーネントの塊([用語](../../../Misc/用語.md#COMとは)) | ADSIは以下のプロバイダーをサポート：<br>・`LDAP://` : **LDAP プロバイダー**<br>・`WinNT:// `: Windows NT プロバイダー（古い形式）<br>・`IIS://` : IIS プロバイダー<br>・`NDS://` : Novell Directory Services プロバイダー |
| **LDAP**（Lightweight Directory Access Protocol） |                                                                       ADとの通信に使用されるプロトコルで、ADの列挙はLDAPに依存。 | `ldapsearch -x -H ldap://dc.corp.com -b "dc=corp,dc=com" "(objectClass=user)"`                                                                                          |
| LDAPS / StartTLS                                |                                                                     LDAPのセキュア版（暗号化）。ポート636やSTARTTLSで使用。 | LDAPS ポート：`636`                                                                                                                                                         |
| **DN**<br>(Distinguish Name)                    | AD内のオブジェクトを一意に識別する名前。==右から左に読む==(階層が上から下へ行く）。LDAPのパスとして使う。<br>==オブジェクトの指定にはDNを使うと確実==（`-Identity <DN>`) | `CN=Stephanie,CN=Users,DC=corp,DC=com`                                                                                                                                  |
| CN(Common Name) / DN(Distinguish Name)          |                                                                                                 右から左に読む | `CN=Stephanie,CN=Users,DC=corp,DC=com`<br>└DCはツリー状になっており、comの配下にcorpなどがある。<br>└CN=Usersがオブジェクト格納場所で、CN=Stephanieがオブジェクトそのもの。                                            |
| グループポリシー（GPO）                                   |                                                                                ユーザー/コンピュータ設定を一括適用する仕組み。 | `Default Domain Policy`、ログオンスクリプト適用                                                                                                                                     |
| スキーマ（Schema）                                    |                                                                                 ADで許容されるオブジェクト・属性の定義集合。 | `user` クラスに `mail`, `displayName` 属性がある                                                                                                                                 |
| FSMO（役割）                                        |                                                             フォレスト/ドメイン内で特定の専任タスクを行う5つの役割（例：PDCエミュレータ等）。 | `Schema Master`, `PDC Emulator`, `RID Master`                                                                                                                           |
| グローバルカタログ（GC）                                   |                                                                      フォレスト内の一部属性を保存する検索用データベース。全域検索で使用。 | GCが有効なDCへのLDAP検索                                                                                                                                                        |
| ACL（アクセス制御リスト）／権限                               |                                                                         オブジェクトに設定されたアクセス権。委任・横展開の経路になる。 | `Read`/`Write`/`GenericAll`、オブジェクトに対する`WriteDacl`権限                                                                                                                     |
| LAPS（Local Administrator Password Solution）     |                                                                   各端末のローカル管理者パスワードを自動管理するMicrosoftの仕組み。 | `ms-Mcs-AdmPwd` 属性にパスワード保存                                                                                                                                              |
| サービスアカウント（Managed Service Account / gMSA）       |                                                                     サービス用に管理されるアカウント。パスワード自動管理されるものもある。 | `corp\sqlsvc$`（gMSA形式）                                                                                                                                                  |
| SIDHistory                                      |                                                                           オブジェクトの過去のSIDを保持する属性（移行時に利用）。 | `objectSID` と `sIDHistory` が存在するユーザー                                                                                                                                    |
| *SPN*<br>(Service Principal Name)               |                                 ・どのホストで、どのサービスを動かすかを示す情報で、サービスアカウントの属性の１つ<br>・１つのサービスアカウントに複数のSPNを登録できる | `{HTTP/web04.corp.com, HTTP/web04, HTTP/web04.corp.com:80}`<br> └web04.corp.comホスト上でHTTPサービスが稼働<br> └複数表記は同一サービスの別表記                                                    |
| SPN重複                                           |                                                            同一SPNが複数アカウントに割り当てられる状態。Kerberos認証問題や攻撃面となる。 | `setspn -L account1` で重複を確認                                                                                                                                             |
| GPOのPreferences（隠し秘密）                           |                                                                     過去にはパスワードを平文保存する等の問題があった箇所。列挙時に要注意。 | GPO XML内の`cpassword`（保護されていない場合）                                                                                                                                        |

---

# Active Directoryの仕組み

## 基本

- Active Directory Domain Services (AD DS) が基盤
- ADはUserやMachineなどのオブジェクトで構成される
- User：PeopleとServiceに分類されるセキュリティプリンシパル
- Machine：マシンアカウントを持つセキュリティプリンシパル
- Security Group：グループ単位で権限を付与する仕組み

## Active Directory Users and Computers

- ユーザーやグループをOU（組織単位）でまとめる
- OUは階層化可能、ポリシー継承あり
- デフォルトでBuiltin, Computersなどのコンテナが存在
- OUには１ユーザーは１つだけ所属できる

![](../../../Images/Pasted%20image%2020230409150913.png)

## User管理

- デフォルトのOUとユーザをまず確認
- OU削除は「Protect object from accidental deletion」を外す必要あり
	- View > Advanced Features > 削除したいOUを右クリック > Properties > Object > Protect object from accidential deletionのチェックを外す
- Delegation of Controlで権限委譲が可能
	1. OUを右クリック > Delegation of Control Wizard > Addで、Enter the object names to selectに権限を委任したいユーザー名を入力
	2. Check Namesでオートコンプリートを実行
	3. ユーザを追加すると、Tasks to Delegateという画面が出てくるので、委任したいタスクにチェックを入れる。
- Powershellでユーザーの操作例
    - パスワード変更：`Set-ADAccountPassword`
    - ログオン時にパスワード変更要求：`Set-ADUser -ChangePasswordAtLogon $true`

## Computer管理

- デフォルトOUであるComputersコンテナにすべて集約するのは非推奨
- 以下のようなOUを作成し、分類する：
    1. Workstations（PC, ラップトップなど）
    2. Servers
    3. Domain Controllers

## Group Policy

- Group Policy Managementを利用
- GPOはOUにリンクし、下位に継承される
- Security Filteringで対象を限定可能
- **SYSVOLディレクトリ**でDC間同期
	- すべてのユーザー、コンピュータなどがSYSVOLネットワーク共有により同期される

### GPOの作成から適用

例：コントロールパネルのアクセス禁止を設定する方法

1. . Group Policy Managementを開く
2. Group Policy Objects配下に新たなポリシーを作成する
3. 作成したポリシーを右クリックし、Editを開く
4. User Configuration -> Administrative Templates -> Control Panel -> Prohibit Access をEnable
5. 部門OUにリンク

![](../../../Images/Pasted%20image%2020230410135945.png)

---

# 認証方法

- ==認証情報はすべてDCに保存==
- ユーザログイン時はDCが認証し、トークンを発行
- 主なプロトコル
    1. Kerberos（デフォルト）：チケット方式
    2. NetNTLM（レガシー）：チャレンジ&レスポンス方式

## Kerberos

- 認証にチケット（TGT, TGS）を利用
- TGTはkrbtgtアカウントのパスワードハッシュで暗号化
- krbtgt漏洩は全アカウントへのアクセスに直結
- サービス利用にはTGSが必要
- Silver Ticketは検証がKDCでなくサービス側なのでステルス性が高い

### Kerberos認証ステップ

1. *AS-REQ*：クライアントから、平文のUsernameと、パスワードによって生成されたユーザーハッシュ（==共通鍵==）で暗号化されたTimestampをKDC(AS)に送る
	- KDC(Key Distribution Center)はDCにインストールされている
	- Timestampの用途は、リプレイ攻撃の検出
2. *AS-REP*：KDCはTGT(Ticket Granting Ticket)を生成しTGTとSession Keyを一緒に返す
	- TGTはUserがサービスにアクセスする資格があることを証明
	- TGTを使うことで、サービスにアクセスしたい時に必要なTicket(TGS)を要求できる
	- TGTのおかげで、サービスにアクセスするたびに認証情報を渡すことなくサービスチケットを要求できる
	- TGTは中身にSession Keyのコピーを保持しており、KDCは必要な時にTGTを復号してSession Keyを得ることができるので、KDC自体にSession keyを保持する必要はない(session keyはUserhashで暗号化)
	- TGTは*krgtgt* hashという、サービスアカウントがTGTを発行するための鍵を内部に保持している = TGTはkrbtgtアカウントのパスワードハッシュを使用して暗号化されているため、ユーザーはTGTの内容を復号できない
	- 💥krbtgtアカウントのパスワードが漏洩した場合、悪意のある攻撃者はWindowsドメイン内のすべてのアカウントにアクセスできる可能性がある→攻撃者に狙われやすい

![](../../../Images/Pasted%20image%2020230410155451.png)

$$TGTのレスポンス$$

3. *TGS-REQ*：TGTを使用してKDCにTGS（Ticket Granting Service）をリクエストするため、クライアントはUsernameとTimestamp(Authenticator)をセッションキーで暗号化して送信し、TGTと、SPNを添付する
	- TGSは、作成された特定のサービスに対してのみ接続を許可するチケット（(ST:==service ticket==ともいう）
4. *TGS-REP*：KDCはアクセスしたいサービスに認証するために必要となるSvc Session KeyとTGSを返す
	- TGSは、Service OwnerのHashからできたキーを使用して暗号化される
	- Service Ownerはサービスを所有するプリンシパル（クライアントではない）
	- TGSは、サービス所有者がTGSを復号することでアクセスできるように、中身（暗号化されたコンテンツ）にSvc session keyのコピーをもつ
		- Svc＝Serviceの略

![](../../../Images/Pasted%20image%2020230410161109.png)

$$TGSのレスポンス$$

5. *AP-REQ*：クライアントはSvc Session Keyで暗号化したUsernameとTimestamp、そしてService Owner Hashで暗号化されているTGSを、サービスが稼働しているSRV（サーバー）に送る
	- SRVは自身の持つService Owner HashでTGSを復号し、Svc Session Keyを取り出す
	- Svc Session Keyを使って、クライアントから送られてきた暗号化されたAuthenticator（UsernameとTimestamp）を復号する
	- 復号した内容（UsernameとTimestamp）を検証する

![](../../../Images/Pasted%20image%2020230410162330.png)

$$TGSでサービスの使用要求$$
- [Silver Ticket](../../../Tools/🥝Mimikatz.md#Silver%20Ticket)にもあるように、Silver Ticket攻撃のステルス性が高いのは、TGSはKDCのTGSにより発行されるが、検証するのはKDCではなくSRVなので、偽造されたTGSがSRVに送られてもKDCのASで検証できないから。要はTGSは認証サーバに送られないから。

### TGT・TGSの中身

#### TGT

- KDC LT Key：krbtgtがTGTを暗号化し==PACに署名する==ためのもの
- Client LT Key：
	- [ADの基本](#Kerberos認証ステップ)でUser Hashとあるもの
	- KDCは、クライアントのClient LT Keyを使用して、そのハッシュ値を検証し、クライアントが正当なユーザーであることを確認する
	- 暗号化されたタイムスタンプの検証や、セッションキーの暗号化に使用
- *PAC(Privilege Attribute Certificate)*： 
	- ユーザーの関連情報をすべて保持し、TGTとともにKDCに送信される
	- PACはKDC LT Keyによって署名され、ユーザーの権限を検証するために使用される
	- これにより、サービスが都度DCに問い合わせることなく、チケットの情報から必要な権限を持つかどうかを確認できる

![](../../../Images/Pasted%20image%2020230413150630.png)

$$TGTのイメージ$$

#### TGS

- Service LT Key：[ADの基本](#Kerberos認証ステップ)でservice owner hashとあるもので、サービスチケットを暗号化する

![](../../../Images/Pasted%20image%2020230413150522.png)

$$TGSのイメージ$$

#### チケットの形式

- 出力形式：`[<セッションID>]-x-x-<フラグ>-<ClientName>@<ServiceName>-<TargetName>.kirbi`

![](../../../Images/Pasted%20image%2020260106093049.png)

$$ticket出力例$$

| セッションID (LUID)  | 一般的な名称          | 特徴・用途                                              |
| --------------- | --------------- | -------------------------------------------------- |
| **0x3e7** (999) | SYSTEM          | OS自体が動作する最も権限の強いセッション。サービスやシステムプロセスが使用する。          |
| 0x3e4 (996)     | NETWORK SERVICE | ネットワーク経由で他のマシンと通信するサービス用。                          |
| 0x3e5 (997)     | LOCAL SERVICE   | ネットワーク権限を持たないローカルサービス用。                            |
| 0x61ec6 など      | User Session    | ユーザー（Administratorや一般ユーザー）がGUIやSSH等でログインした際に生成される。 |
- <u>⚠️チケット注入時の注意点</u>
	- WindowsのKerberosチケットは、LSAに保存されており、このキャッシュはログオンセッション（LUID）ごとに完全に隔離されている
	- `0x3e7` (SYSTEM) のプロセスは、`0x61ec6` (Administrator) のチケットキャッシュを直接見ることはできないため、プロンプトが `0x3e7` (SYSTEM) で動いている状態で`0x61ec6` のチケットをいくら持っていてもOSは「チケットがない」と判断する

| フラグ        | 意味                                              |
| ---------- | ----------------------------------------------- |
| `60810000` | renewable + forwardable + initial               |
| `40c10000` | renewable + forwardable                         |
| `40e10000` | renewable + forwardable + initial + pre_authent |

## NTLM

- NetNTLMも利用可能だが、パスワードハッシュのクラックにそこまで時間は掛からないうえ、PtH攻撃が可能なため、非推奨とされている
- しかし、現実にはKerberosに対応していないマシンなどもあり、停止することによる他システムへの影響などから大掛かりな準備が必要であるため、今もなおNTLM認証が有効な組織は多い

![](../../../Images/Pasted%20image%2020230410163456.png)

$$NetNTLMの認証フロー$$

---

# Active Directoryの構成

## Tree

- 同一名前空間内で複数ドメインをまとめる単位
	- 例：<u>thm.local</u>, uk.<u>thm.local</u>, us.<u>thm.local</u>
- 下図ではthm.localがルートドメイン
- Enterprise Admins（ツリー全体の管理者）、Domain Admins（単一ドメインの管理者）という権限階層がある

![](../../../Images/Pasted%20image%2020230410164027.png)

## Forest

- 異なる名前空間を持つ複数Treeを統合したもの
- 自動で権限は共有されず、Trust関係が必要

![](../../../Images/Pasted%20image%2020230410164545.png)

## Trust

- ドメイン間の信頼関係で、認証情報を相手ドメインであっても受け入れるための仕組み
	- アクセス権限そのものを与えるわけではない
- 単方向または双方向の設定が可能
- Trust関係があっても自動で全リソースにアクセスできるわけではない

![](../../../Images/Pasted%20image%2020230410164755.png)

---