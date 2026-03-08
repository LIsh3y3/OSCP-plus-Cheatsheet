[⚡️SSTI](⚡️SSTI.md)

#### 注目ポイント

- admin権限で利用可能なポスト(POST)の編集機能
- update forgot email機能
- View detailsと"Unfortunately this product is out of stock"という文字
![[Pasted image 20240214205123.png | 300]]

---
## Detect

- Intruderで、下記コマンドの出力結果 (Grep -Matchで77700を指定) やエラーメッセージを観察。
- 出力箇所は、直接のレスポンスやリダイレクト先ではなく、(Stored的に設定値としてinjectされた結果) 別ページになることも多いので、視野狭窄に陥らず機能の連関を把握しておくこと。
	- ↑その場合はIntruderではなく1個ずつ試しが必要。

- 下記の1行目はSSTIに脆弱かどうかのfuzzing。
- 500エラーなど普通とは異なるレスポンスが出力されたら脆弱
```
${{<%[%'"}}%\.,
{fuzzer}
{{fuzzer}}
${fuzzer}
${{fuzzer}}
#{fuzzer}
#{{fuzzer}}
${fuzzer}
${{fuzzer}}
*{fuzzer}
*{{fuzzer}}
<%=fuzzer%>
{% debug %}
777*100
}{777*100
}}{{777*100
}${777*100
}}${{777*100
}#{777*100
}}#{{777*100
}*{777*100
}}*{{777*100
%><% = 777*100
```

---
## Identify テンプレートエンジン

### SSTImap(自動)

```zsh
cd SSTImap
```

基本形(GET)
```zsh
./sstimap.py -u '[脆弱なパラメタ含むURL]'
```
基本形(POST)
```ZSH
./sstimap.py -u '[脆弱なエンドポイントまでのURL]' \
--method POST \
--data 'blog-post-author-display=1&csrf=CSRF_TOKEN' \
```

###### Fuzz対象を指定する場合

```zsh
--marker FUZZ \
```
- `FUZZ`の文字列をファジング対象箇所に注入する(`blog-post-author-display=FUZZ`)

⬇️結果

![[Pasted image 20240214210941.png]]

---
### 手動

- 手動で検出してSSTImapで悪用できるかはビミョー

- 出力箇所が入力箇所と異なる場合にSSTImapで検出できない
- また、`Handlebars`など一部のテンプレートエンジンも無理
- [🔍SSTI](#Detect)のエラーメッセージにテンプレートエンジンが書いてないかをまず確認する
	- 特定した -> [⚡️SSTI](⚡️SSTI.md#手動)
	- 特定できなかった -> [2. SSTIの検出とテンプレートエンジンの特定](2.%20SSTIの検出とテンプレートエンジンの特定.md)