# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux DRM Graphic 显示简单介绍](https://blog.csdn.net/yangkuanqaz85988/article/details/48657615)
> [linux DRM driver 使用示例](https://blog.csdn.net/u012839187/article/details/105833584)
> [DRM实例教程](https://www.jianshu.com/p/f41f98a40455)
> [DRM 驱动程序开发（开篇）](https://blog.csdn.net/hexiaolong2009/article/details/89810355)
> [DRM 驱动程序开发（VKMS）](https://blog.csdn.net/hexiaolong2009/article/details/105180621)
> [最简单的DRM应用程序 （single-buffer）](https://blog.csdn.net/hexiaolong2009/article/details/83721242)
> [drm 驱动是如何创建 fb device 的](https://blog.csdn.net/jingxia2008/article/details/48804859)
> [Linux中的DRM 介绍](https://blog.csdn.net/u013165704/article/details/80599809)
> [Linux Graphic DRI 显示子系统 介绍1](https://blog.csdn.net/u013165704/article/details/80709536)
> [Xilinx DRM KMS driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842520/Xilinx+DRM+KMS+driver)
> [赛灵思xilinx平台drm分析](https://blog.csdn.net/u011414665/article/details/93713569)
> [Linux_GUI加速(1)——GUI系统概述](http://xilinx.eetrend.com/content/2019/100043524.html)
> [Linux_GUI加速(2)——Linux中的DRM-KMS分析](http://xilinx.eetrend.com/content/2019/100043772.html)
> [Linux_GUI加速(3)——加速模块设计](http://xilinx.eetrend.com/content/2019/100043879.html)
> [Getting simple output with Xilinx DRM KMS framework](https://forums.xilinx.com/t5/Embedded-Linux/Getting-simple-output-with-Xilinx-DRM-KMS-framework/td-p/892275)
> [RK3399 探索之旅 / Display 子系统 / 基础概念](https://blog.csdn.net/wuweidonggmail/article/details/111659481)
> [Rockchip DRM主驱动流程梳理](https://blog.csdn.net/gaoyang3513/article/details/82631607)
> [嵌入式linux hdmi分辨率,【Firefly3399Pro】rk3399pro在Framebuffer状态命令行模式中强制HDMI输出固定分辨率...](https://blog.csdn.net/weixin_31720909/article/details/116847869)
> [tools:modetest代码逻辑](https://blog.csdn.net/u012839187/article/details/105833584)

# 架构
![20150930192518294](https://img-blog.csdnimg.cn/20210102230809770.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
重要概念：
- Planes：图层，例如在 rockchip 平台里对应 SOC 内部 VOP 模块的 win 图层;
- CRTC：显示控制器，例如在 rockchip 平台里对应 SOC 内部的 VOP 模块;
- Encoder：输出转换器，指 RGB、LVDS、DSI、eDP、HDMI、CVBS、VGA 等显示接口;
- Connector：连接器，指 encoder 和 panel 之间交互的接口部分;
- Bridge：桥接设备，一般用于注册 encoder 后面另外再接的转换芯片，如 DSI2HDMI 转换芯片;
- Panel：泛指屏，各种 LCD、HDMI 等显示设备的抽象;

Xilinx DRM驱动架构，
![Xlnx DRM KMS](https://img-blog.csdnimg.cn/ee2bf3dc695d4f16b30240b26364d3e9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)
Xilinx DRM SDI Tx驱动架构，可以看到有Bridge这个桥接设备，但这个桥接设备无需任何操作，
![SDI Tx](https://img-blog.csdnimg.cn/4083ece08c674b1aad02854530dd3a90.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)

# 驱动
DRM驱动架构好像都是分辨率固定了，就那么几种，这也符合实际情况，比如某款屏幕支持的分辨率，可以在代码里写死，也可以放到eeprom中，以edid的格式，DRM驱动架构定义了一些默认的`drm_display_mode`在`drivers\gpu\drm\drm_edid.c`中，比如，
```bash
/*
 * Probably taken from CEA-861 spec.
 * This table is converted from xorg's hw/xfree86/modes/xf86EdidModes.c.
 *
 * Index using the VIC.
 */
static const struct drm_display_mode edid_cea_modes[] = {
	/* 0 - dummy, VICs start at 1 */
	{ },
	/* 1 - 640x480@60Hz 4:3 */
	{ DRM_MODE("640x480", DRM_MODE_TYPE_DRIVER, 25175, 640, 656,
		   752, 800, 0, 480, 490, 492, 525, 0,
		   DRM_MODE_FLAG_NHSYNC | DRM_MODE_FLAG_NVSYNC),
	  .vrefresh = 60, .picture_aspect_ratio = HDMI_PICTURE_ASPECT_4_3, },
	/* 2 - 720x480@60Hz 4:3 */
	{ DRM_MODE("720x480", DRM_MODE_TYPE_DRIVER, 27000, 720, 736,
		   798, 858, 0, 480, 489, 495, 525, 0,
		   DRM_MODE_FLAG_NHSYNC | DRM_MODE_FLAG_NVSYNC),
	  .vrefresh = 60, .picture_aspect_ratio = HDMI_PICTURE_ASPECT_4_3, },
...
}
```
这里只有常用的分辨率，可以直接用这里的，也可以生成自己的分辨率，`drm_display_mode `定义如下，
```c
#define DRM_MODE(nm, t, c, hd, hss, hse, ht, hsk, vd, vss, vse, vt, vs, f) \
	.name = nm, .status = 0, .type = (t), .clock = (c), \
	.hdisplay = (hd), .hsync_start = (hss), .hsync_end = (hse), \
	.htotal = (ht), .hskew = (hsk), .vdisplay = (vd), \
	.vsync_start = (vss), .vsync_end = (vse), .vtotal = (vt), \
	.vscan = (vs), .flags = (f), \
	.base.type = DRM_MODE_OBJECT_MODE

/**
 * struct drm_display_mode - DRM kernel-internal display mode structure
 * @hdisplay: horizontal display size
 * @hsync_start: horizontal sync start
 * @hsync_end: horizontal sync end
 * @htotal: horizontal total size
 * @hskew: horizontal skew?!
 * @vdisplay: vertical display size
 * @vsync_start: vertical sync start
 * @vsync_end: vertical sync end
 * @vtotal: vertical total size
 * @vscan: vertical scan?!
 * @crtc_hdisplay: hardware mode horizontal display size
 * @crtc_hblank_start: hardware mode horizontal blank start
 * @crtc_hblank_end: hardware mode horizontal blank end
 * @crtc_hsync_start: hardware mode horizontal sync start
 * @crtc_hsync_end: hardware mode horizontal sync end
 * @crtc_htotal: hardware mode horizontal total size
 * @crtc_hskew: hardware mode horizontal skew?!
 * @crtc_vdisplay: hardware mode vertical display size
 * @crtc_vblank_start: hardware mode vertical blank start
 * @crtc_vblank_end: hardware mode vertical blank end
 * @crtc_vsync_start: hardware mode vertical sync start
 * @crtc_vsync_end: hardware mode vertical sync end
 * @crtc_vtotal: hardware mode vertical total size
 *
 * The horizontal and vertical timings are defined per the following diagram.
 *
 * ::
 *
 *
 *               Active                 Front           Sync           Back
 *              Region                 Porch                          Porch
 *     <-----------------------><----------------><-------------><-------------->
 *       //////////////////////|
 *      ////////////////////// |
 *     //////////////////////  |..................               ................
 *                                                _______________
 *     <----- [hv]display ----->
 *     <------------- [hv]sync_start ------------>
 *     <--------------------- [hv]sync_end --------------------->
 *     <-------------------------------- [hv]total ----------------------------->*
 *
 * This structure contains two copies of timings. First are the plain timings,
 * which specify the logical mode, as it would be for a progressive 1:1 scanout
 * at the refresh rate userspace can observe through vblank timestamps. Then
 * there's the hardware timings, which are corrected for interlacing,
 * double-clocking and similar things. They are provided as a convenience, and
 * can be appropriately computed using drm_mode_set_crtcinfo().
 *
 * For printing you can use %DRM_MODE_FMT and DRM_MODE_ARG().
 */
struct drm_display_mode {
...
}
```

# 设备树
以HDMI为参考，
```bash
If hdcp1.4 is included in the design then key management block node should be
added to the device tree

	hdcp_keymngmt_blk_0: hdcp_keymngmt_blk_top@90000000 {
		clock-names = "s_axi_aclk", "m_axis_aclk";
		clocks = <&zynqmp_clk 71>, <&misc_clk_0>;
		compatible = "xlnx,hdcp-keymngmt-blk-top-1.0";
		reg = <0x0 0x90000000 0x0 0x10000>;
	};

	hdmi_acr_ctrl_0: hdmi_acr_ctrl@a0059000 {
		/* This is a place holder node for a custom IP, user may need to update the entries */
		compatible = "xlnx,hdmi-acr-ctrl-1.0";
		reg = <0x0 0xa0059000 0x0 0x1000>;
	};

	v_hdmi_tx_ss: v_hdmi_tx_ss@80080000 {
		compatible = "xlnx,v-hdmi-tx-ss-3.1";
		reg = <0x0 0xa0080000 0x0 0x80000>, <0x0 0xa0280000 0x0 0x10000>;
		reg-names = "hdmi-txss", "hdcp1x-keymngmt";
		interrupt-parent = <&gic>;
		interrupts = <0 91 4 0 106 4 0 107 4 0 110 4 0 111 4>;
		interrupt-names = "irq", "hdcp14_irq", "hdcp14_timer_irq", "hdcp22_irq", "hdcp22_timer_irq";

		clock-names = "s_axi_cpu_aclk", "link_clk", "s_axis_audio_aclk", "video_clk", "s_axis_video_aclk", "txref-clk", "retimer-clk";
		clocks = <&zynqmp_clk 71>, <&misc_clk_1>, <&zynqmp_clk 71>, <&misc_clk_2>, <&zynqmp_clk 72>, <&si5324 0>, <&dp159>;
		phy-names = "hdmi-phy0", "hdmi-phy1", "hdmi-phy2";
		phys = <&vphy_lane0 0 1 1 1>, <&vphy_lane1 0 1 1 1>, <&vphy_lane2 0 1 1 1>;

		xlnx,input-pixels-per-clock = <0x2>;
		xlnx,max-bits-per-component = <0xa>;
		xlnx,include-hdcp-1-4;
		xlnx,include-hdcp-2-2;
		/* settings for HDCP */
		xlnx,hdcp-authenticate = <0x1>;
		xlnx,hdcp-encrypt = <0x1>;
		xlnx,audio-enabled;
		xlnx,xlnx-hdmi-acr-ctrl = <&hdmi_acr_ctrl_0>;
		xlnx,snd-pcm = <&audio_ss_0_audio_formatter_0>;
		xlnx,vid-interface = <0x1>;

		ports {
			#address-cells = <1>;
			#size-cells = <0>;
			encoder_hdmi_port: port@0 {
				reg = <0>;
				hdmi_encoder: endpoint {
					remote-endpoint = <&dmaengine_crtc>;
				};
			};
		};
	};

	v_drm_dmaengine_drv: drm-dmaengine-drv {
		compatible = "xlnx,pl-disp";
		dmas = <&hdmi_output_v_frmbuf_rd_0 0>;
		dma-names = "dma0";
		xlnx,vformat = "BG24";
		#address-cells = <1>;
		#size-cells = <0>;
		dmaengine_port: port@0 {
			reg = <0>;
			dmaengine_crtc: endpoint {
				remote-endpoint = <&hdmi_encoder>;
			};
		};
	};
```

# modetest

