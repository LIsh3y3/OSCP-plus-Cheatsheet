#### 注目ポイント

- Change password
- Forgot password

###### Change passwordによるパスワードリセット(APPRENTICE)

- tokenパラメタの値を削除する
- [[4. その他認証メカニズムによる脆弱性#パスワードをリセットする]]

###### Forgot passwordによるパスワードリセット

- `current-password`(現在のパスワード)のパラメタと値を削除する
- 具体例：[Github cheat sheet: Current Password](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/tree/main?tab=readme-ov-file#current-password)
- [[⚡️Business logic vuln]]の一種

#### HHIによるパスワードリセット

- [[⚡️HTTP Host header attack#Password reset poisoning]]