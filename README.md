![](OSCP/Images/Pasted%20image%2020260326124528.png)

# OSCP+チートシート

 本リポジトリは、OSCP+（PEN-200）学習を軸に、ペネトレーションテストの 技術・手法をまとめた**日本語チートシート・学習ノート集**です。 試験対策だけでなく、実際のペネトレーションテスト業務でも 参照できるよう、コマンドや手順を体系的に整理しています。

---

# 対象読者

- OSCP+・OSEP取得を目指している方
- ペネトレーションテストの技術を体系的に学びたい方
- 日本語でオフェンシブセキュリティを学びたい方

---

# 目次

## Cheatsheet

### 🐧Linux

- [🔍Linux Enumeration](OSCP/Cheatsheet/🐧Linux/🔍Linux%20Enumeration.md)
- [💥Linux Privilege Escalation](OSCP/Cheatsheet/🐧Linux/💥Linux%20Privilege%20Escalation.md)

### 🪟Windows

- [🔍Windows Local Enumeration](OSCP/Cheatsheet/🪟Windows/🔍Windows%20Local%20Enumeration.md)
- [💥Windows Privilege Escalation](OSCP/Cheatsheet/🪟Windows/💥Windows%20Privilege%20Escalation.md)

#### Active Directory

- [ADの基本](OSCP/Cheatsheet/🪟Windows/Active%20Directory/ADの基本.md)
- [🔍AD Enumeration](OSCP/Cheatsheet/🪟Windows/Active%20Directory/🔍AD%20Enumeration.md)
- [🔍 Credentials Harvesting](OSCP/Cheatsheet/🪟Windows/Active%20Directory/🔍%20Credentials%20Harvesting.md)
- [💥AD認証システムの攻撃](OSCP/Cheatsheet/🪟Windows/Active%20Directory/💥AD認証システムの攻撃.md)
- [💥Lateral Movement & Persistance in AD](OSCP/Cheatsheet/🪟Windows/Active%20Directory/💥Lateral%20Movement%20&%20Persistance%20in%20AD.md)
- [💥AD Exploit](OSCP/Cheatsheet/🪟Windows/Active%20Directory/💥AD%20Exploit.md)

#### Windowsの基礎・前提知識

- [PrivEsc前提知識：Windowsの特権とAccess Controlの仕組み](OSCP/Cheatsheet/🪟Windows/Windowsの基礎・前提知識/PrivEsc前提知識：Windowsの特権とAccess%20Controlの仕組み.md)
- [Windows Internal](OSCP/Cheatsheet/🪟Windows/Windowsの基礎・前提知識/Windows%20Internal.md)

### Common

- [Bind & Reverse Shell・ペイロード・安定化手法](OSCP/Cheatsheet/Common/Bind%20&%20Reverse%20Shell・ペイロード・安定化手法.md)
- [Git Enumeration](OSCP/Cheatsheet/Common/Git%20Enumeration.md)
- [OSINT](OSCP/Cheatsheet/Common/OSINT.md)
- [Password Attack](OSCP/Cheatsheet/Common/Password%20Attack.md)
- [Port Scan & Vuln Scan](OSCP/Cheatsheet/Common/Port%20Scan%20&%20Vuln%20Scan.md)
- [スクリプト・コマンド・シェル操作](OSCP/Cheatsheet/Common/スクリプト・コマンド・シェル操作.md)
- [ファイル操作、ユーティリティ](OSCP/Cheatsheet/Common/ファイル操作、ユーティリティ.md)
- [権限関連の知識、コマンド](OSCP/Cheatsheet/Common/権限関連の知識、コマンド.md)
- [公開エクスプロイトの探索](OSCP/Cheatsheet/Common/公開エクスプロイトの探索.md)
- [公開エクスプロイトの修正](OSCP/Cheatsheet/Common/公開エクスプロイトの修正.md)

### Port Redirection

- [HTTP・DNSトンネリング（Chisel, Ligolo-ng, Dnscat2）](OSCP/Cheatsheet/Port%20Redirection/HTTP・DNSトンネリング（Chisel,%20Ligolo-ng,%20Dnscat2）.md)
- [Port Redirection & SSH Port Forwarding](OSCP/Cheatsheet/Port%20Redirection/Port%20Redirection%20&%20SSH%20Port%20Forwarding.md)

### Web App

