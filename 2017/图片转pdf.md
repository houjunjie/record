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

因为图片是需要`dataUrl`格式，也就是base64的格式，最初的想法是用直接用canvas把图片转成base64格式。但因为图片尺寸太大了都是3000*3000左右的。而且有多张，这样直接转的会很慢。所以这个方法pass了。

然后我就找到了[html2canvas](https://github.com/niklasvh/html2canvas)。这个库大概的原理就是把dom节点在canvas画出来。然后转成dataUrl格式。不过这个库有一些问题，会有图片跨域问题，需要配置，还有一个问题是这个插件在配置里面设置了height,width是不起作用的。明明给了个配置参数居然没用，这就很尴尬了，上网搜了一下说要自己去修改源码，如下：

```
// return renderDocument(node.ownerDocument, options, node.ownerDocument.defaultView.innerWidth, node.ownerDocument.defaultView.innerHeight, index).then(function(canvas) {
var width = options.width != null ? options.width : node.ownerDocument.defaultView.innerWidth;
var height = options.height != null ? options.height : node.ownerDocument.defaultView.innerHeight;
return renderDocument(node.ownerDocument, options, width, height, index).then(function (canvas) {
```
配置代码如下：

```
var img = $('#img .img');
var downImgList = [];
for(let i=0; i<img.length;i++){
var height = img.eq(i).height()
html2canvas(img.eq(i), {
    taintTest: false,
    type:"view",
    useCORS: true, // 支持跨域
    background: '#ffffff', // 设置背景色为白色，因为如果不设置，生成的pdf背景是黑色的
    height:height,
    onrendered: function(canvas) {
        var contentWidth = canvas.width;
        var contentHeight = canvas.height;
        //一页pdf显示html页面生成的canvas高度;
        var pageHeight = contentWidth / 592.28 * 841.89;
        //未生成pdf的html页面高度
        var leftHeight = contentHeight;
        
        var pageData = canvas.toDataURL('image/jpeg', 1.0);
        downImgList.push({
            pageData: pageData,
            contentWidth: contentWidth,
            contentHeight: contentHeight
        })
        if(downImgList.length == img.length) {
            formatPdf(downImgList)
        }
    }
});
}
function formatPdf (lists){
    //注①
    var pdf = new jsPDF('', 'pt', 'a4');
    //a4纸的尺寸[595.28,841.89]，html页面生成的canvas在pdf中图片的宽高
    //一张图片一页
    lists.map(function(item, index) {
        // console.log(item)
        var imgWidth = 595.28;
        var imgHeight = 592.28 / item.contentWidth * item.contentHeight;
        pdf.addImage(item.pageData, 'JPEG', 0, 0, imgWidth,imgHeight);
        if(index < lists.length -1) {
            pdf.addPage();
        }
    })

    pdf.save('xxx.pdf');
}
```

一张图片生成pdf完全没问题。但是我有两张图片的时候就发现了html2canvas截图不全，第二张图片只截了一半，另一半没了。上网找了很多方法，发现并没有用，也只能pass这个插件了

方法可行，就是插件不行，所以我找到了另一个插件[dom-to-image](https://github.com/tsayen/dom-to-image), 这个插件拿来用就可以了，不用想html2canvas那样配置，直接上源码吧。没什么可说的。

```
var img = document.querySelectorAll('#img img');
var downImageLists = [];
for(let i=0; i<img.length;i++){
    domtoimage.toJpeg(img[i])
    .then(function (dataUrl) {
        var contentHeight = img[i].height;
        var contentWidth = img[i].width;
        downImageLists.push({
            pageData: dataUrl,
            contentWidth: contentWidth,
            contentHeight: contentHeight
        })
        if(downImageLists.length == img.length) {
            formatPdf(downImageLists) //如上述的一样
        }
    });
}
```
就这样完美解决了我这个需求，哈哈。不过据说这个插件如果dom节点多的话会很卡。但是我这边只是转图片，并不会很多dom节点。所以这个缺点没什么关系。

之后可能还需要把pdf转成图片，到时候踩到坑在记录一下。233333