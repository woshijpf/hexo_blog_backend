title: Android虚拟激光键盘设计及实现
date: 2015-05-18 19:24:43
categories: 技术
tags:
- Android
- LaserKeyboard
- Linux
---

![Android_VKB](http://7xj51c.com1.z0.glb.clouddn.com/android_vkb_header.jpeg)

## 简介

当第一次在网上看到虚拟激光键盘时，我真的被震惊到了，觉得这键盘好科幻好酷炫啊，完全就是电影里的未来高科技的赶脚啊。看到这么酷炫高达上的东西，其实心中早已按倷不住了，要不自己就动手DIY一个？于是，就在网上各种搜索查找相关的资料，找到一个写得很详细图文并茂的的一个教程，[低成本激光投射虚拟键盘的设计制作-上(原理和硬件)][robopeak_vkb_1]和[低成本激光投射虚拟键盘的设计制作-下(算法与实现)][robopeak_vkb_2]，我也在[RoboPeak公司][robopeak]的产品中看到了这款虚拟激光键盘的产品（其实这家公司是做国内开源硬件和机器人平台的公司，我觉得它们的很多产品都非常有创新性）。

![celluon](http://7xj51c.com1.z0.glb.clouddn.com/celluon.jpg)
当然我也在淘宝上搜过有关虚拟激光键盘的产品，网上在售的主要是韩国Celluon公司的产品（如上图所示，价格在800-1000左右）和国内深圳某公司生产的与之相似的产品（价格只有200-300），它们都是基于蓝牙连接技术的键盘，只要你的设备支持蓝牙与其连接，智能手机、平板电脑、台式电脑、笔记本电脑也都可以使用。总之，虚拟激光键盘技术已经很成熟了，并且蓝牙技术也使得平台兼容性得到很好地解决。

现在移动办公的概念也离我们越来越近，只要一个手机或平板连上WIFI，就能随时随地进种处理各种事务，但是对于移动办公来说，文本文件和图表文件的处理是主要的业务需求，而现在的手机中自带的触屏输入系统存在着输入速度慢、键盘按键小、误输率高等缺点，所以能不能将虚拟激光键盘和移动设备结合在一起，将虚拟激光键盘内置到移动设备中作为其内置的一个输入功能，解决移动设备中触屏输入的缺点呢？于是，我就决定做这么一个基于Android系统的虚拟激光键盘，但这只是一个功能模型，如果能够将各个部件缩小化，然后将其嵌入到移动设备中，上面的设想也就能够实现了。

## 基本原理
本设计的虚拟激光键盘主要由键盘图像投影模块、红外激光定位系统、广角摄像头图像采集系统和Android平板四部分组成。在任意的平整的漫反射表面，通过红色波段的激光器透射刻有键盘图像的光栅，投影出QWERTY式的标准键盘图像，当用户敲击投影出的键盘图像中的某个按键时，通过广角摄像头就能捕捉到键盘区域内的键盘图像信息，再利用图像处理技术识别出图像中用户所敲击键盘的位置坐标，然后根据位置坐标映射到预先测量保存好的键盘布局文件中由此来确定按键的键值，然后将按键的键值封装成Android系统中的按键事件发送到Android系统中，这样就能实现与Android系统自带的软键盘一样的输入功能。

在上面虚拟激光键盘的原理介绍中，有三个核心的问题:
1. 如何产生产生键盘画面？
2. 如何识别按键事件？
3. 如何模拟Android系统中按键事件
<!--more-->


**产生键盘画面**

如果你小时候也和我一样玩过下面这种激光笔的话，相信你就知道我们的键盘图案是怎么产生了
![laser_pen](http://7xj51c.com1.z0.glb.clouddn.com/laser_pen.jpg)
实际上在激光投影键盘产品中，这类画面往往是通过全息投影技术得到的。激光器通过照射先前保存有键盘画面的全息镜片的方式在目标平面上产生相应的画面。还好现在我们有万能的淘宝，我们可以买到这样的[激光键盘投射器][tb1]。Ok，键盘图案就可以通过这个模块很简单地实现了。
![激光键盘发射器](http://7xj51c.com1.z0.glb.clouddn.com/键盘投射器.jpg)

**识别按键事件**

由于键盘画面可以投射在任意的表面上，因此传统的靠物理按钮的手段自然是不可能的（否则也称不上虚拟键盘）。需要非接触的手段来检测。这里给出了几种途径，他们在技术上都是可行的:
1. **通过计算机视觉的方式，通过图像来识别**
通过摄像头捕捉键盘区域的画面并进行分析，判断出键盘输入事件。
2. **通过检测按键发出的声音来判断**
这里假设使用者在按键时会碰触桌面，产生一定的敲击声。通过检测该声音传播时间，可以进行定位。该	  方案在国外的一些研究机构已经实现。
3. **通过超声波雷达手段来判断**
通过发射超声波并检测反射波的传播时间查来检测目标物体（手指）的位置。

这3种方案国内外均有文献表明可以实现，不过相对来说计算机视觉的硬件较为简单，仅仅需要一个摄像头，因此这里我们采用这种方式。
下面这张图是我在Ubuntu系统中通过OpenCV实现虚拟激光键盘的截图
![一个按键检测](http://7xj51c.com1.z0.glb.clouddn.com/一个按键检测.png)

在左边“Source Frame”窗口中的图像中，我接触到桌面的手指指尖有一个白色光斑，我们就可以用通过视觉处理技术在视频画面中捕捉到这样的亮斑来确定用户的按键事件，那么这个是怎么实现的呢？
这里我们就需要另一个器件: [一字线型激光][tb2]
![一字线型激光](http://7xj51c.com1.z0.glb.clouddn.com/一字线型激光.jpg)
它与我们平时常见的激光笔有所不同，我们平时激光笔产生的是一个点，而我们这个一字线型激光它发出的激光产生的是一个平面。我们将线型激光安装平行于桌面，这样它产生的光线就会平行紧贴于桌面之上。此时若手指接近桌面，则会阻挡住激光的通路，产生反射，反射的光点画面会被图中摄像头拍摄到。我们前面的一张图中的白色光斑，正是安装了线激光器后被手指遮挡产生的反射效果。下面这张图就很形象地给出了白色光斑产生的原理: 
![亮斑产生原理](http://7xj51c.com1.z0.glb.clouddn.com/亮斑产生.gif)

**好了，现在我们也可以通过图像处理来获得用户按键的事件，那么我们怎么判断具体按下的是那个键呢？**
还是通过这张图来说明按键识别的过程
![一个按键检测1](http://7xj51c.com1.z0.glb.clouddn.com/一个按键检测1.png)

当我们在桌面上按下某个按键时，这是在摄像头中我们就能捕获到如上图中“Source Frame”窗口中的画面，其中我们按下的位置指尖将会有一个白色光斑，这个就是我们对这张图片最感兴趣的点，而图片中的其他信息我们并不需要，所以可以通过灰度化，阈值化，形态学等图像处理操作来获得我们最感兴趣的指尖，得到如“Result Frame”中的只包含指尖白色光斑的图像。但是，有时我们的图像中可能会有其他的干扰白色光斑，就像图中左侧的干扰光斑。不过，我们可以通过截取我们键盘所在区域作为我们的刚兴趣区域进行识别处理，例如图中“InterestRegion“窗口就是在原图中截取出来的键盘所在的矩形区域，并且我这里通过一个最小包络矩形包围了我们所检测到的指尖白色光斑，并将包络矩形的中心作为我们的指尖的坐标。

现在，我们可以得到了按键位置的指尖坐标了，那如何根据之间坐标来获得对应位置是那个按键呢？

在[低成本激光投射虚拟键盘的设计制作-上(原理和硬件)][robopeak_vkb_1]一文中，作者采用的是3D激光测距的原理来获得用户按键位置在世界空间坐标系中的实际坐标位置（世界空间坐标系就是图像处理中的一个专业词汇，它它描述的是我们实际空间中位置状态，有关该部分介绍请参考[图像坐标系、摄像机坐标系与世界坐标系的关系][image_axis]），然后用尺子测量实际投射键盘中每个按键在作者设定的坐标系中的坐标，并将其保存在一个映射表中（代码中是通过一个struct来实现）。当从图像中处理得到一个用户指尖在世界空间坐标系中的坐标后，通过在映射表中进行查找就能找到对应的按键键值。

我也对作者提到的3D激光测距原理进行了分析，但是我总觉得这个过程有点复杂（最起码我看他画的3D测距的原理图我就看得头都大了），所以我放弃了这种检测按键位置的方法。现在这种方法也是我有一天在使用Matlab进行摄像头标定时突然间想到的一个方法，既然我在获得指尖坐标是在图像坐标系中的像素坐标，那么我为什么不可以将该坐标映射到经过摄像头畸变校正后的键盘图像中每个按键的在图像坐标系中的坐标呢?有时让你很头疼的问题，解决办法就是这么简单。


![按键识别过程](http://7xj51c.com1.z0.glb.clouddn.com/按键识别过程.png)

其实这个方法通俗点讲就是: 如上图所示，我们按下的某个键时所产生的白色光斑在图像校正后的坐标是（20,25），而我们‘q’键在图像中的矩形区域的左上角坐标为（15,20），右下角的坐标为（25,30），那么我们判断可知指尖坐标位于‘q’键的矩形区域内，因此可以确定用户此时按下的键就是‘q’键。

到现在为止，我们已经能够识别到我们按下的什么按键了，下一步就是怎么向系统注入按键事件了。

**模拟Android系统中按键事件**

有关模拟Linux系统和Android系统的按键事件，我将在《Linux系统功能实现》和《移植到Android系统》中在分别做介绍。

**总体框架图**

![总体框架图](http://7xj51c.com1.z0.glb.clouddn.com/激光键盘流程.png)



## 模块器件选择
| 器件 | 核心参数 | 购买链接 | 图片 |
|--------|--------|-------|------|
|  USB广角摄像头   |    可拍摄角度 >= 150度    | [广角摄像头][tb3] | ![广角摄像头](http://7xj51c.com1.z0.glb.clouddn.com/广角摄像头.jpg) |
|  激光键盘发射器   |    无特殊要求             | [激光键盘发射器][tb1] | ![激光键盘发射器](http://7xj51c.com1.z0.glb.clouddn.com/键盘投射器.jpg) |
|  一字线型激光     |    红外激光器    |  [一字线型激光][tb2] | ![一字线型激光](http://7xj51c.com1.z0.glb.clouddn.com/一字线型激光.jpg) |
|  红外带通滤光片   |    800nm左右光谱带通   | [红外带通滤光片][tb4] | ![红外滤光片](http://7xj51c.com1.z0.glb.clouddn.com/滤光片.jpg) |


## 技术路线

总体技术路线图
![Android虚拟激光键盘技术路线图](http://7xj51c.com1.z0.glb.clouddn.com/Android虚拟激光键盘技术路线图.png)

**1. 硬件电路设计与制作**
制作稳压电路板，为键盘投射器，一字线型激光器和摄像头供电

**2. 实物模型制作**
使用PVC板和热熔胶制作虚拟激光键盘模型，将上述模型组装在一起

**3. 软件开发环境搭建**
我使用的是Ubuntu 14.04 64位系统中进行开发调试，安装好Android开发环境，并下载Android4OpenCV库，然后进行相应的配置搭建，最后完成整个项目开发环境的搭建。

**4. Linux系统功能实现**
在Ubuntu系统中使用OpenCV库实现Linux系统中的虚拟激光键盘的功能

**5. 移植到Android系统**
将Ubuntu系统中实现的虚拟激光键盘功能移植到Android系统中

## 具体实现

下面的文章将按照技术路线中的的4个方面来讲解实现过程

1. 硬件电路设计与制作和实物模型制作

2. 软件开发环境搭建

3. Linux系统功能实现

4. 移植到Android系统


## 参考:
[1]  [低成本激光投射虚拟键盘的设计制作-上(原理和硬件)][robopeak_vkb_1]
[2]  [低成本激光投射虚拟键盘的设计制作-下(算法与实现)][robopeak_vkb_2]
[3]  李延平，贺显云.基于OpenCV的激光投影虚拟键盘设计.计算机光盘软件与应用.2014,(21)
[4]  蔡睿妍，激光虚拟键盘的设计与实现.激光与红外.2012,42(8)
[5]  [图像坐标系、摄像机坐标系与世界坐标系的关系][image_axis]
[6]  [自制激光虚拟投影键盘][vkb_sina]

[robopeak_vkb_1]:http://www.csksoft.net/blog/post/lowcost.laserkbd_part1.html
[robopeak_vkb_2]:http://www.csksoft.net/blog/post/lowcost.laserkbd_part2.html
[robopeak]:http://www.robopeak.com
[tb1]:http://item.taobao.com/item.htm?spm=2013.1.20141002.4.jUiGXj&scm=1007.10009.6098.i20236940692&id=10243870309&pvid=879b9fb6-56c1-49b1-955a-2ea3cc753913
[tb2]:http://item.taobao.com/item.htm?spm=a1z09.2.9.64.4FcIi3&id=20236940692&_u=imi3dfvf487
[tb3]:http://item.taobao.com/item.htm?spm=a1z0k.7385961.1997985097.d4918997.tOVHRc&id=44356048924&_u=imi3dfvbb9f
[tb4]:http://item.taobao.com/item.htm?spm=a1z09.2.9.75.4FcIi3&id=8756969933&_u=imi3dfvaedf
[image_axis]:blog.csdn.net/jyc1228/article/details/4209814
[vkb_sina]:http://blog.sina.com.cn/s/blog_4ce0162301016079.html