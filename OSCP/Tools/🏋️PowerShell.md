 [ ] [[Powershellの基本]]と統合する
### PowerShellとは何か

- .NETフレームワークを用いて構築されたWindowsスクリプト言語であり、シェル環境
- これにより、Powershellはシェルから直接.NET関数を実行することも可能。
- Powershellのコマンドのほとんどは、cmdletと呼ばれ、.NETで記述されている。
- 他のスクリプト言語やシェル環境とは異なり、これらのコマンドレットの出力はオブジェクトであり、PowerShellはややオブジェクト指向になっている。
- また、コマンドレットを実行すると、出力されたオブジェクトに対してアクションを実行することができる（あるコマンドレットから別のコマンドレットに出力を渡すのに便利）。
- 通常のコマンドレットの形式は==動詞-名詞==で表現され、例えばコマンドを一覧表示するコマンドレットは Get-Command となる
- 使用される[動詞の一覧](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands?view=powershell-7.3&viewFallbackFrom=powershell-7)

### 関数の使用について（Import-Module)

- メモリへ関数の定義をインポートしないと、関数は使えない
```powershell
Import-Module .\powerview.ps1
```

---

# 基礎的なPowerShell Commands

## Get-Help

- コマンドレットに関する情報を表示する。
- 特定のコマンドに関するヘルプを取得するには、
```powershell
Get-Help [Command-Name]
```
- `-Examples`: このオプションを使用すると、コマンドの使い方の詳細が表示される。表示量膨大。

## Start-Transcript

