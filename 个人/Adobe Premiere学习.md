
# 参考
> [Premiere使用教程（一）新建项目和序列](https://blog.csdn.net/Root915/article/details/85693646)
> [Premiere使用教程（二）界面介绍](https://blog.csdn.net/Root915/article/details/85844860)
> [Adobe Premiere Pro CC入门到精通教程合集!](https://blog.csdn.net/qq_43636466/article/details/89037175)
> [Adobe：使用说明性字幕](https://helpx.adobe.com/cn/premiere-pro/using/working-with-captions.html)
> [Premiere2018怎么使用au给声音降噪?](https://www.jb51.net/softjc/594747.html)
> [「PR」视频剪辑，一键玩转给音频进行降噪处理，自动除去杂声](https://baijiahao.baidu.com/s?id=1644919985889915866&wfr=spider&for=pc)

# Adobe Premiere

1. 导入图片、视频或者音频，软件的左下角，把视频从文件管理器拖入，或者用adobe自带的媒体浏览器浏览文件，在右键菜单可以导入
2. 把导入的素材拖入右下角的轨道上，这里就可以完成视频的剪辑，拼接或者删减
3. 添加字幕，文件新建字幕选择开放式字幕，在左下方字幕窗口可以调整字幕位置，（一开始的pr2018有这个问题，更新2019后无这个问题：但是不要改变字幕的入点和出点，否则软件直接挂死），改变时间在右下方的剪辑窗口里用鼠标拖动字幕的左右边界，是下图1处，不是2，（一开始错误理解了：但是字幕的左右边界要和视频分段的分界重合，否则拖不动），把时间轴放大，操作也是很快捷的，字幕背景透明，有一个透明度，默认是100%，改成0%就可以了
![148](https://img-blog.csdnimg.cn/20200412191758658.png)
4. 按`Ctrl+M`或者菜单，文件，导出，媒体，导出视频的时候记得选H.264，导出为mp4格式，默认是avi，不仅文件大，而且清晰度低
5. 音频降噪，背景里有滋滋滋的声音，在轨道中选择音频，右键在Au中编辑，进入Au之后，全选音频，在效果组里添加自适应降噪，点击下面的应用可以试听效果，或者选则初始一段音频，点击捕捉噪声样本，然后全选音频，在效果组里选择降噪，点击应用可以试听效果，再给多段视频降噪时，可全选，右键进入Au中编辑，但此时不是所有文件都添加进Au，需要手动导入，对每个音频执行降噪，保持，再处理下一个音频文件
6. 视频分割，可利用时间轴左边的剃刀工具，或者右键制作子序列，通过标记出点和入点可以生成子序列视频，相当于分割效果，但这应该用于那种无限重复特效
7. Pr会会把视频输出尺寸默认为你第一个拖入轨道的视频或者图片尺寸，可通过菜单序列，序列设置，帧大小更改，图片可预先处理好尺寸再拖入Pr
8. 添加视频/音频轨道，默认有3条，在v3上方空白处，右键菜单，添加视频轨道

# 配置Adobe Premiere使用独显
可以看到默认使用核显，
![135](https://img-blog.csdnimg.cn/20200316222808282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
桌面右键打开NVIDIA控制面板，选择`管理3D设置`，选择`程序设置`，配置Pr使用独显，不要自动选择。经测试，这种方法失败，后来发现在新建工程的时候可以选择GPU，实锤了，
![140](https://img-blog.csdnimg.cn/20200406191125878.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
可以看到现在GPU1的利用率上来了，该独显干活的时候，它TM的干活啊，不然买电脑的时候就不要买带独显的，徒增功耗，方向错了。
![141](https://img-blog.csdnimg.cn/20200406191214312.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 转码
## handbrake

## VLC
打开VLC，点击左上角的【媒体】，选择【转换/保存】。
