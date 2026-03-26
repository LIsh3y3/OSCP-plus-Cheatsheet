- 特権とは、システムの操作を実行するための特定アカウントの権限を指す
- 操作のための特権が十分かどうかを判断するため、以下の４つの仕組みがある
	- SID
	- Access token
	- Mandatory Integrity Control
	- User Account Control(UAC)

# SIDについて

- 用語：[SID・RID](../../../Misc/用語.md#SID・RID)
- 🔗[well-known SID](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids)があり、その中でもPrivEscの文脈で有用なものがある

| SID                          | 意味                  |
| ---------------------------- | ------------------- |
| S-1-0-0                      | Nobody              |
| S-1-1-0                      | Everybody           |
| S-1-5-11                     | Authenticated Users |
| S-1-5-18                     | Local System        |
| S-1-5-{domainidentifier}-500 | Administrator       |

---

# Access token

- [プロセス・スレッド・プロシージャ](../../../Misc/用語.md#プロセス・スレッド・プロシージャ)

## Access Token の中身（セキュリティコンテキスト）

LSASSが発行し、以下の情報が含まれる：
- ユーザーの SID（誰か）
- 所属しているグループの SID（どこに属してるか）
- 各ユーザーやグループの「権限（privileges）」
- その他の属性（例：どこまでアクセスできるか、制限付きトークンかなど）

## Primary Token

- **プロセス**に付与されるトークン
- 「このプロセスは誰の代わりに動いているか？」を決める
	- 例：ユーザーがアプリを起動 → そのプロセスに「そのユーザーのトークン」が付く
- OS はこのトークンを見て、ファイルやレジストリへのアクセス許可を判断

## Impersonation Token（偽装）

- **スレッド**に一時的に割り当て可能
- プロセスとは「別の身分」でそのスレッドだけを動かせる
- 用途は、サーバーがユーザーの代わりにユーザーの権限でファイル操作を行う場合などに使う


PrimaryとImpersonationの違い：

| 項目   | Primary Token | Impersonation Token |
| ---- | ------------- | ------------------- |
| 対象   | プロセス全体        | スレッド単体              |
| 主な用途 | 通常の実行         | 他人になりすます（偽装）        |
| 継承   | 子スレッドに継承される   | 他スレッドには影響なし         |

---

# Mandatory Integrity Control

- Windowsにおけるセキュリティ強化の一環で、オブジェクトへのアクセス制御を**完全性レベル（Integrity Level）**　によって管理する仕組み。
- 任意アクセス制御リスト（DACL）とは別に適用され、アクセス要求の評価時にDACLよりもまず最初にMICの完全性チェックが行われる。
- 詳細：🔗[Mandatory Integrity Control mechanism - Microsoft](https://learn.microsoft.com/en-us/windows/win32/secauthz/mandatory-integrity-control)

## 完全性レベル（Integrity Level）

- 完全性レベルは、プロセスやオブジェクトの「信頼度」を示す。低い完全性レベルのプロセスは、高い完全性レベルのオブジェクトを**変更できない**（たとえDACLで許可されていても）。
    - オブジェクト：ファイル、ディレクトリ等の操作対象のこと
- Windowsでは以下の5段階が定義されている：

| レベル       | 説明例                           |     |
| --------- | ----------------------------- | --- |
| System    | システムの重要プロセス（例：Winlogon、LSASS） |     |
| High      | 管理者権限で実行されるプロセス               |     |
| Medium    | 標準ユーザーが起動したプロセス（デフォルト）        |     |
| Low       | サンドボックスや制限付きプロセス（例：一部ブラウザ）    |     |
| Untrusted | 非常に制限されたプロセス（一般的には使用されない）     |     |

### 完全性レベルの確認手段

| 確認対象        | 使用ツール / コマンド                                                                                   | 内容                                           |
| ----------- | ---------------------------------------------------------------------------------------------- | -------------------------------------------- |
| プロセス        | 🔗[Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer) | 実行中プロセスの整合性レベルをGUI上で確認可能                     |
| ユーザー        | `whoami /groups`                                                                               | 出力中の「Mandatory Label」や SID でユーザーの整合性レベルを確認可能 |
| ファイル / フォルダ | `icacls <file>`                                                                                | ファイルやディレクトリの整合性ラベルを表示（例：Low/Medium）          |

## 継承の仕組み

- ユーザーが新たにプロセスやオブジェクトを作成すると、自身の完全性レベルを継承する。
- 実行ファイル(オブジェクト)に低い完全性レベルが設定されている場合、どんなユーザーがそのファイルを起動しても、プロセスは低完全性レベルで実行される。
    - これにより、悪意あるコードがユーザーデータへ書き込みを行うことを防止する。
- 新たなプロセスの完全性レベルは、ユーザーの完全性レベルと実行ファイルの完全性レベルとで、どちらか低い方に設定される。
    - 例：管理者がLowレベルの実行ファイルを起動 → プロセスはLowレベルで動作

---

# UAC

- 不正な、あるいは意図しないPrivEscを防ぐ目的
- ==Access token + MIC + UACでセキュリティを担保==
- アプリケーション・タスクを起動するユーザーがAdminであっても、Standard userの権限で実行する
- 仕組みとして、LSASSはユーザーがログオン時に２つの[Access token](#Access%20token)を発行する
    - Adminトークン：必要なときのみ使われる
    - Standard userトークン：通常の操作
- Administratorであっても、まずはStandard userトークン + Medium Integrityでプロセスを実行する
    
- UACプロンプトを通じてはじめてAdminトークン + High Integrityで実行される

![](../../../Images/Pasted%20image%2020250722124456.png) 

$$UACプロンプト$$
- 詳細🔗：[User Account Control overview - Microsoft](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/)