コマンドとその実行内容をテキストファイルにダンプしたいときに使う
	列挙過程で実行した内容をメモするときに便利
	⚠️ファイルは追記される
	⚠️WinRM接続状態では内容が二重で出力されてしまうため、[[#Tee-Object]]を使う
```powershell
# ログ記録を開始
Start-Transcript -Path ".\log.txt"

# ここで色々なコマンドを実行
whoami
Get-LocalUser
Get-Process

# ログ記録を停止
Stop-Transcript

# ログの内容をnotepadで開く
notepad log.txt
```

## Tee-Object

ファイルに出力を記入しつつ、ターミナルに出力結果を表示する
```powershell
.\winPEASx64.exe | Tee-Object -FilePath winpeas_log.txt
```

## Get-Command

- 現在のコンピューターにインストールされているすべてのコマンドレットを表示。
- このコマンドレットでは、次のようなパターンマッチが可能
- `Get-Command Verb-*` ｜ `Get-Command *-Noun`
- 例えば`Get-Command New-*`とする

## Select-String

- ファイルの"中身"を検索する
	- これも`Where-Object`と同じく、grepのような使い方ができる
```powershell
# cmdの場合はfind
tasklist | Select-String "sync"

type <file> | Select-String -Pattern "pass(word)?|pwd|cred(ential)?"
```

- 補足：ファイル名を検索したいなら、[[#オブジェクトのフィルタリング]]

## Job

- [ ] jobの開始、終了、結果受け取りなど方法を記載する
```powershell
# jobの開始(パスは絶対パス)
Start-Job {
C:\Users\ariah\Snaffler.exe -s
}

# job一覧確認
Get-Job

# jobの実行状況確認
Receive-Job -id <job_id>
```
- `Receive-Job`に`-Wait`を追加すると、処理が終わるまでそこで待機
- 参考リンク[[PowerShell] 処理をバックグラウンドで実行する (Start-Job) - 晴耕雨読](https://tex2e.github.io/blog/powershell/background-job)

## オブジェクトの操作

- すべてのコマンドレットの出力はオブジェクト。
- 実際に出力を操作したい場合は、下記のことを理解する
	- 出力を他のコマンドレットに渡す方法　→ `|`を使用
	- 特定のオブジェクト cmdlet を使用して情報を抽出する方法

- 他のシェルと比較して大きな違いは、パイプの後のコマンドにテキストや文字列を渡すのではなく、次のコマンドレットに「オブジェクト」を渡すこと。
- オブジェクトにはメソッドとプロパティが含まれる。メソッドはコマンドレットからの出力に適用できる関数、プロパティはコマンドレットからの出力に含まれる変数と考えられる。
- メソッドやプロパティの詳細を表示するには、コマンドレットからの出力をGet-Memberコマンドレットに渡す
- `Verb-Noun｜Get-Member`
- （例）Get-Command のメンバー(オブジェクトとプロパティ)を表示する
```powershell
Get-Command｜Get-Member -MemberType [Method | Property]
```

### 直前のcmdletからオブジェクトを作成する

- `Select-Object`で行う。
- 直前のコマンドレットの出力からプロパティを取り出し、新しいオブジェクトを作成する
- (例)：ディレクトリをリストアップして、モードと名前だけを選択する
```powershell
Get-ChildItem | Select-Object -Property Mode, Name
```
![[Pasted image 20230604094002.png | 100]]

- また、次のフラグを使用して特定の情報を選択することもできる:
	- first: 最初の x 個のオブジェクトを取得
	- last: 最後の x 個のオブジェクトを取得
	- unique: 一意のオブジェクトのみを表示
	- skip: x 個のオブジェクトをスキップ

- (例)：最初の3つのファイルの名前とサイズを取得する場合
```powershell
Get-ChildItem | Select-Object -First 3 Name, Length
```

💡cmdlet以外でも特定の列のみを表示する方法
```powershell
<cmd> /fo csv | ConvertFrom-Csv | Select-Object <Property>
```
![[Pasted image 20260107140211.png]]

### オブジェクトのフィルタリング

- `Where-Object`で行う
- プロパティの値でフィルタすることができ、bashの`grep`のように使える

- 2番目のバージョンは、`$_`演算子を使用して、`Where-Object`に渡されたすべてのオブジェクトを反復処理する
- `-operator`の部分に入る演算子：
	- `-Contains`：プロパティ値のいずれかの項目が、指定された値と完全に一致する場合  
	- `-EQ`：指定された値と同じである場合  
	- `-GT`：指定された値より大きい場合
	- `-match`：指定された正規表現と一致する場合
	- `-like`：指定された値と近いもの（ワイルドカード使用可能）
	- `-notlike`：指定された値とパターンが一致しないもの
	- `-notin`：指定されたリストを含まないもの
- 🔗[operatorの詳細](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.3&viewFallbackFrom=powershell-6)

- 基本シンタックス
```powershell
# powershell 3.0以降
Where-Object -Property <プロパティ名> -<operator> <value>
# 3.0未満
Where-Object {$_.<プロパティ名> -<operator> <value>}`
```

- 具体例
```powershell
# 停止したプロセスを表示する
Get-Service | Where-Object -Property Status -EQ Stopped

# ⭐️一致する名前を検索
dir | where-object { $_.Name -match '<string>' }

# 結果から除外
ps | Where-Object -Property ProcessName -notin "svchost","sshd"
```

### オブジェクトのソート

- `Sort-Object`で行う
- (例): ディレクトリのリストをソートする
```powershell
Get-ChildItem | Sort-Object
```

### 具体例

###### インストールされたCmdletのトータルの数を表示する

1. まずGet-Commandで使用できるPropertyを一覧表示する
```powershell
Get-Command | Get-Member -MemberType Property
```
- PropertyにCommandTypeがみつかった

2. Get-Commandで使用できるValueの種類を一覧表示する
```powershell
Get-Command | Select-Object -ExpandProperty CommandType -Unique
```

3. カウントする
```powershell
Get-Command | Where-Object -Property CommandType -eq Cmdlet |measure
```
- `measure`: 最小値、最大値、平均値、トータルの数などが表示される

###### ファイルの検索と内容表示

1. 特定のファイルを検索する
```powershell
Get-ChildItem -Path C:\ -Include *[FileName]* -File -Recurse -ErrorAction SilentlyContinue
```
- `-ErrorAction SilentlyContinue`: アクセス権のないディレクトリにアクセスしたときもエラーメッセージを表示せず続行する。

- ファイルの内容を表示する
```powershell
Get-Content "[Path to file]"
```

###### ファイルのハッシュを閲覧する

1. 使用できる**コマンドレットを検索する**
```powershell
Get-Command | Where-Object -Property Name -ilike "*[]*"
```
- `-ilike`: 大文字小文字問わない。
- Get-FileHashコマンドレットが見つかった

2. MD5ハッシュを表示する
```powershell
Get-FileHash -Path "[path to file]" -Algorithm MD5
```

###### パスの確認

- 現在のパスを確認する
```powershell
Get-Location
```

###### ファイルをデコードする
```powershell
certutil -decode "[path to encoded file]" [outputfile which is file decoded]
```

---

# Basic Scripting

- スクリプトを自分で書くと、より複雑で強力な動作ができるようになる。
- ==NmapやPythonが使えない時の代替手段となる==
- PowerShell ISE（Powershellテキストエディター）を使用する。
- Powershellスクリプトは通常.ps1というファイル拡張子を持っている

- 具体例を示すために、ポート番号のリストが与えられ、このリストを使ってローカルポートがリスニングしているかどうかを確認したいというシナリオを想定する。
-  例：
```powershell
$system_ports = Get-NetTCPConnection -State Listen

$text_port = Get-Content -Path C:\Users\Administrator\Desktop\ports.txt

foreach($port in $text_port){

    if($port -in $system_ports.LocalPort){
        echo $port
     }

}
```

## 使い方

- 検索からPowerShell ISEと検索し、アプリを開く。
- .ps1というファイル拡張子をもつファイルをFileタブから開く。
- もしくはFileタブからNewを押し、新たにスクリプトを作成する
- スクリプトを実行するには、PowerShellからファイルを実行するか、ISEの緑のボタンを押して実行する
![[Pasted image 20230604162232.png | 400]]

### ルールや記述方法

- 変数への格納方法
```powershell
$variable_name = value
```
- 例えば
```powershell
$system_ports = Get-NetTCPConnection -State Listen
```

- foreach文
```powershell
foreach($new_var in $existing_var){}
```
- これは、すでに存在する変数(`$existing_var`)を１つずつ取り出し、新しい変数(`$new_var`)に入れる。

- if文
```powershell
if($port -in $system_ports.LocalPort){}
```
- [比較演算子の詳細](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-7.3&viewFallbackFrom=powershell-6)
- `-in`:はリストに含まれているか検証する比較演算子。ここでは`$system_ports`のプロパティである`LocalPort`に`$port`が含まれているかチェックしている。

---

# Intermediate Scripting (Nmap)

- ==Nmapの代替手段==としてのPowerShellスクリプトの作成

手順
- スキャンするIP範囲を決定する
- スキャンするポート範囲を決定する  
- 実行するスキャンの種類を決定する

- 例として、通常のTCP Connectionで、localhostとポートのスキャンをする
```powershell
for($i=[start num]; $i -le [end num]; $i++){ 
	Test-NetConnection [IPAddress(localhost)] -Port $i
}
```
- `-le`: less then。`start num` ~ [end num]まで++する

---
