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

### Part2: Boot Loader

Floppy and hard disks for PCs are divided into 512 byte regions called *sectors*.

If the disk is bootable, the first sector is called the *boot sector*, since this is where the boot loader code resides. 

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses   
0x7c00 through 0x7dff, then jump to CS:IP()
