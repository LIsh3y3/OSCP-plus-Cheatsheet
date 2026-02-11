1. portスキャンでMicrosoft HTTPAPI httpd 2.0がport 3333でopenになっていることを確認
2. httpのエンドポイントがいくつかあり、アクセスすると"can't GET /list-running-procs"と表示されたため、POSTリクエストに変更し、適当なリクエストを送付すると、内部で動作しているプロセスが出力された
3. プロセスにクレデンシャルがあったため、それを用いてsshでアクセス
4. 内部を列挙すると、127.0.0.0:80でローカルサービスが動作していた
5. クレデンシャルを使ってftpでアクセスすると暗号化されたpdfファイルがあったため、保存し、解読
```zsh
pdf2john Infrastructure.pdf > pdf.hash
john --format=PDF --wordlist=/usr/share/wordlists/rockyou.txt pdf.hash 
...
```

6. PDFを閲覧すると、エンドポイントが記載されていた
```
Temporary Command endpoint: http://nickel/?
Backup system: http://nickel-backup/backup
NAS: http://corp-nas/files
```

7. トンネリングし、ligoloのmagic cidrを使ってローカルで動作している80サービスのhttp://nickelにアクセスし、コマンドを実行した
	- 反省：pdfのコメントと細かい表記(`?`)を無視していたため、==無駄なコメントはないと認識すること==
	- ローカルサービスがあれば、十中八九、有益な情報がある

8. ローカルサービスがSYSTEM権限で動作していたため、powershellワンライナーでリバースシェルを取得
