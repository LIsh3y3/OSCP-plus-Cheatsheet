# Feroxbusterとは

- Webサーバに隠されたディレクトリ、ファイルを再帰的に列挙するためのツール
- Rustで書かれており、高速かつ効率的な列挙が可能（⚠️再帰探索でリクエスト数も多いので遅くなりやすい）
- **Gobusterとの主な違い**：
	- ✅ **再帰探索が可能**：発見したディレクトリを自動的に深掘り
	- ✅ ワイルドカード検出が自動
	- ✅ 動的なフィルタリング機能が豊富
	- ❌サブドメイン・vhost列挙は非対応（ディレクトリ/ファイル列挙専用）
 - [Feroxbuster - GitHub](https://github.com/epi052/feroxbuster)

---

## いつFeroxbusterを使うべきか

| 状況           | 推奨ツール       | 理由                |
| ------------ | ----------- | ----------------- |
| ディレクトリ構造が深い  | Feroxbuster | 再帰探索で自動的に深掘り      |
| 1階層だけ高速スキャン  | Gobuster    | 再帰がない分、高速         |
| サブドメイン列挙     | Gobuster    | Feroxbusterは非対応   |
| vhost列挙      | Gobuster    | Feroxbusterは非対応   |
| ワイルドカード対策が面倒 | Feroxbuster | 自動検出・除外           |
| 複雑なフィルタリング   | Feroxbuster | サイズ、ワード数等で柔軟に除外可能 |

推奨アプローチ：
1. まずGobusterで1階層の高速スキャン
2. 気になるディレクトリ(301ステータス、api等)が見つかったらFeroxbusterで再帰的に深掘り

---

# 基本的な列挙コマンド

基本スキャン（再帰探索あり）
```zsh
feroxbuster -u http://<TargetIP|Domain>:<Port>/ -r -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --auto-tune -o feroxbuster.txt
```

拡張子指定 + 再帰探索の深さ制限
```zsh
# ファイルアクセスエラー回避のため、同時に開けるファイルの最大数を調整
ulimit -n 8192

# スキャン
feroxbuster -u http://<TargetIP|Domain>:<Port>/ --depth <num> -r -k -w  /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --auto-tune -o feroxbuster.txt -x '<extensions>'
```
- `-x`にはターゲットのテクノロジースタック等に応じて拡張子を指定する（[[#拡張子リスト]]）

再帰探索を無効化（Gobusterのように1階層のみ）
```zsh
feroxbuster -u http://<TargetIP|Domain>:<Port>/ -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --auto-tune -n -o feroxbuster.txt
```

---

# よく使うオプション

| オプション                | 説明                               |
| -------------------- | -------------------------------- |
| `-t, --threads`      | スレッド数（デフォルト：50）                  |
| `-x, --extensions`   | 拡張子指定（例：`-x php,html,txt`）       |
| `-n, --no-recursion` | 再帰探索を無効化                         |
| `--depth`            | 再帰探索の深さ（デフォルト：4）                 |
| `-k`                 | SSL証明書検証をスキップ                    |
| `-C`                 | 特定のステータスコードを除外（例：`-C 200,401`）   |
| `-s`                 | 表示するステータスコード（例：`-s 200,301,302`） |
| `-S`                 | 特定のサイズを除外（例：`-S 1234`）           |
| `-W`                 | 特定のワード数を除外                       |
| `--auto-tune`        | スレッド数を自動調整（エラー時に減速）              |

---

# トラブルシューティング / Tips

## 拡張子リスト

| スキャンタイプ  | 拡張子リスト                                                    |
| -------- | --------------------------------------------------------- |
| 包括的      | html,htm,txt,sh,php,cgi,asp,aspx,jsp,pl,py,pdf            |
| PHP      | php,html,htm,txt,pdf,bak,old,inc,phtml                    |
| ASP.NET  | aspx,asp,html,htm,txt,pdf,config,cs,ashx,asmx             |
| JSP/Java | jsp,html,htm,txt,pdf,do,action,jspx,jsf                   |
| 静的サイト    | html,htm,txt,pdf,xml,json,css,js                          |
| 設定ミス検出   | bak,old,backup,swp,tmp,zip,tar.gz,sql,log,conf,config,ini |
| API/データ  | json,xml,yaml,yml,csv,txt                                 |
| 軽量/高速    | html,php,txt,pdf                                          |

## よくある問題と対処法

| 問題       | 原因                        | 対策                              |
| -------- | ------------------------- | ------------------------------- |
| 結果が多すぎる  | ワイルドカードDNS / 同じサイズの404ページ | `-C 404 -S <size>` でフィルタリング     |
| スキャンが遅い  | 再帰が深すぎる                   | `--depth 2` で深さを制限              |
| エラーが多い   | スレッド数が多すぎる                | `--auto-tune` を使うか `-t 50` に減らす |
| 何も見つからない | ワードリストが適切でない              | 他のワードリストを試す（[[#補足：利用ワードリスト候補]]） |

## 利用ワードリスト候補

```zsh
# ワード数：4614
/usr/share/wordlists/dirb/common.txt
# ワード数：63088
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
# ワード数：220560
/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

---

