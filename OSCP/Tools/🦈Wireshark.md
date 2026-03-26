# Wiresharkの使い方と解析の基礎

Wiresharkは、ネットワークパケットのキャプチャと解析を行うツール。主にPCAPファイル（Packet CAPture）を扱い、ネットワーク上の通信内容を詳細に分析できる。

## 基本の使い方

1. 起動後、インターフェース名と通信状況を示すグラフが表示される。使用するインターフェースを選び、ダブルクリックでキャプチャ開始

![](../Images/Pasted%20image%2020230326131323.png)

$$立ち上げ画面$$

2. フィルタ設定は任意で、Manage Capture Filtersから事前にキャプチャ対象を絞り込むことも可能

3. 解析済みPCAPファイルを開くには、File > Openから選択

4. キャプチャ後、パケットを右クリックしてFollow > TCP Streamで通信内容を時系列で閲覧。

5. "Find packet"機能でキーワード検索も可能

![](../Images/Pasted%20image%2020230515151532.png)

$$文字列：passwordの検索$$

## 上部タブの使い方一覧表

![](../Images/Pasted%20image%2020250608121029.png)

$$タブ$$

| ボタン名                                   | 機能                        | 主な目的                                     |
| -------------------------------------- | ------------------------- | ---------------------------------------- |
| Capture Options（キャプチャオプション）            | キャプチャ設定ウィンドウを開く           | インターフェース選択やフィルタ設定など詳細なキャプチャ条件を設定         |
| Stop Capturing（キャプチャ停止）                | 現在のパケットキャプチャを停止する         | 必要な分のキャプチャが終わったら手動で停止                    |
| Restart Capturing（再キャプチャ）              | 現在のキャプチャを止めて再スタートする       | キャプチャ条件を変えずに再取得したい時に使用                   |
| Capture Interfaces（インターフェース選択）         | 利用可能なネットワークインターフェースの一覧を表示 | どのインターフェースを監視するかを選ぶ                      |
| Open File（ファイルを開く）                     | 保存済みのPCAPファイルなどを開く        | **過去のキャプチャデータを読み込んで解析する**                |
| Save File（ファイルを保存）                     | 現在のキャプチャを保存する             | 解析用にPCAPファイルとして保存しておく                    |
| Find Packet（パケット検索）                    | 特定のキーワードや条件でパケットを検索       | パスワード、IPアドレス、ドメインなどを検索して該当通信を抽出          |
| Go Back（戻る）                            | 前に表示していたパケットに戻る           | 解析中に前のパケットに戻る                            |
| Go Forward（進む）                         | 次のパケットに進む                 | 戻った後、また次に進む際に使う                          |
| Go to First Packet（最初のパケットへ）           | キャプチャの先頭パケットにジャンプ         | 時系列で最初の通信から調べる時に便利                       |
| Go to Last Packet（最後のパケットへ）            | キャプチャの最後のパケットにジャンプ        | 最新の通信や攻撃などを確認したい時に使用                     |
| Draw packets using your coloring rules | パケットの色分け有無の切り替え           | 通信の種類や異常を視覚的に識別しやすくする                    |
| Packet List Column Preferences（列設定）    | 表示するカラム（列）の設定変更           | 「Time」「Source」「Destination」など表示項目のカスタマイズ |

---


## パケットの色の意味（色別の概要）

色設定は View > Coloring Rules で確認・編集可能。

- 黒背景（白文字）：チェックサムエラーなど深刻なエラー
- 赤：Bad TCP（RST、再送、不正シーケンスなど）
- 緑：HTTP、一般的なTCP通信
- 水色：TCP SYN/ACK（ハンドシェイク）
- 青：DNS、ARP
- 薄紫：通常のTCPデータ転送

---

# キャプチャ方法と前提

## キャプチャ手法

- Network taps：ネットワーク上に物理的なタップ機器を設置して傍受する
- MAC floods：CAMテーブルを溢れさせてスイッチをハブ動作させ、全トラフィックを受信できるようにする（攻撃的手法）
- ARP poisoning：ARPキャッシュを汚染して中間者として通信を傍受する（攻撃的手法）

パケット数が膨大になるため、ディスク容量と処理能力の確保が必要。

## フィルタの使い方

### 表示フィルタ（Display Filter）

- 使用場所：Analyzeタブまたは画面左上のバー
- 主な演算子：`&&`, `||`, `==`, `!=`, `<`, `>`, `contains`, `matches`, `bitwise_and`

### 使用例

- IPアドレス指定： `ip.addr == 192.168.0.1`
- 送信元・宛先の組み合わせ： `ip.src == 10.0.0.1 and ip.dst == 10.0.0.2`
- TCPフィルタ： `tcp.port == 80`
- UDPフィルタ： `udp.port == 53`
- ARPのリプライのみ： `arp.opcode == 2`

---

# パケットの構造解析

パケットはOSI参照モデルに基づいて表示される。

1. Frame（物理層）
2. Ethernet II（データリンク層）：MACアドレス
3. IP（ネットワーク層）：IPv4アドレス
4. TCP/UDP（トランスポート層）：ポート番号など
5. アプリケーション層プロトコル：HTTP、FTP、SMBなど
6. アプリケーションデータ：コンテンツそのもの

---

# 各種プロトコル別の解析

## ARP

![](../Images/Pasted%20image%2020230326141253.png)

$$ARPキャプチャ$$

- ARP Request（リクエスト）とReply（応答）に分かれる
	- Opcodeで区別可能

![](../Images/Pasted%20image%2020230326144125.png)

$$ARPリクエストキャプチャ$$

- `arp.dst.proto_ipv4`で宛先IPv4アドレスを抽出
- View > Name Resolution > Resolve Physical AddressesでMACベンダー名が表示される

![](../Images/Pasted%20image%2020230326143839.png)

$$MACベンダーの表示(SourceにCiscoと表示）$$

## ICMP

- Type 8：Request、Type 0：Reply
- Dataが異常な場合は要注意。
- データのコピーは右クリック → Copy as Hex stream

![](../Images/Pasted%20image%2020230326150954.png)

$$ICMPリクエストのキャプチャ$$

## TCP

- 通常：SYN、ACK、FINなどで通信確立や終了を表現
- nmapスキャン時：RST, ACKで赤く表示：[パケットの色の意味（色別の概要）](#パケットの色の意味（色別の概要）)
- sequence numberとacknowledgment numberをチェック
- ポートが閉じている場合はRST/ACKが返る
- Edit > Preferences > Protocols > TCP > relative sequence number をオフで実番号が見える

## DNS

- 基本はUDP53ポートで、TCP53なら要観察
- QueryとResponseの整合性を確認
- Answerの内容に注目

![](../Images/Pasted%20image%2020230326154730.png)

$$DNSクエリキャプチャ$$

## HTTP

- 暗号化されていないため内容がそのまま見える
- Request URI、File Data、Serverが重要
- Protocol Hierarchyでプロトコル全体像を確認
- File > Export Objects > HTTPで全URIをリストアップ
- Statistics > Endpointsで接続先のIPなどを確認

## HTTPS

- パケットは暗号化されており内容の確認には秘密鍵が必要
- HandshakeとSSL Record Layerの確認が重要
- Edit > Preferences > Protocols > TLS > RSA KEY Listでキー指定すれば復号可能
- ただしTLS 1.3やECDHE/DHEによる鍵交換（前方秘匿性）が使われている場合は、RSA秘密鍵だけでは復号できない


