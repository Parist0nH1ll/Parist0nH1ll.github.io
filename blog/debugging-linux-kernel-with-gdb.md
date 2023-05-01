### Intro

In this blog, I will take a crash that Syzkaller found as an example:

```
WARNING: CPU: 0 PID: 185 at lib/iov_iter.c:629 _copy_from_iter+0x2b6/0xe80
```

Here is the function where the crash occurs:

```
size_t _copy_from_iter(void *addr, size_t bytes, struct iov_iter *i)
{
        if (WARN_ON_ONCE(!i->data_source))
                return 0;

        if (user_backed_iter(i))
                might_fault();
        iterate_and_advance(i, bytes, base, len, off,
                copyin(addr + off, base, len),
                memcpy(addr + off, base, len)
        )

        return bytes;
}
```

You can also find all the places that call this function using the command:

```
grep -rn _copy_from_iter --include \*.c
```

### Verify Crash

Build repro file

```
gcc -o crash crash.c
```

Run vulnerable kernal locally

```
qemu-system-x86_64 -m 2048 \
  -hda ./rootfs.ext2 \
  -kernel ./bzImage \
  -append "root=/dev/sda" \
  -serial stdio -curses \
  -net user,hostfwd=tcp::3333-:22 -net nic 
```

 And copy the repro binary

```
scp -P 3333 crash root@localhost:/root/
```

Login to the machine and run the PoC

```
ssh root@localhost -p 3333
```

### Debugging 

Run qemu with `-S ` means QEMU will hold from booting the kernel until a GDB instance is attached.

```
qemu-system-x86_64 -m 2048 \
  -hda ./rootfs.ext2 \
  -kernel ./bzImage \
  -append "root=/dev/sda nokaslr" \
  -serial stdio \
  -net user,hostfwd=tcp::3333-:22  -net nic \
  -S -gdb tcp::1234 -curses
```

Run the following command to start gdb and attach to the qemu machine

```
gdb vmlinux

(gdb) target remote :1234   # Attach to QEMU
(gdb) hb start_kernel  # Break point at the start of the kernal
(gdb) b _copy_from_iter # Break point at the crash function
(gdb) c  # Continue execution
```

When the break point is hit, we can execute commands below for testing

```
(gdb) p i->data_source # Print variable
(gdb) x/32x i # Print address value
(gdb) n # Execute the next line
(gdb) bt # Print stack trace
```

### Analysis

On normal execution, the memory that `i` points to is:

```
0xffff88800b1dfd90:	0x01010000	0xffff8880
```

On bug trigger, the memory become:

```
0xffff88800cdaf8d0:	0x01000000	0xdffffc00
```

Executing the next line, `WARN_ON_ONCE` macro will check the value of `i->data_source`. It will generate a warning message only once when the param is true. So this means it actually catch the error.

```
(gdb) n
629		if (WARN_ON_ONCE(!i->data_source))
(gdb) n
asm_exc_invalid_op () at ./arch/x86/include/asm/idtentry.h:568
568	DECLARE_IDTENTRY_RAW(X86_TRAP_UD,		exc_invalid_op);
```

Back trace:

```
#0  asm_exc_invalid_op () at ./arch/x86/include/asm/idtentry.h:568
#1  0xffffffff81eec0b6 in _copy_from_iter at lib/iov_iter.c:629
...
```

In this blog, I won't delve into how the code path was reached, but if any interesting findings arise in the future, I might write another blog post to analyze them.

### Useful Notes

Make sure `.config` file contains the following setting to include debug symbols

```
CONFIG_DEBUG_INFO=y
```

Some other gdb commands:

```
shell dmesg # Display kernel dmesg log in GDB shell
info function _copy_from_iter # Display function info
```
