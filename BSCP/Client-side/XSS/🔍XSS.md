[[⚡️XSS]]

🚨必ず一読：[[🔍 Recon#⭐️URLエンコードについて]]
# 注目ポイント

- Post Comment機能
- Search機能
- 怪しげなJavaScriptファイル
- Check Stock機能
- Cookieの値が`username: token`の形式
- Remember me機能：Stay-logged-in Cookie
- (LiveChat機能): [[2. WebSocketの脆弱性と悪用#脆弱性を悪用するためのWebSocketメッセージの操作]]

---
# Detect

## XSStrike

XSStrikeはxssに==脆弱でないものでも検知してしまう==かも。certainに気をつけて
- まずはXSStrikeでScanする。どの種類のXSS(DOM等)でもすぐ脆弱性が見つかる可能性が高い
```
cd /Users/ikedakoshiro/forBurpSuite/XSStrike
```

GETの場合
```zsh
python3 xsstrike.py -u "[URL]"
```
POSTの場合
```
python3 xsstrike.py -u "[URL]" --data "[body部のパラメタと値]"
```
HTTP headerも追加する場合
```
--headers
```

検知できなかった場合は↓へ

## Scanner

### 反射型

#### Search機能

- パラメタの値をScan
![[Pasted image 20240208105323.png | 400]]

#### JSファイル

- もしWeb cache poisoningの脆弱性も併せて検出したら、[[🔍Web cache poisoning]]へ

### 持続型

#### Comment機能

- Comment、Websiteのパラメタ値をScan 
	- OAST：[[⚡️XSS#Comment機能の場合]]
	- 通常↓
![[Pasted image 20240208110822.png | 400]]

##### Cookieが`username:token`の場合

- `username`をScan selected insertion point ->[[⚡️XSS#Cookieがusername tokenの場合]]

### DOM-based XSS

- DOM XSSはScannerじゃ見つからない可能性が高いので[[🔍DOM-based Vuln#Detect]]へ

---

# Fuzz: XSS

- DevToolsのConsoleタブを開いてエラーなど観測してcontextを調整しながら:
```html
<script>alert(1)</script>
```
	or
```html
<img src=1 onerror="alert(1)">
```
	or
```html
<script src="data:,alert(1);//"></script>
```

## 補足：XSStrikeはFuzzのみに利用

- Intruderで選択してScanより速い(XSSに絞ってるから）
- SQLmapと同じでパラメタが必要。cookieも可
- 一瞬でなんの種類か検知する
![[Pasted image 20240218175137.png | 200]]
- ただペイロードが意味わからん
- 例えば以下だとすると
```
/search-results?search=hoge
```
	 ↓
```
{"results":[],"searchTerm":"hoge"}
```

- 検出するペイロードは
```
<DEtails%09ONPOINtErentEr+=+a=prompt,a()%0dx> 
```
	↓
```
<DEtails%09ONPOINtErentEr+=+a=prompt,a()%0dx> 
```
これじゃ悪用できない

結論：Detectには向くけど悪用には期待しない

---


