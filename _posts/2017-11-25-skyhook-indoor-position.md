---
layout: post
author: Robin
title: Skyhook Wi-Fi 定位技术杂谈
tags: 室内定位
categories:
  - 室内定位 
---

&nbsp;Wi-Fi定位是iPhone升级到1.1.3之后新加的应用服务，根据当时的<a href="https://www.macworld.com/article/1131616/iphone_update.html" rel="nofollow" target="_blank">报道</a>，打开地图后，加载了大约2秒钟，所在的街道就立刻闪现在屏幕中央了。难怪乔布斯也说：“It's really cool”，这个功能确实神奇呀。<br>
<br>
而做这项技术是由一家成立于2003年叫<a href="http://www.skyhookwireless.com/" rel="nofollow" target="_blank">Skyhook Wireless</a>的公司。Skyhook是一家位于波士顿的移动定位服务公司，专注于位置定位和场景感知技术。 Skyhook成立于2003年，最初以Wi-Fi接入点地理定位为基础，并发展为融合了Wi-Fi，GPS，手机信号塔，IP地址和设备传感器的混合定位技术可以改善设备位置。 Skyhook在2015年扩大了其产品组合，以提供广告细分和行为见解，并强调他们的广告技术和移动应用程序客户的数据私有化和安全性。 2016年，Skyhook设计了Precision Location，为可穿戴设备和物联网提供动力，并推出了产品Personas和Context Accelerator开发工具，帮助品牌销售商接触到移动消费者。<br>
&nbsp;<br>
&nbsp;<br>
<strong>历史</strong><br>
&nbsp;<br>
Skyhook是由Ted Morgan和Michael Shean于2003年创立的。 Skyhook的数据库早期是通过开着车满大街转悠，边走边采集AP信号，并用GPS定位，从而就有了坐标信息。，当时该公司派出了围绕美国，加拿大，西欧和亚洲某些国家的驾驶员队列出了Wi-Fi热点。 Skyhook曾经或者正在为苹果，三星，索尼，惠普，戴尔，夏普，飞利浦和MapQuest等公司提供基于位置的服务。 Skyhook在2007年获得了第一个专利，现在在美国和海外市场拥有400多项专利。 2014年2月，Skyhook Wireless被Liberty Broadband的子公司TruePosition Inc收购。在2016年，两家公司合并成为Liberty Media家族旗下Liberty Broadband旗下的Skyhook品牌。<br>
&nbsp;<br>
<strong>技术事实</strong><br>
<br>
Wi-Fi热点（也就是AP，或者无线路由器）越来越多，在城市中更趋向于空间任何一点都能接收到至少一个AP的信号。热点只要通电，不管它怎么加密的，都一定会向周围发射信号。信号中包含此热点的唯一全球ID。即使距离此热点比较远，无法建立连接，但还是可以侦听到它的存在。并且热点一般都是很少变位置的，比较固定。<br>
<br>
这样，定位端只要侦听一下附近都有哪些热点，检测一下每个热点的信号强弱，然后把这些信息发送给Skyhook的服务器。服务器根据这些信息，查询每个热点在数据库里记录的坐标，然后进行运算，就能知道客户端的具体位置了，最后坐标告诉客户端。容易理解的是，收到的AP信号越多，定位就会越准。<br>
<br>
不过，一次成功的定位需要两个先决条件：<br>
<ol><li>客户端能上网</li><li>侦听到的热点的坐标在Skyhook的数据库里有</li></ol><br>
<br>
第一条不消说了，不管是Wi-Fi还是Edge，只要能连上Skyhook的服务器就行。<br>
<br>
第二条是Skyhook的金矿所在。对于Skyhook如何知道每个AP的坐标信息有两种说法：<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; 1.&nbsp; 有一种说法是靠网友自己搜集，然后发给Skyhook，Skyhook也提供了专门供网友提交信息的页面：<a href="http://www.skyhookwireless.com/submit-access-point" rel="nofollow" target="_blank">http://www.skyhookwireless.com/submit-access-point</a>。&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp; 2.&nbsp; 第二种说法则是开着车满大街转悠，边走边采集AP信号，并用GPS定位，从而就有了坐标信息。而且他们会定期重新开车采集数据，以适应热点的变化。<br>
<br>
相比之下，第二种就想是我们目前的AP信息采集产品：<strong>Wi-Fi来来，</strong>带上设备，进入某个AP的范围，填写相关位置信息提交即可。<br>
<br>
另外Skyhook网站上还提供了一个查询此服务是否覆盖的电子图：<a href="http://www.skyhookwireless.com/Coverage-Map" rel="nofollow" target="_blank">http://www.skyhookwireless.com/Coverage-Map</a>。可以看到我国大部分城市是有这个服务的。<br>
<br>
<div class="aw-upload-img-list active">
	<a href="http://f.talkingdata.com/uploads/article/20171121/4e879ecebc7662a431327bae731da849.png" target="_blank" data-fancybox-group="thumb" rel="lightbox"><img src="http://f.talkingdata.com/uploads/article/20171121/4e879ecebc7662a431327bae731da849.png" class="img-polaroid" title="Screen_Shot_2017-11-21_at_10.20_.53_PM_.png" alt="Screen_Shot_2017-11-21_at_10.20_.53_PM_.png"></a>
