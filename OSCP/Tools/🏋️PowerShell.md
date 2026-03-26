# PowerShellの基本

## PowerShellとは何か

- .NETフレームワークを用いて構築されたWindowsスクリプト言語であり、シェル環境
- これにより、PowerShellはシェルから直接.NET関数を実行することも可能
- PowerShellのコマンドのほとんどは、cmdletと呼ばれ、.NETで記述されている
- 他のスクリプト言語やシェル環境とは異なり、これらのコマンドレットの出力はオブジェクト
- コマンドレットを実行すると、出力されたオブジェクトに対してアクションを実行することができる（あるコマンドレットから別のコマンドレットに出力を渡すのに便利）
- 通常のコマンドレットの形式は **動詞-名詞** で表現され、例えばコマンドを一覧表示するコマンドレットは `Get-Command` となる
- 🔗[使用される動詞の一覧](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands?view=powershell-7.3&viewFallbackFrom=powershell-7)

---

## 型・変数・表現

### 型

| PowerShell型 | .Net型                        | 備考           |
| ----------- | ---------------------------- | ------------ |
| int         | System.Int32                 |              |
| long        | System.Int64                 |              |
| string      | System.String                | 文字列          |
| double      | System.Double                | 数値、浮動小数点     |
| bool        | System.Boolean               |              |
| array       | System.Object\[]             |              |
| hashtable   | System.Collections.Hashtable |              |

### 変数への格納

```powershell
$variable_name = value
```

### コマンドの実行

- 動詞-名詞の組合せがコマンドの基本
- オブジェクトを変数に格納せずにプロパティ・メソッドにアクセスするには `()` を使う

```powershell
(Get-Item -Path "C:\Windows").FullName
```

---

## 関数の使用について（Import-Module）

- メモリへ関数の定義をインポートしないと、関数は使えない

```powershell
Import-Module .\powerview.ps1
```

---

## コマンドやヘルプの出力

### Get-Help

- コマンドレットに関する情報を表示する

```powershell
Get-Help <command>

# 特定のパラメータについて知りたい
Get-Help <command> -Parameter <param>

# コマンド例を知りたい
Get-Help <command> -Examples

# コマンドについての詳細な説明を別ウィンドウで表示
Get-Help <command> -ShowWindow
```

### Get-Command

- 現在のコンピューターにインストールされているすべてのコマンドレットを表示
- パターンマッチが可能：`Get-Command Verb-*` | `Get-Command *-Noun`

```powershell
# 例
Get-Command New-*
```

### エイリアスの設定

```powershell
New-Alias -Name <エイリアス> -Value <command>
```

---

## ログ・出力記録

### Start-Transcript

コマンドとその実行内容をテキストファイルにダンプしたいときに使う。列挙過程で実行した内容をメモするときに便利。
> [!NOTE]
> - ファイルは追記される
> - WinRM接続状態では内容が二重で出力されてしまうため、`Tee-Object` を使う

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

### Tee-Object

ファイルに出力を記入しつつ、ターミナルに出力結果を表示する

```powershell
.\winPEASx64.exe | Tee-Object -FilePath winpeas_log.txt
```

---

## 関数の定義

- 変数名には型を指定することも、しないこともできる

### 名前付きパラメータを定義するパターン

```powershell
function Get-NameValue {
    [CmdletBinding()]
    param(
        [string]$Name = "",
        $Value
    )
    return "We've got $NAME with value $Value"
}
```

- `[CmdletBinding()]` とすることで、Advanced Functionになる
- **Advanced Function** とは：
    - 共通パラメーターが使えるようになる（`-Verbose` や `-ErrorAction` など）
        - 例: `MyFunction -Verbose` で詳細な出力を表示
    - エラーハンドリングが強化される
        - `$PSCmdlet.ThrowTerminatingError()` などの高度なエラー処理が可能になる
    - `Process` ブロックを使うことで、パイプライン入力を処理しやすくなる

### 名前付きパラメータを定義しないパターン

- `$args` で引数を受け取る
```powershell
function Get-Sum {
    $A = $args[0]
    $B = $args[1]

    $SUM = $A + $B
    Write-Output "SUM: $SUM"
}
```

