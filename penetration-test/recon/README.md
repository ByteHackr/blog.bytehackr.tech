# Recon

https://twitter.com/dc9221/status/1233349429828243456/photo/1

![](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-MRtm2U7Ps4MoTULw68i%2F-MRtm5MYGO89JO6ROC4m%2Fimage.png?alt=media\&token=4916b03c-c750-40d0-a551-bd6770143ece)

### A Web App Tool to Run and Keep all your recon in the same place. <a href="#a-web-app-tool-to-run-and-keep-all-your-recon-in-the-same-place" id="a-web-app-tool-to-run-and-keep-all-your-recon-in-the-same-place"></a>

```
https://docs.reconness.com/
```

![bugcrowd](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-LuP3gyAF959Ou4vUVT0%2F-LuUCuJBMf9bck8Obz3g%2Fimage.png?alt=media\&token=e94f777f-5d25-4aac-a3f3-a751fbfdc794)

![Ways to find Brand / TLD Discovery](https://gblobscdn.gitbook.com/assets%2F-LSy0aAo8OKT4I-Ahftv%2F-LuP3gyAF959Ou4vUVT0%2F-LuUAlzxAbfpBfcfSfK\_%2Fimage.png?alt=media\&token=5d93a91a-6942-4f95-9a69-6e5928d51e9e)

## Discover IP Space <a href="#discover-ip-space" id="discover-ip-space"></a>

```
look for different asns    http://bgp.he.net         prefixes v4        presents different IP ranges inside the CIDR notationwhois -h whois.cymru.com $(dig +short tesla.com) #finds CIDR notation for tesla.com - finds ip address for websitecareful not to pick up shared hosting amassfind subdomains for each of of these ASNsamass intel -asn 123456#find more details about ip ranges/countries etcwhois.arin.netripe.netshodan.io    https://beta.shodan.io/search/filters #useful filters    org:"Tesla"    #maybe some false positives
```

## Discovering New Targets and TLDs <a href="#discovering-new-targets-and-tlds" id="discovering-new-targets-and-tlds"></a>

```
Find different attack surfaces which other people might not have discovered, ie from acquisitionsWikipedia    search for subsiduriesCrunchbase.com    search for org        look for acquisitionsOwler.com    search for org        look for acquisitionsAcquiredby.co    search for org        acquisitionsLinkedIn    affiliated pages or similiar pagesReversewhois    amass intel -d tesla.com -whoisBuiltwith    Relationship Profile        look for anaylitical codes under ID # google tracking codes that are linked to different sitesGoogleDorks    intext:"copyright tesla motors"ShodanDorks    http.favicon.hash:81586312 # Jenkins favicon hash        can narrow it down after that        
```

## Subdomain Enumeration <a href="#subdomain-enumeration" id="subdomain-enumeration"></a>

```
https://github.com/tomnomnom/assetfinder/    assetfinder -subs-only offsecnewbie.comamass enum -d tesla.com -ip    setup config.ini file get access to a lot more data than baseline toolsubfinder # not used much but has a few extra sources    subfinder -d tesla.com -t 25 -timeout 5 -silentDNS bruteforcing https://youtu.be/La3iWKRX-tE?t=802all.txt + goaltdns + commonspeak --> massdns / gobuster3    https://github.com/subfinder/goaltdns  massdns #download all.txt     sed -e 's/$/.tesla.com/' -i all.txt #adds tesla.com to start of each line - overrides file     massdns -r lists/resolvers.txt -t CNAME all.txt -o S > results     #massdns much faster than gobuster but can get you blacklisted from dns resolovers - a lot of false positives and negatives rapid 7 fdns#download latest file here: https://opendata.rapid7.com/sonar.fdns_v2/    pv 2019-10-27-1572199582-fdns_cname.json.gz | pigz -dc | grep -E "\.tesla\.org\"," | jq -r '.name'certstream    certstream | grep -E "\.tesla\.com$"    #Real-time certificate transparency log update stream
```

## Fingerprinting <a href="#fingerprinting" id="fingerprinting"></a>

what is running, narrow down attack surface

```
builtwith.comwhatweb #follows redirection http-->httpsmassscan & nmap    masscan -p1-65535 $(dig +short tesla.com) --rate 1000
```

## Dorking <a href="#dorking" id="dorking"></a>

```
shodan dork    org:"Tesla"    ssl:"Tesla" #accurate and can prove ownership to organisation     ssl:"Tesla" http.component:"Drupal"    ssl:"Tesla" http.title:"Login"censys.io    443.https.tls.certificate.parsed.subject.organizational_unit: Tesla Motorsgithub dork    "tesla.com" password    "tesla.com" key    "tesla.com" apihttps://github.com/condingo/dorky #automate the dorking process - tool to be released​
```

## Content Discovery <a href="#content-discovery" id="content-discovery"></a>

```
Burpcrawler    crawl siteLinkfinder https://github.com/GerbenJavado/LinkFinderjsparser - similar to abovegobuster & recursebuster # recurse only does directory otxurls    echo "www.tesla.com" | otxurls | head -n 300 #attempts to find urls within alienvaultwaybackurls    echo "www.tesla.com" | waybackurls | head -n 300 #similar to above, you can get parameters back which can be useful when fuzzing​
```

## Parameter Discovery <a href="#parameter-discovery" id="parameter-discovery"></a>

```
https://github.com/maK-/parameth
```

## Automation <a href="#automation" id="automation"></a>

```
https://github.com/codingo/InterlaceInterlace #multithreads other tools    interlace -tL domains.txt -c "amass enum-d _target_" -o siubdomains.txt -threads 20LazyRecon #outofdate but still good baseline
```

GHDB + others GUI tool

```
https://www.bishopfox.com/resources/tools/google-hacking-diggity/attack-tools/
```

## Username discovery <a href="#username-discovery" id="username-discovery"></a>

To find out what sites a user is registered:

```
https://namechk.com/https://whatsmyname.app/
```

[\
](https://guide.offsecnewbie.com/cherrytree-oscp-template)
