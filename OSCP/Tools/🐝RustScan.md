# RustScanとは

- [RustScan](https://github.com/bee-san/RustScan)
- Rust言語で構築されたオープンソースのモダンポートスキャナー
- 全65,535ポートを最速で数秒以内にスキャンできる（バッチサイズ・タイムアウト次第）
- **NmapのラッパーとしてNmapを置き換えるものではなく**、補完するツール
    - Openポートを高速で発見し、その結果を自動的にNmapへパイプするという思想
    - Nmapがインストールされていることが前提
- RustScan Scripting Engine（RSE）により、Python・Lua・Shellで書かれた任意のスクリプトを実行可能（[詳細](https://github.com/RustScan/RustScan/wiki/RustScan-Scripting-Engine)）
- Adaptive Learning機能により、実行環境のOSを把握し、スキャン性能を動的に自動調整する

---

# NmapとRustScanの比較

| 項目                | Nmap                       | RustScan                 |
| ----------------- | -------------------------- | ------------------------ |
| 主な用途              | 総合的なネットワーク調査・脆弱性スキャン       | 高速なオープンポート発見 → Nmapへパイプ  |
| スキャン速度            | 遅い（全ポートに数十分かかることもある）       | 速い（全65kポートを数秒〜数十秒でスキャン）  |
| 正確性・安全性           | 高い（偽陽性・偽陰性が少ない）            | 低め（偽陽性が出やすい）             |
| ブルーチームによる検知       | 比較的検知されにくい（オプション次第）        | 検知されやすい（デフォルトは高速・高負荷なため） |
| スクリプト機能           | NSE（Nmap Scripting Engine） | RSE（Python・Lua・Shell対応）  |
| IPv6 / CIDR対応     | ○                          | ○                        |
| Adaptive Learning | なし                         | あり（OS環境に応じて自動調整）         |

**実世界ではNmapの方が推奨。** RustScanはCTFや初期偵察フェーズでの速度重視の場面に向いている。

> [!WARNING]
> - Nmap同様、**結果を信じ込まない**（検出しなかった ≠ サービスが動作していない）
> - 偽陽性・偽陰性を軽減するため、**2回実施すること**
> - `/etc/hosts` にドメイン名を追加してから再度実行することで結果が変わることがある

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