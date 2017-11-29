###图片转pdf的一些记录

最近接到一个需求，是要求把所有要下载的图片转成一个pdf，进行下载。

首先我想到的是用[jspdf](https://github.com/MrRio/jsPDF)这个库来进行pdf的转化。这是一个挺强大的库，使用也简单。根据我的需求我用到方法是`addImage`这个方法

```
/**
 * imagesData 必须是dataUrl
 * type 图片类型JPEG PNG
 * left 左边的距离
 * top 上班的距离
 * width 图片宽度
 * height 图片高度
 */ 
addImage(imageData, type, left, top, width, height)
```
