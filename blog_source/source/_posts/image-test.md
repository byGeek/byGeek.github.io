---
title: image test
date: 2017-06-10 00:22:42
tags:
---

Update 2018/11/09

由于最近七牛云测试域名回收，图片需要绑定自定义域名，外链才正常。参考[绑定加速域名和域名解析流程](https://developer.qiniu.com/fusion/manual/1367/custom-domain-name-binding-process)已经绑定了本站点域名。



使用七牛云作为图床，使用七牛云的图片处理功能，可以对图片进行预处理，使用hexo-qiniu-sync插件可以在_config.yml文件中配置处理默认效果。比如

```
?imageView2/0/w/800/q/75|watermark/2/text/YnlHZWVr/font/YXJpYWw=/fontsize/360/fill/I0ZGRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim
```

控制图片width=800，等比缩放，加上bygeek水印等。只需在_config.yml文件中配置好默认效果之后，在markdown文件中使用一下标记语法即可对图片加上默认处理效果。

```
{% qnimg demo.jpg %}
```

{% qnimg demo.jpg %}

<!--more-->

如果想显示原图效果，markdown中标记如下：

```
{% qnimg demo.jpg normal:yes %}
```

效果即是原图效果，如下：

{% qnimg demo.jpg normal:yes %}

Hexo-qiniu-sync插件自动扫描_config.yml配置的目录local_dir，在执行`hexo s`的时候会自动将local_dir目录下的资源上传同步到七牛云。然后在markdown中直接使用图片名称即可。



更多的图片预处理语法请参考[七牛云图片处理](https://developer.qiniu.com/dora/manual/1279/basic-processing-images-imageview2)。

关于Hexo-qiniu-sync的配置等请参考[官方Github](https://github.com/gyk001/hexo-qiniu-sync)以及[Hexo七牛插件安装与使用](https://juejin.im/post/5a689d93f265da3e283a2fc1)。



