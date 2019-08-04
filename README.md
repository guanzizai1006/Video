# 物联网视频系统
第八届中国软件杯三等奖作品-物联网视频系统
## 一. 摘要
   视频监控系统在各行各业得到广泛的应用。但是由于视频对网络带宽要求比较高，限制了视频系统在某些网络条件苛刻环境的应用。
  物联网采用的网络媒体，一般具有带宽窄，网络连接不安定的特点。很难满足通用视频监控系统对网络的要求。本项目的目标是实现物联网环境下的视频监控解决方案，面向监控对象变化缓慢，降低监控视频的帧率对不会影响监控效果的应用场景；同时满足当有异常事物闯入造成监控画面短时剧烈变化时提高视频帧数以达到监控目的应用场景

## 二. 作品介绍
                                             
  物联网传感器网络的优点是部署方便，功耗低，成本低，可以部署在比较恶劣的环境。但网络带宽低，可靠性差。 
  因此，目前各种物联网协议（如MQTT,GoAP）都为应对这种网络而设计，但这些协议不适合视频监控这类大数据量传输的应用。本题目的目的是设计一套基于UDP协议的轻量级的应用层协议，通过增加数据传输的时间，将视频数据传输出来，再通过服务器端的处理，优化视频的显示效果，满足对低变化率场景的视频监控应用
在大量的野外视频监控系统中，网络部署比较困难，往往采用不安定的无线网络搭建视频监控系统。在该场景下，视频变化率很低，系统只需要扑捉短时的异常变化即可。该系统可以大大降低视频系统对网络的要求，同时不降低视频监控的效果实现大幅降低网络消耗。
## 三. 实现方案（建议包括系统方案、实现原理、硬件框图、软件流程、功能、指标、特色等）                                             

1，功能与指标：
1）系统包括视频采集端和视频服务器端。视频采集端从摄像头获取视频，根据视频画面的变化率决定视频发送到服务器的帧率。服务器接收视频，并保存视频文件到本地磁盘
2）视频采集端采用UDP协议，将视频数据发送给服务器，以适应物联网连接不安定的网络特点。带宽可调：20k-90k
3）视频采集软件在实现视频变化检测算法时，尽可能的考虑各种干扰因素。在视频没有变化时采用固定帧率发送视频，在有异常事物闯入造成视频剧烈变化时，可以尽快传输视频。
4）视频采集端建议采用树莓派开发板（树莓派3b+），摄像头采用海康网络设想头（萤石C6C 1080P云台网络摄像机）,通过调取RTSP协议视频流处理视频，不改变视频摄像头原始分辨率。树莓派安装ubuntu MATE 操作系统，视频采集软件在树莓派启动后自动运行。
2， 实现原理
2.1 客户端
（1）变化率的检测
整体的设计思路是获取视频流后，使用opencv处理视频帧，通过opencv的bsm背景消除建模，将图像处理为二值图像（背景为黑色，前景为白色）：

后遍历图像的像素点，计算出当前图像白色像素所占比，即为图像的变化率
（2）发送视频
拆分视频为图片，通过发送图片的方式到服务端，处理每帧图像，检测变化率，当检测到当前帧变化率超过阈值后，造成视频剧烈变化，就开始发送该的图片帧，当图片的变化率没有超过阈值 5分钟发送一次图像帧，服务端收到图片后进行合成视频
（3）网络通讯线程
将要发送的图片帧放入队列进行缓存，并在帧头部带上处理的时间，作为帧号。
开辟发送图片的线程，在线程中对队列每个图片帧进行压缩并且拆分，一张图片拆分为多个数据包（Internet上的标准MTU值为576，所以每个数据包设置为最大512kb，保证udp的传输），根据自己设定的应用层协议将图片包发送出去
在发送每个图片数据包的间隙，设置不同的延时，从而就能控制带宽
2.2服务端
（1）网络通讯线程
服务端收到消息，根据应用层协议，判断是否符合协议，否则不进行处理将收到的图片包进行重组，并且有丢包检测机制，当发现有丢包情况时，向客户端发送重发数据包，服务端收到图片后将图片以发送的时间为名称，缓存在特定的文件夹，达到一定条件进行合成视频，在nginx服务器上搭建web端，web端实时刷新访问文件夹里面的图片，实现预览效果。
（2）视频合成线程
服务端开启视频合成线程进行视频合成，当检测到图片帧发送时间相差60秒时，或者五分钟内没有收到图片帧，即判断不是一个时间段闯入的，开始进行合成视频
3，硬件框图
1， 客户端：

2， 服务端

4，软件流程

5， 特色

## 四. 性能测试
1，测试设备
客户端：树莓派3B
服务端：本地Ubutun虚拟机服务器
摄像头：萤石C6C 1080P云台网络摄像机
2，测试数据
本地视频文件
萤石C6C 1080P云台网络摄像机RTSP视频流
3,结果分析
通过交叉编译在树莓派运行客户端，在树莓派上运行效果良好，视频显示因树莓派处速度原因稍有延迟，但不影响使用，服务端部署在ubuntu虚拟机上图片合成视频效果良好（详见演示视频）

## 五. 创新性

1， 发送视频的带宽可调，在有异物的情况下发送视频，且带宽可调（19k-90k），在野外监控中，比起传统的视频监控，可以大大降低流量成
2， 自定义UDP应用层协调，实现丢包检测等功能，通过增加数据传输的时间，将视频数据传输出来，再通过服务器端的处理，优化视频的显示效果，满足对低变化率场景的视频监控应用。