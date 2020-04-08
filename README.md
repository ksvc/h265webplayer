# h265webplayer

h265webplayer是金山云的Web端H.265视频播放器，该播放器Web SDK让您可以在支持[WebAssembly](https://caniuse.com/#search=WebAssembly)的浏览器上播放FLVhttp-flv协议的直播视频，mp4格式的点播视频。

![avatar](https://ks3-cn-beijing.ksyun.com/ksplayer/h265/mp4_demo/h265player.png)

## 支持的功能
1、flv格式的直播。
2、mp4格式的点播（音频需是aac格式的）。

## demo 有两种访问方式 

### mp4 demo 访问方式
1、ks3直接访问，链接如下：
```
https://ks3-cn-beijing.ksyun.com/ksplayer/h265/mp4_demo/index.html
```

### flv demo 访问方式

1、ks3直接访问，链接如下：
```
https://ks3-cn-beijing.ksyun.com/ksplayer/h265/outside_demo/v1.1.3/index.html
```

2、获取压缩包后本地创建服务访问，步骤如下：

## 播放器Demo压缩包地址
### flv demo zip
```
https://ks3-cn-beijing.ksyun.com/ai-kie/sdk/h265-pc/h265-pc.zip
```

## 播放器Demo运行说明

### 0. 安装npm包管理器

参见：[Node.js官网](https://nodejs.org/en/)

### 1. 安装http服务器
```
    npm install http-server -g
```

### 2. 启动服务

```
 cd <demo directory>
 npm run start 
```

### 浏览器访问

```
    http://localhost:8000 
```

*说明：* 请替换页面中的拉流地址进行测试

## token鉴权的两种方式

#### token的意义
用于鉴权，验证用户是否拥有访问的权利以及访问的时长

#### 如何获取token
先与商务沟通达成协议后，产品会根据需求提供一个对应的token

### 第一种播放器鉴权token 

```js
    let player = h265js.createPlayer({
    }, 
    {
        token: '8145ee1ec12300ef84f4cd5bcdg84m05'
    });
```
使用：token通过创建player时option中的token字段传入即可使用

过期：一旦token过期，h265 pc sdk将无法使用，过期通知将通过error事件给出

### 第二种 H265Decoder鉴权token
```javascript
let token = '8145ee1ec12300ef84f4cd5bcdg84m05';
qy265decoder.QY265DecoderCreate(null, token, returnCode);
```
使用：token通过创建decoder时的第二个参数传入即可使用

过期：一旦token过期，h265 pc sdk将无法使用，在创建decoder之后检查returnCode值如果等于qy265decoder.QY_TOKEN_INVALID，即为token 过期

```javascript
if(getValue(returnCode, 'i32') === qy265decoder.QY_TOKEN_INVALID) { 
    /*token 过期*/ 
}
```

## 集成h265解码器有两种方式

1、直接使用金山自研的h265播放器（推荐）
2、基于H265Decoder开发使用

## 第一种方式：使用h265播放器，初始化参数配置

### 区分直播点播的参数：isLive （默认值：false）

```js
    let player = h265js.createPlayer({
        isLive: false 
    }, 
    {
        wasmFilePath: 'http://localhost:8000/libqydecoder_0531.wasm',
        url: 'http://localhost:8000/720p_60fps.flv'
    });
```

其中wasmFilePath参数用于指定wasm解码库的位置。

### 是否允许跳帧参数：enableSkipFrame （默认值：false）

在解码器能力不够时，可以将enableSkipFrame 设置为true。此时会跳过一些不重要的非参考帧不进行解码，以便能流畅播放。

```js
    let player = h265js.createPlayer({

    }, 
    {
        url: 'http://localhost:8000/720p_60fps.flv',
        enableSkipFrame: false,
        wasmFilePath: 'http://localhost:8000/libqydecoder_0531.wasm'
    });
```

### 是否启用GPU渲染YUV图像参数： enableYUVrender （默认值：true）

当enableYUVrender为true时，通过WebGL直接渲染YUV图像数据（YUV到RGBA颜色空间的转换在GPU上计算）。

```js
    let player = h265js.createPlayer({

    }, 
    {
        url: 'http://localhost:8000/720p_60fps.flv',
        enableYUVrender: true,
        wasmFilePath: 'http://localhost:8000/libqydecoder_0531.wasm'
    });
```

### 缓存队列相关参数

通过maxLength4ToBeDecodeQueue和maxLength4ToBeRenderQueue参数可以设置待解码缓存队列和待渲染缓存队列（已解码帧队列）的最大长度，以便有效控制内存占用。

```js
    let player = h265js.createPlayer({
    }, 
    {
        url: 'http://localhost:8000/720p_60fps.flv',
        maxLength4ToBeDecodeQueue: 5 * 60, //待解码队列中NALU的最大长度
        maxLength4ToBeRenderQueue: 30, //待渲染帧队列的最大长度
        wasmFilePath: 'http://localhost:8000/libqydecoder_0531.wasm'
    });
```
### 卡顿时长设置： timeToDecideWaiting （默认值：500, 单位：ms）

```js
    let player = h265js.createPlayer({
    }, 
    {
        timeToDecideWaiting: 500
    });
```
### 启播前缓冲视频时长： bufferTime  （默认值：500, 单位：ms）

```js
    let player = h265js.createPlayer({
    }, 
    {
        bufferTime: 500
    });
```

## 播放器相关事件

通过`on`方法可以监听播放器相关事件，`off`方法可以取消监听。

示例代码

```js
    /**
     * @param {object} event player 播放器对象
     * @param {object} data  事件相关信息
     */
    player.on("ready",function(event, data) {
        //Do something   
    });
```

播放器事件列表

```js
h265js.Events = {
    READY: 'ready',                                 //播放器初始化完毕可以播放视频时触发
    PLAY: 'play',                                   //视频由暂停恢复为播放时触发
    PAUSE: 'pause',                                 //视频暂停时触发
    RELOAD: 'reload',                               //音视频时间戳相差超过maxAVTimeStampGap值时触发，建议重新拉流时
    LOADSTART: 'loadStart',                         //播放器开始加载数据时触发
    MEDIAINFO: 'mediaInfo',                         //媒体详细信息
    LOADEDEND: 'loadedEnd',                         //播放器加载到第一帧数据后触发
    VOLUMECHANGE: 'volumechange',                   //当音量改变时触发
    ERROR: 'error',                                 //发生错误时触发
    WAITING: 'waiting',                             //出现卡顿，需要缓存下一帧数据时触发
    PLAYING: 'playing',                             //播放时触发
    STATISTICSINFO: 'statisticsInfo'                //数据统计信息
}
```
## 第二种方式：基于H265Decoder解码器

### JS接口说明

#### 初始化

```js
import H265Decoder from '<decoder direcotory >/h265decoder.js';

let config = {
    wasmFilePath: 'http://localhost:8000/libqydecoder.wasm',
    enableSkipFrame: true
};
let decoder;

//加载并编译wasm解码库
H265Decoder.compileWasmInterfaces(config.wasmFilePath, function () {
    decoder = new H265Decoder(config);
    //设置解码回调
    decoder.set_image_callback(onDecodedFrameCallback);
}

```

#### 向解码器队列送数据

```js
 decoder.toBeDecodeQueue.push({
     nalu: new Uint8Array(naluData),  // naluData 为 ArrayBuffer类型数据
     pts: 0,  //展示时间戳
     isDroppable: false });  // 表示是否可以跳帧
```

- decoder的toBeDecodeQueue属性为保存待解码数据的数组，需自行控制待解码队列缓冲区的长度，避免内存溢出
- decoder会自动取出toBeDecodeQueue中的数据给底层wasm解码器，并在解码后自动释放传入wasm解码器的待解码数据

#### 设置解码输出图像回调 set_image_callback 

```js
    decoder.set_image_callback((image) => {
        let w = image.get_width(); //获取图像宽度
        let h = image.get_height(); //获取图像高度
        let pts = image.get_pts();  //获取图像pts时间戳

        // let image_data = new Uint8ClampedArray(w * h * 4);
        // for (let i = 0; i < w * h; i++) {
        //     image_data[i * 4 + 3] = 255;
        // }
        // //转换image中的YUV数据到image_data中的RGB数据
        // image.transcode(image_data);

        //优化： 直接返回YUV数据
        let yuvData =  image.getYuvDataNew(); //Uint8Array

    });
    
```

**说明：** 解码回调函数的参数image为Image类型，参见h265decoder.js中的定义

#### 其他接口


##### 暂停解码 

```js
decoder.pause();
```

##### 恢复解码 

```js
decoder.resume();
```

##### 检查是否为暂停状态 

```js
    if(decoder.isPaused()) {}
```

##### 启动解码器

```js
decoder.start();
```

**说明:** 默认情况初始化解码器时会自动调用启动解码器

##### 销毁解码器

```js
decoder.free();
```

##### 接收累积跳帧数通知

```js
decoder.on('skip_frame, (skippedframecount) => {
    console.log(skippedframecount);
});
```

### wasm解码器接口说明

#### 接口函数返回码说明
```js
const qy265decoder = {
    QY_OK : (0x00000000),          // Success
    QY_FAIL : (0x80000001),        //  Unspecified error
    QY_OUTOFMEMORY : (0x80000002), //  Ran out of memory
    QY_POINTER : (0x80000003),     //  Invalid pointer
    QY_NOTSUPPORTED : (0x80000004),//  Not support feature encoutnered
    QY_AUTH_INVALID : (0x80000005), //  authentication invalid
    QY_SEARCHING_ACCESS_POINT : (0x00000001), // in process of searching first access point
    QY_REF_PIC_NOT_FOUND : (0x80000007), // encode complete
    QY_NEED_MORE_DATA : (0x00000008),  // need more data 
    QY_BITSTREAM_ERROR : (0x00000009), // detecting bitstream error, can be ignored
    QY_TOKEN_INVALID : (0x0000000A)    //token invalid
```

错误码分为三大类状态：

+ 等于0： QY_OK，表示完全正常
+ 大于0：虽然当前不能正常解码，但解码器本身并没有出错
+ 小于0：解码器本身发生了一些异常和错误

#### 创建解码器 QY265DecoderCreate

```js
let returnCode = _malloc(4);  //为返回码分配空间
setValue(returnCode, 0, "i32"); // 返回码设置为0
let decoder = qy265decoder.QY265DecoderCreate(null, token, returnCode);
if(getValue(returnCode, 'i32') === qy265decoder.QY_OK) { /*解码器创建成功*/ }
```

#### 销毁编码器 QY265DecoderDestroy

```js
qy265decoder.QY265DecoderDestroy(decoder);
```

+ 参数
   + decoder：    通过QY265DecoderCreate接口创建的解码器实例

#### 解码一个NAL单元 QY265DecodeFrameEnSkip
```js
qy265decoder.QY265DecodeFrameEnSkip(decoder, naluData, length, returnCode, pts, shouldSkip);
```
+ 参数
   + decoder：    通过QY265DecoderCreate接口创建的解码器实例
   + naluData：   NAL单元数据，Uint8Array类型
   + length：     NAL单元数据字节数
   + returnCode： 返回码，returnCode对应地址保存0(QY_OK)表示正常
   + pts:         NAL单元对应的显示时间标签
   + shouldSkip:  true/false, 表示是否应该跳过该NAL单元的解码 

#### 获取解码输出 QY265DecoderGetDecodedFrameEm

```js
let frame = qy265decoder.QY265DecoderGetDecodedFrameEm(decoder, returnCode, 0);
```

+ 参数
   + decoder：    通过QY265DecoderCreate接口创建的解码器实例
   + returnCode： 返回码
+ 返回值： 解码输出帧

#### 判断解码输出帧是否有效 QY265DecoderGetFrameValid

```js
let framevalid = qy265decoder.QY265DecoderGetFrameValid(frame);
```
+ 参数
   + frame:   QY265DecoderGetDecodedFrameEm接口返回的解码输出帧 
+ 返回值： 解码输出帧是否有效, 1为有效

#### 归还解码输出帧 QY265DecoderReturnDecodedFrame
通知解码器释放该帧占用的相关内存

```js
qy265decoder.QY265DecoderReturnDecodedFrame(decoder, frame); 
```

+ 参数
    + decoder： 通过QY265DecoderCreate接口创建的解码器实例
    + frame：QY265DecoderGetDecodedFrameEm接口返回的解码输出帧

#### 获取解码输出帧的宽度 QY265GetFrameWidth

```js
    let width = qy265decoder.QY265GetFrameWidth(frame, 0);
```

+ 参数
    + frame：QY265DecoderGetDecodedFrameEm接口返回的解码输出帧
+ 返回值： 解码输出帧的宽度

#### 获取解码输出帧的高度 QY265GetFrameHeight

```js
    let height = qy265decoder.QY265GetFrameHeight(frame, 0);
```

+ 参数
    + frame：QY265DecoderGetDecodedFrameEm接口返回的解码输出帧
+ 返回值： 解码输出帧的高度

#### 获取解码输出帧的显示时间戳 QY265GetFramePts

```js
    let pts = qy265decoder.QY265GetFramePts(frame, 0);
```

+ 参数
    + frame：QY265DecoderGetDecodedFrameEm接口返回的解码输出帧
+ 返回值： 解码输出帧的显示时间戳

#### 获取解码输出帧的YUV某个分量 QY265DecoderGetFramePlane

```js
    let stride = _malloc(2);
    let y = qy265decoder.QY265DecoderGetFramePlane(frame, 0, stride);
    let u = qy265decoder.QY265DecoderGetFramePlane(frame, 1, stride);
    let v = qy265decoder.QY265DecoderGetFramePlane(frame, 2, stride);
```

+ 参数
    + frame：QY265DecoderGetDecodedFrameEm接口返回的解码输出帧
    + index: YUV分量索引，0表示Y分量，1表示U分量，2表示v分量

+ 返回值： YUV某个数据分量的数组，格式为Uint8Array

#### Flush解码器 QY265DecodeFlush

因为解码NAL单元与获取解码输出为异步关系, 所以解码器中可能存在剩余尚未解码完成的若干NAL单元. 调用本函数将使解码器完成所有已经输入的NAL单元的解码. 一般在码流结束或者播放器拖曳时使用.

```js
qy265decoder.QY265DecodeFlush(decoder, bClearCachedPics, returnCode);
```

+ 参数
    + decoder： 通过QY265DecoderCreate接口创建的解码器实例
    + bClearCachedPics: 是否清除缓冲的图像. 在码流结束时, 置为false, 不清除, 得到所有输出帧；在播放器拖曳或其他情况下, 置为true, 清除之前的图像帧, 重新开始
    + returnCode: 返回码，returnCode对应地址保存0(QY_OK)表示正常


## YUV渲染器文档

### 初始化YUV渲染器

```js
import WebGLCanvas from 'yuvrender.min.js'; // yuvrender.min.js在压缩包中的demo目录下

let webGLCanvas = new WebGLCanvas({
                    canvas: this.canvas, //传入一个canvas
                    width: width,        //视频帧宽度
                    height: height       //视频帧高度
                });
```
+ 备注：视频会根据它真实的宽高比自适应视频容器canvas的宽高
```js
<canvas id="videoDisplayCanvas" width="1600" height="600">
```

### 渲染一帧YUV数据

```js
// 假设yuvData为解码输出的一帧图像的YUV表示（Uint8Array类型）
let ylen = width * height; //视频宽高
let uvlen = (width / 2) * (height / 2);
webGLCanvas.drawNextOutputPicture({
    yData: yuvData.subarray(0, ylen),
    uData: yuvData.subarray(ylen, ylen + uvlen),
    vData: yuvData.subarray(ylen + uvlen, ylen + uvlen*2)
});
```

### 获取解码输出的图像帧的YUV表示

参见: [设置解码输出图像回调 set_image_callback](#设置解码输出图像回调-setimagecallback) 

author: xuyang  
email: xuyangfe@163.com  
qq: 1224657942  
