
# 使用说明
本插件是一个云端上传插件，能够将本地的文件包括图片上传到云存储，ImageX是火山引擎推出的专业图像处理的一个服务；具体[引用三方评测](https://blog.csdn.net/weixin_44643524/article/details/112550247)
此外这个服务还提供了便捷的一站式图片上传、存储、处理、分发服务，并提供详尽的服务质量监控和数据报表能力。同时提供客户端图片加载SDK、服务端SDK和上传SDK等。

# 插件自身功能
ImageX功能表
|插件功能| 功能说明 |
|--|--|
| 图像上传 | 提供基本的图像上传、并行上传能力 |
| 文件上传 | 需要联系ImageX提交功能配置文件上传 |
| 上传后转换 | 图片可以转换成任意加载格式，包括avif、webp等格式 |
| 图片预览| 预览图片 |
| 上传性能监控 |传入appid可以查看上传监控指标，监控指标包括：如图|
![上传监控指标](https://img-blog.csdnimg.cn/20210402144153282.png)


# 配合ImageX可实现功能

##### 基础图像分发与管理
|   |   |
| --- | --- |
| 功能列表 | 支持状态 |
| 服务管理 | 10个 |
| 多域名管理 | 10个 |
| https支持 | 全球分发 |
| 域名调度 | 需要服务端SDK |
| 鉴权配置 | 支持图片URL级别鉴权防盗用 |
| 镜像回源 | 支持镜像功能 |
| 精简URL | 支持去参数url精简访问 |
| 回源header | 支持镜像配置header值 |
| 自定义header | 支持自定义header |
| 上传类型限制 | 上传类型限制 图片、文件、任意类型 |
| 资源管理 | 增删改查 |
| 资源禁用 | 禁止url |
| 刷新缓存 | 支持缓存刷新 |
| url拼接 |支持配置url拼接访问  |
| 模板导入带出 |支持样式导入导出  |
| 临时保存数据 | 支持图像管理临时保存的属性|


##### 基础图像实时处理
|  功能 |详细说明   |
| --- | --- |
| 图像格式转换 | 格式转换、GIF 格式优化、渐进显示，支持HEIF、AVIF等动静图格式 |
| 鉴权保护 | 用于水印、文字场景，防止被篡改 |
| 防盗链 | url层面防止盗链 |
| 原图获取 | 开关 |
| 压缩参数 | 0-·-100 |
| 亮度调节 | 0-·-100 |
| 对比度 | 0-·-100 |
| 旋转 | 360度旋转，字体、颜色、透明度 |
| 缩放 | 等比、按照样式缩放等，用于缩略图场景 |
| 裁剪 | 自定义裁剪，支持参数裁剪 |
| 图文水印 | 支持图像裁剪 |
| 内切圆裁剪 | 支持内切圆裁剪 |
| 高斯模糊 | 模糊半径设置 |
| 翻转 | 左右、上下 |
| 锐化 | 0--100 |
|图像信息|获取图片基本信息、主色调等|


##### 基础图像高级处理

**智能识别裁剪**

智能识别图像中的信息内容，然后应用对应的图像变换行为。


**高效压缩格式**

针对图像高压缩场景，推出高效率图像压缩格式HEIF和AVIF，相比于传统图像webp图像格式高出30%以上的压缩比。

针对动图场景HEIF压缩后的体积小2-10倍，同时支持自适应图像编码能力、去压缩失真等新的HEIF特性。


**漫画风格迁移**

通过访问url的方式能够进行图像风格的变化，特别适合社交场景。

**智能识别填充**

通过图像识别能够扩增图片实现智能填充效果。


**智能图像分割**

实现图片前后背景的分割。


# 本插件使用说明

## 1. 引入插件
```
<template>
    <view>
        <imagex-uploader v-bind='options' v-on:change='onUploadChange' />
    </view>
</template>
```

## 2. 准备工作
1. [开通ImageX](https://zjsms.com/e26S9vF/)后在控制台获取[服务ID（serviceID）](https://console.volcengine.com/imagex/service_manage/%29)、[appid](https://console.volcengine.com/baf/)等信息，serviceiD 是为了确定上传的空间、appid确定据监控指标性能信息，此处需要实名认证以及需要创建ImageX的服务（若无）；
2. 上传前需要从服务端获取上传sts2签名，由于签名计算放在前端会暴露 AccessKey和SecretKey，所以签名计算过程放在后端实现（利用签名SDK可以生成临时的 AK、SK等 ），前端通过 http 请求向后端获取签名结果。[签名算法服务端接入](https://zjsms.com/e26JarS/)；
3. 小程序上传需要把网关地址和上传地址添加到小程序的访问白名单中

| 字段名  | 域名  |
| ------------ | ------------ |
|  request 域名 | https://imagex.volcengineapi.com  |
|  uploadFile 域名 | https://tos-lf-x.snssdk.com <br> https://tos-hl-x.snssdk.com <br> https://tos-nc2-slb2.bytecdn.cn <br> https://tos-nc2-slb1.bytecdn.cn |


    <script>
        export default {
            data() {
                return {
                    options: {
                        serviceId: 'your serviceId',
                        stsToken: { /* your stsToken */ }
                }
            },
            methods: {
                onUploadChange(info) {
                    console.log(info)
                }
            }
        }
    </script>

## 3. 参数说明


| 参数 | 说明 | 类型 | 默认值 | 是否必须 |
| -- | -- | -- | -- | -- |
| serviceId | 服务ID | String | '' | 是 |
| stsToken | sts签名 | Object | {} | 是 |
| region | 上传区域 | 'cn'、'us'、'sg' | 'cn' | 否 |
| appId | 应用id，用于定位日志 | Number | null | 否 |
| userId | 用户id，用于定位某一用户日志 | String | null | 否 |
| previewStyle | 预览图样式 | Object | { width: '162rpx', height: '162rpx' } | 否 |
| uploadStyle | 上传样式 | Object | { width: '162rpx', height: '162rpx' } | 否 |
| uploadLimit | 上传数量限制 | Number | 9 | 否 |
| autoUpload | 是否自动上传 | Boolean | true | 否 |
| enableMove | 是否显示移除 | Boolean | true | 否 |
| defaultImageList | 默认预览图片 | Array | [] | 否 |


## 3. 上传完毕后

上传完毕后会获取图片的唯一ID：例如：tos-cn-i-1upkr0djo9/e49f9137ea714be13af8708857d2a5ef.jpeg， 则按照官方要求访问格式为：

> http://wpstatic.e7e7e7.com/tos-cn-i-1upkr0djo9/e49f9137ea714be13af8708857d2a5ef.jpeg~tplv-1upkr0djo9-heic-v3.webp

访问格式：

> 图片地址访问规则: http(s): / /域名/图片URI~模板配置, 其中如上的.webp 可以替换成任意图片格式，比如avif、jpeg、png，ImageX 是都是可以实时处理转换的；详情可以去ImageX研究一下；
