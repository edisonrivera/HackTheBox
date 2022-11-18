![Ignition.jpg](/assets/Tier-1/Ignition/Ignition.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.1.15
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.1.15
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
```


3. Add web page in `/etc/hosts`

4. Visit web page

![web.PNG](/assets/Tier-1/Ignition/web.PNG)

5. With gobuster search directories
```bash
┌──(root㉿kali)-[/home/kali]
└─| gobuster dir -u http://ignition.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

**Output**
```bash
/admin                (Status: 200) [Size: 7092]
```

6. Login in panel with default credentials

| Username | Password |
|:--------:|:--------:|
| `admin`  | `qwerty123`|

![login.PNG](/assets/Tier-1/Ignition/login.PNG)

**Output**

![flag.png](/assets/Tier-1/Ignition/flag.png)

```txt
Flag: 797d6c988d9dc5865e010b9410f247e0
```
