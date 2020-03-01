############################################
与 Linux 移植观察对比
############################################

移植 NetBSD 到这个 SoC 实现起来非常简单，需要填的 *坑* 很少。我以前也做过一些 Linux 移植，但是相比来说，NetBSD 系统的适配比 Linux 更容易一些。事实上，我发现 NetBSD 有几个优势，让它相比于 Linux 更容易移植到一个新的平台，尤其是对于那些 *新人*，比如我这种之前没有 NetBSD 移植经验的人。

* NetBSD 构建自己的 NetBSD 工具链，作为整体构建的一部分。对于 Linux，你需要获取或者构建适用于处理器架构的工具链，以及所使用的 C 库。尽管已经有许多有关这些的说明，但是这仍然是一大挑战。例如，使用了错误的 binutils 或者其他的迷之错误。对于 NetBSD，你只需要运行其内置的构建脚本，工具链就在那里。

* NetBSD 源码包含了整个系统源码 - 内核以及应用程序。


当然，NetBSD 也有一些缺点（一些重大的），特别是对于嵌入式系统。

*
* NetBSD 内核不支持将自身构建成一个自解压镜像（例如 Linux 的 zImage）。内核镜像可以构建为一个压缩镜像，但是需要引导管理器进行解压。当然，NetBSD 确实提供了一个工具 gzboot，它可以用于解压一个 gzip 压缩的内核到 RAM ，然后跳转到内核入口。者提供了与 Linux zImage 类似的功能（非常感谢 Allen Briggs 的 gzboot 方案。）。

Linux 和 NetBSD 都得益于开源系统带来的巨大优势。需要注意的是，在本文的介绍中提到的，由于其他人的工作，以及他们开放分享的精神，NetBSD 和 Linux 以及其他众多的开源系统都非常方便的适配到新的平台。最后，还有什么能够比看到你的控制台输出更让人兴奋的呢（调试信息，等等输出）：

 .. code-block:: raw

    NetBSD/evbarm (Vx115 VEP) booting...

    physmemory: 3840 pages at 0x24100000 -> 0x24ffffff

    Allocating page tables

    freestart = 0x24100000, free_pages = 512 (0x00000200)

    IRQ stack: p0x242f2000 v0xc01f2000

    ABT stack: p0x242f1000 v0xc01f1000

    UND stack: p0x242f0000 v0xc01f0000

    SVC stack: p0x242ee000 v0xc01ee000

    Creating L1 page table at 0x242fc000

    Mapping kernel

    pmap_map_chunk: pa=0x24300000 va=0xc0200000 size=0x183000 resid=0x183000 prot=0x3 cache=1

    SLLLLLLLLPPP

    pmap_map_chunk: pa=0x24483000 va=0xc0383000 size=0x56d000 resid=0x56d000 prot=0x3 cache=1

    PPPPPPPPPPPPPLLLLLLLSSSSLLLLLLLLLLLLLLL

    Constructing L2 page tables

    pmap_map_chunk: pa=0x242f2000 va=0xc01f2000 size=0x1000 resid=0x1000 prot=0x3 cache=1

    P

    pmap_map_chunk: pa=0x242f1000 va=0xc01f1000 size=0x1000 resid=0x1000 prot=0x3 cache=1

    P

    pmap_map_chunk: pa=0x242f0000 va=0xc01f0000 size=0x1000 resid=0x1000 prot=0x3 cache=1

    P

    pmap_map_chunk: pa=0x242ee000 va=0xc01ee000 size=0x2000 resid=0x2000 prot=0x3 cache=1

    PP

    pmap_map_chunk: pa=0x242fc000 va=0xc01fc000 size=0x4000 resid=0x4000 prot=0x3 cache=2

    PPPP

    pmap_map_chunk: pa=0x242fb000 va=0xc01fb000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242fa000 va=0xc01fa000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f9000 va=0xc01f9000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f8000 va=0xc01f8000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f7000 va=0xc01f7000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f6000 va=0xc01f6000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f5000 va=0xc01f5000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    pmap_map_chunk: pa=0x242f4000 va=0xc01f4000 size=0x1000 resid=0x1000 prot=0x3 cache=2

    P

    devmap: 70000000 -> 700fffff @ fd000000

    pmap_map_chunk: pa=0x70000000 va=0xfd000000 size=0x100000 resid=0x100000 prot=0x3 cache=0

    S

    devmap: fc000000 -> fc1fffff @ fc000000

    pmap_map_chunk: pa=0xfc000000 va=0xfc000000 size=0x200000 resid=0x200000 prot=0x3 cache=0

    SS

    freestart = 0x249f0000, free_pages = 1552 (0x610)

    switching to new L1 page table  @0x242fc000...done!

    bootstrap done.

    init subsystems: stacks vectors undefined page pmap done.

    Loaded initial symtab at 0xc038b3f4, strtab at 0xc03b81b0, # entries 11404

    abcdefg

    Copyright (c) 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005

    The NetBSD Foundation, Inc.  All rights reserved.

    Copyright (c) 1982, 1986, 1989, 1991, 1993

    The Regents of the University of California.  All rights reserved.

    pmap_postinit: Allocated 35 static L1 descriptor tables

    NetBSD 3.0.1 (VX115_VEP) #8: Tue Jan  2 19:17:21 EST 2007

        root@localhost.localdomain:/home/agere/vx115_netbsd/build/usr/src/sys/arch/evbarm/compile/obj/VX115_VEP

    total memory = 15360 KB

    avail memory = 5404 KB

    mainbus0 (root)

    cpu0 at mainbus0: ARM926EJ-S rev 5 (ARM9E-S core)

    cpu0: WB enabled EABT

    cpu0: 16KB/32B 4-way Instruction cache

    cpu0: 16KB/32B 4-way write-back-locking-C Data cache

    vx115_ahb0 at mainbus0

    vx115_apb0 at mainbus0

    vx115_clk0 at vx115_apb0 addr 0x700c5000-0x700c5067 intr 9

    vx115_pic0 at vx115_apb0 addr 0x700c1000-0x700c114b

    vx115_com0 at vx115_apb0 addr 0x700e2000-0x700e207b intr 19

    vx115_com0: major = 110: console

    md0: internal 5000 KB image area

    boot device: <unknown>

    root on md0a dumps on md0b

    mountroot: trying msdos...

    mountroot: trying ffs...

    root file system type: ffs

    WARNING: CHECK AND RESET THE DATE!

    init: copying out flags `-s' 3

    init: copying out path `/sbin/init' 11

    Jan  3 00:17:46 init: /etc/pwd.db: No such file or directory

    Enter pathname of shell or RETURN for /bin/sh:

    # ls

    bin     dev     etc     lib     libexec sbin    usr

    # ps

    ps: warning: /var/run/dev.db: No such file or directory

    PID TTY STAT    TIME COMMAND

    5 ?   Ss   0:00.13 -sh

    7 ?   R+   0:00.02 ps

    #
