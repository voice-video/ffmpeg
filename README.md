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

**ffmpeg制作跑马灯效果**
```
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=x=mod(50*t\,main_w):y=abs(sin(t))*h*0.7[out]"
```

**画中画**
> 画中画也可以支持跑马灯效果


**多宫格**


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

**添加水印**
1. 文字水印
在视频中增加文字水印需要准备的条件比较多，需要有文字字库处理的相关文件，在编译FFmpeg时需要支持FreeType、FontConfig、iconv，系统中需要有相关的字库，在FFmpeg中增加纯字母水印可以使用drawtext滤镜进行支持，下面就来看一下drawtext的滤镜参数，具体见下表。

FFmpeg文字滤镜参数：

|参数|类型|说明|
-|-|-|
|text|字符串|文字|
|textfile|	字符串|	文字文件|
|box|	布尔|	文字区域背景框(缺省false)|
|boxcolor|	色彩|	展示字体区域块的颜色|
|font|	字符串|	字体名称（默认为Sans字体）|
|fontsize|	整数|	显示字体的大小|
|x|	字符串|	缺省为0|
|y|	字符串|	缺省为0|
|alpha|	浮点数|	透明度(默认为1)，值从0~1|
 	 	 
举例:
 
（1）将文字的水印加在视频的左上角：
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='hello world':x=20:y=20"`
将字体的颜色设置为绿色：
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='hello world':fontcolor=green"`
如果想调整文字水印显示的位置，调整x与y参数的数值即可。
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='hello world':fontcolor=green:x=400:y=200"`
修改透明度
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='hello world':fontcolor=green:x=400:y=200:alpha=0.5"`
 
（2）文字水印还可以增加一个框，然后给框加上背景颜色：
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='hello world':fontcolor=green:box=1:boxcolor=yellow"`
至此，文字水印的基础功能已经添加完成。
 
（3）有些时候文字水印希望以本地时间作为水印内容，可以在drawtext滤镜中配合一些特殊用法来完成，在text中显示本地当前时间，格式为年月日时分秒的方式，
`ffplay  -i input.mp4 -vf "drawtext=fontsize=60:fontfile=FreeSerif.ttf:text='%{localtime\:%Y\-%m\-%d %H-%M-%S}':fontcolor=green:box=1:boxcolor=yellow"`
 
在使用ffmpeg转码存储到文件时需要加上-re，否则时间不对。
`ffmpeg -re -i input.mp4 -vf "drawtext=fontsize=60:fontfile=FreeSerif.ttf:text='%{localtime\:%Y\-%m\-%d %H-%M-%S}':fontcolor=green:box=1:boxcolor=yellow" out.mp4`
 
（4）在个别场景中，需要定时显示水印，定时不显示水印，这种方式同样可以配合drawtext滤镜进行处理，使用drawtext与enable配合即可，例如每3秒钟显示一次文字水印：
`ffplay -i input.mp4 -vf "drawtext=fontsize=60:fontfile=FreeSerif.ttf:text='test':fontcolor=green:box=1:boxcolor=yellow:enable=lt(mod(t\,3)\,1)"`
在使用ffmpeg转码存储到文件时需要加上-re，否则时间不对。
 
（5）跑马灯效果
`ffplay -i input.mp4 -vf "drawtext=fontsize=100:fontfile=FreeSerif.ttf:text='helloworld':x=mod(100*t\,w):y=abs(sin(t))*h*0.7"`
 
修改字体透明度，修改字体颜色
`ffplay -i input.mp4 -vf "drawtext=fontsize=40:fontfile=FreeSerif.ttf:text='liaoqingfu':x=mod(50*t\,w):y=abs(sin(t))*h*0.7:alpha=0.5:fontcolor=white:enable=lt(mod(t\,3)\,1)"`
2. 图片水印
FFmpeg除了可以向视频添加文字水印之外，还可以向视频添加图片水印、视频跑马灯等，本节将重点介绍如何为视频添加图片水印；为视频添加图片水印可以使用movie滤镜，下面就来熟悉一下movie滤镜的参数，如表1-5所示。
表1-5 FFmpeg movie滤镜的参数

