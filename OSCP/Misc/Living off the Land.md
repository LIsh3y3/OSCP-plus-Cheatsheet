- [ ] [11. Living Off the Land](11.%20Living%20Off%20the%20Land.md)をまとめる

# Powershell Port Scan

シチュエーション：侵害した足場がwindowsマシンで、windowsマシンから内部ネットワークにある機器をスキャンする必要がある場合などに使用(Ligoloを使えば不要)

### Test-NetConnectionによるポートスキャン

- １つのみのポートのTCP接続確認
```powershell
Test-NetConnection -Port <Port> <TargetIP>
```
結果：TcpTestSucceeded : True

### Test-NetConnectionによるホストディスカバリ

特定サブネット(xxx.xxx.xxx.0/24)で特定ポートがopenなホストの探索
```powershell
$subnet = "[第三オクテットまでのIP]"
1..254 | ForEach-Object {
  $ip = "$subnet.$_"
  if (Test-NetConnection -ComputerName $ip -Port <Port> -InformationLevel Quiet) {
    "$ip`:<Port> open"
  }
}
```

### ポートスキャンの自動化（1～1024ポート）

- PowerShellワンライナーで、最初の1024ポートをスキャン可能
- Net.Sockets.TcpClientオブジェクトを作成し、対象IPの各ポートにTCP接続を試行
- 接続成功したポートのみログ表示
```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("<TargetIP>", $_)) "TCP port $_ is open"} 2>$null
```
（`%`は`ForEach-Objet`の言い換え。パイプ(|)と合わせて使用し、パイプの前の値が`$_`に格納されていく。openだったらechoされるし、closedだったらエラーとして`$null`に格納＝非表示）
