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

```ssh
<!-- -i 输入 -->
<!-- -acodec 音频编解码器 -->
<!-- -vcodec 视频编码器用x264 -->
ffmpeg -i test.mp4 -acodec copy -vcodec libx264 -s 1280*1270 test.flv
```

<img width="951" alt="image" src="https://user-images.githubusercontent.com/17528531/184559156-be8dba90-0205-4bdd-ba5d-cc0cafd649b1.png">




