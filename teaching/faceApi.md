# 如何在 h.265 视频流中抓取到人脸并生成图片

## 一、前言

现在的 web 需求也是越来越刁钻了。今天接到了一个奇怪的需求，要求在大屏播放实时直播的视频流，并且还要截取播放时段内一共出现了哪些人。

因此，首先第一步我们要先把视频流在 web 界面中播放出来，不然啥都白瞎。

当视频流播放出来了，我们再来思考抓取人脸的能力。

## 二、视频流播放

### 1、视频流介绍

如果还未接触过视频流的小伙伴肯定对视频流很陌生，那我们就来快速简单的了解一下视频流是什么东西。

视频流逝一种实时通讯技术，就是生成一段在线的链接，渲染在 web 界面上，实时传播别处视频或者直播的内容，你也可以理解为直播。

我们常见的视频流后缀有 `m3u8`,`mp4`,`rstp`,`rtmp` 等等。

而 `rstp`,`rtmp` 通常不适合在 web 上进行播放，因为浏览器限制、协议不兼容等。

因此我们可以用 `m3u8`,`mp4` 来进行开发。这个后缀可以让后端来处理，哪怕是 `rstp`,`rtmp` 的格式，也能通过转码来实现。

### 2、视频流格式

好了马上进入开发。。。等等先别急，我们需要先知道先视频流有哪些格式，我们使用的插件能否这些个时代的播放。

百度文心一言一下马上就知道视频流有 `H.264` 和 `H.265` 两种编码格式。

- `H.264` 是目前最广泛使用的视频编码，以其高效的压缩性能、良好的网络适应性和广泛的兼容性而备受推崇。
- `H.265` 是 `H.264` 的继任者，保持相同视频质量的前提下，能够实现更高的压缩比，从而进一步降低网络带宽的占用。

简单的说就是 `H.265` 更新，更牛。

目前市面上能查询得到的 web 播放插件几乎都只能支持 `H.264`。我这里实现过 `vue-video-player` 这款插件，有兴趣的小伙伴的小伙伴百度下使用教程，百度上说的可比我更详细，这不是今天的重点。

那如何能够能够既支持 `H.264` 又支持 `H.265` 的插件呢？在我查得快要放弃的时候，`@easydarwin/easyplayer` 这个插件竟神奇般的播放出来。

### 3、视频流播放

```bash
# 依赖安装
npm install @easydarwin/easyplayer
```

**这个插件安装完可不算完，还需要一步最重要的 copy 功能，这一步决定了你视频流能不能正常播放。**

```
copy node_modules/@easydarwin/easyplayer/dist/component/EasyPlayer.swf 到 静态文件 根目录
copy node_modules/@easydarwin/easyplayer/dist/component/crossdomain.xml 到 静态文件 根目录
copy node_modules/@easydarwin/easyplayer/dist/component/EasyPlayer-lib.min.js 到 静态文件 根目录
```

就是把 `node_modules` 对应路径下的这三个文件放到项目中的根目录下。

最后在 `index.html` 下引入这三个文件：

```html
<script src="./EasyPlayer-lib.min.js"></script>
<script src="./crossdomain.xml"></script>
<script src="./EasyPlayer.wasm"></script>
```

```js
<EasyPlayer
    style="width: 750px; height: 500px"
    :videoUrl="`http://cdn3.toronto360.tv:8081/toronto360/hd/playlist.m3u8`"
    aspect="16:9"
    live
    :fluent="true"
    :autoplay="true"
    video-id="videoElement"
    ref="easyPlayerRef"
    stretch
></EasyPlayer>
```

使用过于简单。这里提供几个可用的视频流用于小伙伴们测试。

- http://kbs-dokdo.gscdn.com/dokdo_300/_definst_/dokdo_300.stream/playlist.m3u8
- https://sf1-cdn-tos.huoshanstatic.com/obj/media-fe/xgplayer_doc_video/hls/xgplayer-demo.m3u8
- http://cdn3.toronto360.tv:8081/toronto360/hd/playlist.m3u8

## 三、手把手教你人脸抓取

百度一查有且只有 `face-api.js` 提供了该能力，或者说是应用最广的插件。

```bash
yarn add face-api.js
```

在使用前我们还要引入对应的识别模型，这些模型是开发者通过测试实现的模型，当然我们可以对模型进行修改，不过我没改，因为我看不懂～

[官方 gitHub](https://github.com/justadudewhohacks/face-api.js/tree/master/weights) 提供了这些模型，我们只需要将这些模型下载下来，并同样复制到静态文件根目录下就行。

根据插件的开发思路：

- 轮训 <video> 元素播放界面进行查询。
- 将查询到的图片截取出来放置的到 `canvas` 上。然后生成 `base64` 图片。

好了那么我们正式进入开发。

```js
<div class="testPageStyle">
  <EasyPlayer
    style="width: 750px; height: 500px"
    :videoUrl="`http://cdn3.toronto360.tv:8081/toronto360/hd/playlist.m3u8`"
    aspect="16:9"
    live
    :fluent="true"
    :autoplay="true"
    video-id="videoElement"
    ref="easyPlayerRef"
    stretch
  ></EasyPlayer>
  <canvas class="canvasStyle" id="canvas" width="720" height="400"></canvas>
  <img :src="item" v-for="item in imgList" />
</div>
```

```css
.testPageStyle {
  position: relative;

  .canvasStyle {
    position: absolute;
    left: 0;
    top: 0;
    background: transparent;
  }
}
```

很好理解吧，渲染 `video`，渲染 `canvas`，然后将截取出来的图片放置到 `imgList`。

在页面创建后就可以执行下面 `begin` 的代码。

```js
import * as faceapi from 'face-api.js'

