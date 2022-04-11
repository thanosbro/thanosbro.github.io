---
title: My-CMSMS Medium box on Offensive Security Proving Grounds - OSCP Preparation.
tags: [oscp medium box, PG medium box, enumeration, mysql, misc, privilege escalation, sudo, linux]
categories: [OSCP, Proving Grounds]
comments: true
date: 2021-12-21 11:33:00
---

Hello,

We are going to exploit one of [OffSec Proving Grounds](https://portal.offensive-security.com/proving-grounds/play) Medium machines which called `My-CMSMS` and this post is not a fully detailed walkthrough, I will just go through the important points during the exploit process.

## Enumeration:
- Nmap:
![image](../../assets/img/sample/pg-mycmsms/nmap.png)

- Login to the remote mysql using `root:root`:
![image](../../assets/img/sample/pg-mycmsms/mysql-login.png)

- Exploring `cmsms_db` Database:

```
MySQL [cmsms_db]> show tables;
+--------------------------------+
| Tables_in_cmsms_db             |
+--------------------------------+
| cms_additional_users           |
.
.
.
| cms_users                      |
| cms_users_seq                  |
| cms_version                    |
+--------------------------------+
53 rows in set (0.328 sec)
```
![image](../../assets/img/sample/pg-mycmsms/pass-hash.png)

- Updating the admin password:
	
    I couldn't crack the MD5 hash, so let's just try to update the password to something we already know, I found this [blog post](https://cmscanbesimple.org/blog/cms-made-simple-admin-password-recovery) and it shows the MySQL query to update the password:
    ![image](../../assets/img/sample/pg-mycmsms/update-hash.png)

- Executing MySQL Query:
    *  This query will update the admin password to `admin`:
```mysql
update cms_users set password = (select md5(CONCAT(IFNULL((SELECT sitepref_value FROM cms_siteprefs WHERE sitepref_name = 'sitemask'),''),'admin'))) where username = 'admin';
```

### Getting RCE:
1. Generating [bash reverse shell payload](https://bing0o.github.io/posts/reverse-shell-generator/) and starting a Netcat Listener:
![image](../../assets/img/sample/pg-mycmsms/gen-shell.png)

2. Injecting bash payload to the application:
![image](../../assets/img/sample/pg-mycmsms/inject-payload-1.png)

![image](../../assets/img/sample/pg-mycmsms/inject-payload-2.png)
3. hit the submit button, then open the shell:

![image](../../assets/img/sample/pg-mycmsms/open-shell.png)
4. Hit `Run`:

![image](../../assets/img/sample/pg-mycmsms/run-shell.png)
5. We got RCE:
![image](../../assets/img/sample/pg-mycmsms/rce.png)


## Privilege Escalation:
- LinEnum:

![image](../../assets/img/sample/pg-mycmsms/linenum.png)

- Decoding:
![image](../../assets/img/sample/pg-mycmsms/decode.png)

- Creds:
	* User: `armour`
	* Pass: `Shield@123`

- Getting root:
![image](../../assets/img/sample/pg-mycmsms/root.png)


Happy Hacking!