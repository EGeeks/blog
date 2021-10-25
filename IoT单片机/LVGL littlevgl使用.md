# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [lvgl动画效果卡顿的原因](http://bbs.raindi.net/thread-23869-1-1.html)
> [LVGL 双缓冲](https://blog.csdn.net/C_ROOKIES/article/details/114163885)
> [LVGL基础教程 – LVGL 显示接口](https://www.xyhtml5.com/22595.html)
> [stm32 DMA2D使用中断LVGL,提高LVGL帧率](https://www.it610.com/article/1388268791955279872.htm)
> [LVGL | lvgl最新版本在STM32上的移植使用](https://blog.csdn.net/zhengnianli/article/details/113931569)
> [lvgl - 移植文件系统](https://www.jianshu.com/p/d0faa09598e6)

# 字体
> [分享的在littlevgl 调用freetye显示文字, 而不用通过字模软件生成](https://whycan.cn/t_1496.html)
> [Online TTF to C Array Unicode Font Converter](https://littlevgl.com/ttf-font-to-c-array)
> [Littlevgl 显示汉字](https://blog.csdn.net/xinxiaoci/article/details/86136793)
> [LittlevGL中使用FreeType问题](https://blog.csdn.net/weixin_41863685/article/details/90717602)

How to use the generated fonts in LittlevGL?
1. Copy the result C file into your LittlevGL project
2. In a C file of your application declare the font as: extern lv_font_t my_font_name; or simply LV_FONT_DECLARE(my_font_name);
3. Set the font in a style: style.text.font = &my_font_name;

# 在linux上使用lvgl
> [littlevgl用fb形式移植](https://blog.csdn.net/xujun3614/article/details/82898079)
> [LVGL的linux_frame_buffer项目加入FB双缓](https://whycan.com/t_5887.html)