- [Burp Suiteの基本操作・トラブルシューティング](OSCP/Cheatsheet/Web%20App/Burp%20Suiteの基本操作・トラブルシューティング.md)
- [ブラウザ開発者ツール](OSCP/Cheatsheet/Web%20App/ブラウザ開発者ツール.md)
- [API testing](OSCP/Cheatsheet/Web%20App/API%20testing.md)
- [File Inclusion](OSCP/Cheatsheet/Web%20App/File%20Inclusion.md)
- [File upload vulnerability](OSCP/Cheatsheet/Web%20App/File%20upload%20vulnerability.md)
- [Path traversal](OSCP/Cheatsheet/Web%20App/Path%20traversal.md)
- [SQL Injection](OSCP/Cheatsheet/Web%20App/SQL%20Injection.md)
- [WebShell](OSCP/Cheatsheet/Web%20App/WebShell.md)
- [XSS](OSCP/Cheatsheet/Web%20App/XSS.md)

### Evasion(OSCP+試験範囲外)

- [AV Evasion：概念と理論](OSCP/Cheatsheet/Evasion(OSCP+試験範囲外)/AV%20Evasion：概念と理論.md)
- [AV Evasionのテクニック](OSCP/Cheatsheet/Evasion(OSCP+試験範囲外)/AV%20Evasionのテクニック.md)
- [PE構造とシェルコード](OSCP/Cheatsheet/Evasion(OSCP+試験範囲外)/PE構造とシェルコード.md)
- [Web攻撃の難読化](OSCP/Cheatsheet/Evasion(OSCP+試験範囲外)/Web攻撃の難読化.md)

## Misc

- [Cypher文法](OSCP/Misc/Cypher文法.md)
- [C言語](OSCP/Misc/C言語.md)
- [Python文法](OSCP/Misc/Python文法.md)
- [SQL基本文法](OSCP/Misc/SQL基本文法.md)
- [コンパイル・ビルド](OSCP/Misc/コンパイル・ビルド.md)
- [コンピュータ・アセンブラ・バイナリの基礎知識](OSCP/Misc/コンピュータ・アセンブラ・バイナリの基礎知識.md)
- [Living off the Land](OSCP/Misc/Living%20off%20the%20Land.md)
- [JENKINS](OSCP/Misc/JENKINS.md)
- [Nmap Live Host Discovery](OSCP/Misc/Nmap%20Live%20Host%20Discovery.md)
- [Normal Informations](OSCP/Misc/Normal%20Informations.md)
- [仮想マシンの設定、便利機能](OSCP/Misc/仮想マシンの設定、便利機能.md)
- [用語](OSCP/Misc/用語.md)

### Client-side Attacks

- [Client-Side Attacks：概要とFingerprinting](OSCP/Misc/Client-side%20Attacks/Client-Side%20Attacks：概要とFingerprinting.md)
- [Microsoft Officeのエクスプロイト（マクロ）](OSCP/Misc/Client-side%20Attacks/Microsoft%20Officeのエクスプロイト（マクロ）.md)
- [Phishing](OSCP/Misc/Client-side%20Attacks/Phishing.md)
- [Windows Library Filesの悪用](OSCP/Misc/Client-side%20Attacks/Windows%20Library%20Filesの悪用.md)

## Tools

- [☠️Msfvenom](OSCP/Tools/☠️Msfvenom.md)
- [🏋️PowerShell](OSCP/Tools/🏋️PowerShell.md)
- [🐈‍⬛Password Crack - JtR・Hashcat](OSCP/Tools/🐈‍⬛Password%20Crack%20-%20JtR・Hashcat.md)
- [🐉THC-Hydra](OSCP/Tools/🐉THC-Hydra.md)
- [🐙FFUF](OSCP/Tools/🐙FFUF.md)
- [🐝RustScan](OSCP/Tools/🐝RustScan.md)
- [🐶Bloodhound](OSCP/Tools/🐶Bloodhound.md)
- [👻Gobuster](OSCP/Tools/👻Gobuster.md)
- [👽Nikto](OSCP/Tools/👽Nikto.md)
- [📓WPScan](OSCP/Tools/📓WPScan.md)
- [🥝Mimikatz](OSCP/Tools/🥝Mimikatz.md)
- [🦈Wireshark](OSCP/Tools/🦈Wireshark.md)
- [🦝FeroxBuster](OSCP/Tools/🦝FeroxBuster.md)
- [🧰Immunity debugger](OSCP/Tools/🧰Immunity%20debugger.md)
- [🛠️Windows Sysintarnals](OSCP/Tools/🛠️Windows%20Sysintarnals.md)

---

# 免責事項

本ノートは学習・研究目的のみを対象としています。
許可のないシステムへの攻撃は違法です。

>[!Important]
>**自動エクスプロイトツールの使用は試験で禁止されています。**  
>例：`sqlmap` の自動エクスプロイト機能など。
>本リポジトリに記載のツール・手法を試験で使用する場合は、必ず事前に最新の試験ガイドを確認してください。
>- [OSCP Exam Guide](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide)
>- [Proctored Exams](https://help.offsec.com/hc/en-us/sections/360008126631-Proctored-Exams)
