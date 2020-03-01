############################################
构建 NetBSD 系统组成
############################################

我在一个 x86 Linux 系统上完成了大部分的开发工作，因此希望能够从 Linux 环境下交叉构建 NetBSD 系统。幸运的是，NetBSD 提供了一个强大的构建系统，支持交叉编译（构建一个与构建主机不同的处理器架构），以及跨平台构建（在非 NetBSD 构建系统上构建）。

构建过程包含几个步骤:

* 构建工具链
* 构建内核
* 构建用户层库和应用程序
* 构建根文件系统

NetBSD 带来了一个有趣的 shell 脚本，build.sh ，让这一切变得简单。

构建工具链
=======================================

构建工具链可以使用 build.sh 脚本完成：只需要在顶层源码目录执行

.. code-block ::

   ./build.sh -u -m evbarm tools


参数如下：

* ``-u`` : 在构建之前，不执行 ``make clean`` ；如果做了修改（对于首次构建，影响不大），这将会阻止重构所有内容。
* ``-m evbarm`` ：机器架构类型；这里用于 ARM 评估板。
* ``tools`` ：只构建工具链，而不是整个系统。

我确实发现了一个问题，在通过 Fedora Core 5 系统构建工具链时，由于本地 Fedora 工具（用于构建 NetBSD 工具链）版本太新(gcc 4.1)。NetBSD 工具链只能够使用 gcc 版本3 工具。幸运的是，Fedora 提供了可选的 GCC 3.2 工具链，安装方式如下：

.. code-block :: shell

   yum install compat-gcc-3.2

然后工具链构建过程如下：

.. code-block :: shell

   export HOST_CC=gcc32
   export HOST_CXX=g++32
   ./build.sh -u -m evbarm tools 

确保 gcc-3 工具可以构建工具链。需要注意，这些工具（以及 export 声明）在 NetBSD 工具链构建完之后就不需要了，他们只是用来实现工具链自举。同时，最新的 NetBSD 发行版应该可以通过 gcc 4 工具构建，因此这一步也不必要了。

构建内核
=======================================

构建 NetBSD 内核也可以通过 build.sh 脚本完成，同样也只需要在顶层源码目录执行：

.. code-block ::

   ./build.sh -u -m evbarm kernel=VX115_VEP

参数如下：

* ``-u`` 在构建之前，不执行 ``make clean`` ；如果做了修改，这将会阻止重构所有内容
* ``-m evbarm`` 机器架构类型；这里用于 ARM 评估板。
* ``kernel=VX115_VEP`` 指定构建使用的内核配置文件；这里使用的是 VX115_VEP 评估板。内核配置文件位于 sys/arch/evbarm/conf 。

上述命名将会执行以下几个操作：

* 生成构建目录 sys/arch/evbarm/compile/obj/VX115_VEP
* 在 VX115_VEP 配置文件上执行 nbconfig 配置工具，创建本次构建所需要的源文件。该配置指令打印很多东西；它实际上根据执行的系统和设备配置的自动配置框架，生成源文件。具体细将会在 ``_ 中更多讨论。
* 在源文件上执行 ``make depend`` 创建源文件依赖树。
* 执行 ``make`` 构建内核和驱动。

在命令完成后，将会在 :file:`sys/arch/evbarm/compile/obj/VX115_VEP` 目录下生成内核对象文件。这些文件包括 ELF 内核文件，以及配置文件中定义的其他不同格式的输出文件。

* netbsd : ELF 对象文件
* netbsd.bin : *netbsd* 转换后的纯二进制文件，用于直接加载到内存。
* netbsd.bin.srec : *netbsd.bin* 转换后的 S-record 格式，用于加载到 flash 。
* netbsd.gdb : 附带调试符号表的 ELF 对象文件（用于调试）


构建用户空间库和应用程序
=======================================

由于 NetBSD 发行版包含了所有的用户空间库和应用程序，build.sh 脚本也可以处理；只需要在源码顶层目录运行：

.. code-block ::

   ./build.sh -u -m evbarm

它将会创建 :file:`obj/destdir.evbarm` 目录，包含所有的用户空间文件。

构建根文件系统
=======================================

NetBSD 的默认构建，将会生成一个桌面系统的文件系统，对于嵌入式系统，将会非常庞大。我只需要一个支持控制台输出的最小的基本库和应用程序，放到一个单独的目录 :file:`rootfs_ramdisk`。（事实上，这虽然不是裸机最小化要求，但是也足够小，足以放到 ramdisk，所以我不需要进行进一步的裁剪。）

* rootfs_ramdisk

  * bin
    * [ cat chmod cp date df echo ed expr hostname kill ln ls mkdir mv ps pwd rm rmdir sh sleep stty sync test
  * dev
    * 自动生成的内容，后续讨论
  * etc
    * mkttys passwd.conf rc.conf rc.lkm rc.shutdown ttys group motd rc rc.local rc.subr
    * rc.d
      * local motd mountall root
  * lib
    * libcrypt.so  libcrypt.so.0  libcrypt.so.0.2  libc.so  libc.so.12  libc.so.12.128.2  libedit.so  libedit.so.2  libedit.so.2.9  libevent.so  libevent.so.0  libevent.so.0.2  libkvm.so  libkvm.so.5  libkvm.so.5.2  libm.so  libm.so.0  libm.so.0.2  libtermcap.so  libtermcap.so.0  libtermcap.so.0.5  libtermlib.so  libtermlib.so.0  libtermlib.so.0.5  libutil.so  libutil.so.7  libutil.so.7.6  libz.so  libz.so.0  libz.so.0.4
  * libexec
    * ld.elf_so
  * sbin
    * dmesg  fsck  fsck_ffs  init  init.shell  ldconfig  mknod  modload  modunload  mount  mount_ffs  mount_filecore mount_kernfs  mount_mfs  mount_null  mount_overlay  mount_procfs  mount_ufs  nologin  rndctl  savecore   sysctl ttyflags  umount
  * usr
    * libexec
      * getty