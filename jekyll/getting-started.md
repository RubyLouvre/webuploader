---
layout: post
title: 快速开始
name: Getting started
group: 'nav'
weight : 1
styles:
  - /css/webuploader.css
scripts:
  - /js/webuploader.js
  - /js/getting-started.js
---

## 引入资源

使用Web Uploader文件上传需要引入三种资源：JS, Css, Swf。您可以选择[下载]({{site.baseurl}}/download.html)本站默认包，也可以直接使用由[staticfile.org](http://www.staticfile.org)提供的在线文件。

```html
<!-- 最新压缩版样式文件 -->
<link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">

<!-- 最新压缩版JS文件 -->
<script src="//netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
```

## 简单文件上传

WebUploader只包含文件上传的底层实现，不包括UI部分。所以交互方面可以自由发挥，以下将演示如何去实现一个简单的版本。

<div id="uploader" class="wu-example">
    <div id="thelist" class="uploader-list"></div>
    <div class="btns">
        <div id="picker">选择文件</div>
        <button id="ctlBtn" class="btn btn-default">开始上传</button>
    </div>
</div>

### Html部分
首先准备dom结构，包括存放信息的容器、选择按钮和控制按钮三个部分。

```html
<div id="uploader" class="wu-example">
    <div id="thelist" class="uploader-list"></div>
    <div class="btns">
        <div id="picker">选择文件</div>
        <button id="ctlBtn" class="btn btn-default">开始上传</button>
    </div>
</div>
```

### 初始化Web Uploader
具体说明，请看一下代码中的注释部分。

```javascript
var uploader = WebUploader.create({

    // 不压缩image
    resize: false,

    // swf文件路径
    swf: BASE_URL + '/js/Uploader.swf',

    // 文件接收服务端。
    server: 'http://webuploader.duapp.com/server/fileupload.php',

    // 选择文件的按钮。可选。
    // 内部根据当前运行是创建，可能是input元素，也可能是flash.
    pick: '#picker'
});
```
### 显示用户选择
由于webuploader不处理UI逻辑，所以需要自己去监听`fileQueued`事件来实现。

```javascript
// 当有文件添加进来的时候
uploader.on( 'fileQueued', function( file ) {
    $list.append( '<div id="' + file.id + '" class="item">' +
        '<h4 class="info">' + file.name + '</h4>' +
        '<p class="state">等待上传...</p>' +
    '</div>' );
});
```

### 文件上传进度
文件上传中，Web Uploader会对外派送`uploadProgress`事件，其中包含文件对象和该文件当前上传进度。

```javascript
// 文件上传过程中创建进度条实时显示。
uploader.on( 'uploadProgress', function( file, percentage ) {
    var $li = $( '#'+file.id ),
        $percent = $li.find('.progress .progress-bar');

    // 避免重复创建
    if ( !$percent.length ) {
        $percent = $('<div class="progress progress-striped active">' +
          '<div class="progress-bar" role="progressbar" style="width: 0%">' +
          '</div>' +
        '</div>').appendTo( $li ).find('.progress-bar');
    }

    $li.find('p.state').text('上传中');

    $percent.css( 'width', percentage * 100 + '%' );
});
```

### 文件成功、失败处理

```javascript
uploader.on( 'uploadSuccess', function( file ) {
    $( '#'+file.id ).find('p.state').text('已上传');
});

uploader.on( 'uploadError', function( file ) {
    $( '#'+file.id ).find('p.state').text('上传出错');
});

uploader.on( 'uploadComplete', function( file ) {
    $( '#'+file.id ).find('.progress').fadeOut();
});

```


## 上传图片
与普通文件上传相比，此demo演示了，文件过滤，图片预览，图片压缩上传功能。

<div id="uploader-demo" class="wu-example">
    <div id="fileList" class="uploader-list">
    </div>
    <div id="filePicker">选择图片</div>
</div>


要实现如上demo，首先需要准备一个按钮，和一个用来存放添加的文件信息列表的容器。

```html
<!--dom结构部分-->
<div id="uploader-demo">
    <!--用来存放item-->
    <div id="fileList" class="uploader-list"></div>
    <div id="filePicker">选择图片</div>
</div>
```

然后创建Web Uploader实例

```javascript
// 初始化Web Uploader
var uploader = WebUploader.create({

    // 自动上传。
    auto: true,

    // swf文件路径
    swf: BASE_URL + '/js/Uploader.swf',

    // 文件接收服务端。
    server: 'http://webuploader.duapp.com/server/fileupload.php',

    // 选择文件的按钮。可选。
    // 内部根据当前运行是创建，可能是input元素，也可能是flash.
    pick: '#filePicker',

    // 只允许选择图片文件。
    accept: {
        title: 'Images',
        extensions: 'gif,jpg,jpeg,bmp,png',
        mimeTypes: 'image/*'
    }
});
```

由于Web Uploader并没有实现任何交互方面的功能，所以对应的文件选择列表需要自己动手。
只需监听Web Uploader的`fileQueued`事件便可以完成此功能。

```javascript
// 当有文件添加进来的时候
uploader.on( 'fileQueued', function( file ) {
    var $li = $(
            '<div id="' + file.id + '" class="file-item thumbnail">' +
                '<img>' +
                '<div class="info">' + file.name + '</div>' +
            '</div>'
            ),
        $img = $li.find('img');


    // $list为容器jQuery实例
    $list.append( $li );

    // 创建缩略图
    // 如果为非图片文件，可以不用调用此方法。
    // thumbnailWidth x thumbnailHeight 为 100 x 100
    uploader.makeThumb( file, function( error, src ) {
        if ( error ) {
            $img.replaceWith('<span>不能预览</span>');
            return;
        }

        $img.attr( 'src', src );
    }, thumbnailWidth, thumbnailHeight );
});
```

然后剩下的就是上传状态提示了，当文件上传过程中, 上传成功，上传失败，上传完成都分别对应`uploadProgress`,
`uploadSuccess`, `uploadError`, `uploadComplete`事件。

```javascript
// 文件上传过程中创建进度条实时显示。
uploader.on( 'uploadProgress', function( file, percentage ) {
    var $li = $( '#'+file.id ),
        $percent = $li.find('.progress span');

    // 避免重复创建
    if ( !$percent.length ) {
        $percent = $('<p class="progress"><span></span></p>')
                .appendTo( $li )
                .find('span');
    }

    $percent.css( 'width', percentage * 100 + '%' );
});

// 文件上传成功，给item添加成功class, 用样式标记上传成功。
uploader.on( 'uploadSuccess', function( file ) {
    $( '#'+file.id ).addClass('upload-state-done');
});

// 文件上传失败，显示上传出错。
uploader.on( 'uploadError', function( file ) {
    var $li = $( '#'+file.id ),
        $error = $li.find('div.error');

    // 避免重复创建
    if ( !$error.length ) {
        $error = $('<div class="error"></div>').appendTo( $li );
    }

    $error.text('上传失败');
});

// 完成上传完了，成功或者失败，先删除进度条。
uploader.on( 'uploadComplete', function( file ) {
    $( '#'+file.id ).find('.progress').remove();
});
```

更多细节，请查看[js源码]({{site.baseurl}}/js/getting-started.js)。