|参数	|类型	|说明|
|-|-|-|
|filename	|字符串|	输入的文件名，可以是文件，协议，设备|
|format_name, f	|字符串	|输入的封装格式|
|stream_index,   si|	整数|	输入的流索引编号|
|seek_point,   sp|	浮点数|	Seek输入流的时间位置|
|streams,   s	|字符串	|输入的多个流的流信息|
|loop	|整数	|循环次数|
|discontinuity|	时间差值|	支持跳动的时间戳差值|
 
`ffmpeg -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=x=10:y=10[out]" output.mp4`
Ø 原始视频文件路径：input.mp4

Ø 水印图片路径：logo.png

Ø 水印位置：(x,y)=(10,10)<=(left,top)距离左侧、顶部各10像素；

Ø 输出文件路径：output.mp4


overlay过滤器:

描述：前景窗口(第二输入)覆盖在背景窗口(第一输入)的指定位置。

语法：overlay[=x:y[[:rgb={0, 1}]]

参数 x 和 y 是可选的，默认为 0。

参数 rgb 参数也是可选的，其值为 0 或 1，默认为 0。  

|参数说明：| |
|-|-|
|x    |               从左上角的水平坐标，默认值为 0|
|y    |               从左上角的垂直坐标，默认值为 0|
|rgb  |             值为 0 表示输入颜色空间不改变，默认为 0；值为 1 表示将输入的颜色空间设置为 RGB |

|参数	|说明|
|-|-|
|main_w 或 W |   	视频单帧图像宽度|
|main_h 或 H  |  	视频单帧图像高度|
|overlay_w	|水印图片的宽度|
|overlay_h	|水印图片的高度|

对应地可以将overlay参数设置成如下值来改变水印图片的位置：
|水印图片位置	|overlay值|
|-|=|
|左上角|	10:10|
|右上角  |  	main_w-overlay_w-10:10|
|左下角   | 	10:main_h-overlay_h-10|
|右下角	|main_w-overlay_w-10:main_h-overlay_h-10|

在FFmpeg中加入图片水印有两种方式，一种是通过movie指定水印文件路径，另外一种方式是通过filter读取输入文件的流并指定为水印，这里重点介绍如何读取movie图片文件作为水印。
（1）图片logo.png将会打入到input.mp4视频中，显示在x坐标50、y坐标20的位置

`ffplay -i input.mp4 -vf "movie=logo.png[logo];[in][logo]overlay=50:10[out]"`

由于logo.png图片的背景色是白色，所以显示起来比较生硬，如果水印图片是透明背景的，效果会更好，下面找一张透明背景色的图片试一下：

`ffplay -i input.mp4 -vf "movie=logo2.png[watermark];[in][watermark]overlay=50:10[out]"`

（2）显示位置
```
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=10:10[out]"
 
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=main_w-overlay_w-10:10[out]"
 
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=10:main_h-overlay_h-10[out]"
 
ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=main_w-overlay_w-10:main_h-overlay_h-10[out]"
```
（3）跑马灯效果
`ffplay -i input.mp4 -vf "movie=logo.png[watermark];[in][watermark]overlay=x=mod(50*t\,main_w):y=abs(sin(t))*h*0.7[out]"`
 
3. FFmpeg生成画中画
在使用FFmpeg处理流媒体文件时，有时需要使用画中画的效果。在FFmpeg中，可以通过overlay将多个视频流、多个多媒体采集设备、多个视频文件合并到一个界面中，生成画中画的效果。在前面的滤镜使用中，以至于以后的滤镜使用中，与视频操作相关的处理，大多数都会与overlay滤镜配合使用，尤其是用在图层处理与合并场景中，下面就来了解一下overlay的参数，具体见表1-6。

表1-6 FFmpeg滤镜overlay基本参数
|参数	|类型	|说明|
|-|-|
|x	|字符串|	X坐标|
|y	|字符串|	Y坐标|
|eof_action|	整数|	遇到eof表示时的处理方式，默认为重复
Ø  repeat(值为0)：重复前一帧
Ø  endcall(值为1)：停止所有的流
Ø  pass(值为2)：保留主图层|
|shortest	|布尔	|终止最短的视频时全部终止（默认false）|
|format|	整数	|设置output的像素格式，默认为yuv420
Ø  yuv420 (值为0)
Ø  yuv422 (值为1)
Ø  yuv444 (值为2)
Ø  rgb (值为3)|
 
从参数列表中可以看到，主要参数并不多，但实际上在overlay滤镜使用中，还有很多组合的参数可以使用，可以使用一些内部变量，例如overlay图层的宽、高、坐标等。


（1）显示画中画效果
```
ffplay -i input.mp4 -vf "movie=sub_320x240.mp4[sub];[in][sub]overlay=x=20:y=20[out]"
 
ffplay -i input.mp4 -vf "movie=sub_320x240.mp4[sub];[in][sub]overlay=x=20:y=20:eof_action=1[out]"
 
ffplay -i input.mp4 -vf "movie=sub_320x240.mp4[sub];[in][sub]overlay=x=20:y=20:shortest =1[out]"
```
缩放子画面尺寸
`ffplay -i input.mp4 -vf "movie=sub_320x240.mp4,scale=640x480[sub];[in][sub]overlay=x=20:y=20[out]"`
（2）跑马灯

`ffplay -i input.mp4 -vf "movie=sub_320x240.mp4[test];[in][test]overlay= x=mod(50*t\,main_w):y=abs(sin(t))*main_h*0.7[out]"`
 
4. FFmpeg视频多宫格处理
视频除了画中画显示，还有一种场景为以多宫格的方式呈现出来，除了可以输入视频文件，还可以输入视频流、采集设备等。从前文中可以看出进行视频图像处理时，overlay滤镜为关键画布，可以通过FFmpeg建立一个画布，也可以使用默认的画布。如果想以多宫格的方式展现，则可以自己建立一个足够大的画布，下面就来看一下多宫格展示的例子：
ffmpeg -i 1.mp4 -i 2.mp4 -i  3.mp4 -i  4.mp4 -filter_complex "nullsrc=size=640x480[base];[0:v] setpts=PTS-STARTPTS,scale=320x240[upperleft];[1:v]setpts=PTS-STARTPTS,scale=320x240[upperright];[2:v]setpts=PTS-STARTPTS, scale=320x240[lowerleft];[3:v]setpts=PTS-STARTPTS,scale=320x240[lowerright];[base][upperleft]overlay=shortest=1[tmp1];[tmp1][upperright]overlay=shortest=1:x=320[tmp2];[tmp2][lowerleft]overlay=shortest=1:y=240[tmp3];[tmp3][lowerright]overlay=shortest=1:x=320:y=240" out.mp4
 
1.2.3.4.mp4为文件路径，out.MP4为输出文件路径，通过nullsrc创建overlay画布，画布大小640:480,
使用[0:v][1:v][2:v][3:v]将输入的4个视频流去除，分别进行缩放处理，然后基于nullsrc生成的画布进行视频平铺，命令中自定义upperleft,upperright,lowerleft,lowerright进行不同位置平铺。


只叠加左上右上的命令：
`ffmpeg -i 1.mp4 -i 2.mp4 -i  3.mp4 -i  4.mp4 -filter_complex "nullsrc=size=640x480[base];[0:v]setpts=PTS-STARTPTS,scale=320x240[upperleft];[1:v]setpts=PTS-STARTPTS,scale=320x240[upperright];[base][upperleft]overlay=shortest=1[tmp1];[tmp1][upperright]overlay=shortest=1:x=320" out2.mp4`


