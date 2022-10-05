# ffmpeg编程入门

## ffmpeg8个常用库
1. AVUtil: 核心工具库
2. AVFormat: 文件格式和协议库
3. AVCodec: 编解码库
4. AVFilter: 音视频滤镜库
5. AVDevice: 输入输出设备库
6. SwrRessample: 音频重采样，声道数，数据格式，采样率等多种基本信息的转换
7. SWScale: 图像格式转换，比如YUV数据转换成RGB, 缩放尺寸1280x720变为800x480
8. PostProc: 后期处理，当使用AVFilter时，需要打开PostProc的开关

## ffmpeg 函数
1. av_register_all() 注册所有组件，4.0已弃用
2. avdevice_register_all() 对设备进行注册，比如V4L2等
3. avformat_network_init() 初始化网络库以及网络加密协议相关的库（比如openssl）

### ffmpeg函数-封装格式相关
- avformat_alloc_context() 负责申请一个AVFormatContext结构的内存，并进行简单初始化
- avformat_open_input() 打开输入的视频文件
- avformat_find_stream_info() 获取视频文件的信息
- avformat_read_fream() 读取音视频包
- avformat_seak_file() 定位文件，拖动某时刻播放, 如果是直播流 seak就是无效的
- av_seak_frame() 定位文件
- avformat_free_context() 释放该结构里面的所有东西以及该结构本身
- avformat_close_input() 关闭解复用器。关闭后就不需要avformat_free_context()进行释放

### ffmpeg函数-解码器相关
- avcodec_alloc_context3() 分配解码器上下文
- avcodec_find_decoder() 根据ID查找解码器
- avcodec_find_decoder_by_name() 根据名字查找解码器
- avcodec_open2() 打开编解码器
- ~~avcodec_decode-video2()~~ 解码一帧视频数据 3.x版本以下
- ~~avcodec_decode-audio4()~~ 解码一帧音频数据 3.x版本以下
- avcodec_send_packet() 发送解码数据包
- avcodec_receive_frame() 接收解码后的数据
- avcodec_free_context() 释放解码器上下文，包含了avcodec_close()
- avcodec_close() 关闭解码器