</div>
<br>
&nbsp;<br>
<strong>Skyhook和Apple定位的乱谈</strong><br>
&nbsp;<br>
我们知道iOS设备的定位方式有三种：<br>
<ol><li>卫星定位</li><li>蜂窝基站定位</li><li>Wi-Fi定位</li></ol><br>
<br>
卫星定位和蜂窝基站定位的原理大家应该都有了解，因为卫星和基站都是有位置信息的，根据有位置信息的东西算出自身的位置信息，这个比较容易理解。但是Wi-Fi的定位是如何进行的呢？貌似Wi-Fi里并没有位置信息啊，但通过Wi-Fi定位算出来的坐标通常是又快又比较准确，这是发生了什么？<br>
&nbsp;<br>
其实这开始于一个始乱终弃的故事。最早做Wi-Fi定位的是&nbsp;Skyhook Wireless&nbsp;，该公司通过装备了Wi-Fi天线和高灵敏度的GPS接收器的汽车在城市间兜转，以获取众多Wi-Fi热点的网络标识和信号强弱信息以及相关GPS坐标存储到服务器进行整理和分析，然后设备就可以通过附近Wi-Fi的信息向服务器获取位置信息了。 2008年，Apple发布了3G版的iPhone，改款iPhone自带的定位服务就是Apple，Skyhook以及Google三家联合开发，但是Apple发布iOS4开始抛弃了Skyhook，自己建立数据库。这也怪不得苹果这个高富帅，拥有众多的iOS设备，采集这些信息的时候根本不需要开车满大街跑啊。 总的来讲，Wi-Fi初期还是要依赖于卫星定位和蜂窝基站定位得出其初步的位置信息才可以往下展开定位工作。Wi-Fi发展到现在，其算法不断改进，定位速度和精度也越来越高。到现在，iOS设备不需要连上Wi-Fi，只需要附近有Wi-Fi信号就可以进行定位了。其实只是用户的设备会缓存相关的位置信息，这里有&nbsp;<a href="http://www.apple.com/pr/library/2011/04/27Apple-Q-A-on-Location-Data.html" rel="nofollow" target="_blank">声明</a>&nbsp;，通过这些缓存的信息，可以根据Wi-Fi信号而不需要连上就可以算出位置信息了。 现在是不是觉得懂得了Wi-Fi定位的原理，太天真了，事情远没有这么简单的……<br>
&nbsp;<br>
Wi-Fi定位方法基本上可以分为两大类：<br>
<br>
<strong>1.不基于RSSI</strong><br>
<ul><li>TOA（time ofarrival）：到达时刻</li><li>TDOA（time difference of arrival）：到达时间差</li><li>AOA（angle of arrival）：到达角度</li></ul><br>
<br>
因为这些值的获取需要特殊的Wi-Fi模块，目前智能机上并没有这些模块，因此无法获取，这类方法无法使用。<br>
<br>
<strong>2.基于RSSI</strong><br>
<br>
在智能手机上，可以通过系统SDK获取到周围各个AP（Access Point）发送的信号强度RSSI（Received Signal Strength Indicator）及AP地址，目前利用RSSI来定位看来是最可行的方法，基于RSSI定位主要有两个算法:<br>
<ul><li>三角定位</li></ul><br>
<br>
如果我们已经知道了这些AP的位置，我们可以利用信号衰减模型估算出移动设备距离各个AP的距离,然后根据智能机到周围AP距离画圆，其交点就是该设备的位置。由于三角定位算法需要我们提前知道AP的位置，因此对于环境变化较快的场合不适合使用。<br>
<ul><li>指纹算法</li></ul><br>
<br>
指纹算法类似于机器学习算法，分为两个阶段:<br>
<br>
离线训练阶段：将需要室内定位区域划分网格，建立采样点（间距1~2m） 使用wifi接受设备逐个采样点采样，记录该点位置、所获取的RSSI及AP地址。 对采样数据进行处理（滤波、均值等）<br>
<br>
在线定位阶段:用户持移动设备在定位区域移动，实时获取当前RSSI及AP地址，将该信息上传到服务器进行匹配（匹配算法有NN、KNN、神经网络等） 得到估算位置。 匹配算法有NN、KNN、神经网络等。<br>
<br>
指纹算法相比较三角定位算法精度更高。三角定位算法需要提前知道所有AP的位置 指纹算法需要提前绘制一幅信号Map。智能手机基于WIFI的室内定位应用，更适合使用基于RSSI信号的指纹算法，原因在于我们不需要提前知道所有AP的位置，而且指纹算法可以应对AP位置或状态的改变。可以提前将测绘指纹数据库储存到服务器上，移动设备在定位区域将自己得到的周围AP信息实时发送给服务器，由服务器进行匹配并返回坐标位置给客户端。一旦AP状态或位置变化，只需要更新定位区域数据库而并不需要在客户端作出改变。<br>
&nbsp;<br>
&nbsp;<br>
参考资料<br>
<ul><li><a href="https://www.macworld.com/article/1159528/smartphones/how-iphone-location-works.html" rel="nofollow" target="_blank">how iPhone location works？</a></li><li><a href="http://www.apple.com/pr/library/2011/04/27Apple-Q-A-on-Location-Data.html" rel="nofollow" target="_blank">Apple-Q-A-on-Location-Data</a></li><li><a href="https://www.zhihu.com/question/20593603" rel="nofollow" target="_blank">Wi-Fi 定位的原理是什么？</a></li><li><a href="http://blog.csdn.net/doubleuto/article/details/40080533" rel="nofollow" target="_blank">iOS定位原理和使用建议</a></li></ul><br>
                              </div>