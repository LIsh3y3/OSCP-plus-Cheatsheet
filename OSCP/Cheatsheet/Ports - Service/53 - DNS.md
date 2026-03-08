# DNS列挙の目的

- ターゲットドメインの構成を把握してアタックサーフェースを明らかにすること
- 脆弱そうなドメインやサーバーに当たりをつけること
- 💡OSCPにおいては最初からIPアドレスが提供されているため、列挙の優先度低い

---

# Enumeration

## 手動

| ツール      | 対応OS    | 特徴・用途                    | 代表コマンド例                                 |
| -------- | ------- | ------------------------ | --------------------------------------- |
| host     | Linux   | Linuxデフォルト<br>シンプル・初心者向け | `host -t [record type] <domain>`        |
| nslookup | Windows | Windowsデフォルト(LOLBIN)     | `nslookup -type=[record type] <domain>` |
| dig      | Linux   | 高機能・詳細な情報取得向け            | `dig ANY <domain>`<br>（ANY =  一括取得）     |
（record type : [53 - DNS](#DNS%20record)）

---

## 自動

### dnsrecon

- pythonベースではやい
- 様々なモードを選択し、ほしい情報を選択して取得できる

一括スキャン
```sh
dnsrecon -d <domain> -n <name_serverIP> -a 
```

逆引きスキャン
```zsh
dnsrecon -r <IP/subnet→例：51.222.169.0/24> -n <name_serverIP>
```

細かいスキャン（`-a`の実行内容を細分化）
```sh
# スタンダートスキャン
dnsrecon -d <domain> -t std

# ブルートフォース
dnsrecon -d <domain> -D <wordlist> -t brt

# ゾーン転送
dnsrecon -d <domain> -t axfr
```

### dnsenum

- モードを選択できないし遅い
- 1コマンドでレコード列挙・逆引き・サブドメイン列挙をまとめて実施

```zsh
dnsenum <domain>
```

### gobuster

ブルートフォースでサブドメイン列挙する時に使う
```zsh
gobuster dns -d <domain> -w <wordlist> -t 10
```

---

# 補足情報

## DNS列挙のセオリー

大きな流れとしては、
1. ターゲットのホスト名を解決し、DNSレコードの情報を入手する
2. 入手したIPアドレスの情報から、サブネットの範囲を推測し、同じサブネットに属するホストのドメイン名を列挙する（PTRレコードを活用）

## 自動化ツール（dnsrecon・dnsenum）の動作原理

同じサブネットに属するドメインを逆引き検索
```zsh
for ip in $(seq 200 254); do host $ip; done | grep -v "not found"
```

サブドメインを列挙する
```zsh
for sub in $(cat <wordlist>); do host $sub.<domain>; done | grep -v "not found"
```
- `grep -v`: invert match = 該当しないものを出力する
- `seq`：連番

---

# Misc

## DNS record一覧表

| レコード名 | 読み方            | 用途・役割                            | 例                                           |
| ----- | -------------- | -------------------------------- | ------------------------------------------- |
| NS    | Name Server    | 権威DNSサーバの指定（そのドメインを管理するDNSサーバ）   | `NS ns1.example.com`                        |
| A     | Address        | ホスト名→IPv4アドレスを解決                 | `A 192.168.1.1`                             |
| AAAA  | Quad A         | ホスト名→IPv6アドレスを解決                 | `AAAA 2001:db8::1`                          |
| MX    | Mail Exchange  | メールサーバの指定                        | `MX mail.example.com`                       |
| PTR   | Pointer        | **逆引き**用。IP→ホスト名を解決              | `PTR www.example.com`                       |
| CNAME | Canonical Name | 別名（エイリアス）の設定                     | `CNAME www.example.com`                     |
| TXT   | Text           | **任意のテキスト**情報。認証情報やSPFレコードなどにも使う | `TXT "v=spf1 include:_spf.google.com ~all"` |

## 参考

- [Pentesting DNS - HackTricks](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-dns.html)