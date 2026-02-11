# RustScanコマンド実践手順

- 安全性・正確性より速度を優先するときに使う。
- **2回実施すること**（偽陽性、偽陰性を軽減するため）
- `/etc/hosts/`にドメイン名を追加してから再度実行すると、より詳細な結果が得られるかも

- Openポートを探索し、Openポートにのみ詳細なスキャン(rustscanの仕様)
```zsh

```

## [[Port Scan & Vuln Scan|Port Scan & Vuln Scan]]と比較した際の注意点

- はやいので、偽陽性(false positive)や、ブルーチームによる検知があがりやすい（速度は調整できる）
- 実世界では、Nmapの方が安全

## 用途・前提

- Nmap同様、==結果を信じ込まない==。3回実行しても表示されないポートもある。initial access後のマシンで`netstat -ano`を実行すること
- Nmapより素早くスキャンしたい時に使う
- RustScan Scripting Engine(RSE)により、任意のスクリプトを実行可能([詳細](https://github.com/RustScan/RustScan/wiki/RustScan-Scripting-Engine))
- RustScanの速度は、RustScan自体ではなく、自身のCPUやネットワークなどの環境による
- RustScanはスキャン対象やユーザーの環境に動的に適応する。この適応型学習機能により、スキャン速度や精度を最大化しつつ、リソースの過負荷や非効率なスキャンを防ぐ。
- 実行環境のOSを把握し、動的に調整するという独自機能を持つ。

---

# RustScanコマンド解説

- 標準のスキャン
```zsh
rustscan -a $IP
```
	→空いているポートだけ表示

- nmapのコマンドを用いたスキャン
```zsh
rustscan -r [ports] -a $IP -- [nmap commands]
```

- 複数IP同時スキャン
```zsh
rustscan -a [IP1],[IP2]
```

- ホスト(FQDN)スキャン
```zsh
rustscan -a www.google.com
```

- CIDRでスキャン範囲指定
```zsh
rustscan -a [$IP]/[subnet mask]
```

- スキャン対象のリスト一覧が記載されたファイルを用いて対象を同時にスキャン
（例）hosts.txt
```txt
192.168.0.1
192.168.0.2
google.com
192.168.0.0/30
127.0.0.1
```
```zsh
rustscan -a 'hosts.txt'
```

- 特定ポートのスキャン
```zsh
rustscan -a $IP -p [port]
```

- 複数ポートのスキャン
```zsh
rustscan -a $IP -p [port1],[port2]
```

- ポートの範囲指定スキャン
```zsh
rustscan -$IP --range [start]-[end]
```

- ランダムな順序でのスキャン
```zsh
rustscan -a $IP --scan-order "Random"
```

## RustScanの調整

- [RustScan Usage](https://github.com/RustScan/RustScan/wiki/Usage)

- バッチサイズの調整（指定したポート数を一度にスキャン）
```zsh
rustscan -b [batch size]
```

- タイムアウト時間の調整(5秒＝5000ミリセカンド)
```zsh
rustscan -T [miliseconds]
```

---

# RustScanインストール

1. [RustScan release page]([https://github.com/RustScan/RustScan/releases](https://github.com/RustScan/RustScan/releases))から.deb.zipファイルをダウンロード
2. unzip
3. dpkgでインストール
```zsh
sudo dpkg -i rustscan_<version>_amd64.deb
```