### スクリプトブロック（関数定義をしないで小さな処理をするパターン）

```powershell
Invoke-Command -ScriptBlock {Write-Output "hoge"}
```

---

## オブジェクトの操作

- すべてのコマンドレットの出力はオブジェクト
- 他のシェルと比較して大きな違いは、パイプの後のコマンドにテキストや文字列を渡すのではなく、次のコマンドレットに「オブジェクト」を渡すこと
- オブジェクトにはメソッドとプロパティが含まれる
    - メソッド：コマンドレットからの出力に適用できる関数
    - プロパティ：コマンドレットからの出力に含まれる変数

```powershell
# メソッドやプロパティの詳細を表示
Verb-Noun | Get-Member -MemberType [Method | Property]
```

### 表示形式調整

```powershell
# 特定のプロパティを表示
xxxx | Select-Object Id, ProcessName

# テーブル形式で表示
xxxx | Format-Table

# リスト形式で表示
xxxx | Format-List

# すべてのプロパティをリスト形式で表示
xxxx | Format-List *

# グリッドビューGUI（Excel形式のようなもの）で表示（データの抽出と操作が可能）
xxxx | Select-Object * | Out-GridView

# オブジェクトで利用可能なプロパティ・メソッドの取得
xxxx | Get-Member -Type Property
xxxx | Get-Member -Type Method

# Linuxでいうmoreコマンド（表示数が多いとき利用）
xxxx | Out-Host -Paging
```

### オブジェクトの抽出（Select-Object）

- 直前のコマンドレットの出力からプロパティを取り出し、新しいオブジェクトを作成する

```powershell
# ディレクトリをリストアップして、モードと名前だけを選択する
Get-ChildItem | Select-Object -Property Mode, Name
```

![](../Images/Pasted%20image%2020230604094002.png)

```powershell
# 最初の3つのファイルの名前とサイズを取得する
Get-ChildItem | Select-Object -First 3 Name, Length
```

使用できるフラグ：
- `-First <x>`：最初のx個のオブジェクトを取得
- `-Last <x>`：最後のx個のオブジェクトを取得
- `-Unique`：一意のオブジェクトのみを表示
- `-Skip <x>`：x個のオブジェクトをスキップ

💡 cmdlet以外でも特定の列のみを表示する方法
```powershell
<cmd> /fo csv | ConvertFrom-Csv | Select-Object <Property>
```

![](../Images/Pasted%20image%2020260107140211.png)

### オブジェクトのフィルタリング（Where-Object）

- プロパティの値でフィルタすることができ、bashの `grep` のように使える

```powershell
# PowerShell 3.0以降
Where-Object -Property <プロパティ名> -<operator> <value>

# 3.0未満
Where-Object {$_.<プロパティ名> -<operator> <value>}
```

#### Where-Objectで用いられる演算子

`Get-Help Where-Object -ShowWindow` でも表示可能。

| 演算子       | 概要                            | 例                          |
| --------- | ----------------------------- | -------------------------- |
| -EQ       | 値が等しい                         | ProcessName -EQ "explorer" |
| -NE       | 値が等しくない                       | ProcessName -NE "explorer" |
| -Match    | 文字列が正規表現に一致する                 | ProcessName -Match "ex.*"  |
| -NotMatch | Match演算子の逆                    |                            |
| -Like     | 文字列がワイルドカード表現に一致する            | ProcessName -Like "ex*"    |
| -NotLike  | Like演算子の逆                     |                            |
| -ilike    | 大文字小文字を問わずワイルドカード表現に一致する      |                            |
| -GT       | より大きい                         | ProcessName -GT "ex"       |
| -LT       | より小さい                         |                            |
| -Contains | プロパティ値のいずれかの項目が指定された値と完全一致    |                            |
| -in       | 指定されたリストに含まれているか              |                            |
| -notin    | 指定されたリストを含まない                 |                            |
| -le       | less than or equalで、「以下」を意味する |                            |

