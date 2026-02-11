OSINT = Passive Recon

-  [wappalyzer](https://www.wappalyzer.com/)
	Web サイトで使用されているテクノロジーを特定する

- [Wayback Machine](https://archive.org/web/)
	ドメイン名を検索すると、サービスが Web ページをスクレイピングしてコンテンツを保存した回数が常に表示される。このサービスは、現在の Web サイトでまだアクティブな可能性がある古いページを発見するのに役立つ。読み込んだurlとかもわかる

- [GitHub](https://github.co.jp/)
	GitHub の検索機能を使用して、会社名または Web サイト名を検索し、ターゲットに属するリポジトリを見つけられるかも。見つけたら、まだ発見していないソース コード、パスワード、またはその他のコンテンツにアクセスできる可能性がある

- S3 bucket 
	amazonのストレージサービス。設定ミスで機密情報を公開しているかも。nameのとこに会社名とかいれて　**{name}** -assets、  **{name}** -www、  **{name}** -public、  **{name}** -private など。

- [SSL/TLS 証明書 - crt.sh](https://ui.ctsearch.entrust.com/ui/ctsearchui)
	ドメインに属するサブドメインを発見することができる。→攻撃可能な範囲を広げる

- [Sublist3r](https://www.kali.org/tools/sublist3r/)
	サブドメインを列挙する
```zsh
sublist3r -d [domain] -t [thread] -e [search engine(bing等)]
```

---
