我是用的是wsl2，ubuntu版本24.04.3进行部署，无法进行硬件编码和调用摄像头，因此利用ffmepg工具和mediamtx进行的以网络为基础的rtsp推流，成功部署并进行推理。

deepstream环境配置和部署：
cuda下载：https://developer.nvidia.com/cuda-downloads
Cudnn: https://developer.nvidia.com/cudnn-downloads
tensorrt: https://developer.nvidia.com/tensorrt/download
deepstream: https://catalog.ngc.nvidia.com/orgs/nvidia/collections/deepstream_sdk
注意！注意！注意！：
deepstream的部署有着严苛的版本要求，配置之前先要知道自己的系统版本，nvidia驱动版本，python绑定，适配版本查询（cuda,Cudnn ...）,用发及下载，参考：https://catalog.ngc.nvidia.com/orgs/nvidia/collections/deepstream_sdk
推荐下载tbz2文件，适配性较好。ex：（deepstream_sdk_v8.0.0_x86_64.tbz2）
配置完成后有时会发现缺少依赖，先在终端搜索可用的相关依赖，再下载，re:有时会提示找不到相关依赖的下载路径
完成之后：deepstream-app --version验证
没问题的话下载deepstream-yolo：https://github.com/marcoslucianops/DeepStream-Yolo
（有详细教程）
最终参考（全）：https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Quickstart.html

rtsp：
1.确保网络良好，ip不加密。
mediamtx: https://github.com/bluenviron/mediamtx/releases
mediamtx作为一个ffmpeg的服务端，可以将视频流发送到mediamtx服务器上，再在其他地方进行推流输出结果。
ffmpeg下载：直接在终端输入sudo apt install ffmpeg即可。
使用：看情况修改mediamtx的yaml配置文件，再运行meidamtx，打开另一个终端进行ffmpeg推流，示例：ffmpeg -f dshow -rtbufsize 1024M -i video="USB2.0 HD UVC WebCam" -c:v libx264 -preset ultrafast -tune zerolatency -f rtsp rtsp://localhost:8554/camerastream
正常情况下视频流会成功传到服务器中，我们可以进行调用，先测试视频流是否可调用，测试调用示例:ffplay rtsp://localhost:8554/camerastream.
如果成功再进行deepstream调用，调用需要修改localhost主机IP（在配置文件里）该文件里的配置即版本图片仅供参考。

Gstreamer_python:Gstreamer作为一种强大的跨平台多媒体框架，支持各种音视频格式的处理，包括图片，视频，视频流等。deepstream依赖gstreamer的插件处理这些音视频文件，因此可直接使用gstreamer访问deepstream的内部元素以及处理库。

下面我为大家说明一下提取deepstream元元素的两种方式:1.通过deepstream-python绑定(需要编译运行setup，处理较为复杂)，python绑定之后可以通过python语言间接访问deepstream底层的c/c++结构，并提取出deepstream处理之后的元元素，详情请参考https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Quickstart.html
2.通过gstreamer插件直接访问deepstream的内部元素(前面我已经提到过deepstream是基于gstreamer的多媒体处理框架，那么使用gstreamer访问deepstream也不容置疑)，那么还有一个问题--deepstream处理之后元元素的提取。这里我们可以使用探针来获取，方便快捷。这个方法相对于前者来说学习成本较高，需要使用者熟悉gstreamer的插件的使用和结果的格式转化，而对于ai流行的现代来说问题应该不大，优点是不需要python绑定即可使用deepstream，且性能较高。详情请参考我们提供的Getreamer-python(使用deepatream-yolo做出修改得到的)里的python文件。

此外，不同架构使用deepstream-yolo差异不大，只需简单修改代码即可。关于deepstream-yolo的详细用法请参考https://github.com/marcoslucianops/DeepStream-Yolo
