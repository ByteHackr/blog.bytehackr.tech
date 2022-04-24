# Passive OSINT

## Netcraft.com

Finds underlying OS, web server version uptime

## Find subdomains

Sometimes SSL is a goldmine of information

```
crt.sh
```

```
#!/bin/bash
# a basic script to pull information from crt and present it
# example ./crt.sh offsecnewbie.com
# author rowbot
if [[ $# -eq 0 ]] ;

then
	echo "Usage: ./crt.sh domain. Also you might have to install jq - 'apt get install jq'"
	exit 1

else

curl -s https://crt.sh/\?q\=\%.$1\&output\=json | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u > $1

fi

```

If you can't get jq installed - try this script

```
#!/bin/bash
# a basic script to pull information from crt and present it
# example ./crt.sh offsecnewbie.com
# author rowbot

if [[ $# -eq 0 ]] ;

then
	echo "Usage: ./crt.sh domain"
	exit 1

else

curl -s "https://crt.sh/?q=%.$1" -o rawdata; cat rawdata | grep "<TD>" | grep -vE "style" | cut -d ">" -f 2 | grep -Po '.*(?=....$)' | sort -u | grep -v "*" > $1

fi
```

Compare subdomains found using theHavester with crt.sh script as some will be missing - not all domains have ssl.

```
theHarvester -d offsecnewbie.com -l 500 -b google
```

IP addresses from subdomains

```
for i in $(cat subdomains.txt); do dig $i | grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' | grep -vE "10.*"; done
```

Use Virustotal to find subdomains

```
https://www.virustotal.com
```

Also you can play about with a nice entity diagram

## Read

```
https://www.bugcrowd.com/blog/discovering-subdomains/
```

FireFox addon - passive recon

## Google hacking

```
examples here
site:offsecnewbie.com doctype:docx
inurl:/etc/passwd%00 intext:root
also check out doc meta info, ie doc creator, where doc was stored, created with Office 2010, saved on this network share eg its IP address
```

also heck out doc meta info, gives info such as where doc was stored - network share ip address, who created it, what was it created with etc

```
https://github.com/ElevenPaths/FOCA
```

## Social Media Search

Search for people on social media

```
/opt/sherlock/sherlock.py
```

## Recon

A giant inventory of recon tools is available via the Skip Tracing Framework

```
https://makensi.es/stf/
```
