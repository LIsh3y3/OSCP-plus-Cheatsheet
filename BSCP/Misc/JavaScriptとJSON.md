
## 文法

- ⭐️JavaScriptにはオブジェクトのプロパティにアクセスする方法が2種類ある(参考記事: [Qiita](https://qiita.com/TakanoriOkawa/items/7de0b158afa17ef2dae8))
	- ドット記法(`.`): `document.cookie`
	- ブラケット記法(`['']`): `document['cookie']`
- `const`: 定数
- `オブジェクト.メソッド`の形
- `document`オブジェクト：　HTMLの要素を取得できる
- `window.location.href`オブジェクト：　現在のURLを取得する
- `const function = () => {}`: [参考：アロー関数](https://it-biz.online/web-design/arrow-function/)
- `オブジェクト.length == 2 ? A : B;`: 三項演算子。真ならばA、偽ならばBを実行
- ⭐️`this`は現在のオブジェクトを指す ([参考：JavaScriptのthisを理解する](https://qiita.com/takkyun/items/c6e2f2cf25327299cf03))
	- `this.password`＝現在のオブジェクトのpassword

- ↓文字列の連結：結果は今日はいい天気ですね
```js
var str = 'ですね！'
var result = '今日は' + '良い天気' + str;
 
console.log( result );
```

---
## XSS関連のJavaScriptやHTMLタグの動作

- `<script>`: HTMLを描画する前に表示される。ただし、解析されるのは他のHTMLタグの解析が終わったあと
- `<img src=1 onerror=...>`: 画像として実行する。src=1では存在しないので必ずerrorイベントが起きるので、イベントハンドラの`onerror`が発火する
- `イベントハンドラ=1`: 何も実行されない。このイベントハンドラが有効かの検証に使用される
- `<iframe src=[target_server]?[vuln_param]=[payload] onload=this.style.width='100px'>`: 反射型XSSで使用。`onload`はHTML要素が完全に読み込まれた後に発火するイベント。

- `{method:'post',body:'/post?postId=1}` のとき、制御可能な値がpostIdだとすると、`&'}`で現在のオブジェクトを閉じることができる
	- -> `{method:'post',body:'/post?postId=2&'}`

---
## メソッド

###### 基本

- グローバルオブジェクト(`window`など)は、あえて指定せずともそのメソッドを呼び出せる
```js
window.location
```
```js
location
```

###### toString
- すべてのオブジェクトから使用できるメソッド

###### getElementsByClassName()
-   ()で指定した名前が含まれるすべてのHTML要素を集めた"HTMLCollection"という配列風のオブジェクトを作成する
	- `const example = getElementsByClassName('example')[0];`

###### location.search
- クエリストリングの値をオブジェクトに入れる

###### document.write
- HTMLに書き込む
```js
document.write('<select name="storeId">');
```
- →HTMLに`<select name="storeId">`と書き込まれる。

###### innerHTML
- 意：HTMLのタグの中身を取得したり、中身を指定した値で書き換える
- [参考記事](https://www.sejuku.net/blog/26442)

---
## イベントについて

- [Mozila:イベント入門](https://developer.mozilla.org/ja/docs/Learn/JavaScript/Building_blocks/Events)
- [イベントの具体例](https://tcd-theme.com/2021/05/javascript-eventhandler.html)

- イベントに反応するには、イベントに**イベントハンドラー**を取り付ける
- イベントハンドラー ＝ イベントが発行されたときに実行するコードのブロック（作成した JavaScript 関数）
- このようなコードのブロックが、イベントに応答して実行するように定義されている場合、**イベントハンドラーを登録**していると言う。
- イベントハンドラー≒イベントリスナー。
	- 厳密に言えば別のもの。イベントリスナーはイベントの発生を待ち受けし、イベントハンドラーは発生したイベントに応答して動作するコード

- イベントハンドラーは`on ＋ イベント名`のような形式
- イベントハンドラ内では`<scrit>`タグは必要ないが、自作関数を呼び出すときは別途関数の定義が必要
```html
<p id="text">Click this button.</p>
<input type="button" value="button" onclick="changeText()">

<script>
  function changeText() {
    let getText = document.getElementById('text');
    getText.textContent = 'You clicked!';
  }
</script>
```

---
## JSONについて

- JSONはJavaScriptの文法とほぼ同じ
- JavaScriptだけでなくどの言語でも使用可能

---
## テンプレートリテラルとは

- 文字列の中にJSの式を組みこむ文字列リテラル
- 文字列を\`\`で囲み、式(プレースホルダ）は`${}`で囲む
```js
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

---
## プロトタイプについて

###### 参考

- 詳細 [PortSwigger: JavaScript prototypes and inheritance](https://portswigger.net/web-security/prototype-pollution/javascript-prototypes-and-inheritance)
- [Qiita](https://qiita.com/awakia/items/8ff451ca5f8ae0122be7)

#### prototypeとは

- 言葉自体の意味は、あとで何か作る時のための最初の模型
- ==JavaScriptにおいてはほとんどのデータ型がオブジェクトとして振る舞う==事ができる。
- プリミティブ型(int, char)でもオブジェクトのように振る舞える
- それぞれのオブジェクト(ほとんどのデータ型のこと)は、割り当てられたプロトタイプ(組み込みプロトタイプ)のプロパティを自動的に継承する

###### 要点

1. 基本の概念:
      - JavaScriptのすべてのオブジェクトは他のオブジェクトをプロトタイプとして継承します。この継承されたオブジェクトを「プロトタイプオブジェクト」と呼びます。

2. 自動的な継承:
	-  新しく作成されたオブジェクトは、JavaScriptに組み込まれているプロトタイプ（例: `Object.prototype`, `String.prototype` 等）を自動的に継承します。

3. プロトタイプからの継承:
-  オジェクトはプロトタイプオブジェクトのプロパティやメソッドを継承します。そのため、あるオブジェクトが持っているプロパティやメソッドは、プロトタイプチェーンを通じて継承オブジェクトでも使用できます。

4. 組み込みプロトタイプ(グローバルプロトタイプ[[#グローバルプロトタイプの例]]):
	- JavaScript には組み込みのプロトタイプがいくつかあります。例えば、文字列には `String.prototype` が、配列には `Array.prototype` があります。これらの組み込みプロトタイプには、データ型ごとに便利なメソッドやプロパティが予め定義されています。

5. メソッドの利用:
	- 文字列オブジェクトが `toLowerCase()` メソッドを持っているのは `String.prototype` にこのメソッドが定義されているからです。このように、プロトタイプに定義されたメソッドは、そのプロトタイプを継承するすべてのオブジェクトで利用可能になります。

###### グローバルプロトタイプの例

```js
let myObject = {};
Object.getPrototypeOf(myObject);    // ⭐️Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);	    // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype
```

#### prototype chain

- すべてのprototypeはObject.prototypeに帰結する。ちなみにObject.prototypeのprototypeはnull。
- チェーン上のすべてのオブジェクトからプロパティを継承する
	- usernameはString.prototypeとObject.prototypeの2つとも

![[Pasted image 20240203160441.png | 600]]

#### オブジェクトのprototypeへのアクセス方法

- `__proto__`というプロパティでprototypeにアクセスする方法がデファクトスタンダード
- `__proto__`プロパティを使ってprototypeとそのプロパティを読み取ったり、必要に応じて再割り当てすることも可能
###### 例

- 上記画像の例でusernameオブジェクトを取り上げる
- ↓アクセス方法(通常のオブジェクトのプロパティへのアクセスと一緒)
```js
username.__proto__
username['__proto__']
```
- prototype chainの階層を遡る
```js
username.__proto__                        // String.prototype
username.__proto__.__proto__              // ⭐️Object.prototype
username.__proto__.__proto__.__proto__    // null
```

#### prototypeの編集

- prototypeのプロパティを追加・上書きすることが可能
```js
String.prototype.removeWhitespace = function(){
    // 前と後ろの空白を消す処理
}
```
- prototype chainの継承先でも使える
```js
let searchTerm = " example ";
searchTerm.removeWhitespace(); //"example"
```

---
## トラブルシューティング

###### View Exploitでうまくできるのに、クリアできない

- `Content-Type: application/javascript`の場合は以下のみでOK
```js
alert(document.cookie)
```
- `<script>`タグはいらない。この状態でView Exploitしてもプレーンテキストが表示されるのみだが、気にしない