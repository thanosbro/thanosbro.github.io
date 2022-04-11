---
title: HAwordy Medium box on Offensive Security Proving Grounds - OSCP Preparation.
tags: [oscp medium box, PG medium box, enumeration, wordpress, misc, privilege escalation, suid, linux]
categories: [OSCP, Proving Grounds]
comments: true
date: 2021-12-18 11:33:00
---

Hello,

We are going to exploit one of [OffSec Proving Grounds](https://portal.offensive-security.com/proving-grounds/play) Medium machines which called `HAwordy` and this post is not a fully detailed walkthrough, I will just go through the important points during the exploit process.

## Enumeration:
- Nmap:
![nmap](../../assets/img/sample/pg-hawordy/nmap.png)

- Using `wpscan` against wordpress running on port `80`, path `/wordpress`:
![image](../../assets/img/sample/pg-hawordy/wpscan.png)

- The Exploit:
![image](../../assets/img/sample/pg-hawordy/exploit.png)

- phtml file to upload:

![image](../../assets/img/sample/pg-hawordy/phtml.png)

- RCE:
![image](../../assets/img/sample/pg-hawordy/rce.png)

## Privilege Escalation:
- the wget binary on the system has the SUID bit:
![image](../../assets/img/sample/pg-hawordy/suid.png)

- we can add our root user to the system by overwriting `/etc/passwd` file using wget:
    1. copying `/etc/passwd` file to our attack machine.
    
    2. creating new user named `bingo` with password `pwned` using `openssl`:
    ![image](../../assets/img/sample/pg-hawordy/openssl.png)
    
    3. adding the new user to the downloaded `passwd` file:
    ![image](../../assets/img/sample/pg-hawordy/localpwd.png)
    
    4. uploading the new `passwd` file to the target machine and overwriting the remote `/etc/passwd` using wget:
    ![image](../../assets/img/sample/pg-hawordy/root.png)



Happy Hacking!