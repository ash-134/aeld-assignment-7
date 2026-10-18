# Kernel Oops Analysis for faulty Driver

## 1. Triggering the Kernel Oops
Command executed on QEMU target:
    echo "hello_world" > /dev/faulty

## 2. Kernel Oops Output Summary
    Unable to handle kernel NULL pointer dereference at virtual address 0000000000000000
    Mem abort info:
      ESR = 0x0000000096000046
      EC = 0x25: DABT (current EL), IL = 32 bits
      FSC = 0x06: level 2 translation fault
    Data abort info:
      ISV = 0, ISS = 0x00000046
      CM = 0, WnR = 1
    Internal error: Oops: 96000046 [#1] PREEMPT SMP
    Modules linked in: hello scull faulty(O)
    pc : faulty_write+0x10/0x20 [faulty]
    lr : vfs_write+0xc4/0x290
    Call trace:
     faulty_write+0x10/0x20 [faulty]
     vfs_write+0xc4/0x290
     ksys_write+0x68/0xf4

## 3. Root Cause and Line Location Analysis
1. Virtual Address and WnR Flag:
   The message Unable to handle kernel NULL pointer dereference at virtual address 0000000000000000 along with WnR = 1 (Write not Read) indicates the kernel attempted to write to memory address 0x0.

2. Program Counter (pc) and Call Trace:
   The Program Counter (pc : faulty_write+0x10/0x20 [faulty]) pinpoints the exact crash offset at +0x10 bytes inside faulty_write in misc-modules/faulty.c:
       ssize_t faulty_write (struct file *filp, const char __user *buf, size_t count, loff_t *pos) {
           *(int *)0 = 0;
           return 0;
       }

3. Conclusion:
   Line *(int *)0 = 0; explicitly dereferences address 0 and attempts to write 0 to it, causing the MMU translation fault and triggering the Kernel Oops.
