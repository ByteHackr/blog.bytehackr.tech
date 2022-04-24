# 80, 443

## Checklist <a href="#checklist" id="checklist"></a>

* View SSL certificates for usernames
* View Source code
* Check /robots.txt, .htaccess, .htpasswd
* Check HTTP Request
* Run Burp Spider
* View Console
* Use Nikto
* Check OPTIONS
* HTTP PUT / POST File upload
* Parameter fuzzing with wfuzz
* Browser response vs Burp response
* Shell shock (cgi-bin/status)
* Cewl wordlist and directory bruteforce
* `nmap --script http-enum 192.168.10.55`
* Apache version exploit & other base server exploits
* **Port 443 :**
  * nmap -Pn -sV --script ssl\* -p 443 10.10.10.60 -A -T5
  * Heartbleed (`sslyze --heartbleed <ip>`)
  * Heartbleed exploit code ([https://gist.github.com/eelsivart/10174134](https://gist.github.com/eelsivart/10174134))
  * Shellshock
  * Poodle

**IIS :**

* ​[https://book.hacktricks.xyz/pentesting/pentesting-web/iis-internet-information-services](https://book.hacktricks.xyz/pentesting/pentesting-web/iis-internet-information-services)​
* Try changing file.asp file to file.asp.txt to reveal the source code of the files

**Apache :**

* Struts ([https://github.com/LightC0der/Apache-Struts-0Day-Exploit](https://github.com/LightC0der/Apache-Struts-0Day-Exploit))
* Shell shock ([https://www.exploit-db.com/exploits/34900](https://www.exploit-db.com/exploits/34900))
* OpenFuck ([https://github.com/exploit-inters/OpenFuck](https://github.com/exploit-inters/OpenFuck))

## **Directory Enumeration** <a href="#directory-enumeration" id="directory-enumeration"></a>

**Apache :** x -> php, asp, txt, xml, bak

**IIS : x->** asp, aspx, txt, ini, tmp, bak, old

Gobuster **quick** directory busting

```
gobuster dir -w /usr/share/seclists/Discovery/Web_Content/common.txt -t 80 -u http://
```

Gobuster search with file **extension**

```
gobuster dir -w /usr/share/seclists/Discovery/Web_Content/common.txt -t 100 -x txt,php,csv,md,json,js,html,py,sh -u http://url​gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,csv,md,json,js,html,py,sh -t 100 -u http://url ​gobuster dir -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -x txt,php,csv,md,json,js,html,py,sh -u http://url 
```

Gobuster **comprehensive** directory busting

```
gobuster -s 200,204,301,302,307,403 -w /usr/share/seclists/Discovery/Web_Content/big.txt -t 100 -x txt,php,csv,md,json,js,html,py,sh -u http://url
```

* `​`
* `gobuster dir -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,csv,md,json,js,html,py,sh -k -u http://10.10.10.x`
* \-k (ignore ssl verification)
* \-x specific extension
* Dirbuster
* Change wordlists (Wfuzz, dirb)
*   Custom directory enumeration (HTB Obscurity)

    * wfuzz -c -z file,common.txt -u [http://10.10.10.168:8080/FUZZ/SuperSecureServer.py](http://10.10.10.168:8080/FUZZ/SuperSecureServer.py)​

    ​

**Parameter Fuzzing**

**WFUZZ**

* hc - status code to ignore
* hw - word length to ignore
* hh - char length to ignore
* hl - line length to ignore

```
wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 --hw 12 http://192.168.0.119/index.php?FUZZ=id
```

## Wordpress <a href="#wordpress" id="wordpress"></a>

**Wpscan**

```
wpscan --url http://10.10.10.10 -e u,vp // enumerate users & vulnerable plugins​wpscan --url 10.10.10 --passwords rockyou.txt --usernames elliot
```

**Metasploit**

```
use auxiliary/scanner/http/wordpress_login_enum
```

**Username Enumeration via Bruteforce**[SecurityCompass/wordpress-scriptsContribute to SecurityCompass/wordpress-scripts development by creating an account on GitHub.github.com](https://github.com/SecurityCompass/wordpress-scripts/blob/master/wp\_login\_user\_enumeration.py)

`python wp_brute.py -t` [`http://10.10`](http://10.0.0.10) `-u usernames.txt`[\
](https://blog.adithyanak.com/oscp-preparation-guide/enumeration/bruteforce)
