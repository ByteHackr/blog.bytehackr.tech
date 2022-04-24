# Binary

**Linux BOF :**

* check ASLR : cat /proc/sys/kernel/randomize\_va\_space
  * 0 - ASLR Disable
  * 1 - ASLR Enabled
* gdb checksec
* ldd \<binary>
* ltrace \<binary>
* Lib2retc attack - HTB Frolic
* [https://github.com/david942j/one\_gadget](https://github.com/david942j/one\_gadget) (One Gadget tool for finding RCE in libc)
* [https://snowscan.io/htb-writeup-frolic/](https://snowscan.io/htb-writeup-frolic/#)

**Buffer Overflow Practice :**

* SLmail
* ftpfreefloat
* minishare
* Ftpfreefloat

**Tools** :

* GDB Peda ([https://github.com/longld/peda](https://github.com/longld/peda))
