
- Scannerでは検知できないことが多い。
- 以下の脆弱性を直接悪用することはなくても考え方は大事(だと思う)

##### 攻撃に使いたい異常値

- 数値
    - 負値
    - 境界値
    - 異常にでかい、異常に小さい
- 文字列
    - 異常に長い
    - `%00` (null文字) 含む

---
### 異常値をうまく処理できない欠陥を悪用する

###### オーバーフロー(商品購入)

- [[2. -1 事例：Business logic vuln#整数型のオーバーフローを起こす]]


---
### 意図されたワークフローを無視する

- [Lab: Insufficient workflow validation](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-insufficient-workflow-validation)

---
## Misc :

###### クライアント側の制御に対する過度の信頼　- APPERENTICE

- [[3. 多要素認証の脆弱性#ログイン状態のみアクセスできるページへの直接アクセス]]

###### パスワードリセット

- [[⚡️Password reset#Forgot passwordによるパスワードリセット]]