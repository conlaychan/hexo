---
title: 播放监控摄像机的视频流的方案
date: 2022-01-10 10:40:00
tags:
  - 海康
  - 大华
  - 天地伟业
  - 监控
  - VLC
  - HLS
  - m3u8
categories: 其他
description: 国内主流监控设备视频流的播放
---

## 监控设备 rtsp 流的格式
* 大华
  * 主码流 <font color="green">rtsp://账号:密码@摄像头IP:554/cam/realmonitor?channel=1&subtype=0</font>
  * 辅码流 <font color="green">rtsp://账号:密码@摄像头IP:554/cam/realmonitor?channel=1&subtype=1</font>
* 海康
  * 主码流 <font color="green">rtsp://账号:密码@摄像头IP:554/h264/ch1/main/av_stream</font>
  * 辅码流 <font color="green">rtsp://账号:密码@摄像头IP:554/h264/ch1/sub/av_stream</font>
  * <font color="blue">注意：实际情况下 URL 中使用 h264 或 h265 并没有区别，这是由设备来决定的，拉流方决定不了！</font>
* 天地伟业
  * 主码流 <font color="green">rtsp://账号:密码@NVR服务器IP:554/100/1</font>
  * 辅码流 <font color="green">rtsp://账号:密码@NVR服务器IP:554/100/2</font>
  * <font color="blue">注意：示例中的100表示通道号为99！设备在厂商系统中的唯一编号为99！</font>
* 大华 HLS 流地址格式
  * 主码流 <font color="green">http://服务器IP:端口/live/cameraid/设备编号%24通道号/substream/1.m3u8</font>
  * 辅码流 <font color="green">http://服务器IP:端口/live/cameraid/设备编号%24通道号/substream/2.m3u8</font>
  * 示例：channelid 为 <font color="red">1000000</font>$1$0$<font color="red">33</font> 的设备的辅码流地址是
  
  > http://<span></span>192.168.0.1:7086/live/cameraid/<font color="red">1000000</font><font color="blue">%24</font><font color="red">33</font>/substream/2.m3u8
  > 其中的 <font color="blue">%24</font> 是固定片段。
  > 大华 HLS 服务的端口通常是7086。

## 播放器

### 桌面软件
推荐 [VLC media player](https://www.videolan.org/) ，能播放 h264/h265 的 rtsp/HLS。

### HTML无插件方案
* rtsp：使用 [ZL media Kit](https://github.com/ZLMediaKit/ZLMediaKit) 将 rtsp 转码为 http-flv 或 ws-flv 然后用 [Jessibuca](http://jessibuca.monibuca.com/) 播放（务必设置forceNoOffscreen为true），支持 h264/h265。
* HLS：当前没有播放 h265-HLS 的方案，播放 h264-HLS 的 js 库有很多，最简单的便是 [hls.js](https://hls-js.netlify.app/api-docs/) 。

------
![福利](/images/骚图/三国杀/双乔3.jpg)

