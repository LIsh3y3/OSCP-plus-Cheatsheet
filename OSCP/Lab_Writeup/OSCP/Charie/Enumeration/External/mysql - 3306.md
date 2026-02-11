# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router|firewall
```

---

# Enumeration


- リモート接続試行（パスワードありなしにかかわらず、ログインできないよう）
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ mysql -u root -h 192.168.115.112 -p
Enter password: 
ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host '192.168.49.115' is not allowed to connect to this MariaDB server
                                                                                      
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ mysql -u root -h 192.168.115.112 -p 
Enter password: 
ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host '192.168.49.115' is not allowed to connect to this MariaDB server

```


---

# Screenshot


