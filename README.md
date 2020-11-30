# 247ctf-Stack-my-Pivot-Writeup
Try this challenge for yourself [here](https://247ctf.com/).


I found no writeup online for this challenge so here is one.

So There is the challenge:
![alt text](./screenshots/247Dashboard.png "Challenge infos")

We learn that the buffers will be small.
Let's look at it inside ghidra.

## Ghidra

![alt text](./screenshots/ghidra_main.png "ghidra main")

Nothing interesting in the main function, let's look at the chall function.

![alt text](./screenshots/ghidra_chall.png "ghidra chall")

So two buffers, the first one is large but cannot be overflowed the second is very small but can be overflowed by 8 bytes.

It appears simple enough, put some shellcode inside the first one, jump to it from the second one.

Let's analyze all this in gdb.



## GDB

First let's check security and permissions.
> checksec ./stack_my_pivot

```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX disabled
PIE:      No PIE (0x400000)
RWX:      Has RWX segments
```

No PIE, nice. 64bit, wait we only read 24bytes, that's not a lot. Well this probably wont be a problem later.

I use gdb gef but use whatever you want

> gef> vmmap

```
0x0000000000400000 0x0000000000401000 0x0000000000000000 r-x /home/emanone/Documents/CTF/247CTF/Pwn/StackPivot/stack_my_pivot
0x0000000000600000 0x0000000000601000 0x0000000000000000 r-- /home/emanone/Documents/CTF/247CTF/Pwn/StackPivot/stack_my_pivot
0x0000000000601000 0x0000000000602000 0x0000000000001000 rw- /home/emanone/Documents/CTF/247CTF/Pwn/StackPivot/stack_my_pivot
0x00007ffff7dcf000 0x00007ffff7dd1000 0x0000000000000000 rw- 
0x00007ffff7dd1000 0x00007ffff7df7000 0x0000000000000000 r-- /usr/lib/libc-2.32.so
0x00007ffff7df7000 0x00007ffff7f44000 0x0000000000026000 r-x /usr/lib/libc-2.32.so
0x00007ffff7f44000 0x00007ffff7f90000 0x0000000000173000 r-- /usr/lib/libc-2.32.so
0x00007ffff7f90000 0x00007ffff7f93000 0x00000000001be000 r-- /usr/lib/libc-2.32.so
0x00007ffff7f93000 0x00007ffff7f96000 0x00000000001c1000 rw- /usr/lib/libc-2.32.so
0x00007ffff7f96000 0x00007ffff7f9c000 0x0000000000000000 rw- 
0x00007ffff7fca000 0x00007ffff7fce000 0x0000000000000000 r-- [vvar]
0x00007ffff7fce000 0x00007ffff7fd0000 0x0000000000000000 r-x [vdso]
0x00007ffff7fd0000 0x00007ffff7fd2000 0x0000000000000000 r-- /usr/lib/ld-2.32.so
0x00007ffff7fd2000 0x00007ffff7ff3000 0x0000000000002000 r-x /usr/lib/ld-2.32.so
0x00007ffff7ff3000 0x00007ffff7ffc000 0x0000000000023000 r-- /usr/lib/ld-2.32.so
0x00007ffff7ffc000 0x00007ffff7ffd000 0x000000000002b000 r-- /usr/lib/ld-2.32.so
0x00007ffff7ffd000 0x00007ffff7fff000 0x000000000002c000 rw- /usr/lib/ld-2.32.so
0x00007ffffffde000 0x00007ffffffff000 0x0000000000000000 rwx [stack]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 --x [vsyscall]
```

Executable stack, nice. Let's assume ASLR is on (it is).

Ok let's see what registers we can overwrite.
We now the read is 0x1c so 24.

> pattern create 28
```
[+] Generating a pattern of 28 bytes
aaaaaaaabaaaaaaacaaaaaaadaaa
[+] Saved as '$_gef0
```

We know the first one cannot be overflowed so let's not bother trying it.

> gef➤  c
```
Continuing.
[+] Welcome!
[+] What's your first name?
smth 
[+] What's your surname?
aaaaaaaabaaaaaaacaaaaaaadaaa
```

![alt text](./screenshots/register_overflow.png "register overflow")



## Thanks
I was really stuck on this challenge and the creator [Razvi](https://twitter.com/Razvieu) had the kindness of helping me ! Thanks to him for his time and for the challenge.