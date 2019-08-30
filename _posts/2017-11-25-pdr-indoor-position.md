---
layout: post
author: Robin
title: 行人航位推算（PDR）与室内定位技术综述
tags: 室内定位
categories:
  - 室内定位 
---

<div class="article-message">
                                <br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/b0786ba709e6ab87cd48d8c355501a57.png" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/b0786ba709e6ab87cd48d8c355501a57.png" class="img-polaroid" title="1447663276491509.png" alt="1447663276491509.png"></a>
</div>
<br>
&nbsp;<br>
&nbsp;近年来，室内定位技术越来越受到人们的重视，基于室内定位技术的应用也越来越多，例如：监视病人在医院的位置、消防员在失事建筑物内的位置等等。<br>
&nbsp;<br>
室内定位技术也已经有了很大的技术积累，比如利用在不同位置的短距离信号（如WiFi、RFID、红外线等）的信号强度不同建立射频地图。通过检测信号强度的变化，利用三角定位法等确定人的位置。但是这些方法需要提前在建筑物内部署大量的信标节点（虽然现在这已经不是问题。），而且信号很容易受到环境的干扰和多径效应等。<br>
&nbsp;<br>
相反的，行人航位推算（Pedestrian Dead Reckoning，PDR）系统无需在建筑物内预装信标节点，利用惯性传感器（如加速度传感器、陀螺仪、磁力计等）计算人的步长和方向，即推测出行人在建筑物内的踪迹。<br>
&nbsp;<br>
<strong>1. PDR系统概述</strong><br>
&nbsp;<br>
PDR系统所用的是<strong>PDR算法</strong>，如下图所示，它是一种相对定位算法。<br>
<br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/f8e64b15af16ed376f62e7e81280b0d1.jpg" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/f8e64b15af16ed376f62e7e81280b0d1.jpg" class="img-polaroid" title="pdr_argolthm.jpg" alt="pdr_argolthm.jpg"></a>
</div>
<br>
&nbsp;<br>
假设已知初始位置信息 (x1,y1)，推算下一点位置信息 (x2,y2)，推算公式为：<br>
<br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/86111fa83668f052e776c741ed03d878.jpg" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/86111fa83668f052e776c741ed03d878.jpg" class="img-polaroid" title="2015-12-1314-M6-7.jpg" alt="2015-12-1314-M6-7.jpg"></a>
</div>
<br>
&nbsp;<br>
同理可得往后每一步的递推公式。由递推公式可知，整个计算过程中有两个关键因素：行走位移S和方向角θ。位移S可以通过典型的<strong>步频-步长模型</strong>来估算，方向角可以通过方向传感器直接获取或者通过与陀螺仪两者组合并结合地图约束信息来得到。<br>
&nbsp;<br>
&nbsp;<br>
人体的自然行走运动包括前向、侧向以及垂直向3个分量，其3个分量以及手机坐标轴的定义如上图所示，将手机屏幕朝上水平放置在手掌中，3个运动分量与手机坐标轴的对应关系为：垂直轴与Z轴重合，前向轴与Y轴重合，侧向轴与X轴重合。步态检测中常用的一种算法是<strong>波峰检测算法</strong>，其优点是算法实现简单、计算量小，可以方便地做到实时检测。但手机传感器的硬件设备精度不高，以及行人行走状态的随机变化造成采集到的加速度信号含有噪声，使得波峰检测算法精确度不高。目前并没有一种非常准确的步态识别算法，多数情况下都是在信号上下功夫，进行特征分析处理等。<br>
&nbsp;<br>
<strong>PDR 系统的结构框架如下图：</strong><br>
<br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/d8474756ff056f1f1faf077583f3cd05.png" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/d8474756ff056f1f1faf077583f3cd05.png" class="img-polaroid" title="Screen_Shot_2017-11-24_at_2.22_.29_PM_.png" alt="Screen_Shot_2017-11-24_at_2.22_.29_PM_.png"></a>
</div>
<br>
&nbsp;<br>
<strong>2. 步态检测</strong><br>
&nbsp;<br>
步态检测是基于惯性传感器的行人定位系统中的模块之一，步态检测算法分类如下：<br>
&nbsp;<br>
<strong>&nbsp; &nbsp; 2.1&nbsp; 时域分析</strong><br>
&nbsp;<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;人在行走的时候，随身携带的智能手机中，加速度轨迹子啊时域上会呈现周期性的变化。通常情况下，会利用手机内的三轴加速度传感器记录加速度轨迹，再从加速度轨迹中检测阈值，当然在行走过程中，数据会有一些异常的抖动产生，如下图。时域分析方法除了阈值检测法之外，还有动态时间规整法等等。<br>
<br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/a3044fec31378b12a89b06634532a9ab.jpg" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/a3044fec31378b12a89b06634532a9ab.jpg" class="img-polaroid" title="pretreatment.jpg" alt="pretreatment.jpg"></a>
</div>
<br>
&nbsp;<br>
&nbsp;<br>
<strong>&nbsp; 2.2&nbsp;频域分析</strong><br>
&nbsp;<br>
&nbsp; &nbsp; &nbsp; &nbsp; 频域分析的原理是人行走时的频率稳定在 2 Hz左右，而其他行为的频率一般不在 2 Hz附近，利用这一特点，可以使用<strong>短期傅里叶变换（STFT）</strong>提取出人行为的频率，若在 2 Hz附近，则认为是跨了一步。<br>
&nbsp; &nbsp; &nbsp; &nbsp; 时域分析较为直观易懂，缺点是时域上容易受到其他噪声的影响，而频域分析方法主要是利用行走时所特有的 2 Hz来区分行走的其他行为，但是不够直观，比如很难区分人抬脚和落脚两种行为，因为其频率差别很小。下图是从技术、计算成本、检测错误率方面总结比较了一些基于手机的步伐检测结果：<br>
&nbsp;<br>
&nbsp; &nbsp;*&nbsp; 文献1：&nbsp;&nbsp;<a href="http://ieeexplore.ieee.org/document/6817854/" rel="nofollow" target="_blank">An improved indoor localization method using smartphone inertial sensors</a><br>
&nbsp; &nbsp;*&nbsp; 使用的技术：&nbsp; 阈值检测<br>
&nbsp; &nbsp;*&nbsp; 计算成本：&nbsp; 较低<br>
&nbsp; &nbsp;*&nbsp; 检测错误率：2%左右<br>
&nbsp;&nbsp;<br>
&nbsp; &nbsp;*&nbsp; 文献2：&nbsp;&nbsp;<a href="https://dl.acm.org/citation.cfm?id=2370280" rel="nofollow" target="_blank">A reliable and accurate indoor localization method using phone inertial sensors</a><br>
&nbsp; &nbsp;*&nbsp; 使用的技术：&nbsp; 动态时间规整<br>
&nbsp; &nbsp;*&nbsp; 计算成本：&nbsp; 中等<br>
&nbsp; &nbsp;*&nbsp; 检测错误率：&lt; 2%<br>
&nbsp;<br>
&nbsp; &nbsp;*&nbsp; 文献3：&nbsp;&nbsp;<a href="https://dl.acm.org/citation.cfm?id=2493449" rel="nofollow" target="_blank">Walk detection and step counting on unconstrained smartphones</a><br>
&nbsp; &nbsp;*&nbsp; 使用的技术：&nbsp; 频域分析<br>
&nbsp; &nbsp;*&nbsp; 计算成本：&nbsp; 中等<br>
&nbsp; &nbsp;*&nbsp; 检测错误率：&lt; 2%<br>
&nbsp;<br>
<strong>3.&nbsp; 步长推算</strong><br>
由于每个人的身高、走路方式不同，所有每个人的步长也是不一样的，关于步长推算方面的总结如下：<br>
&nbsp;<br>
<strong>&nbsp; &nbsp; &nbsp;3.1 常数模型</strong><br>
<strong>&nbsp; &nbsp; &nbsp; &nbsp; </strong>推算步长最直观的方法就是将一段测得的行走距离除以步数得到的步数，得到平均步长，即认为步长是常数。但是由于实际上人在行走时的姿态会有所变化，所以步长也会改变。<br>
<br>
<strong>&nbsp; &nbsp; &nbsp;3.2&nbsp;线性频率模型<br>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;模型A：</strong><br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 文献<a href="https://dl.acm.org/citation.cfm?id=2370280" rel="nofollow" target="_blank">A reliable and accurate indoor localization method using phone inertial sensors</a>中，通过收集23个不同身高的人行走4 000步的数据，分析得到步长和频率呈现线性关系，提出了线性频率模型：<br>
<strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; L = a · f + b</strong><br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 其中，a 和 b 是通过大量的线下训练得到，此方法计算成本较小，但是计算精度也较低。<br>
<br>
<strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;模型B：</strong><br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 文献<a href="http://www.mdpi.com/1424-8220/12/7/8507" rel="nofollow" target="_blank">Step Length Estimation Using Handheld Inertial Sensors</a>&nbsp;中，提出基于步频和行人身高的步长推算模型：<br>
<strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;L = h ·（a·f + b）+ c</strong><br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 式中L是步长，h是身高，f是步频，K={a，b，c}是针对每个人的系数集合。实验结果表明该模型的步长推算错误率为5.7%，而计算成本依旧较低。<br>
&nbsp;<br>
<br>
<strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;模型C：<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;</strong>文献<a href="http://ieeexplore.ieee.org/document/6817854/" rel="nofollow" target="_blank">An improved indoor localization method using smartphone inertial sensors</a>中，提出基于步频和加速度方差的步长推算模型：<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/c64111bcae342b5db6dd1d4c7bab58ea.jpg" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/c64111bcae342b5db6dd1d4c7bab58ea.jpg" class="img-polaroid" title="1.jpg" alt="1.jpg"></a>
</div>
<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;式中f是步频，ν是每一步的加速度方差，K={α，β，γ}是每个人的系数集合。实验结果表明该模型步长推算精度较高，同时其计算成本也较大。<br>
<br>
<strong>&nbsp; &nbsp; &nbsp;3.3&nbsp;经验模型</strong><br>
&nbsp;<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;文献&nbsp;<a href="https://link.springer.com/article/10.1186/1687-6180-2014-65" rel="nofollow" target="_blank">Pedestrian dead reckoning for MARG navigation using a smartphone</a>&nbsp;中提出了一种经验模型：<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171124/2ce835b8c6c37cc302a041ba409a2a6b.jpg" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171124/2ce835b8c6c37cc302a041ba409a2a6b.jpg" class="img-polaroid" title="2.jpg" alt="2.jpg"></a>
</div>
<br>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;式中amax和amin分别是步态检测过程中的加速度最大值和最小值，C是比例系数，Tian Z等人采用了一种反向传播的神经网络来计算C的值，计算成本很高。<br>
&nbsp;<br>
&nbsp;<br>
&nbsp;<br>
<strong>4. 方向推算</strong><br>
&nbsp;<br>
已知了步长，还必须知道不行方向才能够计算出行人的位置。通常智能手机上都有数字罗盘，背后也就是磁力计和陀螺仪传感器，它能测出手机的Y轴投影到水平面时和地磁北极的夹角，即手机的方向角。但此方法存在地磁偏差和位置偏差。为了克服偏差，很多地方都提出融合其他的传感器，根据使用传感器类型不同，可以为了如下两类：<br>
&nbsp;<br>
<strong>&nbsp; &nbsp; &nbsp;4.1 融合惯性传感器</strong><br>
&nbsp; &nbsp; &nbsp; 由于智能手机里集成了很多惯性传感器（如加速度传感器、陀螺仪），它们可以和数字罗盘结合起来使用。例如用加速度传感器测得的加速度轨迹可以用来确定一类时间点，在这类时间点上的位置偏差和在起点人把手机放进衣袋后的位置偏差相同，这样只要测出在起点的位置偏差，再结合在每一步的推断点上测到的手机方向角，二者相加即为人走每一步时的行走方向。<br>
&nbsp;<br>
<strong>&nbsp; &nbsp; 4.2 融合照相机</strong><br>
&nbsp; &nbsp; &nbsp; 天花板的直线边缘可以作为参考来推算行人方向。文献&nbsp;<a href="https://dl.acm.org/citation.cfm?id=2493434" rel="nofollow" target="_blank">zero-configured heading acquisition for indoor mobile devices through multimodal context sensing</a>&nbsp;中先是利用计算机视觉技术从手机拍到的照片中提取出天花板边缘，再计算手机Y轴相对天花板边缘的方向偏差。由于建筑物水平界面大多是长方形的，所以天花板边缘相对建筑物水平或垂直，这时再测量建筑物的绝对方向，相当于天花板边缘的绝对方向，再结合前面手机相对天花板边缘的方向偏差，就能得到手机的方向。该方案能取得1°左右的精度，缺点是计算量巨大，耗能也很大。<br>
&nbsp;<br>
&nbsp;<br>
&nbsp;<br>
<strong>5.&nbsp;开放性研究问题</strong><br>
<br>
在行人航位推算应用于室内定位的过程中，依旧存在着一些研究问题。<br>
<br>
<strong>&nbsp; &nbsp;5.1 不同方案系统的融合</strong><br>
&nbsp; &nbsp;将几种定位技术结合起来使用可以有效提高定位的精度、可靠性，同时能节省能耗成本。而且，能根据人所在的环境以及定位需求的不同选择最合适的定位技术，从而实现无缝切换。比如WiFi指纹定位和行人航位推算相结合的定位。<br>
<br>
<strong>&nbsp; &nbsp;5.2 利用外部环境提高精度</strong><br>
&nbsp; &nbsp; 除了内部优化步伐探测、步长推算和方向推算算法，还可以借助外部环境提高定位精度，例如借助地标。在某个位置的手机传感器读数若有明显的特征，则认为该位置是一个地标。比如人乘坐电梯时手机的加速度传感器读数会有明显的特征，可以把电梯位置作为地标。<br>
<br>
<strong>6 结束语</strong><br>
<br>
行人航位推算系统（PDR）不需要在室内预装信标节点就能实现室内定位，跟踪行人轨迹。本文回顾了PDR系统中三个模块：步伐检测、步长推算、方向推算的各种算法方案，对它们进行了简单介绍和比较。最后列举了一些热门的开放性问题。<br>
&nbsp;<br>
<strong>7. 参考资料</strong><br>
&nbsp;<br>
<ol><li><strong><a href="http://www.chinaaet.com/article/3000014656" rel="nofollow" target="_blank">http://www.chinaaet.com/article/3000014656</a></strong></li><li><a href="http://journal.cqupt.edu.cn/jcuptnse/html/2016/1673-825X-28-2-233.html" rel="nofollow" target="_blank">一种改进的行人导航算法研究</a></li><li><a href="http://html.rhhz.net/CHXB/html/2015-12-1314.htm#outline_anchor_7" rel="nofollow" target="_blank">WiFi-PDR室内组合定位的无迹卡尔曼滤波算法</a></li><li><a href="http://d.wanfangdata.com.cn/Thesis/Y1816858" rel="nofollow" target="_blank">基于GPS和自包含传感器的行人室内外无缝定位算法研究</a></li></ol><br>
                              </div>