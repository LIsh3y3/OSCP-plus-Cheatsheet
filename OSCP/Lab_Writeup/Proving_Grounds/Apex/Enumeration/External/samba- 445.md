# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
```

---

# Enumeration

## 総合列挙

### enum4linux

```sh
================================( Share Enumeration on 192.168.227.145 )================================

do_connect: Connection to 192.168.227.145 failed (Error NT_STATUS_IO_TIMEOUT)

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        docs            Disk      Documents
        IPC$            IPC       IPC Service (APEX server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

```

### smbmap

```sh
# 基本列挙（Null Session / ゲスト確認）
smbmap -H $TargetIP

[+] IP: 192.168.227.145:445     Name: 192.168.227.145           Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        docs                                                    READ ONLY       Documents
        IPC$                                                    NO ACCESS       IPC Service (APEX server (Samba, Ubuntu))
[*] Closed 1 connections    
```

### docs共有へのアクセス -> nothing

```sh
mkdir mount_point
sudo mount -t cifs //$TargetIP/docs mount_point/
cd mount_point
```

- webサイトからもアクセスできる情報
```sh
┌──(koshi㉿kali)-[~/ProvingGrounds/Apex/mount_point]
└─$ ls
'OpenEMR Features.pdf'  'OpenEMR Success Stories.pdf'
```

---

# Screenshot


