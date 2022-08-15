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
ffmpeg -i test.mp4 -acodec copy -vcodec libx264 -s 1280x720 test.flv


-c:v：指定视频编码器 / -vcodec 两者效果一样
-c:a：指定音频编码器 / -acodec 两者效果一样

ffmpeg -i test.mp4 -c:a copy -c:v libx264 -s 12800x720 test.flv
```

<img width="951" alt="image" src="https://user-images.githubusercontent.com/17528531/184559156-be8dba90-0205-4bdd-ba5d-cc0cafd649b1.png">

