他ユーザーの認証情報など、横展開につかる情報を探す
- →なさそうなので、次のマシンの列挙に移行する

# Shadow

```
root@milan:/etc# scp shadow koshi@192.168.45.249:/home/koshi/PEN-200/Skylark/VM15
root@milan:/etc# scp passwd koshi@192.168.45.249:/home/koshi/PEN-200/Skylark/VM15
```

```sh
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15/Unshadows]
└─$ ls
passwd_milan  passwd_root_VM15  passwd_sarah  shadow  shadow_milan  shadow_root_VM15  shadow_sarah

┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15/Unshadows]
└─$ unshadow passwd_milan shadow_milan > milan_unshadow.txt
    
┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15/Unshadows]
└─$ unshadow passwd_sarah shadow_sarah > sarah_unshadow.txt

┌──(koshi㉿kali)-[~/PEN-200/Skylark/VM15/Unshadows]
└─$ unshadow passwd_root_VM15 shadow_root_VM15 > root_VM15_unshadow.txt
```

- 3分まって解析できなかったら、次の解析をし、すべて無理だったらこれはベクターではない→無理だったので、VM18の列挙に移行

---


# Manual

## ユーザー


---

## システム情報


---

## 実行中のプロセス


---

## ネットワーク


---

## Cron


---

## インストール済みアプリケーション


---

## アクセス制限が不十分なファイル/ディレクトリ


---

## マウントされていないドライブ


---

## 高権限をもつバイナリの列挙



---

## システムデーモン



---

## 共有フォルダ、SNMP情報


---

## 興味深い情報

- ssh秘密鍵はなさそう
```sh
root@milan:/etc/ssh# find / -name .ssh -type d 2>/dev/null
/root/.ssh
/home/milan/.ssh
root@milan:/etc/ssh# ls -la /root/.ssh
total 8
drwx------  2 root root 4096 Nov 23  2022 .
drwx------ 16 root root 4096 Feb 15 10:01 ..
root@milan:/etc/ssh# ls -la /home/milan/.ssh
total 8
drwx------  2 milan milan04 4096 Nov 14  2022 .
drwxr-xr-x 15 milan milan04 4096 Jan 24  2023 ..
```


---

## デバイスドライバ・カーネルモジュール



