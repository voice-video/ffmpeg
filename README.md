# ffmpeg实战

- ffmpeg: 音视频编码器
  - ffmpeg -h 基本信息
  - ffmpeg -h long 高级信息
  - ffmpeg -h full 所有信息

    ffmpeg -h full > ffmpeg_h_full.log
- ffplay: 媒体播放器
  - ffplay -h
- ffprobe: 多媒体流分析器
  - ffprobe -h

## ffmpeg命令

FFmpeg 的命令行参数非常多，可以分成五个部分。
```
ffmpeg {1} {2} -i {3} {4} {5}
```

上面命令中，五个部分的参数依次如下。

```
1. 全局参数
2. 输入文件参数
3. 输入文件
4. 输出文件参数
5. 输出文件
```

例如：
```ssh
<!-- -i 输入 -->
<!-- -acodec 音频编解码器 audio-->
<!-- -vcodec 视频编码器用x264 video-->
<!-- -s [：stream_specifier]大小（输入/输出，每个流） 设置窗口大小-->
ffmpeg -i test.mp4 -acodec copy -vcodec libx264 -s 1280x720 test.flv


-c:v：指定视频编码器 / -vcodec 两者效果一样
-c:a：指定音频编码器 / -acodec 两者效果一样

ffmpeg -i test.mp4 -c:a copy -c:v libx264 -s 12800x720 test.flv
```

<img width="951" alt="image" src="https://user-images.githubusercontent.com/17528531/184559156-be8dba90-0205-4bdd-ba5d-cc0cafd649b1.png">


**查看添加水印，文字等**
```
ffmpeg -filters 
```
----------------------------------
**基础命令小结**
```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...

ffmpeg -i [输入文件名] [参数选项] -f [格式] [输出文件] 

参数选项： 
1、-an: 去掉音频 
2、-vn: 去掉视频 
3、 -acodec: 设定音频的编码器，未设定时则使用与输入流相同的编解码器。音频解复用在一般后面加copy表示拷贝 
4、 -vcodec: 设定视频的编码器，未设定时则使用与输入流相同的编解码器，视频解复用一般后面加copy表示拷贝 
5、–f: 输出格式（视频转码）
6、-bf: B帧数目控制 
7、-g: 关键帧间隔控制(视频跳转需要关键帧)
8、-s: 设定画面的宽和高，分辨率控制(352*278)
9、-i:  设定输入流
10、-ss: 指定开始时间（0:0:05）
11、-t: 指定持续时间（0:05）
12、-b: 设定视频流量，默认是200Kbit/s
13、-aspect: 设定画面的比例
14、-ar: 设定音频采样率
15、-ac: 设定声音的Channel数
16、-r: 提取图像频率（用于视频截图）
17、-c:v:  输出视频格式
18、-c:a:  输出音频格式
19、-y:  输出时覆盖输出目录已存在的同名文件
20、-b 修改码率
```
---------------------------------

**查看是否支持x264的编码**
```
ffmpeg -encoders | findstr x264
```


**提取YUV数据**
```
// 提取3秒数据，分辨率和原视频一致
ffmpeg -i test.mp4 -t 3 -pix_fmt yuv420p yuv420p_orig.yuv
```

```
// 提取3秒数据，分辨率转为320*240
ffmpeg -i test.mp4 -t 3 -pix_fmt yuv420p -s 320x240 yuv420p_320_240.yuv
```

**提取RGB数据**
```
// 提取3数据，分辨率转为320*240
ffmpeg -i test.mp4 -t 3 -pix_fmt rgb24 -s 320x240 rgb24_320_240.rgb
```

**RGB和YUV之间的转换**
```
ffmpeg -s 320x240 -pix_fmt yuv420p -i yuv420p_320_240.yuv -pix_fmt rgb24  rgb24_320_240.rgb
```


**ffplay播放yuv数据**
```
ffplay -pixel_format yuv420p -video_size 320x240 -i yuv420p_320_240.yuv
```

