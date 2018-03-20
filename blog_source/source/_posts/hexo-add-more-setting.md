---
title: hexo添加第三方功能支持
date: 2018-03-20 10:33:13
tags:
- hexo
categories:
- hexo
keywords:
description:

---



hexo添加第三方插件，可以实现很多功能，如seo优化，生成feed等。



<!--more-->



## SEO优化

seo优化简单的说如何让搜索引擎收录你的网站。参考[此博文](https://segmentfault.com/a/1190000009254968)。

需注意一点的是，因为网站未备案的原因，直接使用http无法访问自己的博客。而hexo中自动生成的sitemap都是默认以http来访问，导致提交sitemap到谷歌的时候，会无法抓取。所以我手动修改了一下sitemap的模板文件。



- hexo-generator-sitemap

  在`node_modules\hexo-generator-sitemap\sitemap.xml`中，手动加入`https://`

  ```xml
  <loc>https://{{ post.permalink | uriencode }}</loc>
  ```

- hexo-generator-baidu-sitemap

  在`node_modules\hexo-generator-baidu-sitemap\baidusitemap.ejs`中，手动加入`https://`

  ```xml
  <loc>https://<%- encodeURI(url + post.path) %></loc>
  ```



## 添加feed

安装feed插件

```bash
npm install hexo-generator-feed --save
```

在站点_config.yml文件中添加

```json
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content: true
  content_limit: 140
  content_limit_delim: ' '
```





## 添加评论系统

Valine是一款极简的评论系统，基于leancloud。具体参考[此博文](https://ioliu.cn/2017/add-valine-comments-to-your-blog/)。



## 参考链接

- [SEO](https://segmentfault.com/a/1190000009254968)
- [Feed github](https://github.com/hexojs/hexo-generator-feed)

