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

**查看是否支持x264的编码**
```
ffmpeg -encoders | findstr x264
```

**ffmpeg制作跑马灯效果**
```
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=x=mod(50*t\,main_w):y=abs(sin(t))*h*0.7[out]"
```

**画中画**
> 画中画也可以支持跑马灯效果


**多宫格**