**ffplay播放rgb数据**
```
ffplay -pixel_format rgb24 -video_size 320x240 -i rgb24_320_240.rgb 
```

**ffmpeg命令提取PCM数据**
```
ffmpeg -i test.mp4 -ar 48000 -ac 2 -f s16le 48000_2_s16le.pcm
```

**ffmpeg命令修改帧率**
```
ffmpeg -i test.mp4 -r 15 out.mp4

// 注意修改帧率涉及到重新编码，所以不能加编码拷贝, 下面的这种方式不生效
ffmpeg -i test.mp4 -r 15 -codec copy out.mp4
```

**多个视频拼接**
- 对于 MPEG 格式的视频，可以直接连接：
```c
/**  这种方式的适用场景是：视频容器是MPEG-1, MPEG-2 PS或DV等可以直接进行合并的。换句话说，其实可以直接用cat或者copy之类的命令来对视频直接进行合并。很多文章介绍了这种方法，但适用性却没有提及。这并不是一个通用的方法
*/
ffmpeg -i "concat:test.mp4|test1.mp4|test2.mp4" -codec copy out.mp4
```

- 对于非 MPEG 格式容器，但是是 MPEG 编码器（H.264、DivX、XviD、MPEG4、MPEG2、AAC、MP2、MP3 等），可以包装进 TS 格式的容器再合并。在新浪视频，有很多视频使用 H.264 编码器，可以采用这个方法
```
ffmpeg -i input1.flv -c copy -bsf:v h264_mp4toannexb -f mpegts input1.ts
ffmpeg -i input2.flv -c copy -bsf:v h264_mp4toannexb -f mpegts input2.ts
ffmpeg -i input3.flv -c copy -bsf:v h264_mp4toannexb -f mpegts input3.ts
ffmpeg -i "concat:input1.ts|input2.ts|input3.ts" -c copy -bsf:a aac_adtstoasc -movflags +faststart output.mp4
// 保存 QuickTime/MP4 格式容器的时候，建议加上  -movflags +faststart。这样分享文件给别人的时候可以边下边看。
```

**[FFmpeg concat 分离器]方式合并视频**
- 先创建一个文本文件 filelist.txt：
```
file 'input1.mkv'
file 'input2.mkv'
file 'input3.mkv'
```

- `ffmpeg -f concat -i filelist.txt -c copy output.mkv`
> 注意：使用 FFmpeg concat 分离器时，如果文件名有奇怪的字符，要在  filelist.txt 中转义。
拼接需要注意：只有视频编码一样，音频编码一样，音频参数(采样率/声道等)也统一，才可以拼接

**ffmpeg命令截取照片**
```
ffmpeg -i test.mp4 -y -f image2 -ss 00:00:02 -vframes 1 -s 640x360 test.jpg
```

- image2 一种格式
- vframes 帧，如果大于1，需要加%03d test%03d.jpg

**视频转图片**
提取前2秒数据， 每秒10帧
```
ffmpeg -i test.mp4 -t 2 -s 640x360 -r 10 test%03d.jpg
```
**图片转视频**
```
ffmpeg -f image2 -i test%03d.jpg -r 10 test2.mp4
```

**视频转gif**
```
ffmpeg -i test.mp4 -t 5 -r 1 img.gif
```

**gif转视频**
```
ffmpeg -f gif -i img.gif  test.mp4
```


**视频裁剪**
- 图片裁剪
```
// 0:0 表示起始坐标， iw/3 表示宽度的1/3
ffmpeg -i input.jpg -vf crop=iw/3:ih:0:0 out.jpg

ffplay -i input.jpg -vf crop=iw/3:ih:0:0 // 可以清楚的看到截出的效果
```
- 视频裁剪
> 和上面一样

**过滤器-文字水印**



**过滤器-图片水印**


**ffmpeg制作跑马灯效果**
```
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=x=mod(50*t\,main_w):y=abs(sin(t))*h*0.7[out]"
```

**画中画**
> 画中画也可以支持跑马灯效果


**多宫格**