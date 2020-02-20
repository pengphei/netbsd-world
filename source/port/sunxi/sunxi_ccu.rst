############################################
sunxi 移植 ccu 时钟控制驱动说明
############################################

在 NetBSD 中进行全志 CCU 时钟控制单元进行移植需要建立文件包括（以 suniv f1c100s 为例）：

* suniv_f1c100s_ccu.c
* suniv_f1c100s_ccu.h

sunxi ccu 整体架构
======================================


sunxi ccu 中间层
---------------------------------------

* sunxi_ccu
* sunxi_ccu_div
* sunxi_ccu_fixed_factor
* sunxi_ccu_fractional
* sunxi_ccu_gate
* sunxi_ccu_nkmp
* sunxi_ccu_nm
* sunxi_ccu_phase
* sunxi_ccu_prediv

在 ``struct sunxi_ccu_clk`` 中可以定义的时钟类型目前包括以下几种：

SUNXI_CCU_UNKNOWN, 未知类型
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_GATE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_GATE 表示门类型，仅用于使能/失能相关时钟配置。

* 具体的接口实现参考 sunxi_ccu_gate.c 文件
* 具体的宏定义参考 sunxi_ccu.h 文件，包括类型定义，以及宏定义如下：

  .. code-block:: c

    struct sunxi_ccu_gate {
        bus_size_t	reg;    // 寄存器地址
        uint32_t	mask;   // 寄存器标志位偏移
        const char	*parent;
    };
    
    #define	SUNXI_CCU_GATE(_id, _name, _pname, _reg, _bit)		\
    [_id] = {						\
        .type = SUNXI_CCU_GATE,				\
        .base.name = (_name),				\
        .base.flags = CLK_SET_RATE_PARENT,		\
        .u.gate.parent = (_pname),			\
        .u.gate.reg = (_reg),				\
        .u.gate.mask = __BIT(_bit),			\
        .enable = sunxi_ccu_gate_enable,		\
        .get_parent = sunxi_ccu_gate_get_parent,	\
    }
* 具体的例子参考 sun4i_a10_ccu.c 中的 ``AHB_GATING_REG0`` ，也就是全志 Allwinner A10 User manual 芯片手册中的 **6.4.16.AHB Module Clock Gating Reg0** 中的定义。

  .. code-block:: c
        
        /* AHB_GATING_REG0 */
        SUNXI_CCU_GATE(A10_CLK_AHB_OTG, "ahb-otg", "ahb",
            AHB_GATING_REG0, 0),

  * ``AHB_GATING_REG0`` 指代相应的寄存器，即 **0x060**。
  * ``0`` 指代 **Gating AHB Clock for USB0(0: mask, 1: pass).** 表明的 USB0 的时钟门偏移位。

SUNXI_CCU_NM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_NM 通常用于分频寄存器，其分频输出的时钟频率为：

.. code-block:: c
   output = source/n/m


* 根据 ``flag`` 不同字段表示，最终的输出分频频率为：

  1. 仅标记位 `SUNXI_CCU_NM_POWER_OF_TWO` :

     .. code-block:: c

        output = source/(1<<n)/m
    
  2. 仅标记位 `SUNXI_CCU_NM_DIVIDE_BY_TWO` :
    
     .. code-block:: c

        output = source/n/(m*2)

* 具体的接口实现参考 sunxi_ccu_nm.c 文件
* 具体的宏定义参考 sunxi_ccu.h 文件，包括类型定义，以及宏定义如下：

  .. code-block:: c

    struct sunxi_ccu_nm {
        bus_size_t	reg;
        const char	**parents;
        u_int		nparents;
        uint32_t	n;
        uint32_t	m;
        uint32_t	sel;
        uint32_t	enable;
        uint32_t	flags;
    #define	SUNXI_CCU_NM_POWER_OF_TWO	__BIT(0)
    #define	SUNXI_CCU_NM_ROUND_DOWN		__BIT(1)
    #define	SUNXI_CCU_NM_DIVIDE_BY_TWO	__BIT(2)
    };
    
    #define	SUNXI_CCU_NM(_id, _name, _parents, _reg, _n, _m, _sel,	\
            _enable, _flags)				\
    [_id] = {						\
        .type = SUNXI_CCU_NM,				\
        .base.name = (_name),				\
        .u.nm.reg = (_reg),				\
        .u.nm.parents = (_parents),			\
        .u.nm.nparents = __arraycount(_parents),	\
        .u.nm.n = (_n),					\
        .u.nm.m = (_m),					\
        .u.nm.sel = (_sel),				\
        .u.nm.enable = (_enable),			\
        .u.nm.flags = (_flags),				\
        .enable = sunxi_ccu_nm_enable,			\
        .get_rate = sunxi_ccu_nm_get_rate,		\
        .set_rate = sunxi_ccu_nm_set_rate,		\
        .set_parent = sunxi_ccu_nm_set_parent,		\
        .get_parent = sunxi_ccu_nm_get_parent,		\
    }
