# Port Scan

以下、必要に応じて[FW回避用オプション](#FW回避用オプション)をつけること。

quick scan
```zsh
sudo nmap -sC -sV -oN Nmap/quickscan.nmap -T4 <target_IP>
```

full scan
```zsh
# ポートのリストを変数に代入：２回実施して差分を検証すること
ports=$(sudo nmap <target_IP> -p- --min-rate=1000 | grep '^[0-9]' | awk -F'/' '{print $1}' | tr '\n' ',' | sed 's/,$//')
```
```zsh
# openポートを詳細スキャン
sudo nmap -sC -sV -O -p $ports -oA Nmap/fullscan <target_IP>
```

UDPスキャン & SYNスキャン
```zsh
nmap -sU -sS -sV <target_IP> -T4 -oN Nmap/udpscan.nmap --top-ports 100
```
- SYNスキャンと組み合わせることでUDPスキャンの信頼性が高まる
- SNMP、TFTP、DNSなどが要注意

Rustscan
```zsh
rustscan -a <target_IP> --ulimit 5000 -- -sC -sV -O -oN Nmap/rustscan.nmap
```


> [!TIP] 
> - **スキャンは最低2回実施すること！**(3回やっても見逃しはある)
> - `/etc/hosts`にドメイン名登録し、再スキャンすることで、より正確な結果が表示される
> - `-sV`によって表示されるサービス名は、Administratorによって非正規のサービス名に変更されている可能性があることを考慮する
> - 詳細なサービスはPort Scanだけではわからない可能性があるので、必要に応じてNSEを実行

---

# PortScan x Service Scan w/ AutoRecon

Openポートを発見し、さらにポートに合わせてenum4linuxなどでサービスの自動列挙+NmapのsTスキャンを実施
```zsh
# $portsは上記Nmapで作成した環境変数で、Openポートのリストを格納してある
sudo autorecon <target_IP> -p $ports 
```
- 出力の`_manual_commands.txt`に手動で列挙する方法が記載されている

出力に書き込みできるようにする
```zsh
sudo chown -R <attacker_username>:<attacker_username> results
```

> [!WARNING] 
> ヌケモレや誤検知が多いので、過信せず、Nmapと併用すること。

---

# Vuln Scan w/ NSE

- 🔗[NSEドキュメント](https://nmap.org/nsedoc/)

カテゴリに属するNSEすべてを実行（[NSEのカテゴリ一覧表](#NSEのカテゴリ一覧表)）
	⚠️トラフィックと情報量が膨大なのであまり実行すべきでない
```zsh
sudo nmap --script　"<category>" <target_IP> -p <port>
```

サービスのバージョンをもとに脆弱性をスキャン
	⚠️トラフィックと情報量が膨大なのであまり実行すべきでない
```zsh
sudo nmap -sOV --script "vuln" <target_IP> -p <port>
```

✅特定のスクリプトのみ実行
	トラフィックと情報量を必要な量に抑える
```zsh
# nseをピンポイントで指定
sudo nmap --script "<xxx.nse>" <target_IP> -p <port>
```
```zsh
# 特定サービスのすべてのnseに絞って実行
sudo nmap --script "http-*" <target_IP> -p <port>
```

## NSEのダウンロード

Nmap標準のスクリプトに存在しないNSEをダウンロードして使うことができる。

1. 該当の脆弱性のNSEを検索

![](../../Images/Pasted%20image%2020250309222541.png)

$$nse検索例$$

2. 既存のNSEに存在しなければダウンロード

3. NSEのパスに保存
```zsh
sudo cp /home/kali/Downloads/http-vuln-cve-2021-41773.nse /usr/share/nmap/scripts/http-vuln-cve2021-41773.nse
```

4. script.dbをアップデート
```zsh
sudo nmap --script-updatedb
```

## NSEの使い方

| NSE指定方法                                   | 実行されるスクリプト                       |
| ----------------------------------------- | -------------------------------- |
| `--script vuln`                           | `vuln` カテゴリの全スクリプト               |
| `--script vuln,safe`                      | `vuln` と `safe` の両方に属するスクリプト     |
| `--script "default and safe"`             | `default` かつ `safe` に属するスクリプト    |
| `--script "intrusive or vuln"`            | `intrusive` または `vuln` に属するスクリプト |
| `--script "</path/to/custom_script.nse>"` | 指定したカスタムスクリプトのみ                  |

カテゴリごとの利用可能なスクリプト一覧 | スクリプトごとの引数を表示
```zsh
nmap --script-help "<category | script>"
```

NSE、カテゴリを表示
```zsh
cat /usr/share/nmap/scripts/script.db
```
↓出力
```
...Entry { filename = "memcached-info.nse", categories = { "discovery", "safe", } }...
```

> [!WARNING] 
> - 外部のNSEをダウンロードする時は注意が必要で、悪意あるコードが外部のNSEに含まれており、そのNSEを使うと悪意ある第三者がフルアクセスをゲットできてしまう可能性がある
> - スキャナが発見できるのは、スキャナが設定した脆弱性だけである
> - カテゴリごとにNSEがシステムにどのような影響を与えるかを確認してから実行すること
> 	-  [NSEのカテゴリ一覧表](#NSEのカテゴリ一覧表)

---
---

# Nmap/NSE 補足

## 主要フラグチートシート

| スイッチ            | 説明                                                                                                                                                 |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| -vv             | 詳細表示（verbose）                                                                                                                                      |
| -sS             | SYNスキャン（ステルス）                                                                                                                                      |
| -sT             | TCPコネクションスキャン                                                                                                                                      |
| -sU             | UDPスキャン                                                                                                                                            |
| -sn             | pingスイープ（ポートスキャンなし）                                                                                                                                |
| -O              | OS検出                                                                                                                                               |
| -sV             | バージョン検出                                                                                                                                            |
| -oA             | 全フォーマットで結果保存                                                                                                                                       |
| -A              | `-sV -O -sC --traceroute`相当（⚠️[なぜ-Aを使わないのか - Reddit](https://www.reddit.com/r/nmap/comments/pz6tvl/why_nmap_sc_sv_when_you_can_use_nmap_a/?tl=ja)） |
| -T1~5           | スキャン速度調整                                                                                                                                           |
| --min-rate      | 最小パケット送信レート                                                                                                                                        |
| -p              | ポート指定スキャン                                                                                                                                          |
| -p-             | 全ポートスキャン                                                                                                                                           |
| --script        | NSEスクリプト実行                                                                                                                                         |
| --script=<カテゴリ> | カテゴリ全スクリプト実行                                                                                                                                       |
| -sC             | `--script=default`相当                                                                                                                               |

---

## スキャンタイプごとの特徴・内部動作

### TCPコネクションスキャン (-sT)

- 3ウェイハンドシェイクを実施
- 閉じたポートにはRSTで応答
- 遅く目立ちやすいが、==FW越えはしやすい==

### SYNスキャン (-sS)

- ステルススキャン
- `sudo`時にデフォルトで使われる
- syn/ackにはRSTで返答（接続確立前に終了）
- 速く目立ちにくいが、==特殊なフィルタには弱い==

### UDPスキャン (-sU)

- 非常に遅く不確実
- ICMP unreachable応答があるとclosedポートと判断し、応答がないとopen|filteredと判断する
- 応答がない→ open or filtered と判断する
- 対策: `--top-ports`：一般的なUDPポートのリストで絞る    
```zsh
nmap -sU --top-ports <num> <target_IP>
```

- SYN Scanと組み合わせることでホストへの理解がより鮮明になる
```zsh
sudo nmap -sU -sS <target_IP>
```
  
### NULL/FIN/Xmasスキャン

- ステルス性高いが信頼性低い
- Windowsでは全て「closed」に見える
- SYNフラグを含まないため、特定フィルタ回避に有効

### ICMPネットワークスキャン (-sn)

- ホスト発見用
- CIDR範囲で使用
```zsh
nmap -sn 192.168.0.0/24
```
- ICMP Echo Requestだけでなく、TCP/80,443へのリクエスト、ICMP Timestamp requestも送信し、あらゆる角度からホストの生死を判断する

---

## NSEのカテゴリ一覧表

⚠️NSEのカテゴリを理解せずに使うことは厳禁

| カテゴリ      | 説明                                                    | システム安定性への影響                             |
| --------- | ----------------------------------------------------- | --------------------------------------- |
| auth      | 認証関連のテストを行うスクリプト。認証バイパスやデフォルトクレデンシャルのチェックを行う。         | 影響あり（一部のスクリプトはアカウントのロックアウトを引き起こす可能性がある） |
| broadcast | ネットワークブロードキャストを利用して、LAN 内の情報を収集する。                    | 影響なし（ブロードキャストを使うため、安全）                  |
| brute     | ブルートフォース攻撃を試行し、弱いパスワードを特定する。                          | 影響あり（大量の試行によりアカウントロックや検知リスクがある）         |
| default   | `--script default` で実行される基本的なスクリプト群。一般的な情報収集やスキャンを行う。 | 影響なし（安全なスクリプトのみ）                        |
| discovery | ネットワーク探索を目的としたスクリプト。ホストのリストアップやサービスの検出を行う。            | 影響なし（主に情報収集用）                           |
| dos       | 標的に対する DoS（Denial of Service）攻撃を試行し、影響を評価する。          | 影響大（サービスのダウンやシステムクラッシュを引き起こす可能性がある）     |
| exploit   | 既知の脆弱性を悪用し、システムに対する攻撃を試みる。                            | 影響大（標的システムに対して深刻な影響を及ぼす可能性がある）          |
| external  | 外部のサービス（例：Vulners、Shodan）を利用して情報を取得する。                | 影響なし（外部 API を利用するため安全）                  |
| fuzzer    | サービスに対してランダムなデータを送信し、異常動作やバグを引き起こすか検証する。              | 影響あり（サービスがクラッシュする可能性がある）                |
| intrusive | システムに影響を与える可能性のある積極的なスキャンを行うスクリプト。                    | 影響あり（サービスがクラッシュする可能性あり）                 |
| malware   | マルウェア感染の兆候を検出するスクリプト。                                 | 影響なし（スキャンのみを実施）                         |
| safe      | システムに影響を与えない安全なスクリプト。                                 | 影響なし（常に安全に実行可能）                         |
| version   | サービスのバージョンを特定するスクリプト。                                 | 影響なし（通常は情報収集のみ）                         |
| vuln      | 既知の脆弱性を検出し、影響を評価するスクリプト。                              | 影響あり（一部のスクリプトはサービスに影                    |

---

## FW回避用オプション

| スイッチ                    | 説明                                                                                                                                              |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| -f                      | パケットを断片化する（つまり、より小さな断片に分割する）ために使用され、パケットがファイアウォールやIDSによって検出される可能性を低くする。                                                                         |
| --mtu \<number>         | -fの代案。送信されるパケットに使用する最大伝送単位サイズを指定。これは8の倍数である必要がある。                                                                                               |
| --scan-delay \<time>ms  | 送信するパケットの間に遅延を追加するために使用。これは、ネットワークが不安定な場合に非常に便利だが、時間ベースのFWやIDSのトリガーを回避するためにも使用される                                                               |
| --badsum                | パケットに無効なチェックサムを生成するために使用。実際のTCP/IPスタックはこのパケットをドロップするが、ファイアウォールはパケットのチェックサムを確認することなく、自動的に応答する可能性がある。そのため、このスイッチを使用して、ファイアウォールやIDSの存在を確認することができる。 |
| --data-length \<number> | 指定したバイト数のランダムデータを付加する。処理速度は低下するが、スキャンを幾分でも目立たなくすることができる                                                                                         |
| **-Pn**                 | ICMPブロックを回避する。Windowsマシンへのスキャンにはほぼ必須。==遅くなる。==                                                                                                  |

---

## ADに所属しているかどうかの判断方法（Windows）

- DNS_Domain_Name の形式
    - FQDN形式 (例: `domain.com`) → AD所属
    - 単一名 (例: `HOSTNAME`) → AD非所属
- DNS_Tree_Name の有無
    - 存在する → AD所属
    - 存在しない → AD非所属
- 上記両方とも表示されない場合は、NetExecでsmbのドメイン名を検証する
	- smbもopenでない場合は、ワークグループと仮定する

```zsh
# AD所属
rdp-ntlm-info: 
  Target_Name: RELIA
  NetBIOS_Domain_Name: RELIA
  DNS_Domain_Name: relia.com          # 着目
  DNS_Computer_Name: login.relia.com  # 着目（FQDN）
  DNS_Tree_Name: relia.com      　　　 # 着目
  
# AD非所属（単純なワークグループ）
rdp-ntlm-info: 
  Target_Name: EXTERNAL
  NetBIOS_Domain_Name: EXTERNAL
  NetBIOS_Computer_Name: EXTERNAL     # 着目
  DNS_Domain_Name: EXTERNAL           # 着目
  DNS_Computer_Name: EXTERNAL         # 着目
```
- 関連ノート：[ADの基本](../🪟Windows/Active%20Directory/ADの基本.md#Active%20Directoryとは)

---

## 参考リンク🔗

- [Nmap公式チートシート](https://www.stationx.net/nmap-cheat-sheet/)
- [NSEスクリプト一覧](https://nmap.org/nsedoc/)
