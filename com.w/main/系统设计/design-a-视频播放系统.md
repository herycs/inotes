# 视频点播

# 基本策略

## 视频文件处理

1. 下载后播放

    通过HTTP 协议从服务器上下载，而后进行播放

2. 实时播放

    rtmp协议连接媒体服务器进行实时播放（造价高，需要搭建流媒体服务器）

3. 近实时播放

    使用HLS协议（协议将视频封装为ts格式，视频编码为H264，音频编码为MP3，AAC或AC3），连接HTTP服务器

## 播放器选择

1. flash（额外安装播放器）
2. h5（无需安装外置播放器）
3. 安装播放器插件
4. video.js（默认使用h5，当不支持h5时会切换为flash）

# 操作程序

FFmpeg记录转换视频，音频，并能将其转换为流的开源计算机程序。

使用：QQ影音，暴风影音等

# 基础参数

ffmpeg [参数]

操作视频文件过程：x.avi ---> x.mp4 --->m3u8

# 引用

1. 页面引入video.css, video.js, video-contrib-hls.js

2. 页面使用

    ```html
    <video src="http://www.sss.com/video/v1/xxx.m3u8">标签引入视频</video>
    ```

    