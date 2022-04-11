---
title: Doing Recon on a Large Scope
tags: [bug hunting, recon, enumeration, OSINT]
categories: [bug hunting, recon]
comments: true
date: 2020-04-08 11:33:00
---

![image](../../assets/img/sample/doing-recon/image.png)

hello Hackers, this is the first post on my blog and I hope it will be useful for some of you, I'm gonna talk about Recon on Targets with a large scope and how to collect its domains, subdomains and IPs.

## Why Recon!

As you all know Recon is the first stage when you're doing a penetration testing or bug hunting, and if you can get an information ( domain/ip/github account ... and so on) other hackers didn't reach to it yet, it's a great advantage for you to exploit it before other hackers and get a bounty, sounds great right! :D, it's all based on your Recon-fu, also always keep you data organized and easy to reach, because these days there's a lot of vulnerabilities being found by security researchers and when the POC is published you have to be ready to find some of your targets vulnerable to it, take care of your recon data and it will take care of you, trust me.

our recon will go through 3 stages:
 - Finding SubDomains.
 - Finding Domains related to the same Target.
 - Finding IPs.

## Finding SubDomains:

let's say our target is `target.com` and we want to get all possible subdomains like `api.target.com` and `admin.target.com` ... Etc, there's a lot of tools to do that but i'm gonna pick some of them ( based on my own opinion ), and we're gonna use a bash script to use these tools and combine the results and sort them into one file! 


### Automate The Process:

> Automation is your best friend ;)

so we're gonna use 4 tools and 3 APIs:
 1. Tools:
   - [Findomain](https://github.com/Edu4rdSHL/findomain)
   - [SubFinder](https://github.com/projectdiscovery/subfinder)
   - [Amass](https://github.com/OWASP/Amass)
   - [AssetFinder](https://github.com/tomnomnom/assetfinder)
 1. APIs:
   - [WayBackMachine](http://web.archive.org/)
   - [crt.sh](https://crt.sh/)
   - [BufferOver](https://dns.bufferover.run/)

basically, you can use each tool or API separated, then combine the results, but this gonna take time and not productive for me. 

so I made a bash script to automate this and make it easier and faster, you can use a specific tools or exclude a specific one as you like: [SubEnum](https://github.com/bing0o/SubEnum/)

![subenum's help](../../assets/img/sample/doing-recon/domains.sh.png)

this tool will be enough to get all the possible subdomains.

also running a brute force tool like [knockpy](https://github.com/guelfoweb/knock) is a good idea to find more subdomains especially if you use a custom wordlist.

## Finding Domains related To The Same Organization:
this means if the main domain of our target is `target.com` and we got all the subdomains of that domain in the previous stage, but maybe our target own other domains like `target.net`, `targetshop.com` ... etc, we need to find those too.

in this stage we're gonna use a reverse whois lookup, first we do a normal whois against `target.com`, usually i use [viewdns.info](https://viewdns.info/whois/) to get the Registrant (Name OR Email Address), we're gonna need one or both of this informations to do a reverse whois lookup and get all the domains that registred using that name or email address (`Email Address Recommended here`).

Let's use some APIs to collect the domains:

1. [viewdns.info](https://viewdns.info/reversewhois/): this website is good but it gives you only the first 500 results for free and you have to pay for the rest, so if your target has more than that as domains, it's not good for you.

2. [whoisxmlapi.com](ttps://reverse-whois-api.whoisxmlapi.com/): I used this API lately and I liked it, it's free and very good, but I'm not sure if it's limited or not!, but at least it does the job :). 

### Let's bash it:

since we're gonna use [whoisxmlapi.com](ttps://reverse-whois-api.whoisxmlapi.com/) API, first you have to create an account and get your `API KEY`.

let's make a bash scrip to get the results:


```bash
#!/bin/bash

#check if the first argument doesn't exist.
[ -z "$1" ] && { echo "[!] ./alldomains.sh <Registrare Email/Name>"; exit 1; }

curl -s -XPOST https://reverse-whois-api.whoisxmlapi.com/api/v2 -d \
"{\"apiKey\": \"YOUR_API_KEY\",\"mode\": \"purchase\",\"basicSearchTerms\": {\"include\": [\"$1\"]}}" | 
tr ',\|[' '\n' | 
cut -d '"' -f 2 | 
grep -v "domainsCount\|domainsList"
```

don't forget to change `YOUR_API_KEY` with your API key :)

save this in a file then run:

```bash
./alldomains.sh domains@target.com > domains.txt 
```

PS: `domains@target.com` we got this from the previous stage (whois lookup)


here don't forget that you have to do the first stage again `subdomains finding` with this new domains, and get more results ... It gets bigger :D.

I want to mention that this script [SubEnum](https://github.com/bing0o/SubEnum/) is able to use a list of domains, so you can just feed your list of domains to this tool and you don't have to use all the tools and APIs to make it faster, like this:

![SubEnum](../../assets/img/sample/doing-recon/ListOfDomain.png)

## Finding IPs:

here's two ways to get list of IPs:

### Using Amass:
you can use amass to get the IP for each subdomain or domain you find using this tool like this:

```bash
amass enum -d target.com -ip -o results.txt 
```
this will give you the IP of each SubDomain and save the results in results.txt file.

### Using ASN:
**Recomended**

first you need the ip address of your `target.com`, then use this [site](https://bgp.he.net/) to get [autonomous system number (ASN)](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)), we're gonna use this number to get a bunch of [CIDRs](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) owned by our target, 

Enter the IP address in the search bar and you will get something like this:

||:----:||
||Announced By||
|:--|:--|--:|
|Origin AS|Announcement|Description|
|AS1X1|192.168.32.0/19|Target Services, Inc|  

the result of **Announcement** is The CIDR, click on the result of **Origin AS** you will find some more AS numbers related to the organization, but be carefull, don't get out of scope, if your scope only for `Target Services. Inc` , you have to exclude `Target Corp, Inc` .... etc.


### let's bash it:

as always i can't do this boring stuff without automation, this is a bash script [ASNenum.sh](https://github.com/bing0o/bash_scripting/blob/master/ASNenum.sh), it takes two kind of arguments, IP or ASN, if you enter an IP it will gives you the ASN and one CIDR of the entred IP that awned by your target organization, if you enter ASN the result will be a list of CIDRs, but The manuall way gives you more results.

and here is a python script if you want to translate CIDR to a list of IPs:


```python
#!/usr/bin/env python

import ipaddress, sys

try:
    cidr = sys.argv[1]
except:
    print("[!] Usage: ips <CIDR>")
    exit(1)

ips = ipaddress.ip_network(unicode(str(cidr)))

for ip in ips:
    print(ip)

``` 

lately i found more than 380+ CIDR for my target, the list of IPs got more than 44 million IP, of course not all of them are up, and i had to filter them, but hey with tools like masscan and zmap it's not a problem at all it tooks just a few minutes to go through the entire list and scan ports 80,443 :D

## conclusion:

well, let's see:
 1. Finding the SubDomains: Done.
 2. Finding domains: Done.
 3. Finding IPs: Done.

we didn't go deep in our recon, i think it's enough at this point , next time we will talk about how to deal with all of this data, and filter this results, to find your entry point and start the real job :D. 

if you learnt something from this post please let me learn something from you too, if you have any recommandation or question leave it in the comments section, or find me on [twitter](https://twitter.com/hack1lab).

don't forget to share everywhere :D.

Happy Hacking!
