## 型・変数・表現

###### 型

| powershell型 | .Net型                        | 備考       |
| ----------- | ---------------------------- | -------- |
| int         | System.Int32                 |          |
| long        | System.Int64                 |          |
| string      | System.String                | 文字列      |
| string      | System.Double                | 数値、浮動小数点 |
| bool        | System.boolean               |          |
| array       | System.Object\[]             |          |
| hashtable   | System.Collections.Hashtable |          |


###### コマンドの実行

- 動詞-名詞の組合せがコマンドの基本
- オブジェクトを変数に格納せずにプロパティ・メソッドにアクセスするには()を使う
```powershell
(Get-Item -Path "C:\Windows").FullName
```

---

## コマンドやヘルプの出力

- Get-Commandでコマンドの列挙
- Get-Helpでコマンドの使い方
```powershell
Get-Help [command]
#特定のパラメタについて知りたい
Get-Help [command] -Parameter [param]
#コマンド例を知りたい
Get-Help [command] -Example
#コマンドについての詳細な説明を別ウィンドウで表示
Get-Help [command] -ShowWindow
```

- エイリアスの設定
```powershell
New-Alias -Name [エイリアス] -Value [command]
```

---

## 関数の定義


- 変数名には[[#型]]を指定することも、しないこともできる

#### 名前付きパラメータを定義するパターン

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

- `[CmdletBinding()]`とすることで、Advanced Functionになる。
- *Advanced Function*とは：
	- 共通パラメーターが使えるようになる。
		- `-Verbose` や `-ErrorAction` などの **共通パラメーター** を自動的に使えるようになる。
		- 例: `MyFunction -Verbose` で詳細な出力を表示。
	
		- エラーハンドリングが強化される
			- `$PSCmdlet.ThrowTerminatingError()` などの **高度なエラー処理** が可能になる。
			- パイプライン入力が簡単になる

	- `Process` ブロックを使うことで、パイプライン入力を処理しやすくなる。

#### 名前付きパラメータを定義しないパターン

- argsで引数を受け取る
```powershell
function Get-Sum {
	$A = $args[0]
	$B = $args[1]

	$SUM = $A + $B
	Write-Output "SUM: $SUM"
}
```

#### 関数定義をしないで小さな処理をするパターン（スクリプトブロック）

```powershell
Invoke-Command -ScriptBlock {Write-Output "hoge"}
```

---

## オブジェクトの表示・抽出・整列

### 表示

- (オブジェクトとは、この文脈では、PowerShellの実行結果として出力された値)
	- オブジェクトなので、値1つでもプロパティやメソッドが使える。

###### 表示形式調整コマンド

- 特定のプロパティを表示(id, processname)
```powershell
xxxx | Select-Object Id, ProcessName
```
- 出力形式を変更
```powershell
#テーブル形式で表示
xxxx | Format-Table
#リスト形式で表示
xxxx | Format-List
#上記ですべて表示
xxxx | Format-List *
#グリッドビューGUI(Excel形式のようなもの)で表示（データの抽出と操作が可能）
xxx | Select-Object * | Out-GridView
```
- オブジェクトで利用可能なプロパティ・メソッドの取得
```powershell
xxxx | Get-Member -Type Property
xxxx | Get-Member -Type Method
```
- Linuxでいう`more`コマンド(表示数が多いとき利用)
```powershell
xxxx | Out-Host -Paging
```

### 抽出・整列

- 特定のプロパティのみを表示
```powershell
xxxx | Where-Object [Property] -EQ "[value]"
```
- オブジェクトの整列
```powershell
xxxx | Sort-Object [column]
# 降順：-Descending、ユニーク：-Uniqueを付与
```
- グループ化


###### Where-Objectコマンドで用いられる演算子

以下以外は、`Get-Help Where-Object -ShowWindow`で表示可能

| 演算子       | 概要                 | 例                          |
| --------- | ------------------ | -------------------------- |
| -EQ       | 値が等しい              | ProcessName -EQ "explorer" |
| -NE       | 値が等しくない            | ProcessName -NE "explorer" |
| -Match    | 文字列が正規表現に一致する      | ProcessName -Match "ex.\*" |
| -NotMatch | Match演算子の逆         |                            |
| -Like     | 文字列がワイルドカード表現に一致する | ProcessName -Like "ex*"    |
| -NotLike  | Like演算子の逆          |                            |
| -GT       | より大きい              | ProcessName -GT "ex"       |
| -LT       | より小さい              |                            |