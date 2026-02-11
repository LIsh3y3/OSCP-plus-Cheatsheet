[[⚡️Web cache poisoning]]

###### 特徴

- 単独の攻撃というより、攻撃配布の手段と捉えると良い。
- Scanすると、他の脆弱性 + Web cache poisoningの脆弱性が表示される
- ==XSS==や==open redirect==と一緒に検出されることが多い。基本はXSSを疑う
![[Pasted image 20240210153905.png]]

---
#### 注目ポイント

- 怪しげなJavaScript
	- `/resources/js/tracking.js`
	- `/js/geolocate.js`等

- `GET /`リクエスト

---
## Detect

- [[#注目ポイント]]をActive scanにかける
- Hostヘッダの値でレスポンスを検索していく。該当する箇所でリソースの読み込みなどしているかも
```html
<link rel="canonical" href='//TARGET_NET/?hogehoge=aaa'/>
```


- もし絶対に存在しないファイル名にリクエストを送り、レスポンスにそのファイル名が反射されていたら -> [[⚡️Web cache poisoning]]の正規化されたCache keyの悪用を疑う
