# 使用说明
ImageX图片服务提供了便捷的一站式图片上传、存储、处理、分发服务，并提供详尽的服务质量监控和数据报表能力。同时提供客户端图片加载SDK、服务端SDK和上传SDK等。
## 1. 引入插件
```
<template>
    <view>
        <imagex-uploader v-bind='options' v-on:change='onUploadChange' />
    </view>
</template>
```

## 2. 准备工作
1. 开通ImageX后在控制台获取serviceId
2. 上传前需要从服务端获取上传sts2签名，由于签名计算放在前端会暴露 AccessKey和SecretKey，所以签名计算过程放在后端实现（利用签名SDK可以生成临时的 AK、SK等 ），前端通过 http 请求向后端获取签名结果。签名算法服务端接入可参考 https://www.volcengine.com/docs/508/20045
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
| previewStyle | 预览图样式 | Object | { width: '162rpx', height: '162rpx' } | 否 |
| uploadStyle | 上传样式 | Object | { width: '162rpx', height: '162rpx' } | 否 |
| uploadLimit | 上传数量限制 | Number | 9 | 否 |
| autoUpload | 是否自动上传 | Boolean | true | 否 |
| enableMove | 是否显示移除 | Boolean | true | 否 |
| defaultImageList | 默认预览图片 | Array | [] | 否 |



