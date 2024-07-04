# 图片添加水印、文件转图片、图片转文件、html2canvas截屏功能

工作中遇到了关于图片的处理方法，做个记录分享给小伙伴们。

## 一、图片添加水印(包括图片压缩)

做法: 

- 将图片转为 `canvas`
- 对图片进行内容填充
- 将图片转化成 `base64` 的格式

下面先提供代码，并对代码做进一步的讲解：

```js
picWaterMark = ({ // 1
  url = '',
  textAlign = 'left',
  font = "30px Microsoft Yahei",
  fillStyle = 'rgba(255, 255, 255, 0.8)',
  content = '请勿外传',
  callback = null,
} = {}) => {
  const img = new Image(); // 2
  img.src = url;
  img.crossOrigin = 'anonymous';  // 3
  img.onload = function () {
    const canvas = document.createElement('canvas');
    canvas.width = img.width;  // 4
    canvas.height = img.height;

    const ctx = canvas.getContext('2d');

    ctx.drawImage(img, 0, 0, imageWidth, imageHeight);
    ctx.textAlign = textAlign;  // 5
    ctx.font = font;
    ctx.fillStyle = fillStyle;
    ctx.fillText(content, 10, 20);

    let base64Url = '';
    base64Url = canvas.toDataURL("image/jpeg", 0.5); // 6
    callback && callback(base64Url); // 7
  }
}
```

代码中注释的数字，是我下文要说明的内容哈。

### 1、传参

传参我这里定义成一个对象，小伙伴可以自行定义哈，对于上面的传参值，小伙伴们也可以直接写死在代码中。

- url: 作为图片的内容，这个应该是必传项，传入 `base64` 格式的图片或者图片的 `url`。
- callBack: 图片添加完水印后的回调事件，这个也应该是必传项。

### 2、创建图片对象

创建一个图片对象用来存放我们要改造的图片。

### 3、crossOrigin

解决跨域问题。

当在 `canvas` 中绘制一张外链图片时，我们会遇到一个跨域问题。打开浏览器调试会发现以下错误：

> img.html:23 Uncaught DOMException: Failed to execute 'toDataURL' on 'HTMLCanvasElement': Tainted canvases may not be exported.


这是受限于 CORS 策略，会存在跨域问题，虽然可以使用图像，但是绘制到画布上会污染画布，一旦画布被污染，就无法提取画布上的数据。

比如无法使用使用画布 `toBlob()`,`toDataURL()`,或 `getImageData()` 方法；当使用这些方法的时候 会抛出上面的安全错误。

加上第三点的代码就能解决这个问题。

### 4、设置画布大小

这里如果将原图的长宽缩小，是可以对图片进行压缩处理的。

### 5、水印内容