// 查询 video 元素的 id 值
getVideoElementId() {
    const player = this.$refs.easyPlayerRef
    if (player) {
    const videoElement = player.$el.querySelector('video')
    if (videoElement) {
        console.log('Video Element ID:', videoElement.id)
        return videoElement.id
    } else {
        console.log('Video Element not found.')
    }
    } else {
    console.log('EasyPlayer component not found.')
    }
    return ''
},
begin() {
    const that = this
    const video = document.getElementById(this.getVideoElementId())
    const canvas = document.getElementById('canvas')
    const context = canvas.getContext('2d')
    console.log(faceapi.nets);

    // 加载模型
    Promise.all([
    faceapi.nets.tinyFaceDetector.loadFromUri('/models'),
    faceapi.nets.faceRecognitionNet.loadFromUri('/models'),
    faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
    faceapi.nets.faceLandmark68Net.loadFromUri('/models'),
    ]).then(() => {
        startVideo()
    })
},
```

在开始执行抓取之前，需要先获取到关键元素的 id，并且加载对应的模型。当使用 `EasyPlayer` 后，`<video>` 被插件包裹了，导致没办法直接定义 `id`，我只好写一个方法来获取对应的id。也还好插件有配置id，不然我岂不是要便利元素来获取了～。

加载模型的代码可能一开始看不懂，不知道对应的模型分别有什么功能。

这里可以打印一下 `faceapi.nets` 看看包含了哪些模型。

- ssdMobilenetv1：人脸检测模型，基于 MobileNetV1 的单阶段人脸检测器，检测器较小，速度较快，适合在浏览器中使用
- tiny_face_detector： 更小版本的人脸检测器，它是 ssd_mobilenetv1 的一个子集。它的模型大小更小，检测速度更快，但是精度稍低。
- face_expression_model：情绪识别模型，用于识别表情（笑、悲伤、恐惧等）
- age_gender_model：年龄和性别识别模型
- ssd_mobilenetv1_age_gender_model：结合了年龄和性别的检测模型，与人脸检测模型共享权重
- mtcnn：一个多阶段的人脸检测和对齐模型，它的精度高于 ssd_mobilenetv1，但是需要更多的计算资源和时间
- faceLandmark68Net：人脸68个点位检测

进入 `startVideo` 的开发，`startVideo` 直接写在 `begin` 里即可。

```js
begin() {
    // 省略部分代码
    ... 
    // 加载模型
    Promise.all([
        // 省略部分代码
        ...
    ]).then(() => {
        startVideo()
    })

    async function startVideo() {
        console.log('开始检测～')
        const displaySize = { width: video.width, height: video.height }
        faceapi.matchDimensions(canvas, displaySize) // 准备覆盖画布了

        // 开启自拍功能，这里开起来可以测试用。
        const stream = await navigator.mediaDevices.getUserMedia({ video: {} })
        video.srcObject = stream

        setInterval(async () => {
            // 清空画布内容
          context.clearRect(0, 0, canvas.width, canvas.height)

            // 人脸检测
          const detections = await faceapi
            .detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())
            // .detectAllFaces(video, new faceapi.SsdMobilenetv1Options({ minConfidence }))
            .withFaceLandmarks() // 人脸68个点位检测
            .withFaceDescriptors() // 人脸检测

            // 如果没查询到人脸数据，就不用继续了
          if (!detections.length) return;

          detections.forEach(async (detection) => {
            const { x, y, width, height } = detection.detection.box
            // 下面这三行的代码可以随便调整来截取到自己心仪的图片大小
            canvas.width = width 
            canvas.height = height 
            context.drawImage(video, x, y, width , height , 0, 0, width, height)

            if (detection.landmarks) {
              that.faceList.push(detection)
              const base64Image = canvas.toDataURL('image/png')
              that.imgList.push(base64Image)
            }
          })
        }, 500)
      }
},
```

上面的代码和注释应该是很清晰了。编写好代码后打开浏览器测试，你就能马上看到自己帅气的截图保持0.5秒一张疯狂的渲染在界面上。

## 四、去重

那我们如何才能做到对图片去重呢。

那就对人脸检测出来的列表数据去重处理：

```js
let newDetections = []
    if (!!that.faceList.length) {
    detections.forEach((fd1) => {
        let minNum = 0.45
        that.faceList.forEach((fd2) => {
        const checkNum = faceapi.euclideanDistance(fd1.descriptor, fd2.descriptor)
        if (minNum > checkNum) minNum = checkNum
        })
        if (minNum >= 0.45) newDetections.push(fd1)
    })
    } else {
    newDetections = detections
}
```

逻辑就是我将生成出来的图片的 `detections` 保存起来。而后每次渲染出来的图片都遍历下进行对比。

如果通过 `faceapi.euclideanDistance` 比对出来的值，如果小于 0.6 那么就代表是同一个人，反之就代表是不同的人。

所以当我们拿到最新的一张照片后，就应该和存储起来的所有图片都一一比对一下，只要每次比对的结果都大于0.6 那么我就有理由相信这是一个新的人像图片。

但是当我在实际测试的时候发现。0.6 有点太高了。所以我自己在实现需求的时候用 0.45。

## 功能实际思路

对于上述比对值用 `0.45` 就会出现同一个人多次出现的情况，导致图片列表里会生成很多相同的人像。

但是并不用担心。这里的人脸检测并不会很快，基本上只有你脸转动，或者表情变化，脸部被遮挡等情况才会出现新图。

我们在开发这种需求的时候肯定不会单独前端开发，要配合后端进行数据存储或者人员信息查询。

因此当后端通过人像比对出来的人员信息相同的时候，那么就剔除掉对应的图片就行啦。

## 结语

看了这么多，不得点个赞再走？～
