# RT-Thread LVGL 触摸屏对接记录

LVGL(轻巧而多功能的图形库)是一个免费的开放源代码图形库，它提供创建具有易于使用的图形元素，精美的视觉效果和低内存占用的嵌入式GUI所需的一切。

RT-Thread 目前已经支持 LVGL，4.1.1 版本之后（含4.1.1）移植 LVGL 的时候，只需要完善`lv_conf.h` 、`lv_port_disp.c`、 `lv_port_indev.c` 这三个文件即可，这三个文件的作用在后面介绍。

开始移植之前，需要检查使用的显示和输入设备是否已经有驱动支持，如果没有驱动支持的话需要自行对接，不对接到 RT-Thread 的设备框架，直接驱动也是可以的。

## 准备工作

这里默认显示和触摸的驱动已经没有问题，这次移植我使用的触摸屏显示芯片为 st7796 ，触摸芯片为 gt911 。

首先可以直接复制一个已经对接好的 LVGL 文件作为模板，再在此基础上进行修改，例如 `rt-thread-master\bsp\stm32\stm32l475-atk-pandora\applications\lvgl` ，结合板卡情况修改其中的`lv_conf.h` 、`lv_port_disp.c`、 `lv_port_indev.c` 三个文件即可, `SConscript` 自行按需修改。

## LVGL 配置文件

`lv_conf.h` 为 lVGL 配置文件，其中需要配置一些显示器的主要参数，移植时至少要配置好一下三个选项

- `LV_HOR_RES_MAX` 显示器的水平分辨率。
- `LV_VER_RES_MAX` 显示器的垂直分辨率。
- `LV_COLOR_DEPTH` 颜色深度，其取值对应如下：
  - 8 - RG332
  - 16 - RGB565
  - 32 - (RGB888和ARGB8888)

## 对接显示接口

在`lv_port_disp.c`中对接 LVGL 的显示接口，根据已有的模板，最重要的是对接以下接口

```c
/*Flush the content of the internal buffer the specific area on the display
 *You can use DMA or any hardware acceleration to do this operation in the background but
 *'lv_disp_flush_ready()' has to be called when finished.*/
static void disp_flush(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
{
    /* color_p is a buffer pointer; the buffer is provided by LVGL */
    //在这里填入对应显示驱动的加载接口
    //示例：
    //lcd_load(area->x1, area->x2, area->y1, area->y2, color_p);

    /*IMPORTANT!!!
     *Inform the graphics library that you are ready with the flushing*/
    lv_disp_flush_ready(disp_drv);
}
```

在注释的地方填入自己显示驱动的对应接口即可，需要注意传入的参数顺序和类型。

还需要初始化 `lv_disp_buf_t` 和 `lv_disp_drv_t` 变量，具体可以查看：[Display interface — LVGL documentation](https://docs.lvgl.io/latest/en/html/porting/display.html)

## 对接触摸输入接口

在`lv_port_indev.c`中对接 LVGL 的输入接口，LVGL 支持多种类型的输入设备，例如触摸，键盘，编码器等，我这里使用的是触摸类型。

输入接口必须初始化 `lv_indev_drv_t` 变量，最重要的是实现其 `read_cb` 回调函数，从而获取触摸信息。其余具体配置可以查看：[Input device interface — LVGL documentation](https://docs.lvgl.io/latest/en/html/porting/indev.html)



对接完成以后，记得检查 Kconfig 以及 scons 的相关配置。可以运行 demo 进行测试基本功能，例如触摸点是否准确，点击滑动等是否正常，显示刷新是否正常等。

需要提醒大家注意的一个点：确保触摸和显示的坐标是否对应。

可以看出只要显示和输入设备的驱动没有问题，对接 LVGL 是很快的。那么行动起来，让 LVGL 运行起来吧！