[打开这里，可以查看到比较全的 Canvas 参考手册](https://www.w3school.com.cn/tags/html_ref_canvas.asp)

有了这份参考手册和上面的代码做参考，对于自定义水印内容应该是没有难度了。

### 6、提取画布数据

将画布数据提取出来，`toDataURL` 的第二个参数是对图片进行压缩，是个选填值，小伙伴可以根据需要自行定义。

### 7、callBack

生成新的图像数据后进行接下来的操作。


## 二、文件转图片

如果我们使用的 `input`, 设置 `type='file'`，进行图片上传，需要将文件数据转化成 `base64` 的形式。

```js

const Page: FC = () => {

  //  将图片转化为 base64 的格式
  const transformFileToDataUrl = (file) => {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = e => {
        const { result } = e.target;
        resolve({
          result,
        });
      };
    });
  }

  const fileChange = files => {
    const file = files.target.files[0];
    transformFileToDataUrl(file).then(data => {
      console.log(data.result);
    })
  };

  return (
    <input type='file' onChange={fileChange} />
  )
}
```

`html5` + `canvas` 进行移动端手机照片上传时，发现ios手机上传竖拍照片会逆时针旋转90度，横拍照片无此问题；Android手机没这个问题。

因此解决这个问题的思路是：获取到照片拍摄的方向角，对非横拍的ios照片进行角度旋转修正。

利用 `exif.js` 读取照片的拍摄信息。可到[官网](http://code.ciaoca.com/javascript/exif-js/) 查看详细文档。

这里主要用到 `Orientation` 属性。

`Orientation` 属性说明如下：

| 旋转角度  | 参数 |
| :-------- | :--- |
| 0°        | 1    |
| 顺时针90° | 6    |
| 逆时针90° | 8    |
| 180°      | 3    |

根据旋转角度进行修正。

修改上述 `transformFileToDataUrl()` 的方法,如下：

```js
//  将图片转化为 base64 的格式
const transformFileToDataUrl = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = e => {
      const { result } = e.target;

      EXIF.getData(file, function () {
        EXIF.getAllTags(this);
        const orientation = EXIF.getTag(this, "Orientation");
        resolve({
          result,
          orientation
        });
      });
    };
  });
}
```

## 三、图片转文件

```js
export const dataURLtoFile = (dataurl, filename) => {
  var arr = dataurl.split(","),
    mime = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]),
    n = bstr.length,
    u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([u8arr], filename, { type: mime });
}
```

## 四、压缩图片

以下代码为网络上 copy 过来，侵删。

入参先写在前头：

- data: 这里的 `data` 是个对象，里面放两个字段，`dataUrl`: base64 图片数据格式，`orientation`: 可能需要用到的旋转字段，如果不需要设置固定值为 `1` 即可。
- callback: 回掉函数
- compressionRatio: 压缩比例，默认值为20
- compress: 是否开启压缩，默认值为 `true`
- cross: 是否设置开启跨域，建议设置为 `true`

```js
export const compress = (
  data,  // { dataUrl: '', orientation: 默认值为1 }
  callback,
  compressionRatio = 20,
  compress = true,
  cross = false,
) => {
  /**
   * 压缩图片
   * @param data file文件 数据会一直向下传递
   * @param callback 下一步回调
   * @compressionRatio 压缩比例
   * @compress 是否压缩
   */
  // const imgCompassMaxSize = 200 * 1024; // 超过 200k 就压缩
  // const imgFile = data.file;
  const orientation = data.orientation || 1;
  const img = new window.Image();
  img.src = data.dataUrl;

  if (cross) {
    // img.setAttribute("crossOrigin", 'Anonymous')
    img.crossOrigin = "*";
  }

  img.onload = function () {
    let drawWidth, drawHeight, width, height;

    drawWidth = this.naturalWidth;
    drawHeight = this.naturalHeight;

    // 改变一下图片大小
    let maxSide = Math.max(drawWidth, drawHeight);

    if (maxSide > 1024) {
      let minSide = Math.min(drawWidth, drawHeight);
      minSide = (minSide / maxSide) * 1024;
      maxSide = 1024;
      if (drawWidth > drawHeight) {
        drawWidth = maxSide;
        drawHeight = minSide;
      } else {
        drawWidth = minSide;
        drawHeight = maxSide;
      }
    }

    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");

    canvas.width = width = drawWidth;
    canvas.height = height = drawHeight;
    // 判断图片方向，重置 canvas 大小，确定旋转角度，iphone 默认的是 home 键在右方的横屏拍摄方式

    switch (orientation) {
      // 1 不需要旋转
      case 1: {
        // ctx.drawImage(img, 0, 0, drawWidth, drawHeight);
        ctx.clearRect(0, 0, width, height);
        ctx.drawImage(img, 0, 0, width, height);
        break;
      }
      // iphone 横屏拍摄，此时 home 键在左侧 旋转180度
      case 3: {
        ctx.clearRect(0, 0, width, height);
        ctx.translate(0, 0);
        ctx.rotate(Math.PI);
        ctx.drawImage(img, -width, -height, width, height);
        break;
      }
      // iphone 竖屏拍摄，此时 home 键在下方(正常拿手机的方向) 旋转90度
      case 6: {
        canvas.width = height;
        canvas.height = width;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.translate(0, 0);
        ctx.rotate((90 * Math.PI) / 180);
        ctx.drawImage(img, 0, -height, width, height);
        break;
      }
      // iphone 竖屏拍摄，此时 home 键在上方 旋转270度
      case 8: {
        canvas.width = height;
        canvas.height = width;
        ctx.clearRect(0, 0, width, height);
        ctx.translate(0, 0);
        ctx.rotate((-90 * Math.PI) / 180);
        ctx.drawImage(img, -width, 0, width, height);
        break;
      }
      default: {
        ctx.clearRect(0, 0, width, height);
        ctx.drawImage(img, 0, 0, width, height);
        break;
      }
    }

    let compressedDataUrl;
    if (compress) {
      compressedDataUrl = canvas.toDataURL(
        "image/jpeg",
        compressionRatio / 100
      );
    } else {
      compressedDataUrl = canvas.toDataURL("image/jpeg");
    }
    data.compressedDataUrl = compressedDataUrl;
    delete data.orientation;
    callback(compressedDataUrl, data);
  };
};
```

## 五、html2canvas截屏功能，以及遇到的问题

```js
//两个参数：所需要截图的元素id，截图后要执行的函数， canvas为截图后返回的最后一个canvas
 html2canvas(document.getElementById('id')).then(function(canvas) {document.body.appendChild(canvas);});
```

### API

`html2canvas` 可以设置一些参数，入参请看[官网](http://html2canvas.hertzen.com/configuration)

### FAQ

#### 1、截屏失效

如果是在移动端进行截屏尝试的话，可能会存在大屏手机无法截屏的情况。

这是因为 canvas 有面积大小的限制，所以建议减少 `canvas` 的高度来保证 `html2canvas` 的正常使用。

#### 2、预览失效问题

举个例子，在钉钉微应用的开发中，官方有提供一个图片预览的方法。

截屏出来的 `base64` 数据在 ios 上能够正常的展示，但是在安卓机子上确失效。

建议：`base64` 的数据先上传到服务器上，用返回回来的 url 进行图片预览来达到目的。


 




















