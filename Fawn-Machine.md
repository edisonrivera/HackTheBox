![Fawn.png](/assets/fawn.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.166.28
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.166.28
```

**Output**
```bash
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
```

3. Connect with **ftp**
```bash
┌──(root㉿kali)-[/home/kali]
└─| ftp 10.129.166.28 
Connected to 10.129.166.28.
220 (vsFTPd 3.0.3)
Name (10.129.166.28:kali):
```
+ In `Name (10.129.166.28:kali):` write **anonymous**.
+ We do this to access to system without password.


4. List files of system and get the `flag.txt` file.
```bash
ftp> ls
229 Entering Extended Passive Mode (|||12844|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> get flag.txt
```

5. Go to path **/home/kali** list files and read `flag.txt` 
```bash
┌──(root㉿kali)-[/home/kali]
└─| cat flag.txt           
035db21c881520061c53e0536e44f815 

```



**By: EsT**