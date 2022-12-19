1. Decompress **.zip** file.

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Challenges/BehindScenes]
└─| 7z x Behind\ the\ Scenes.zip
```

**Output**
```bash
total 12
drwxr-xr-x  3 root root 4096 Dec 19 16:39 .
drwxr-xr-x 17 root root 4096 Dec 19 14:47 ..
drwxr-xr-x  2 root root 4096 Dec 11 18:12 rev_behindthescenes
```

2. Use **hexeditor** to see information about file `behindthescenes`

```bash
┌──(root㉿kali)-[/home/…/Machines/Challenges/BehindScenes/rev_behindthescenes]
└─| hexeditor behindthescenes
```
* Use `Ctrl` + `W` to search for string **HTB** or **password**

![info.PNG](/assets/Challenges/Reversing/info.PNG)

* **Flag:** `HTB{Itz_0nLy_UD2}`