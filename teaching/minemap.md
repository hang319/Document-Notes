# 手把手教你使用 minemap

## 一、前言

今天来看一款比较小众的商用前端地图开发能力 [minemap](https://minedata.cn/nce-support/webDev/MineMap-2D?type=summary)

说它小众，是因为它没有高德地图、百度地图那样的拥有足够的知名度，但并不是说它不好，它文档也相对完善，快速阅读和使用起来很方便，所以很多甲方都愿意付费选择它，这也是它的优势之一。

那从未入手过该地图的小伙伴，怎么样能够，在半小时内掌握它，并且 10 分钟内上手开发需求呢？

那笔者肯定会用最简练的文字描述和重点代码来开发我们常用的需求，并且会标注好需要注意的点～

你先别管你学会了没，你只要你能快速完成需求～

## 二、目录

- 地图初始化
- 地图飞行
- 点标记
- 点标记窗体
- 点聚合
- 绘制路线
- 绘制多边形、圆

## 三、地图初始化

`minemap` 只支持 cdn 的形式引入。

因此，记得在 `index.html` 里加上引用

```js
<link rel="stylesheet" href="https://minemap.minedata.cn/minemapapi/v2.1.1/minemap.css" />
<script src="https://minemap.minedata.cn/minemapapi/v2.1.1/minemap.js"></script>

// 需要图形绘制请同步引入下面的链接
<link rel="stylesheet" href="https://minemap.minedata.cn/minemapapi/minemap-plugins/edit/minemap-edit.css"/>
<script src="https://minemap.minedata.cn/minemapapi/minemap-plugins/edit/minemap-edit.js"></script>
```

那这里就要注意了，我们引入的是外网上可使用的链接，如果客户是内网环境访问，上面的链接就行不通了，需要问下客户内网所能访问的 `minemap` 链接。所以关于 `minemap` 的 `ip` 和版本可以通过走配置文件，在 `Vue -> main.js` 通过动态配置。

至此，我们的配置就完成了。马上创建个 `div` 看看地图的效果吧～

```vue
<template>
  <div class="mapStyle" id="mapId" />
</template>

<style>
.mapStyle {
  width: 200px;
  height: 200px;
}
</style>

<script>
export default {
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      // 不要看配置那么多，小脑袋瓜那么聪明的你肯定看出了一些规律～
      minemap.domainUrl = "https://minemap.minedata.cn";
      minemap.dataDomainUrl = "https://minemap.minedata.cn";
      minemap.serverDomainUrl = "https://minemap.minedata.cn";
      minemap.spriteUrl =
        "https://minemap.minedata.cn/minemapapi/v2.1.1/sprite/sprite";
      minemap.serviceUrl = "https://service.minedata.cn/service";
      minemap.appKey = "*****这不得你自己填呐～*****";
      minemap.solution = 11001; // 这个是序号，可填可不填

      let params = {
        container: "mapId",
        style: "https://service.minedata.cn/map/solu/style/11001", // 这个样式不一定是白色的，可以有很多种颜色，让公司的 ui 提供下颜色序号就好啦。
        /*底图样式*/
        center: [116.46, 39.92],
        /*地图中心点*/
        zoom: 14,
        /*地图默认缩放等级*/
        pitch: 0,
        /*地图俯仰角度*/
        maxZoom: 17,
        /*地图最大缩放等级*/
        minZoom: 3,
        /*地图最小缩放等级*/
        projection: "MERCATOR",
      };

      this.amap = new minemap.Map(params);
      this.amap.on("load", () => {
        console.log("地图加载好啦～");
      });
    },
  },
};
</script>
```

至此，创建地图的第一步就成功了，界面上一定会出现地图。如果没有出现，不要怀疑，一定是你上面的步骤没对好，好好检查下吧～。看起来挺多代码的，但实际上就是引入+一个方法的事～

## 四、地图飞行

[官网 飞行 demo](https://minedata.com.cn/support/api/demo/js-api/zh/map/state/map-fly)

```js
/**
 * 参数项说明：
 * center：表示飞行结束后的地图中心坐标值；
 * zoom：表示飞行结束后的地图缩放等级；
 * bearing：表示飞行结束后的地图旋转等级；
 * pitch：表示飞行结束后的地图倾斜等级；
 * duration：表示飞行时长，单位为毫秒；
 */
this.amap.flyTo({
  center: [116.46, 39.92],
  zoom: 17,
  bearing: 10,
  pitch: 30,
  duration: 2000,
});
```

## 五、点标记

[官网 点标记 demo](https://minedata.com.cn/support/api/demo/js-api/zh/overlay/marker/marker-add)

根据官网 demo，我们可以来梳理下研发逻辑，在此之前，我们要先熟悉这个方法需要传递进来哪些参数

首先第一个 `obj` 是用来存放以下的数据：

- list: 点位的数组，每个对象要包含 `lng`，`lat` 的字段代表经纬度
- url: 图片的路径
- key: 这些点位的 key(一般打点需求都会针对不同类型的数据做不同的打点图标，因此我这边做了一个 key，方便于后续的删除操作)

至于第二个参数就是存放一些属性，比如我当前存放了图片的大小，小伙伴们有需要什么就丢进去就行。

研发逻辑：

- 创建一个 `div` 存放点位的图片，并设置到大小，图片路径等基本参数
- 通过 `new minemap.Marker` 来实现打点，并设置好经纬度
- 对点位的 id 和 key 进行数据存储
- 给点位增加点击能力，用于后续的操作

```js
initPoints(obj, { width = '35px', height = '45px' }) {
    const that = this
    const { list = [], url, key } = obj
    const imgWdith = width || '35px';
    const imgHeight = height || '35px';
    if (list && !!list.length) {
    list.forEach((item) => {
        if (item.lng && item.lat) {
        var el = document.createElement('div')
        el.id = `marker-${key}`
        el.style['background-image'] = 'url(' + url + ')'
        el.style['background-size'] = '100% 100%'
        el.style['background-repeat'] = 'no-repeat'
        el.style.width = imgWdith
        el.style.height = imgHeight
        // el.style['border-radius'] = '50%'
        const _marker = new minemap.Marker(el, { offset: [-25, -50] })
            .setLngLat([parseFloat(item.lng), parseFloat(item.lat)])
            .addTo(this.amap)

        // 保存当前的点位信息，用于后续删除操作
        this.markers.push({
            key,
            _marker,
            id: item.id,
        })

        el.addEventListener('click', (e) => {
            setTimeout(() => {
            console.log('给点标记提供打点能力', e);
            }, 50)
        })
        }
    })
    }
}

// 删除点标记
delMarkIdPoints() {
    if (this.amap && this.markers.length !== 0) {
        for (let i = 0; i < this.markers.length; i++) {
            this.markers[i]._marker.remove()
        }
        this.markers = []
    }
}
```

使用：

```js
initPoints(
  {
    list: [{}, {}, {}],
    url: "图片路径～//minedata.com.cn/support/static/api/demo/js-api/zh/images/park.png",
    key: "test",
  },
  {}
);

delMarkIdPoints();
```

看似内容很多的一个方法，经过拆解是不是也变得很好理解？

## 六、点标记窗体

[官网 点标记窗体 demo](https://minedata.com.cn/support/api/demo/js-api/zh/overlay/popup/popup-add)

在笔者看来，官网提供的窗体方案会比较死板，样式不容易开发

并且窗体的数据需要在打点时就传递进去，这就很麻烦了，很多时候，我们是点击后，通过 id 去查询详细信息，再进行弹窗的。

因此，这里笔者就不复用官网的能力，有需要的小伙伴自行看下官网就行。

笔者这里推荐自己写个窗体，根据点击带过来的页面绝对定位来实现点位窗体的渲染。

同理，先说下开发逻辑，代码都可以复用，小伙伴可以直接 copy 下面的代码，减少点工作量，效率至上呗

- 首先还记得上一大点我们留了一个点标记点击事件的口子吧，在这里把能力抛出来
- 将传递出来的点位数据、x 轴、y 轴数据保存起来
- 写个定位打点上去

```js
el.addEventListener("click", (e) => {
  setTimeout(() => {
   this.pointPopup(e, { key, ...item })
  }, 50);
});

pointPopup(e, obj) {
    /**
     * 如果有需要的话这里可以请求下接口再打点
     */

    this.point = {
        x: e.clientX,
        y: e.clientY,
        visible: true,
        data: obj
    }
}
```

```html
<div
  v-if="point.visible"
  :style="{ top: `${point.y}px`, left: `${point.x}px` }"
>
  <div>这是自定义的窗体</div>
</div>
```

有使用的 demo 窗体的小伙伴肯定已经发现了。相比于官网的代码量。笔者的代码量少，简洁，灵活，可控性更高。

[官网 点聚合 demo](https://minedata.com.cn/support/api/demo/js-api/zh/layer/common/point-cluster)

## 七、点聚合

有真实开发打点经验的小伙伴就会发现点标记的弊端，就是点位一多，界面上就密密麻麻的，毫无美感可言。因此对于大几百，甚至是大几万的数据来说，我们用点聚合的方式来渲染就比较合理美观。

官网 demo 的点聚合会根据聚合量来改变颜色，比如 0 ～ 100 的聚合量渲染绿色、100 ～ 1000 渲染黄色、1000 以上渲染红色。

我这里提供的颜色渲染有些不同，如果很多类型的点聚合在地图上，其实是很难一眼看出来这些聚合点分别是什么类型。

因此，我做的改动是不同类型的聚合点，配置不同的颜色。

同理，我们先聊一下开发思路，再贴代码：

- 点聚合的图片时需要提前注册进去的，而不是在打点的时候才渲染，因此我们可以提前将所有点聚合的图标都进行注册。
- 对数据进行封装
- 将封装好的数据添加到地图中
- 添加外圈图层
- 添加内圈图层
- 添加数字图层

看似方法很长，但是拆分成上面几个小块，每一块就是几行代码～

```js
/**
 * 增加点聚合图片
 * 这里只是新增了一个图片，如果有多个的话，可以在方法里遍历添加
 * 并通过 imageClassArr 存储起来
 */
addAggregationImage(e) {
    this.amap.loadImage('图片在线路径~', (err, image) => {
        if (!this.amap.hasImage(`icon-test-img`)) {
            this.amap.addImage(`icon-test-img`, image)
            this.imageClassArr.push(`icon-test-img`)
        }
    })
}

/**
 * 初始化点聚合数据
 * @param {Array} dataArr 需处理数据源
 * @param {Object} config 取值对象集合
 * @returns {{features: *[], type: string}}
 */
addAggregationSource(dataArr, config) {
    let data = {
        type: 'FeatureCollection',
        features: [],
    }

    dataArr.forEach((item, index) => {
        data.features.push({
            type: 'Feature',
            geometry: {
                //设置经纬点
                type: 'Point',
                coordinates: [Number(item[config.lngKey]), Number(item[config.latKey])],
            },
            properties: {
                // 设置文字字段
                text: `${item[config.textKey]}`,
                pointIcon: 'icon-test-img',
                ...item,
            },
        })
    })
    return data
},

/**
 * 点聚合
 */
pointAggregation({ list, key }) {
    const that = this

    const bgColor = { r: 118, g: 144, b: 255, a: 1 };

    let newAggLayoutArr = []

    //初始化数据
    let dataSource = this.addAggregationSource(newList, {
        latKey: 'lat',
        lngKey: 'lng',
        textKey: 'id',
        classKey: 'icon',
    })

    this.amap.addSource(`${key}-data-point`, {
        type: 'geojson',
        data: dataSource,
        cluster: true,
        /* 最大聚合层级 */
        clusterMaxZoom: 15,
        clusterRadius: 50 /* 聚合半径 */,
    })


    //添加非聚合图层 加载图片到地图样式中
    this.amap.addLayer({
        id: `${key}-unclustered-points`,
        type: 'symbol',
        source: `${key}-data-point`,
        layout: {
            //布局属性
            //   这个 pointIcon 对应上面的 addAggregationSource 的参数
            'icon-image': '{pointIcon}',
            'icon-size': 0.4,
            'text-field': `{text}`,
            'text-offset': [0, 3.5],
            'text-size': 12,
            visibility: 'visible',
        },
        paint: {
            //绘制属性
            'text-color': '#357fdf',
        },
    })

    newAggLayoutArr.push(`${key}-unclustered-points`) //将图层id添加到图层id数组中

    that.amap.addLayer({
        id: `${key}-point-outer-cluster`,
        type: 'circle',
        source: `${key}-data-point`,
        paint: {
            'circle-color': `rgba(${bgColor.r}, ${bgColor.g}, ${bgColor.b}, 0.5)`,
            'circle-radius': 20,
        },
        filter: ['>=', 'point_count', 0],
    })
    newAggLayoutArr.push(`${key}-point-outer-cluster`)

    that.amap.addLayer({
        id: `${key}-point-inner-cluster`,
        type: 'circle',
        source: `${key}-data-point`,
        paint: {
            'circle-color': `rgba(${bgColor.r}, ${bgColor.g}, ${bgColor.b}, 0.8)`,
            'circle-radius': 15,
        },
        filter: ['>=', 'point_count', 0],
    })
    newAggLayoutArr.push(`${key}-point-inner-cluster`) //将图层id添加到图层id数组中

    //添加数量图层
    that.amap.addLayer({
        id: `${key}-cluster-count`,
        type: 'symbol',
        source: `${key}-data-point`,
        layout: {
            'text-field': '{point_count}',
            'text-size': 10,
        },
        paint: {
            'text-color': 'rgba(0, 0, 0, 0.75)',
        },
        filter: ['has', 'point_count'],
    })
    newAggLayoutArr.push(`${key}-cluster-count`) // 将图层id添加到图层id数组中

    this.initPointAggregationClick(key)
}

/**
 * 初始化点聚合点击事件
 */
initPointAggregationClick(key) {
    this.amap.on('click', (e) => {
        var features = this.amap.queryRenderedFeatures(e.point, {
            layers: [`${key}-unclustered-points`],
        })
        var feature = features.length > 0 ? features[0] : null
        if (!feature) {
        // 点击地图空白区域
            return
        }
        console.log('就是在这里获取到点聚合的点击事件', e, { key, ...feature.properties });
    })
}

/**
 * 删除点集合
 */
clearAggregation(key) {
    this.aggLayoutArr.forEach((item, index) => {
        // 图层存在再删除
        if (item.indexOf(key) !== -1 && this.amap.getLayer(item)) {
            this.amap.removeLayer(item)
            if (typeof this.amap.getSource(`${key}-data-point`) != 'undefined') {
            // 判断是否有该id的数据源
            this.amap.removeSource(`${key}-data-point`) // 清空数据源
            }
        }
    })
}
```

可以和官网的 demo 做一个比对，因为功能有些许不同，所以代码有些许不同，但是总体一致。

代码量虽大，但是拆分开模块来看，其实不难理解。看不懂，多看几遍就懂了。

## 八、绘制路线

[官网 绘制路线 demo](https://minedata.com.cn/support/api/demo/js-api/zh/layer/special/symtracking-layer)

绘制路线我们根据传入的经纬度数组，会按顺序在界面上连接成一条线。逻辑很简单，代码也很简单。但是这条线没办法标识方向。

我过了一遍官网的 demo，目前没有找到能够打箭头的标识路线的能力。但是我找了能够在路线上迁移图标的能力。因此我们给路线加上小车行进路线，也能代替路线方向。

那么我们先来绘制一条路线出来

```js
addLine(latLngArr, key,{ lineColor = '#3BBCFF' }) {
    const lineLatLngData = latLngArr.map((item) => [parseFloat(item.lng), parseFloat(item.lat)]);
    let jsonData = {
        type: 'FeatureCollection',
        features: [
            {
                type: 'Feature',
                geometry: {
                    type: 'LineString',
                    coordinates: lineLatLngData
                },
                properties: {}
            }
        ]
    };
    this.amap.addSource(`lineSource-${key}`, {
        type: 'geojson',
        data: jsonData
    });

    this.amap.addLayer({
        id: `lineLayer-${key}`,
        type: 'line',
        source: `lineSource-${key}`,
        layout: {
            'line-join': 'round',
            'line-cap': 'round'
        },
        paint: {
            'line-width': 6,
            'line-color': {
                type: 'categorical',
                property: 'kind',
                stops: [[1, lineColor]],
                default: lineColor
            },
            'line-blur': 0.9
        },
        minzoom: 7,
        maxzoom: 17.5
    });
}

/**
 * 删除路线
 */
 delLine(key) {
    if (this.amap.getLayer(`lineLayer-${key}`)) {
        this.amap.removeLayer(`lineLayer-${key}`);
    }
    if (typeof this.amap.getSource(`lineSource-${key}`) != 'undefined') {
    // 判断是否有该id的数据源
        this.amap.removeSource(`lineSource-${key}`); // 清空数据源
    }
}
```

路线出来了，小车也要跟着一起出来。**这里要打开动画效果**，这里笔者踩过坑，没有打开动画效果，小车不会行走。

```js
addLine() {
    // 省略部分代码...

    this.amap.repaint = true;
    var jsonDriverData = minemap.Template.util.pointArrayToSymtrackingGeoJson(lineLatLngData, false);

    this.amap.addSource(`lineSource-symtrack-${key}`, {
        type: 'geojson',
        data: jsonDriverData
    });

    this.amap.addLayer({
        type: 'symtracking',
        source: `lineSource-symtrack-${key}`,
        id: `lineLayer-vehicle-${key}`,
        layout: {
        'icon-image': 'n-vehicle',
        'icon-allow-overlap': false,
        'icon-ignore-placement': true,
        'symbol-avoid-edges': false,
        'symbol-placement': 'line',
        'symtracking-fps': jsonDriverData.features.length / 10, // 小车速度
        'symtracking-time-segment': 10, //总共要运行多少秒
        'compatible-mode': false //notice  采用新的数据样式，保证小车不消失
        },
        paint: {
        'icon-color': '#ff0000',
        'symtracking-delay': 0 //notice 小车更新延迟为零，保证小车能够循环播放，小车循环播放的条件是：此项为零 并且 symtracking-fps * symtracking-time-segment ≈ myGeojson.features.length
        }
    });
}

/**
 * 删除路线
 */

delLine(key) {
    // 省略部分代码...

    if (this.amap.getLayer(`lineLayer-vehicle-${key}`)) {
        this.amap.removeLayer(`lineLayer-vehicle-${key}`);
    }
    if (typeof this.amap.getSource(`lineSource-symtrack-${key}`) != 'undefined') {
    // 判断是否有该id的数据源
        this.amap.removeSource(`lineSource-symtrack-${key}`); // 清空数据源
    }
}
```

## 九、绘制图形

[官网 绘制图形 demo](https://minedata.com.cn/support/api/demo/js-api/zh/edit/base/edit-custom)

在绘制图形之前，我们需要回到初始化 `minemap` 的模块，确认 `minemap-edit.js` 是否有引入。

确认引入后我们需要初始化绘制能力。

```js
this.edit = new minemap.edit.init(map, {
    boxSelect: true /* 是否支持拉框选择 */,
    touchEnabled: true /* 是否支持手指触屏 */,
    displayControlsDefault: true /* 是否启用编辑控件 */,
    showButtons: false /* 是否启用默认控件按钮 */,
});
```

当然**绘制的样式是支持自定义**的，比如线的宽度，线的颜色，面的颜色等等。

这个就可以参考官网提供的 [edit api 文档](https://minedata.com.cn/minemapapi/minemap-plugins/edit/edit-api.html)

```js
this.edit = new minemap.edit.init(map, {
    boxSelect: true /* 是否支持拉框选择 */,
    touchEnabled: true /* 是否支持手指触屏 */,
    displayControlsDefault: true /* 是否启用编辑控件 */,
    showButtons: false /* 是否启用默认控件按钮 */,

    userStyles: {
        inactive: {
            // ...
        },
        active: {
            // ...
        },
        static: {
            // ...
        }
    },
});
```

不过在 `v.2.1.0` 的版本中使用无效。笔者曾使用过的 `v.2.0.0` 有效。不过该写的代码还是要写的，就是需要看下对应的版本是否能够产生对应的效果。

好了。 `edit init` 好了后，接下来就是开始绘制图形的代码啦

```js
// 允许进行edit
this.edit.enableDraw(); 

// 绘制多边形
this.edit.onBtnCtrlActive('polygon') 
```

好了话不多说，执行 `onBtnCtrlActive(mode)` 就可以在页面上绘制了，地图支持绘制各种各样的图形，进入官网demo 就能看到对应的图形对应不同的 `mode`。

绘制完怎么接收到我们绘制出来的点位经纬度呢？

**在地图初始化完成后，我们就应该创建绘制的监听**

```js
initEditCreate() {
    const that = this
    this.amap.on('edit.record.create', (res) => {
        const chooseLocation = []
        var coordinates_Polygon = res.record.features[0].geometry.coordinates[0] //获取图形所有经纬度
        for (var i = 0; i < coordinates_Polygon.length - 1; i++) {
            chooseLocation.push(coordinates_Polygon[i])
        }
        conosle.log('可以对 chooseLocation 进行保存');

        // 地图绘制后就要调整为不可编辑的状态，否则点击区域会有可编辑的操作
        this.edit.disableDraw()
    })
}
```

删除用户绘制的区域

```js
const { features = [] } = this.edit.draw.getAll();

if (features && !!features.length) {
    this.edit.enableDraw();
    this.edit.removeFeatures([features[0].id]);
    this.edit.disableDraw()
}
```

最后，如果我对绘制出来所部区域都进行了经纬度的存储，现在要回填。那么：

```js
setPointBoundary({ list = [], key }) {
   const bgColor = { r: 118, g: 144, b: 255, a: 1 };
    const ids = []

    this.drawDefault({ data: list, id: key })
    ids.push(key);

    this.edit.enableDraw()
    this.edit.setSelected(ids)
    const params = {
        fillOutlineColor: `rgba(${bgColor.r}, ${bgColor.g}, ${bgColor.b}, 1)`,
        lineColor: `rgba(${bgColor.r}, ${bgColor.g}, ${bgColor.b}, 1)`,
        custom_style: 'true',
    }
    this.edit.setCustomStyle(params)
    this.edit.disableDraw()
},

setPointBoundary({
    list: [['lng', 'lat'],['lng', 'lat'],['lng', 'lat'],['lng', 'lat']],
    key: 'boundary_id'
});
```

上面的方法应该是很好理解：

我们对传入的列表经纬度执行 `drawDefault()` 的方法，绘制地图，这个方法下面来看。

绘制成功后，我们可以对对应的key 进行背景色，线颜色的改造。

```js
// 默认值绘制
drawDefault({ data, id }) {
    if (this.amap && this.edit) {
        if (data.length > 2) {
            const list = [...data]
            const eBegin = JSON.stringify(data[0])
            const eEnd = JSON.stringify(data[data.length - 1])
            if (eBegin !== eEnd) {
                list.push(data[0])
            }
            this.edit.draw.add({
                id,
                type: 'Feature',
                geometry: {
                    type: 'Polygon',
                    coordinates: [list],
                },
            })
        }
    }
}
```

这里要重点注意：绘制区域的话，最后一个经纬度一定要和第一个经纬度一致，这样地图才会认为这个是一个有闭环的区域，否则画不出区域来。

因此我们在写这个方法的时候，可以多做一层判断，如果前后点位不同的话，就帮它补上一条，保证绘制的成功性。