1. ポートスキャンをしたところ、http 80と3000、samba 445、freeswitch-event 8021がopenであることを確認

2. sambaをsmbmapのguest認証で列挙したところ、`backup`というsmb shareがあり、backupの中に"freeswitch"と"cassandra"を発見

3. smbをマウントした上で、freeswitchのパスワードが保存されているという以下のファイルにアクセスし、パスワードがデフォルトの`ClueCon`であることを確認
```zsh
mkdir mount_point
sudo mount -t cifs //192.168.1.1/backup mount_point/ -o guest
cd mount_point

# freeswitchのパスワード保存フォルダ
/etc/freeswitch/autoload_configs/event_socket.conf.xml
```

4. [pentesting freeswitch]( https://x7331.gitbook.io/boxes/services/tcp/8021-freeswitch)を確認し、Exploit-DBのPoC[47799](https://www.exploit-db.com/exploits/47799)を使用してRCEを実行しようとするtも、Authentication Failedとなるため、`backup` shareに保存されている`ClueCon`は古いパスワードであると推測

5. http 3000で動作していたCassandraという、Facebookが開発したオープンソースの分散型NoSQLデータベースには、リモートファイル読み取りの脆弱性があり、Exploit-DBのPoC[49362](https://www.exploit-db.com/exploits/49362)を用いてファイルを読み取ることができる（影響バージョンは0.5だが、バージョン情報を見る術がないため試行）

6. 同PoCに記載のパスを読み取ると、Cassandra db用のパスワードを入手
```zsh
python 49362.py 192.168.224.240 /proc/self/cmdline

/usr/bin/ruby2.5/usr/local/bin/cassandra-web-ucassie-pSecondBiteTheApple330
```

7. cassandra db用のパスワードで`cassie`にssh接続を試行するも失敗

8. `/etc/passwd`をcassandraのfile readを使って読み取る限り、ユーザー`cassie`がシェルを持つこと、Cassandra db用のパスワードにはユーザー`cassie`の名前があることから、`cassie`のSSHキーを読み取ろうとするも失敗

9. freeswitchのRCEを実行するために、現状のパスワードをcassandraの脆弱性を使って読み取ると、`ClueCon`ではないパスワードが使われていたため、[47799](https://www.exploit-db.com/exploits/47799)のパスワードを変更した上で実行し、シェルを獲得
	- ポートを試行錯誤したが、80しか通らなかった
	- **tips**：80以外のすべてのポートがFWで遮断されているときでも、`base64`を使えばファイルは転送できる（[ファイル操作、ユーティリティ](../Cheatsheet/Common/ファイル操作、ユーティリティ.md#base64)）
	- **反省**：最新のpspyを動作させたところ、GLIBC xxx not foundと表示され諦めたが、これはそのGLIBCを使っているバージョンを使えばいいだけのこと（[コンパイル・ビルド](../Misc/コンパイル・ビルド.md#GLIBC_xxx%20not%20found)）

10. freeswitchユーザーで列挙すると、cassieのid_rsaが`/home/cassie/id_rsa`にあることを確認したので、Cassandraの脆弱性で読み取り、cassieとしてアクセスするも、失敗した

11. `/etc/ssh/sshd.config`を確認すると、`Allow User root anthony`となっており、cassieはsshでログインできないことがわかり路頭に迷う

12. cassandra db用のパスワードで`su cassie`としたところ、成功し、`sudo -l`としたところ、`cassandra-web`をNO PASSWORDで実行できることを確認
	- **反省**：sshで使えないパスワードが、完全にcassandra用のパスワードと思い込んでおり、`su`で使えることを失念

13. Cassandra-webをrootで起動できれば、Exploit-DBのPoC[49362](https://www.exploit-db.com/exploits/49362)でroot権限でファイルを読み取れると考えた
```zsh
# 起動中となり、操作を受け付けなくなるので、&でバックグラウンド実行
sudo cassandra-web -u cassie -p SecondBiteTheApple330 -B 0.0.0.0:5555 &
```

14. rootで起動したcassandra-webにリモートからncで接続できなかっため、PoCの中身を読み、ディレクトリトラバーサルであることを確認の上、内部からアクセス
```zsh
curl --path-as-is http://0.0.0.0:5555/../../../../../../../../../home/anthony/.ssh/id_rsa
```

15. anthonyのid_rsaを使ってログインできなかったため、anthonyのbash_historyを見ると、rootにもコピーしていることを確認
```zsh
curl 0.0.0.0:5555/../../../../../../../../home/anthony/.bash_history --path-as-is

clear
ls -la
ssh-keygen
cp .ssh/id_rsa.pub .ssh/authorized_keys
sudo cp .ssh/id_rsa.pub /root/.ssh/authorized_keys
exit
```
- ==ユーザーは面倒になりrootのパスワードや鍵を使い回すことがある==

16. anthonyの秘密鍵を使ってroot としてssh接続成功