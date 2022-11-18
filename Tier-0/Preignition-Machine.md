![Explosion.png](/assets/Tier-0//Explosion/Explosion.jpg)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 1 10.129.79.180
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.79.180
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
```


3. Visit web page

![web.PNG](/assets/Tier-0/Preignition/web.PNG)


4. Search directories with **gobuster**
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| gobuster dir -u 10.129.79.180 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

* If you can search files with especific extensions user `-x <extension>`

**Output**
```bash
/admin.php            (Status: 200) [Size: 999]
```

5. Visit the file **admin.php**


![login.PNG](/assets/Tier-0/Preignition/login.PNG)

6. Use default credentials
   
| Username | Password |
|:--------:|:--------:|
| `admin`  | `admin`  |
| `root`   | `root`   |
| `administrator` | `administrator` |

7. In this case the correct credentials are `admin` and `admin`

![flag.PNG](/assets/Tier-0/Preignition/lfag.PNG)