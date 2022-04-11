---
title: Exfiltrated Easy box on Offensive Security Proving Grounds - OSCP Preparation.
tags: [oscp easy box, PG easy box, enumeration, cms, cve, privilege escalation, cron, linux]
categories: [OSCP, Proving Grounds]
comments: true
date: 2022-01-03 11:33:00
---

Hello,

We are going to exploit one of [OffSec Proving Grounds](https://portal.offensive-security.com/proving-grounds/play) Easy machines which called `Exfiltrated` and this post is not a fully detailed walkthrough, I will just go through the important points during the exploit process.

## Enumeration:
- Nmap:
![image](../../assets/img/sample/pg-exfiltrated/nmap.png)

- Port 80 is running `Subrion` CMS version `4.2.1` as shown in the `/panel`:
![image](../../assets/img/sample/pg-exfiltrated/port80.png)

- We can login with `admin:admin` to the CMS.
- Using Searchsploit to look for exploits:
![image](../../assets/img/sample/pg-exfiltrated/searchsploit.png)

- Exploit:
![image](../../assets/img/sample/pg-exfiltrated/exploit.png)


## Privilege Escalation:
- Running [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS):

![image](../../assets/img/sample/pg-exfiltrated/linpeas.png)

As shown in the picture above there's a cronjob running this script `/opt/image-exif.sh` every minute as root, and this is the content of the script:
```bash
#! /bin/bash
#07/06/18 A BASH script to collect EXIF metadata 

echo -ne "\\n metadata directory cleaned! \\n\\n"


IMAGES='/var/www/html/subrion/uploads'

META='/opt/metadata'
FILE=`openssl rand -hex 5`
LOGFILE="$META/$FILE"

echo -ne "\\n Processing EXIF metadata now... \\n\\n"
ls $IMAGES | grep "jpg" | while read filename; 
do 
    exiftool "$IMAGES/$filename" >> $LOGFILE 
done

echo -ne "\\n\\n Processing is finished! \\n\\n\\n"
```
- it runs `exiftool` against every jpg file on this directory `/var/www/html/subrion/uploads`, let's exploit that:

I used this script to create the exploit: [https://github.com/OneSecCyber/JPEG_RCE](https://github.com/OneSecCyber/JPEG_RCE) .

1. Generate a [payload](https://bing0o.github.io/posts/reverse-shell-generator/):

![image](../../assets/img/sample/pg-exfiltrated/payload.png)

2. Inject the payload into a jpg file:
![image](../../assets/img/sample/pg-exfiltrated/inject.png)

3. Upload the `runme.jpg` file to the remote target on `/var/www/html/subrion/uploads` directory and wait a minute:
![image](../../assets/img/sample/pg-exfiltrated/root.png)


Happy Hacking!