>TFTP（Trivial File Transfer Protocol）は、最もシンプルなファイル転送プロトコルの一つです。UDPポート69番で動作し、ユーザー認証や暗号化を必要とせずにファイル転送が可能です。TFTPのシンプルさは、VoIP端末などのデバイスへの設定ファイルやROMイメージの展開といった内部ネットワーク操作に有効ですが、同時に深刻なセキュリティリスクも招きます。

# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
69/udp    open          tftp           Netkit tftpd or atftpd
```

---

# Enumeration

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

---

# Screenshot


