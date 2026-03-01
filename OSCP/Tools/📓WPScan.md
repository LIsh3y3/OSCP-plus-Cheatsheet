WPSCanとはなにか、なにをスキャンするかは、公式ドキュメントを参照→[WPScan docs - Github](https://github.com/wpscanteam/wpscan/wiki/WPScan-User-Documentation)

- 用途は、**Wordpress**のテーマ、プラグインから脆弱性を検出する
	- Wordpress本体（コア）は開発者が迅速にセキュリティパッチを適用するが、コミュニティプラグインやテーマは脆弱性が放置されたまま残ってることが多い

- ❗️ターゲットがWordpressを使っていたら、必ず実行する

---

# 基本スキャン

WPScanのデータベースを更新する
```zsh
wpscan --update
```

ターゲットのWordPressの有名なプラグイン・テーマとその脆弱性を列挙
```zsh
wpscan --url http://<target_IP> --no-banner --enumerate p,t --plugins-detection aggressive -o WebEnum/wpscan --api-token <API_token>
```
- `[+]`発見情報、`[!]`脆弱性情報
- 💡`out of date`に着目
- Error：Scan aborted -> Kaliからwpscan.comに接続できない場合は、[[vmware-kali linux#トラブルシューティング]]を参照

## 補足：APIトークンについて

- APIトークンがないと、脆弱性情報は表示されない（プラグインやテーマは表示される）
- 無料版は1日25リクエストまで
- ただし、プラグイン・テーマごとにリクエストするので、1日25回スキャンできるわけではない
	- API使えないときは、`--api-token`なしでスキャンし、searchsploitする
- APIトークン：[https://wpscan.com/profile](https://wpscan.com/profile)
![[Pasted image 20250426154502.png]]

## オプション一覧

- ログインページのURI（デフォルト: `/wp-login.php`）が異なる場合：
	- `--login-uri URI`

- `Scan Aborted: The url supplied 'http://alvida-eatery.org/' seems to be down (Timeout was reached)`
	- →`-t 1`: スレッド数減少で対策

- `--enumerate` オプション一覧表：

| オプション | 内容                     | 備考     |
| ----- | ---------------------- | ------ |
| `p`   | よく使われるプラグインの列挙         |        |
| `vp`  | 脆弱なプラグインの列挙            |        |
| `ap`  | すべてのプラグインの列挙           | 時間がかかる |
| `t`   | よく使われるテーマの列挙           |        |
| `vt`  | 脆弱なテーマの列挙              |        |
| `at`  | すべてのテーマの列挙             | 時間がかかる |
| `cb`  | configファイルのバックアップの探索   |        |
| `tt`  | Timthumbの脆弱性を含むファイルの探索 |        |
| `dbe` | データベースのエクスポートファイル探索    |        |
| `u`   | ユーザー名列挙（ID 1〜10）       |        |
| `m`   | メディアIDの列挙（ID 1〜10）     |        |
（引数なしで`--enumerate`を使った場合のデフォルト値：  `vp,vt,tt,cb,dbe,u,m`）

---

# ユーザー名の列挙とブルートフォース攻撃

1. ユーザー名を列挙する：​
```zsh
wpscan --url http://<target.com>/ --enumerate u
```

2. 見つかったユーザー名に対してパスワードのブルートフォース攻撃を行う：​
```zsh
wpscan -o wpscan.txt -f cli-no-color --no-banner --url https://<target.com>/ --usernames <username> --passwords 
<wordlist>
```

---

# ステルスモードとバイパス

プロキシを使用してスキャンを行う：​
```zsh
wpscan --url https://<target.com>/ --proxy socks5://127.0.0.1:9050
```

ランダムなユーザーエージェントを使用する：​
```zsh
wpscan --url https://<target.com>/ --random-user-agent
```

リクエスト間に遅延を追加する：​
```zsh
wpscan --url https://<target.com>/ --throttle 5
```

---

# その他のオプション

カスタムのユーザーエージェントを指定する：​
```zsh
wpscan -o wpscan.txt -f cli-no-color --no-banner --url https://<target.com>/ --user-agent "CustomUserAgent/1.0"
```

- アグレッシブスキャン
	- ✅見逃しが少ない
	- ❌時間がかかる
	- ❌ステルス性低い
```zsh
wpscan -o wpscan.txt -f cli-no-color --no-banner\
	--random-user-agent \
	--enumerate ap,at,cb,dbe,u \
	--detection-mode aggressive \
	--plugins-detection aggressive \
	--plugins-version-detection aggressive \
	--url http://<target.com>/ \
	--api-token [MY_API_TOKEN]
```