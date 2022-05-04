### Part 1: PC Bootstrap 

For 32bit OS, the physical memory layout is like this:

```
+------------------+  <- 0xFFFFFFFF (4GB)
|      32-bit      |
|  memory mapped   |
|     devices      |
|                  |
/\/\/\/\/\/\/\/\/\/\

/\/\/\/\/\/\/\/\/\/\
|                  |
|      Unused      |
|                  |
+------------------+  <- depends on amount of RAM
|                  |
|                  |
| Extended Memory  |
|                  |
|                  |
+------------------+  <- 0x00100000 (1MB)
|     BIOS ROM     |
+------------------+  <- 0x000F0000 (960KB)
|  16-bit devices, |
|  expansion ROMs  |
+------------------+  <- 0x000C0000 (768KB)
|   VGA Display    |
+------------------+  <- 0x000A0000 (640KB)
|                  |
|    Low Memory    |
|                  |
+------------------+  <- 0x00000000
```

- the "Low Memory" 640KB area is the only RAM that an early PC could use.   
  // 早起的16位Intel 8086只能寻址1MB内存空间.
- the 384KB memory area from 0xA0000 to 0xFFFFF is reserved for hardware special use, e.g video display buffer and firmware.
  + the most import part is 64KB region from 960KB~1024KB, which is for BIOS. BIOS does performing basic system initialization   
     such as activating the video card and checking the amount of memory installed, and then loads OS from disk, network... and   
     pass the control to OS.

#### ROM BIOS

When the BIOS runs, it sets up an interrupt descriptor table and initializes various devices such as the VGA display.

After initializing the PCI bus and all the important devices the BIOS knows about, it searches for a bootable device   
such as a floppy, hard drive, or CD-ROM. Eventually, when it finds a bootable disk, the BIOS reads the boot loader   
from the disk and transfers control to it.   

*Exercise 2*
use `si` to trace into ROM BIOS for a few instructions and gusess what it may be doing.

term1: make qemu-nox-gdb   
term2: make gdb   

```assembly
[f000:fff0]    0xffff0: ljmp   $0xf000,$0xe05b
(gdb) si
[f000:e05b]    0xfe05b: cmpl   $0x0,%cs:0x6ac8
(gdb) ni
[f000:e062]    0xfe062: jne    0xfd2e1
(gdb) ni
[f000:e066]    0xfe066: xor    %dx,%dx
(gdb) ni
[f000:e068]    0xfe068: mov    %dx,%ss
(gdb) ni
[f000:e06a]    0xfe06a: mov    $0x7000,%esp
(gdb)
[f000:e070]    0xfe070: mov    $0xf34c2,%edx
(gdb) si
[f000:e076]    0xfe076: jmp    0xfd15c
(gdb)
[f000:d15c]    0xfd15c: mov    %eax,%ecx
(gdb) si
[f000:d15f]    0xfd15f: cli   # clear IF(Interrupt flag)
(gdb) si
[f000:d160]    0xfd160: cld   # clear DF(Direction flag)
(gdb) si 
[f000:d161]    0xfd161: mov    $0x8f,%eax
(gdb) si
[f000:d167]    0xfd167: out    %al,$0x70  # write AL to IO port address 0x70 (CMOS/RTC index register)
(gdb) si
[f000:d169]    0xfd169: in     $0x71,%al  # read from IO port address 0x71(CMOS/RTC data register) to AL
(gdb) si
[f000:d16b]    0xfd16b: in     $0x92,%al
0x0000d16b in ?? ()
(gdb)
[f000:d16d]    0xfd16d: or     $0x2,%al
0x0000d16d in ?? ()
(gdb)
[f000:d16f]    0xfd16f: out    %al,$0x92
0x0000d16f in ?? ()
(gdb)
[f000:d171]    0xfd171: lidtw  %cs:0x6ab8   # load cs:0x6a8 into IDTR(interrupt descriptor table register)
0x0000d171 in ?? ()
(gdb)
[f000:d177]    0xfd177: lgdtw  %cs:0x6a74   # load cs:0x6a74 into GDTR(global descriptor table register)
```

### Part2: Boot Loader

Floppy and hard disks for PCs are divided into 512 byte regions called *sectors*.

If the disk is bootable, the first sector is called the *boot sector*, since this is where the boot loader code resides. 

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses   
0x7c00 through 0x7dff, then jump to CS:IP()
