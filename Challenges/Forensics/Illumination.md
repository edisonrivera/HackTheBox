1. Decompress **.zip** file.

```bash
┌──(root㉿kali)-[/home/…/Machines/Challenges/Forensics/Illumination]
└─| 7z x Illumination.zip
```

**Output**
```bash
┌──(root㉿kali)-[/home/…/Challenges/Forensics/Illumination/Illumination.JS]
└─| ls -la
total 20
drwx------ 3 root root 4096 May 30  2019 .
drwx------ 4 root root 4096 Dec 19 14:29 ..
-rw-r--r-- 1 root root 2635 May 30  2019 bot.js
-rw-r--r-- 1 root root  199 May 30  2019 config.json
drwx------ 7 root root 4096 May 30  2019 .git
```

2. If you watch all files, we have a **.git** file. List all commits in current project
```bash
┌──(root㉿kali)-[/home/…/Challenges/Forensics/Illumination/Illumination.JS]
└─| git log --online
```

**Output**
```bash
edc5aab (HEAD -> master) Added some whitespace for readability!
47241a4 Thanks to contributors, I removed the unique token as it was a security risk. Thanks for reporting responsibly!
ddc606f Added some more comments for the lovely contributors! Thanks for helping out!
335d6cf Moving to Git, first time using it. First Commit!
```

3. In `47241a4` commit we have a **token**.
```bash
┌──(root㉿kali)-[/home/…/Challenges/Forensics/Illumination/Illumination.JS]
└─| git show 47241a4
```

**Output**
```bash
┌──(root㉿kali)-[/home/…/Challenges/Forensics/Illumination/Illumination.JS]
└─| git show 47241a4
```
* You can put the first letters of commit, isn't require all commit.

**Output**
```js
-       "token": "SFRCe3YzcnNpMG5fYzBudHIwbF9hbV9JX3JpZ2h0P30=",
+       "token": "Replace me with token when in use! Security Risk!",
        "prefix": "~",
        "lightNum": "1337",
        "username": "UmVkIEhlcnJpbmcsIHJlYWQgdGhlIEpTIGNhcmVmdWxseQ==",
```

4. Previous token is encoded in base64, only decode this and you get the flag.
```bash
┌──(root㉿kali)-[/home/…/Challenges/Forensics/Illumination/Illumination.JS]
└─| echo "SFRCe3YzcnNpMG5fYzBudHIwbF9hbV9JX3JpZ2h0P30=" | base64 -d
```

**Output**
```bash
HTB{v3rsi0n_c0ntr0l_am_I_right?}
```