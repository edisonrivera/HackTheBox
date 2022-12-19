1. Decompress **.zip** file.

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Challenges/Canvas]
└─| 7z x Canvas.zip
```

**Output**
```bash
4 drwx------ 2 root root 4096 Nov  6  2020 css
4 -rw-r--r-- 1 root root   19 Nov  4  2020 dashboard.html
4 -rw-r--r-- 1 root root  513 Nov  4  2020 index.html
4 drwx------ 2 root root 4096 Nov  6  2020 js
```

2. Search infor in **js directory**
```bash
┌──(root㉿kali)-[/home/…/Machines/Challenges/Canvas/js]
└─| cat login.js
```

**Output**
```js
var _0x4e0b=['\x74\x6f\x53\x74\x72\x69\x6e\x67','\x75\x73\x65\x72\x6e\x61\x6d\x65','\x63\x6f\x6e\x73\x6f\x6c\x65','\x67\x65\x74\x45\x6c\x65\x6d\x65\x6e\x74\x42\x79\x49\x64','\x6c\x6f\x67','\x62\x69\x6e\x64','\x64\x69\x73\x61\x62\x6c\x65\x64','\x61\x70\x70\x6c\x79','\x61\x64\x6d\x69\x6e','\x70\x72\x6f\x74\x6f\x74\x79\x70\x65','\x7b\x7d\x2e\x63\x6f\x6e\x73\x74\x72\x75\x63\x74\x6f\x72\x28\x22\x72\x65\x74\x75\x72\x6e\x20\x74\x68\x69\x73\x22\x29\x28\x20\x29','\x20\x61\x74\x74\x65\x6d\x70\x74\x3b','\x76\x61\x6c\x75\x65','\x63\x6f\x6e\x73\x74\x72\x75\x63\x74\x6f\x72','\x59\x6f\x75\x20
...
```

* This file is a js obfuscate, use a deobfuscate js online

3. Use [DeobfuscateJS](https://www.dcode.fr/javascript-unobfuscator)

![djs.PNG](/assets/Challenges/Misc/Canvas/djs.PNG)

* In output we can look important information

![flag_en.PNG](/assets/Challenges/Misc/Canvas/flag_en.PNG)

4. Decode this message to get flag, you can use [DupliChecker](https://www.duplichecker.com/ascii-to-text.php)

![decode.PNG](/assets/Challenges/Misc/Canvas/decode.PNG)

* **Flag:** `HTB{W3Lc0m3_70_J4V45CR1p7_d30bFu5C4710N}`