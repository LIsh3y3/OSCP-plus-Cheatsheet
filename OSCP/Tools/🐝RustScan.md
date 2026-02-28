# Nmapと比較した特徴

- 安全性・正確性よりも速度を優先するときに使うもので、偽陽性(false positive)や、ブルーチームによる検知があがりやすい（速度は調整できる）
- 仕様として、Rustで構築されていることと、Openポートを探索してからOpenポートにのみ詳細なスキャンするため、速い
- 実行環境のOSを把握し、動的に調整するという独自機能を持つ
- RustScan Scripting Engine(RSE)により、任意のスクリプトを実行可能([詳細](https://github.com/RustScan/RustScan/wiki/RustScan-Scripting-Engine)
- ==実世界では、Nmapの方が推奨==
　
> [!WARNING]
> - Nmap同様、==結果を信じ込まない== （検出しなかった≠サービスが動作していない）
> - 偽陽性、偽陰性を軽減するため、**2回実施すること**
> - `/etc/hosts/`にドメイン名を追加してから再度実行することで結果が変わることがある

---

# RustScanコマンド解説

標準のスキャン
```zsh
rustscan -a <target_IP>
```
→空いているポートだけ表示

nmapのコマンドを用いたスキャン
```zsh
rustscan -r ports] -a <target_IP> -- [nmap commands]
```

複数IP同時スキャン
```zsh
rustscan -a <target_IP1>,<target_IP2>
```

ホスト(FQDN)スキャン
```zsh
rustscan -a <www.target.com>
```

CIDRでスキャン範囲指定
```zsh
rustscan -a <target_IP>/<subnet_mask>
```

スキャン対象のリスト一覧が記載されたファイルを用いて対象を同時にスキャン
```txt

# hosts.txt
192.168.0.1
192.168.0.2
google.com
192.168.0.0/30
127.0.0.1
```
```zsh
rustscan -a 'hosts.txt'
```

特定ポートのスキャン
```zsh
rustscan -a <target_IP> -p [port]
```

複数ポートのスキャン
```zsh
rustscan -a <target_IP> -p [port1],[port2]
```

ポートの範囲指定スキャン
```zsh
rustscan -<target_IP> --range [start]-[end]
```

ランダムな順序でのスキャン
```zsh
rustscan -a <target_IP> --scan-order "Random"
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