* 具体的例子参考 sun4i_a10_ccu.c 中的 ``APB1_CLK_DIV_REG`` ，也就是全志 Allwinner A10 User manual 芯片手册中的 **6.4.14.APB1 Clock Divide Ratio** 中的定义。

  .. code-block:: c
        
        SUNXI_CCU_NM(A10_CLK_APB1, "apb1", apb1_parents,
            APB1_CLK_DIV_REG,		/* reg */
            __BITS(17,16),		/* n */
            __BITS(4,0),		/* m */
            __BITS(25,24),		/* sel */
            0,				/* enable */
            SUNXI_CCU_NM_POWER_OF_TWO),

  * ``apb1_parents`` 字段，表示可以设置的时钟源，根据用户手册描述，其可以设置的时钟源如下表：

    .. list-table:: 时钟源对应表
       :widths: 15 10 30
       :header-rows: 1

       * - apb1_parents 数组
         - APB1_CLK_SRC_SEL描述 
         - 数值对应
       * - osc24m
         - OSC24M
         - 0
       * - pll_periph
         - PLL6 (set to 1.2GHz)
         - 1
       * - losc
         - 32KHz
         - 2
    对于 ``apb1_parents`` ，其在代码中的定义如下，其中各个定义的数序，必须与描述相一致：

    .. code-block:: c
       
       static const char *apb1_parents[] = { "osc24m", "pll_periph", "losc" };

  * ``reg`` 字段，设置的 ``APB1_CLK_DIV_REG`` 指代相应的寄存器，即 **0x058**。
  * ``n`` 字段，设置的 ``__BITS(17,16)`` 表示寄存器描述中的 **CLK_RAT_N** 。
  * ``m`` 字段，设置的 ``__BITS(4,0)`` ，表示寄存器描述中的 **CLK_RAT_M** 。
  * ``sel`` 字段，设置为 ``__BITS(25,24)``，表示寄存器描述中的 **APB1_CLK_SRC_SEL**，用于选择对应的时钟源。
  * ``enable`` 字段，设置为 ``0``，因为对于该寄存器，其不具有 `使能` 设置位，所以设置为0，否则应该设置为对应的使能位。
  * ``flag`` 字段，设置为 ``SUNXI_CCU_NM_POWER_OF_TWO`` ，因为根据 **CLK_RAT_N** 中的描述，其设置的值在进行处理时，需要用2的指数，即 ``2^n`` 。


SUNXI_CCU_NKMP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_NKMP 与 SUNXI_CCU_NM 类似，也用于分频寄存器，但是计算方法不同，其分频输出的时钟频率为：

.. code-block:: c

   output = (source*n*k)/(m*p)


