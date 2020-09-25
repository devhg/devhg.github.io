---
title: HTML5+app实现文件上传(java后端)
date: 2019-08-13 20:18:59
tags:
  - Web前端
  - Java
categories:
  - HTML5+APP
---

以拍照上传，相册选择图片上传为例



### HTML5 Plus 拍照或者相册选择图片上传到服务器

<br>

起因：正在写一个人脸识别打卡签到的webApp，其中需要一个拍摄照片并上传服务器的功能

通过阅读[h5+官方文档](http://www.html5plus.org/doc)  了解到有相册 、相机、 文件上传等接口



#### 下面学习这几个api：



1、从图库选择图片

```javascript
plus.gallery.pick(
  function(path) {       // path为选择图片的路径
    // 下面将图片显示在界面
    hui('#img3 div').hide();
    hui('#img3 img').attr('src', path);
    hui('#img3 img').show();
    // 上传文件
    imgUpload(path);
  },
  function(e) {
    hui.toast("上传失败");
  }, {
    filter: 'image',
    system: false
  }
)
// 选择多张图片
plus.gallery.pick(
  function(paths){
    for(i in paths.files){
      hui.toast(paths.files[i]);
      //imgUpload(path);
    }
  },
  function(e){
    hui.toast('取消了选择');
  },{
    filter: 'image',
    multiple: true,
    maximum: 5
  }
);

```

<br>

2、相机获取图片

```javascript
plus.camera.getCamera().captureImage(
  function(path) {
    // 这个path 不能直接使用 是相对的  需要进行路径转换
    var url = "file://" + plus.io.convertLocalFileSystemURL(path); // 路径转换
    hui('#img2 div').hide();
    hui('#img2 img').attr('src', path);
    hui('#img2 img').show();
    imgUpload(path);
  },
  function(e) {
    hui.toast("上传失败");
  }, {
    index: 2 // 拍照时默认的摄像头 1后置 2 前置
  }
)
```

<br>

3、uploader文件上传

原理应该就是通过http 的post请求上传文件

```javascript
/**
 * 图片上传（java后端测试成功）
 * @param {Object} path
 */
function imgUpload(path) {
  plus.nativeUI.showWaiting();
  var task = plus.uploader.createUpload(
    'http://192.168.1.142:8080/img/upload.do', {
      method: "POST"
    },
    function(resp, status) {
      if (status == 200) {
        plus.nativeUI.closeWaiting();
        console.log(resp.responseText);
        mui.toast('上传成功');
      } else {
        mui.toast('上传失败');
        plus.nativeUI.closeWaiting();
      }
    }
  );
  task.addFile(path, {key: 'file'}); // 这里必须和 java后端的 @RequestParam(value = "file") 对应
  task.addData("name", "test");
  task.start();
}
```

代码就是上述所写，server为上传的服务端接口地址，如果上传成功，则回调的status会返回200，不成功或者接口参数有问题会返回400或者500。

resp.responseText  服务端返回的结果，一般服务端会返回json，解析一下json就可以使用了。

<br>

传输其他文件时如果还想添加其他参数，用.addData(key,value),

添加图片用.addFile(图片路径，{key:后端接收文件的名字})， **这个key必须和后端接收名字对应**

配合后端代码看会好理解，后端我用java接收的：

<br>

4、Java后端接收并保存

用标准的MultipartFile接收即可。注意xml限制的大小设定

springMVC.xml

```xml
<!--配置文件上传-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
  <property name="defaultEncoding" value="UTF-8" />
  <!--单位字节-->
  <property name="maxUploadSize" value="20480000"/>
</bean>
```



```java
@RequestMapping(value = "/upload", produces = "text/html;charset=utf-8")
@ResponseBody
public String uploadImg(@RequestParam(value = "file") MultipartFile file, HttpServletRequest request) {
  Map<String, Object> map = new HashMap<>();

  try {
    String oriFilename = file.getOriginalFilename();

    // 获取文件后缀
    String fileType = oriFilename.substring(oriFilename.lastIndexOf(".") + 1, oriFilename.length()).toLowerCase();
    // 存储路径
    String basePath = "/Users/qxqzx/Desktop/img/";
    // 保存的文件名字
    String saveName = String.valueOf(new Date().getTime()) + "." + fileType;
    File dst = new File(basePath, saveName);

    if (!dst.getParentFile().exists()) {
      dst.mkdirs();
    }
    file.transferTo(dst); // 写入本地

    map.put("success", true);
    map.put("code", "200");
    map.put("msg", "图片上传成功！");
  } catch (IOException e) {
    e.printStackTrace();
    map.put("success", false);
    map.put("code", "-200");
    map.put("msg", "图片上传失败！");
  }


  return JSON.toJSONString(map);

}

```



暂时总结到这, 等后续补充

