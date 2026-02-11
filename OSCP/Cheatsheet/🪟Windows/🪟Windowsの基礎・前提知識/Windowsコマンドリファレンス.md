
# ユーザー操作

ユーザーのパスワードを変更
```powershell
# ADの場合は末尾に/domainを追加
net user <username> Password123!
```
- パスワードポリシーを満たせるように、大文字、小文字、数字、記号を入れること

ユーザーをグループに追加
```powershell
# ADの場合は末尾に/domainを追加
net localgroup <group> <username> /add
```

# Webページ操作

現在ログオンしているユーザーの認証情報をつかってリクエスト
```powershell
Invoke-WebRequest -UseDefaultCredentials <url>
```

レスポンスの中身を閲覧
```powershell
(Invoke-WebRequest <url>).Content
```