* 根据 ``flag`` 不同标志位，最终的输出分频频率为：

  1. 仅标记位 `SUNXI_CCU_NKMP_DIVIDE_BY_TWO` :

     .. code-block:: c

        output = (source*n*k)/((m*2)*p)
    
  2. 仅标记位 `SUNXI_CCU_NKMP_FACTOR_N_EXACT` :
    
     .. code-block:: c

        output = (source*(n+1)*k)/(m*p)

  3. 仅标记位 `SUNXI_CCU_NKMP_SCALE_CLOCK` :
    
     该标志位不影响最终的时钟分频结果，但是会影响不同标志位设置的时间顺序，以一定的时间顺序设置更新相应的标志位。

  4. 仅标记位 `SUNXI_CCU_NKMP_FACTOR_P_POW2` :
    
     .. code-block:: c

        output = (source*n*k)/(m*(1<<p))

  5. 仅标记位 `SUNXI_CCU_NKMP_FACTOR_N_ZERO_IS_ONE` :
    
     .. code-block:: c

        n = (0==n) ? 1：n;
        output = (source*n*k)/(m*p)

  6. 仅标记位 `SUNXI_CCU_NKMP_FACTOR_P_X4` :
    
     .. code-block:: c

        p = p ? 4：1;
        output = (source*n*k)/(m*p)

  7. 仅标记位 `SUNXI_CCU_NKMP_MULTIPLY_BY_TWO` :
    
     .. code-block:: c

        output = (source*(n*2)*k)/(m*p)


* 具体的接口实现参考 sunxi_ccu_nkmp.c 文件
* 具体的宏定义参考 sunxi_ccu.h 文件，包括类型定义，以及宏定义如下：

  .. code-block:: c

    struct sunxi_ccu_nkmp_tbl {
        u_int		rate;
        uint32_t	n;
        uint32_t	k;
        uint32_t	m;
        uint32_t	p;
    };

    struct sunxi_ccu_nkmp {
        bus_size_t	reg;
        const char	*parent;
        uint32_t	n;
        uint32_t	k;
        uint32_t	m;
        uint32_t	p;
        uint32_t	lock;
        uint32_t	enable;
        uint32_t	flags;
        const struct sunxi_ccu_nkmp_tbl *table;
    #define	SUNXI_CCU_NKMP_DIVIDE_BY_TWO		__BIT(0)
    #define	SUNXI_CCU_NKMP_FACTOR_N_EXACT		__BIT(1)
    #define	SUNXI_CCU_NKMP_SCALE_CLOCK		__BIT(2)
    #define	SUNXI_CCU_NKMP_FACTOR_P_POW2		__BIT(3)
    #define	SUNXI_CCU_NKMP_FACTOR_N_ZERO_IS_ONE	__BIT(4)
    #define	SUNXI_CCU_NKMP_FACTOR_P_X4		__BIT(5)
    #define	SUNXI_CCU_NKMP_MULTIPLY_BY_TWO		__BIT(6)
    };
    
    #define	SUNXI_CCU_NKMP_TABLE(_id, _name, _parent, _reg, _n, _k, _m, \
		       _p, _enable, _lock, _tbl, _flags)	\
        [_id] = {						\
            .type = SUNXI_CCU_NKMP,				\
            .base.name = (_name),				\
            .u.nkmp.reg = (_reg),				\
            .u.nkmp.parent = (_parent),			\
            .u.nkmp.n = (_n),				\
            .u.nkmp.k = (_k),				\
            .u.nkmp.m = (_m),				\
            .u.nkmp.p = (_p),				\
            .u.nkmp.enable = (_enable),			\
            .u.nkmp.flags = (_flags),			\
            .u.nkmp.lock = (_lock),				\
            .u.nkmp.table = (_tbl),				\
            .enable = sunxi_ccu_nkmp_enable,		\
            .get_rate = sunxi_ccu_nkmp_get_rate,		\
            .set_rate = sunxi_ccu_nkmp_set_rate,		\
            .get_parent = sunxi_ccu_nkmp_get_parent,	\
        }

    #define	SUNXI_CCU_NKMP(_id, _name, _parent, _reg, _n, _k, _m,	\
                _p, _enable, _flags)			\
        SUNXI_CCU_NKMP_TABLE(_id, _name, _parent, _reg, _n, _k, _m, \
                    _p, _enable, 0, NULL, _flags)

