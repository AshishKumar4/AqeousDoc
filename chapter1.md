# The Dive - boot.s

I'll start by explaining the code in _boot.s_. This file is the start of everything. Since we're relying on   grub, we've made sure to make it multiboot compatible \(v1.6\).  
The necessary code is explained below.

### MULTIBOOT

```
MAGIC equ 0xE85250D6
ARCH equ 0
HEADER_LENGTH equ multiboot_end - multiboot_end
CHECKSUM equ -(MAGIC + ARCH + HEADER_LENGTH)
ALIGN 8, db 0
section .multiboot
multiboot_start:
        DD MAGIC
        DD ARCH
        DD HEADER_LENGTH
        DD CHECKSUM
multiboot_end:
```

The code is self-explanatory. _MAGIC_ is a magic number that must always be present at the start of the multiboot header. _ARCH_ is a number that gives information about the architecture. `0 means x86_64`  
One thing to note is that the header MUST be 8 byte aligned. We put it into it's own section because things are easier when linking the whole OS.

### The rest of the OS

We first declare a stack because we don't have one available when the system boots.

```
ALIGN 4
section .bss
stack_top:
    resb 16384
stack_bottom:
```

We align it on a 4 byte page because it's easier to work with in protected mode. This OS as of now **DOES NOT** support 64 bit operation.

We can now finally start talking about the OS code now.

```
section .text
GLOBAL _start:function (_start.end - _start)
_start:
        mov esp, stack_bottom
    mov ax, 0
extern ksetup
        call ksetup
        cli
.hang:  hlt
        jmp .hang
.end:
```

`_start` is the entry point of our OS. The start of the code that GRUB will load.  
We first load the address of the bottom of the stack into the stack pointer `esp`. Then we declare an external function `ksetup()` this function does everything needed to set up the hardware, and then tranfers to `kernel_main()`, where the actual kernel code resides.
###boot_bc.s
This is a backwards `boot.s` file clone that uses Multiboot v0.6. Use this only if you are unable to use Multiboot 2 v1.6. There isn't really anything else to this file.
####Multiboot v0.6
```
	MAGIC equ 0x1BADB002
	A_4K equ 1
	MEM_INFO equ 2
	FLAGS equ A_4K | MEM_INFO
	CHECKSUM equ -(MAGIC + FLAGS)
	ALIGN 4
section .multiboot
	DD MAGIC
	DD FLAGS
	DD CHECKSUM
```
It's similar to the previous file. The only difference is that there is no `HEADER_LENGTH` field, and there is a new `FLAGS` field, for more info, look up the documentation

That concludes everything we have to say about the booting procedure and boot.s

