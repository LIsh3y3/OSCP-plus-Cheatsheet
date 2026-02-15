>TFTP（Trivial File Transfer Protocol）は、最もシンプルなファイル転送プロトコルの一つです。UDPポート69番で動作し、ユーザー認証や暗号化を必要とせずにファイル転送が可能です。TFTPのシンプルさは、VoIP端末などのデバイスへの設定ファイルやROMイメージの展開といった内部ネットワーク操作に有効ですが、同時に深刻なセキュリティリスクも招きます。

# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
69/udp    open          tftp           Netkit tftpd or atftpd
```

---

# Enumeration

- ファイルがいくつか見つかったので、tftpでアクセスし、ファイルをダウンロードした
```sh
$ nmap -n -Pn -sU -p69 -sV --script tftp-enum $TargetIP
PORT STATE SERVICE VERSION 
69/udp open tftp Netkit tftpd or atftpd 
 tftp-enum 
   backup.cfg
   sip-confg
   sip.cfg
   sip_327.cfg
```

- backup.cfg
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM14]
└─$ cat backup.cfg                         
FTP credentials for umbraco web application upgrade:

ftp_jp
~be<3@6fe1Z:2e8
```

- sip-confg
```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM14]
└─$ cat sip-confg 
# Properties common for all protocols
net.java.sip.communicator.service.protocol.DTMF_METHOD=AUTO_DTMF
net.java.sip.communicator.service.protocol.DTMF_MINIMAL_TONE_DURATION=70
# SIP specific properties
net.java.sip.communicator.service.protocol.sip.SERVER_PORT=5060
...
```


---

# Screenshot