* 具体的例子参考 sun4i_a10_ccu.c 中的 ``APB1_CLK_DIV_REG`` ，也就是全志 Allwinner A10 User manual 芯片手册中的 **6.4.14.APB1 Clock Divide Ratio** 中的定义。

  .. code-block:: c
        
        SUNXI_CCU_NM(A10_CLK_APB1, "apb1", apb1_parents,
            APB1_CLK_DIV_REG,		/* reg */
            __BITS(17,16),		/* n */
            __BITS(4,0),		/* m */
            __BITS(25,24),		/* sel */
            0,				/* enable */
            SUNXI_CCU_NM_POWER_OF_TWO),

  * ``apb1_parents`` 字段，表示可以设置的时钟源，根据用户手册描述，其可以设置的时钟源如下表：

    .. list-table:: 时钟源对应表
       :widths: 15 10 30
       :header-rows: 1

       * - apb1_parents 数组
         - APB1_CLK_SRC_SEL描述 
         - 数值对应
       * - osc24m
         - OSC24M
         - 0
       * - pll_periph
         - PLL6 (set to 1.2GHz)
         - 1
       * - losc
         - 32KHz
         - 2
    对于 ``apb1_parents`` ，其在代码中的定义如下，其中各个定义的数序，必须与描述相一致：

    .. code-block:: c
       
       static const char *apb1_parents[] = { "osc24m", "pll_periph", "losc" };

  * ``reg`` 字段，设置的 ``APB1_CLK_DIV_REG`` 指代相应的寄存器，即 **0x058**。
  * ``n`` 字段，设置的 ``__BITS(17,16)`` 表示寄存器描述中的 **CLK_RAT_N** 。
  * ``m`` 字段，设置的 ``__BITS(4,0)`` ，表示寄存器描述中的 **CLK_RAT_M** 。
  * ``sel`` 字段，设置为 ``__BITS(25,24)``，表示寄存器描述中的 **APB1_CLK_SRC_SEL**，用于选择对应的时钟源。
  * ``enable`` 字段，设置为 ``0``，因为对于该寄存器，其不具有 `使能` 设置位，所以设置为0，否则应该设置为对应的使能位。
  * ``flag`` 字段，设置为 ``SUNXI_CCU_NM_POWER_OF_TWO`` ，因为根据 **CLK_RAT_N** 中的描述，其设置的值在进行处理时，需要用2的指数，即 ``2^n`` 。

SUNXI_CCU_PREDIV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_PREDIV 与 SUNXI_CCU_NM 类似，也用于分频寄存器，但是计算方法不同，其分频输出的时钟频率为：

.. code-block:: c

   output = (source)/pre/div


* 具体的例子参考 sun8i_h3_ccu.c 中的 ``AHB1_APB1_CFG_REG`` ，也就是全志 Allwinner H3 Datasheet 芯片手册中的 **4.3.5.11 AHB1/APB1 Configuration register** 中的定义。

  .. code-block:: c
        
	SUNXI_CCU_PREDIV(H3_CLK_AHB1, "ahb1", ahb1_parents,
	    AHB1_APB1_CFG_REG,	/* reg */
	    __BITS(7,6),	/* prediv */
	    __BIT(3),		/* prediv_sel */
	    __BITS(5,4),	/* div */
	    __BITS(13,12),	/* sel */
	    SUNXI_CCU_PREDIV_POWER_OF_TWO),


  * ``reg`` ，设置为 ``AHB1_APB1_CFG_REG`` ，表示 **AHB1_PRE_DIV** 寄存器设置
  * ``prediv``，设置为 ``__BITS(7,6)`` ，表示 **AHB1_PRE_DIV** 寄存器设置
  * ``prediv_sel``，设置为 ``__BIT(3)``，表示 **** 寄存器设置。
  * ``div`` 设置为 ``__BITS(5,4)``，表示 **AHB1_CLK_DIV_RATIO** 。
  * ``sel`` 设置为 ``__BITS(13,12)`` 表示 **AHB1_CLK_SRC_SEL**，分别对应于 ``ahb1_parents`` 数组相关内容：
    
    .. code-block:: c
       
       static const char *ahb1_parents[] = { "losc", "hosc", "axi", "pll_periph0" };

  * ``flag`` 设置为 ``SUNXI_CCU_PREDIV_POWER_OF_TWO`` 用于声明 ``div`` 字段对应的数值，在设置时必须为2的指数。
  
SUNXI_CCU_DIV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_PHASE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_FIXED_FACTOR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SUNXI_CCU_FRACTIONAL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^