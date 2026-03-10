[Port Scan & Vuln Scan](../../Cheatsheet/Common/Port%20Scan%20&%20Vuln%20Scan.md)
# スキャンタイプ一覧表

| スキャンタイプ                | コマンド例                                                      |
| ---------------------- | ---------------------------------------------------------- |
| ARP Scan               | `sudo nmap -PR -sn $IP/24`                                 |
| ICMP Echo Scan         | `sudo nmap -PE -sn $IP/24`                                 |
| ICMP Timestamp Scan    | `sudo nmap -PP -sn $IP/24`                                 |
| ICMP Address Mask Scan | `sudo nmap -PM -sn $IP/24`                                 |
| TCP SYN Ping Scan      | `sudo nmap -PS22,80,443 -sn $IP/30`                        |
| TCP ACK Ping Scan      | `sudo nmap -PA22,80,443 -sn $IP/30`                        |
| UDP Ping Scan          | `sudo nmap -PU53,161,162 -sn $IP/30`                       |
| snより正確                 | `nmap -sT -A --top-ports=20 $IP/30 -oG top-port-sweep.txt` |

---

# オプション一覧表

| オプション | 目的                                                                                          |
| ----- | ------------------------------------------------------------------------------------------- |
| `-n`  | reverse-DNS lookupをしない                                                                      |
| `-R`  | オンライン・オフライン問わず全てのホストにreverse-DNS lookup                                                     |
| `-sn` | ホスト検出のみ。ポートスキャンはしない。ICMP Echo request(ping)だけでなく、80,443へのリクエスト、ICMP timestamp requestを実施する。 |

---

# ホストディスカバリにおける注意点

- `-Pn`を使ってはいけない
	- 実際に生きているかどうかをチェックしない、という意味で、ホストディスカバリの意味がなくなる
	- 使うと、存在しないホストでも、"Host is up"と表示される

---

# ホストディスカバリ実行コマンド

- どのホストがオンラインかわからない場合はまずこれを実行
- `-Pn`は使わないこと [Nmap Live Host Discovery](#ホストディスカバリにおける注意点)

複数ホストをスキャンしてオンラインホストの一覧を作成（ping sweep(`-sn`)より正確）
```zsh
# 80番or443番を対象にスキャン
sudo nmap -p 80 192.168.50.1-253 -oG web-sweep.txt

# 抽出&整形
grep open web-sweep.txt | cut -d " " -f2
```

ping sweep
```zsh
sudo nmap -sn 192.168.1.0/24 -oG ping-sweep.txt
```

Nmapが使えないとき：
```zsh
# サブネットに存在する機器に対し、特定ポートがopenかどうかを検証
for i in $(seq 1 254); do nc -zv -w 1 [サブネットIP第三オクテット].$i <Port>; done
```
- Windowsの場合は[Living off the Land](../../Misc/Living%20off%20the%20Land.md#Test-NetConnectionによるポートスキャン)
