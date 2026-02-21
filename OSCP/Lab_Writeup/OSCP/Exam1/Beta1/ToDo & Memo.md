- http://localhost/launch6
	- が気になる

- sashaとして、phishing送信の可能性もある（ただ、smtpサーバーポートがopenになっていないので、可能性としては低い）

---

状況の整理：

- iisstart.htmのferoxbuster?
- ~~マシンをrevertして、再度アクセス試行？（万が一アカウントロックがかかっていたらいけないしね。）~~
- userame == passwordで認証試行？
	- 
- そもそもemailの読めるやつ以外は、ちゃんと解凍できているのか？
	- →できている

- ~~rpcをもう少し詳細に列挙してみる~~
	- https://github.com/camercu/oscp-prep/blob/main/CHEATSHEET.md#211-msrpc-and-netbios---135137139

- ~~launch6でauthenticatedなんとかで試行~~


- autoreconをport 80と443に実行する
```zsh
sudo autorecon 192.168.115.111 -p 80,443
```