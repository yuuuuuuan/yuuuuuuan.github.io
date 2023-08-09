---
layout: post
title:  "[verilog]设计:基于FPGA的VGA显示原理及其实现"
date:   2022-12-28 
categories: verilog
---
[源码在这里](https://github.com/yuuuuuuan/FPGA_VGA)	FPGA使用verilog语言设计的VGA模块

## 介绍

- **VGA（Video Graphics Array)**

视频图形阵列是IBM于1987年随PS/2机一起推出的一种使用模拟信号的视频传输标准，具有分辨率高、显示速率快、颜色丰富等优点，在彩色显示器领域得到了广泛的应用。不支持热插拔，不支持音频传输。

![image](/assets/v2-4316a4e0a54140040e5ad54c4a16bd61_1440w.png)

- **色彩原理**

由于人的肉眼有感知**红、绿、蓝**三种不同颜色的锥体细胞，因此色彩空间通常可以由三种基本色来表达。三种基色是相互独立的，任何一种基色都不能有其它两种颜色合成。**红绿蓝**是三基色，这三种颜色合成的颜色范围最为广泛。我们的RGB信号就是三基色的运用，对这三个信号赋予不同的数值，混合起来便是不同的色彩。

## 原理

- **VGA显示原理**

VGA显示图像使用扫描的方式，从第一行的第一个像素开始，逐渐填充，第一行第一个、第一行第二个、、、、第二行第一个、第二行第二个、、、、第n行最后一个。

![image](/assets/VGA.PNG)

- **VGA时序标准**

VGA有两个非常重要的信号，一个是**行同步信号(HSYNC)**，另一个是**场同步信号(VSYNC)**。通过这两个信号完成一帧图像的像素点扫描，在数字电路中，时序是我们进行程序设计的重要依据。

也就是当HSYNC的高电平来临，VGA显示器就知道，我要开始接收这一行要显示的像素信息的，当下一个高电平来临时，那么这一行的图像信息就传输完成了，其中真正要显示的图像信息就蕴含在这一周期中。 - 同理场同步也如此

VGA可以显示不同分辨率的图像，不同分辨率及帧率对应参数如下：

![image](/assets/100046083-84484-5.png)

## **代码实现**

初始化定义

```
//parameter define
parameter H_VALID = 10'd640 , 	//行有效数据
V_VALID = 10'd480 ; 				//场有效数据

parameter RED = 24'hFF0000, 	//红色
ORANGE = 24'hFC000, 			//橙色
YELLOW = 24'hFFFF00, 			//黄色
GREEN = 24'h00FF00, 			//绿色
CYAN = 24'h00FFFF, 				//青色
BLUE = 24'h0000FF, 				//蓝色
PURPPLE = 24'hFF00FF, 			//紫色
BLACK = 24'h000000, 			//黑色
WHITE = 24'hFFFFFF, 			//白色
GRAY = 24'h35141A;				//灰色

    parameter h_begin = 100;
    parameter v_begin = 300;
    parameter h_diff = 100;
    parameter v_diff = 240;
    parameter h_lose = 256;
    parameter v_lose = 224;
    parameter h_start = 240;
    parameter v_start = 134;
    parameter pass_diff = 200;
    parameter length = 20;
	 parameter	vidon = 1;
```

显示像素点:

```
if((pix_x >= h_diff)&&(pix_x <h_diff + length)&&(pix_y >= v_diff)&&(pix_y < v_diff + length)&&(state == 0))
        begin isbody <= 1; color <= 1; end
    else if((pix_x >= h_diff + pass_diff)&&(pix_x < h_diff + pass_diff + length)&&(pix_y >= v_diff)&&(pix_y < v_diff + length)&&(state == 0))
        begin isbody <= 1; color <= 2; end
    else if((pix_x >= h_diff + pass_diff*2)&&(pix_x < h_diff + pass_diff*2 + length)&&(pix_y >= v_diff)&&(pix_y < v_diff + length)&&(state == 0))
        begin isbody <= 1; color <= 3; end
    else if((pix_x >= h_all[9:0] )&&(pix_x < h_all[9:0] + length)&&(pix_y >= v_all[9:0])&&(pix_y < v_all[9:0] + length)&&(state != 0))
        begin isbody <= 1; color <= 3; end
    else if((pix_x >= h_all[19:10])&&(pix_x < h_all[19:10] +length)&&(pix_y >= v_all[19:10])&&(pix_y < v_all[19:10] + length)&&(num >= 2)&&(state != 0))
        begin isbody <= 1; color <= 1; end
    else if((pix_x >= h_all[29:20])&&(pix_x < h_all[29:20] + length)&&(pix_y >= v_all[29:20])&&(pix_y < v_all[29:20] + length)&&(num >= 3)&&(state != 0))
        begin isbody <= 1; color <= 5; end
    else if((pix_x >= h_all[39:30])&&(pix_x < h_all[39:30] + length)&&(pix_y >= v_all[39:30])&&(pix_y < v_all[39:30] + length)&&(num >= 4)&&(state != 0))
        begin isbody <= 1; color <= 1; end
    else if((pix_x >= h_all[49:40])&&(pix_x < h_all[49:40] + length)&&(pix_y >= v_all[49:40])&&(pix_y < v_all[49:40] + length)&&(num >= 5)&&(state != 0))
        begin isbody <= 1; color <= 5; end
    else if((pix_x >= h_all[59:50])&&(pix_x < h_all[59:50] + length)&&(pix_y >= v_all[59:50])&&(pix_y < v_all[59:50] + length)&&(num >= 6)&&(state != 0))
        begin isbody <= 1; color <= 1; end
    else if((pix_x >= h_all[69:60])&&(pix_x < h_all[69:60] + length)&&(pix_y >= v_all[69:60])&&(pix_y < v_all[69:60] + length)&&(num >= 7)&&(state != 0))
        begin isbody <= 1; color <= 5; end
    else if((pix_x >= h_all[79:70])&&(pix_x < h_all[79:70] + length)&&(pix_y >= v_all[79:70])&&(pix_y < v_all[79:70] + length)&&(num >= 8)&&(state != 0))
        begin isbody <= 1; color <= 1; end
    else if((pix_x >= h_all[89:80])&&(pix_x < h_all[89:80] + length)&&(pix_y >= v_all[89:80])&&(pix_y < v_all[89:80] + length)&&(num >= 9)&&(state != 0))
        begin isbody <= 1; color <= 5; end
    else if((pix_x >= h_all[99:90])&&(pix_x < h_all[99:90] + length)&&(pix_y >= v_all[99:90])&&(pix_y < v_all[99:90] + length)&&(num >= 10)&&(state != 0))
        begin isbody <= 1; color <= 1; end
```