```powershell
# 停止したプロセスを表示する
Get-Service | Where-Object -Property Status -EQ Stopped

# 一致する名前を検索
dir | Where-Object { $_.Name -match '<string>' }

# 結果から除外
ps | Where-Object -Property ProcessName -notin "svchost","sshd"
```

### オブジェクトのソート（Sort-Object）

```powershell
Get-ChildItem | Sort-Object

# 降順
xxxx | Sort-Object <column> -Descending

# ユニーク
xxxx | Sort-Object <column> -Unique
```

---

## 主要コマンドレット

### Select-String

 ファイルの"中身"を検索する（grepのような使い方）
```powershell
# cmdの場合はfind
tasklist | Select-String "sync"

type <file> | Select-String -Pattern "pass(word)?|pwd|cred(ential)?"
```
- ※ ファイル名を検索したい場合はWhere-Objectを使う

### Job（バックグラウンド処理）

```powershell
# jobの開始（パスは絶対パス）
Start-Job {
    C:\Users\hoge\Snaffler.exe -s
}

# job一覧確認
Get-Job

# jobの実行状況確認
Receive-Job -id <job_id>
```
- `Receive-Job` に `-Wait` を追加すると、処理が終わるまでそこで待機
- 参考：[[PowerShell] 処理をバックグラウンドで実行する (Start-Job) - 晴耕雨読](https://tex2e.github.io/blog/powershell/background-job)

---

## 具体例

### インストールされたCmdletのトータルの数を表示する

```powershell
# 1. 使用できるPropertyを一覧表示する
Get-Command | Get-Member -MemberType Property

# 2. CommandTypeのValueの種類を一覧表示する
Get-Command | Select-Object -ExpandProperty CommandType -Unique

# 3. カウントする（measureで最小値・最大値・平均値・トータルの数などが表示される）
Get-Command | Where-Object -Property CommandType -eq Cmdlet | measure
```

### ファイルの検索と内容表示

```powershell
# 特定のファイルを検索する
Get-ChildItem -Path C:\ -Include *<FileName>* -File -Recurse -ErrorAction SilentlyContinue
# -ErrorAction SilentlyContinue: アクセス権のないディレクトリにアクセスしたときもエラーメッセージを表示せず続行する

# ファイルの内容を表示する
Get-Content "<path_to_file>"
```

### ファイルのハッシュを閲覧する

```powershell
# 1. コマンドレットを検索する
Get-Command | Where-Object -Property Name -ilike "*<keyword>*"

# 2. MD5ハッシュを表示する
Get-FileHash -Path "<path_to_file>" -Algorithm MD5
```

### パスの確認

```powershell
Get-Location
```

### ファイルをデコードする

```powershell
certutil -decode "<path_to_encoded file>" <output_file>
```

---

## Basic Scripting

- スクリプトを自分で書くと、より複雑で強力な動作ができるようになる
- NmapやPythonが使えない時の代替手段となる
- PowerShell ISE（PowerShellテキストエディター）を使用する
- PowerShellスクリプトは通常 `.ps1` というファイル拡張子を持つ
- スクリプトを実行するには、PowerShellからファイルを実行するか、ISEの緑のボタンを押して実行する

![ 400](../Images/Pasted%20image%2020230604162232.png)

### ルールや記述方法

```powershell
# foreach文
foreach($new_var in $existing_var){}

# if文（-inはリストに含まれているか検証する比較演算子）
if($port -in $system_ports.LocalPort){}
```

- [比較演算子の詳細](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-7.3&viewFallbackFrom=powershell-6)

### 例：ポートのリスニング確認スクリプト

```powershell
$system_ports = Get-NetTCPConnection -State Listen

$text_port = Get-Content -Path C:\Users\Administrator\Desktop\ports.txt

foreach($port in $text_port){
    if($port -in $system_ports.LocalPort){
        echo $port
    }
}
```

---

## Intermediate Scripting（Nmapの代替）

手順：
- スキャンするIP範囲を決定する
- スキャンするポート範囲を決定する
- 実行するスキャンの種類を決定する

```powershell
# 通常のTCP Connectionでlocalhostのポートをスキャンする
for($i=<start num>; $i -le <end num>; $i++){
    Test-NetConnection <IPAddress> -Port $i
}
```