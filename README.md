## FFmpeg简介
    FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序，可以运行在Windows、Mac OS X，Android，iOS等系统，
[FFmpeg库下载页](http://ffmpeg.org/)

## 常用命令
    系统：Mac OS X，  FFmpeg版本：4.2.2
#### 1. 查看机器有哪些音视频输入设备
```java
ffmpeg -f avfoundation -list_devices true -i  //显示机器挂载的音视频设备

//例如输出以下信息，视频设备两个(摄像头，桌面（录屏常用))
[AVFoundation indev @ 0x7fe3a2700300] AVFoundation video devices:
[AVFoundation indev @ 0x7fe3a2700300] [0] FaceTime高清摄像头（内建）
[AVFoundation indev @ 0x7fe3a2700300] [1] Capture screen 0
[AVFoundation indev @ 0x7fe3a2700300] AVFoundation audio devices:
[AVFoundation indev @ 0x7fe3a2700300] [0] MacBook Pro麦克风
```
#### 2. 录制命令
```java
//  音频录制
ffmpeg  -f avfoundation -i :0 out.wav
//视频录制，输入源：桌面
ffmpeg -f avfoundation -i 1 -r 30 out.mp4 //格式也可以是yuv等等
//视频录制，输入源：摄像头
ffmpeg -framerate 30 -f avfoundation -i 0 out.mp4
```
