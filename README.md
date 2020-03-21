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
//音频录制
ffmpeg  -f avfoundation -i :0 out.wav
//视频录制，输入源：桌面
ffmpeg -f avfoundation -i 1 -r 30 out.mp4 //格式也可以是yuv等等
//视频录制，输入源：摄像头
ffmpeg -framerate 30 -f avfoundation -i 0 out.mp4
//录制音频 + 桌面视频，1 代表视频设备，0代表音频设备
ffmpeg -f avfoundation -i 1:0 -r 30 out.mp4
//录制音频 + 摄像头视频
ffmpeg -framerate 30 -f avfoundation -i 0:0 out.mp4
//录制视频原始数据，录制时终端会显示大小和格式，如：1280x720，uyvy422，播放时需要指定参数
ffmpeg -f avfoundation -i 1 -r 30 out.yuv
//录制音频原始数据
ffmpeg  -f avfoundation -i :0 -ar 44100 -f s16le out.pcm
```
#### 3. 音视频分离合并
```java
//acodec: 指定音频编码器，copy 指明只拷贝，不做编解码，vn: v 代表视频，n 代表 no 也就是无视频的意思
ffmpeg -i input.mp4 -acodec copy -vn out.aac
//vcodec: 指定视频编码器，copy 指明只拷贝，不做编解码，an: a 代表视频，n 代表 no 也就是无音频
ffmpeg -i input.mp4 -vcodec copy -an out.h264
//音视频合并
ffmpeg -i out.h264 -i out.aac -vcodec copy -acodec copy out.mp4
//抽取视频原始数据(yuv)，-c:v rawvideo 指定将视频转成原始数据，-pixel_format yuv420p 指定转换格式为yuv420p
ffmpeg -i input.mp4 -an -c:v rawvideo -pixel_format yuv420p out.yuv
ffplay -video_size 720x1280 -pixel_format yuv420p out.yuv//播放视频原始数据
//YUV转H264:
ffmpeg -f rawvideo -pix_fmt yuv420p -s 720x1280 -r 30 -i out.yuv -c:v libx264 -f rawvideo out.h264
//抽取音频原始数据
ffmpeg -i input.mp4 -vn -ar 44100 -ac 2 -f s16le out.pcm
ffplay -ar 44100 -ac 2 -f s16le -i out.pcm//播放音频原始数据
```
#### 4. 添加水印，移除水印
```java
//添加水印 -vf中的 movie 指定logo位置，scale 指定 logo 大小，overlay 指定 logo 摆放的位置
ffmpeg -i input.mp4 -vf "movie=logo.png,scale=64:48[watermask];[in][watermask] overlay=30:10 [out]" water.mp4
//移除水印
ffplay -i input.mp4 -vf delogo=x=20:y=30:w=210:h=100:show=1   //观察视频绿框，调整x,y,w,h找到水印位置
ffmpeg -i input.mp4 -vf delogo=x=20:y=30:w=210:h=100 output.mp4 //绿框不显示，移除水印
```
#### 5. 视频缩放裁剪
```java
// 视频宽高缩小50%
ffmpeg -i input.mp4 -vf scale=iw/2:-1 scale.mp4
//视频裁剪
ffmpeg -i input.mp4 -vf scale=iw/2:-1 scale.mp4
ffmpeg -ss 00:00:05 -t 10 -i input.mp4 -vcodec copy -acodec copy input_10.mp4//略有误差，时间点不是关键帧
//时间精确，将输入的视频先转换成所有的帧都为关键帧的视频，其实就是将所有的帧的编码方式转为帧内编码
ffmpeg -i input.mp4 -qscale 0 -intra input_keyword.mp4  //转换为关键帧
ffmpeg -ss 00:00:05 -t 10 -i input_keyword.mp4 -vcodec copy -acodec copy input_10.mp4//起始时间5s，截取10s
//截取一段音频
ffmpeg -ss 00:00:05 -t 10 -i input_keyword.mp3 -vn -acodec copy input_10.mp4//截取10s
//从指定的x、y(10, 130)位置裁剪成指定的w/2、h/2
ffmpeg -i input.mp4 -vf crop=iw/2:ih/2:10:130 -c:v libx264 -c:a copy output.mp4
//从坐标(10, 130)开始裁剪大小为：原视频宽度 * 1000(固定值：1000) 的视频
ffmpeg -i input.mp4 -vf crop=iw:1000:10:130 -c:v libx264 -c:a copy output.mp4
```
#### 6. 倍速播放
```java
//视频倍速播放：
//视频滤波器通过改变每一个 pts时间戳
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" output.mp4
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output.mp4
//音频倍速播放
ffmpeg -i input.mp4 -filter:"atempo = 2.0" -vn output.mp4
//atempo filter 配置区间在0.5和2.0之间，如果需要更高倍，可以使用多个 atempo filter 串在一起来实现，下面是实现4倍的参考
ffmpeg -i input.mp4 -filter:"atempo=2.0,atempo=2.0" -vn output.mp4
//音视频同步倍速
ffmpeg -i input.mp4 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" output.mp4
//-filter_complex 复杂滤镜，[0:v]表示第一个（文件索引号是0）文件的视频作为输入。setpts=0.5PTS表示每帧视频的pts时间戳都乘0.5 ，也就是差少一半
```
#### 7. 对称视频
```java
//水平方向
ffmpeg -i input.mp4 -filter_complex "[0:v]pad=w=2*iw[a];[0:v]hflip[b];[a][b]overlay=x=w" duicheng.mp4
//垂直方向
ffmpeg -i input.mp4 -filter_complex "[0:v]pad=w=2*iw[a];[0:v]vflip[b];[a][b]overlay=x=w" duicheng.mp4
```
#### 8. 画中画
```java
ffmpeg -i input.mp4 -i input1.mp4 -filter_complex "[1:v]scale=w=176:h=144:force_original_aspect_ratio=decrease[ckout];[0:v][ckout]overlay=x=W-w-10:y=0[out]" -map "[out]" -movflags faststart new.mp4
ffmpeg -i input.mp4 -i input1.mp4 -filter_complex "[1]scale=iw/2:ih/2[pip];[0][pip]overlay=main_w-overlay_w-10:main_h-overlay_h-10" new.mp4
// main_w 为 [0] 宽度 main_h 为 [0] 高度 overlay_w 为 [pip] 宽度 overlay_h 为 [pip] 高度
//[0] 表示第一个源（-i） [1] 表示第二个源（-i）
//[1]scale=iw/2:ih/2[pip] 缩放视频源[1]为源宽（iw）二分之一，高（ih）二分之一，得到的结果为[pip]
//main_w-overlay_w-10，main_h-overlay_h-10是子画处于父画面的位置
ffmpeg -i input.mp4 -i input1.mp4 -filter_complex "[1]scale=iw/2:ih/2[pip];[0][pip]overlay=(main_w-overlay_w)/2:(main_h-overlay_h)/2" new.mp4
//子画面居中
````
#### 9. 多路视频拼接
```java
ffmpeg -f avfoundation -i "1" -framerate 30 -f avfoundation -i "0:0" -r 30 -c:v libx264 -preset ultrafast -c:a aac -profile:a aac_he_v2 -ar 44100 -ac 2 -filter_complex "[0:v]scale=1280:720[a];[a]pad=2560:720[b];[b][1:v]overlay=1440:0[out]" -map "[out]" -movflags faststart -map 1:a out.mp4
//mac 桌面截屏和摄像头视频合为一路视频，pad用作边界扩充
ffmpeg -f concat -i test.txt -c copy output.mp4
//test.txt文件内容：file 'input.mp4'换行file 'input1.mp4'
```
#### 10. 视频图片互转
```java
\\视频转JPEG
ffmpeg -i input.mp4 -r 1 -f image2 image-%3d.jpeg
\\视频转gif
ffmpeg -i input.mp4 -ss 00:00:00 -t 10 out.gif
\\图片转视频
ffmpeg -f image2 -i image-%3d.jpeg images.mp4
```
#### 11. 直播相关
```java
//推流： 
ffmpeg -re -i out.mp4 -c copy -f flv rtmp://server/live/streamName
//拉流保存：
ffmpeg -i rtmp://58.200.131.2:1935/livetv/hunantv -c copy dump.mp4
//转流：
ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v copy -f flv rtmp://server/live/h264Stream
//实时推流：
ffmpeg -framerate 15 -f avfoundation -i “1” -s 1280x720 -c:v libx264 -f flv rtmp://localhost:1935/live/room
```
