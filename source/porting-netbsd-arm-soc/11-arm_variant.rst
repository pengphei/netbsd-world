############################################
支持 ARM926EJ-S 差异
############################################

在这个 SoC 中使用的处理器是一个 ARM9 核心的衍生，即 ARM926EJ-S。为了支持这个衍生型号，我们需要对常规的 ARM 配置代码做一些修改。需要主要的是，这与提供衍生核心全部新特性的支持不同，我们只需要让系统能够将该处理器识别为 ARM9 型号即可。同时最新的 NetBSD 完全支持该处理器，因此这些工作将不再需要。

arch/arm/arm32/cpu.c
===========================

* 在 ``cpuids[]`` 数组中增加 CPU 识别码：

  .. code-block:: c

     { CPU_ID_ARM926EJS, CPU_CLASS_ARM9ES, "ARM926EJ-S", generic_steppings }

* 修改 `wtnames[14]` 最新内核版本号：

  .. code-block:: c

     "**unknown 14**" to "write-back-locking-C"

arch/arm/include/armreg.h
=======================================

* 增加 ARM9 CPU 架构类型类型定义标识符：

  .. code-block:: c

     CPU_ID_ARCH_V5TEJ   0x00060000

* 增加 CPU ID 定义：

  .. code-block:: c

     #define CPU_ID_ARM926EJS    0x41069260
