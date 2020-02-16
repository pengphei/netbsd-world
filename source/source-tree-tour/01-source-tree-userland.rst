*******************************************
第一部分 - 用户层
*******************************************

NetBSD 是这个星球上主要的开源操作系统之一。而且，它整个源代码都可以通过各种方式获取，比如FTP，SUP，rsync，匿名 CVS，并且许多供应商会通过 CD 光盘进行售卖，最常见的还是通过 NetBSD 操作系统。如果解压 NetBSD 源码，它将会导出几百兆的源码到 `/usr/src` 目录。这包括了用户层素有的源码，包括编译器，X 窗口系统，以及构建从 NetBSD 包集合(Package Collection)囊括的第三方软件的构建指令，当然也包括 NetBSD 内核。

在本节，我们将会简单介绍 NetBSD 源码树的用户层部分。第二部分将会介绍适用于应用软件开发者的库，而第三部分将会深度介绍内部部分。

这里介绍的所有文件和目录都位于 `/usr/src` ，这样我们可以不用每次都输入这个位置。例如，我们引用的 `games` 目录，你就清除我们说的是 `/usr/src/games` 。

现在让我们看看 `/usr/src` 里面有什么！

- Makefile:

   NetBSD 主 Makefile，尽量只基于合适的 NetBSD make 更新版本进行本地构建；这个 Makefile 包含了如何构建整个源码树，以及通过它安装一个工作系统的描述。两个有意思的目标是 `build` 和 `release` 。前一个编译所有的源码和库，并且将它们安装到系统，而 `make release` 将会创建发行版压缩文件，例如构建一个完整的 NetBSD 发布。

   有一些变量可以影响构建过程，其中最没影响的是 DESTDIR，它允许构建和安装新系统到一个不同的目录，这对于测试非常有用，例如一个 chroot 环境，测试构建是否正常，或者用于后续拷贝文件到 live 系统进行安装。另一个有意思的变量是 RELEASEDIR，用于告诉 `make relase` 安装的地方。

- build.sh：

   

- include:

- bin:

- sbin:

- usr.bin, usr.sbin:

- libexec:

- dist:

- gnu:

- crypto:

- games:

- distrib:

- etc:

- regress:

- share:

- lib:

- sys:

- xsrc:

- pkgsrc: