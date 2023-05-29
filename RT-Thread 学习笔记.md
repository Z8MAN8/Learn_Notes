# RT-Thread 学习笔记

- 程序**内存分布**：

  - Code：代码段，存放程序的代码部分；
  - RO-data：只读数据段，存放程序中定义的常量；
  - RW-data：读写数据段，存放初始化为非 0 值的全局变量；
  - ZI-data：0 数据段，存放未初始化的全局变量及初始化为 0 的变量；

  stm32 上电默认从 **Flash** 启动，启动后将 RW 段中的 RW-data（初始化的全局变量）搬运到 **RAM** 中，另外根据编译器给出的 ZI 地址和大小分配出 ZI 段，并将这块 RAM 区域清零，剩余未使用的 RAM 空间做动态内存堆。

- 数值越小的优先级越高，0 为最高优先级。

- 时间片仅对**优先级相同**的**就绪态**线程有效。

- **空闲线程**完成最后的线程删除动作。

- 用户提供的栈首地址需做系统对齐。

- create动态创建的线程使用delete释放；init静态初始化的线程使用detach脱离

- 线程本身不应调用 `rt_thread_detach()` **脱离线程本身**。

-  `rt_thread_suspend()` 只能使用来挂起当前线程（即**自己挂起自己**），而且在挂起线程自己后，需要立刻调用 `rt_schedule()` 函数进行手动的线程上下文切换。

- **空闲线程**是一个线程状态**永远为就绪态**的线程，因此设置的钩子函数必须保证空闲线程在任何时刻都不会处于挂起状态，由于 **malloc、free** 等内存相关的函数内部使用了信号量作为临界区保护，因此在钩子函数内部也不允许调用此类函数！（基本上不允许调用系统 API)

- 函数指针的定义就是将“函数声明”中的“函数名”改成“（*指针变量名）”。但是这里需要注意的是：“（*指针变量名）”两端的括号不能省略，括号改变了运算符的优先级。如果省略了括号，就不是**定义函数指针**而是一个函数声明了，即声明了一个返回值类型为指针型的函数。

  - 函数指针的妙用 （其中**回调函数**讲的也很好）：[C语言--函数指针的用法总结](https://blog.csdn.net/faihung/article/details/80329925)

  - （RM通讯不同帧处理可以参考回调函数）

- 线程优先级通过维护一个掩码mask，优先级对应的位数上为1，32位处理器刚好32级优先级

- 通过**与**操作判断线程状态，高效





### SPI 相关

```c
/* spi 使用示例，主机发送 */
    /* 查找 spi 设备获取设备句柄 */
	rt_hw_spi_device_attach("spi3", "wspi", GET_PINS(0, 4));
    spi_dev = (struct rt_spi_device *)rt_device_find(name);
    if (!spi_dev)
    {
        rt_kprintf("spi sample run failed! can't find %s device!\n", name);
    }
    else
    {
			
        /* 方式1：使用 rt_spi_send_then_recv()发送命令读取ID */
        //rt_spi_send(spi_dev, send_buffer, sizeof(send_buffer));
        
		/* config spi */
        struct rt_spi_configuration cfg;
        cfg.data_width = 8;
        cfg.mode = RT_SPI_MODE_0 | RT_SPI_MSB; /* SPI Compatible: Mode 0 and Mode 3 */
        cfg.max_hz = 1 * 1000 * 1000; /* 1M */
        rt_spi_configure(spi_dev, &cfg);
			
			  struct rt_spi_message msg1, msg2;
        msg1.send_buf   = read_id;
        msg1.recv_buf   = RT_NULL;
        msg1.length     = 6;
        msg1.cs_take    = 1;
        msg1.cs_release = 0;
        msg1.next       = &msg2;

        msg2.send_buf   = RT_NULL;
        msg2.recv_buf   = RT_NULL;
        msg2.length     = 5;
        msg2.cs_take    = 0;
        msg2.cs_release = 1;
        msg2.next       = RT_NULL;

        rt_spi_transfer_message(spi_dev, &msg1);
				rt_kprintf("send sucessed\n");
    }
```

