---
title: obs+node-media-server+flv.js实现录播和直播
date: 2018-05-22 17:12:43
tags:
---
### 实现思路
- 下载obs软件，进行视频的录制
- 通过node-media-server开启一个服务，在obs中推流到该服务器
- 通过flv.js配合html5的video标签实现node-media-server中视频源的播放

### 开始实现

#### obs的使用
obs的下载请移步[官网](https://obsproject.com/)，有windows, mac, linux 三个平台的版本可供下载
我这里使用的是mac版，其他版本的使用应该也差不多
- 首先需要新建一个场景
![新建场景.png](/images/obs1.png)
这里有很多种场景可以使用，我用显示捕获来示范一下吧...
![新建场景.png](/images/obs2.png)
可以对场景进行命名，我直接使用默认的名字，点击确定
![新建场景.png](/images/obs3.png)

再次点击确定，这个时候场景就创建成功了，拖动场景可以将场景进行缩放，缩放到遮住黑色的背景就好了
![缩放场景.png](/images/obs4.png)
- 推流
视频的本质其实是一张张截下来的图片，我们需要将这一张张图片放到一个地方，然后前端就可以从这个地方读取，从而展示出来，因此在这之前我们需要开启一个服务，作为前端获取视频的源地址

#### node-media-server开启服务
1. 新建一个空白的文件夹，执行`npm init`, 根据提示输入相关信息后，下载node-media-server
```
npm install node-media-server --save
```
2. 新建一个入口文件index.js
```
const NodeMediaServer = require('node-media-server');

const config = {
  rtmp: {
    port: 1935,
    chunk_size: 60000,
    gop_cache: true,
    ping: 60,
    ping_timeout: 30
  },
  http: {
    port: 8000,
    allow_origin: '*'
  }
};

var nms = new NodeMediaServer(config)
nms.run();
```
3. 然后在命令行中执行
```
node index.js
```
如果看到下面的提示，表示我们已经成功开启node-media-server服务了
![开启服务.png](/images/obs5.png)

#### flv.js
flv.js是来自Bilibli的开源项目。它解析FLV文件喂给原生HTML5 Video标签播放音视频数据，使浏览器在不借助Flash的情况下播放FLV成为可能。具体的介绍请自行google哈，继续刚才的项目

- 新建一个index.html文件
 ```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>直播</title>
</head>
<body>
    <script src="https://cdn.bootcss.com/flv.js/1.4.0/flv.min.js"></script>
    <video id="videoElement" width="100%" controls></video>
    <script>
        if (flvjs.isSupported()) {
            var videoElement = document.getElementById('videoElement');
            var flvPlayer = flvjs.createPlayer({
                type: 'flv',
                url: 'http://localhost:8000/live/hello.flv'
            });
            flvPlayer.attachMediaElement(videoElement);
            flvPlayer.load();
            flvPlayer.play();
        }
    </script>
</body>
</html>

```
> 这里遇到了一个坑，可能是mac的原因，默认视频是没有自动播放的，而且一开始video标签我也没有加上controls，所以网页上一直显示的是一张静态的图片，偶然才发现原来是视频处于暂停状态 =_=!!

#### 可以进行录播啦~
- 点击obs中的设置，进入设置页面，点击流，如果是在本地直播的话，流类型选择自定义流媒体服务器，url填写如图所示，流名称填写index.html设置的名字，本项目是`hello`

> 我们也可以通过bilibili等直播平台进行播放，这里就填写你bilibili上的直播链接和名称

![image.png](/images/obs6.png)

- 点击obs的开始推流按钮
![image.png](/images/obs7.png)

这时双击在浏览器打开index.html就可以看到直播啦，记得点击视频下方的开始按钮~












