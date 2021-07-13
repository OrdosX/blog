---
title: 在基于nginx的vue项目中启用gzip预压缩
date: 2020-07-17 10:07:26
categories:
- 前端
tags:
- nginx
- vue
---
开启gzip压缩能提高网页的加载速度。基于nginx的gzip压缩有实时压缩和预压缩两种方式。在vue项目中，通过安装并启用compression-webpack-plugin，可以在`npm run build`的时候生成预压缩的`.js.gz`和`.css.gz`文件，减轻服务器的处理负担。

<!--more-->

## vue的配置

首先安装compression-webpack-plugin插件

```
npm i compression-webpack-plugin
```

在vue项目的根目录下（也就是有`package.json`的目录）新建一个名为`vue.config.js`的文件，输入以下内容

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin')
module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      config.plugins.push(
        new CompressionWebpackPlugin({
            test: /\.js$|\.html$|\.css$/,
            threshold: 4096
        })
      )
    }
  }
}
```

`threshold`是触发压缩的最小体积，如果设得过小可能越压越大，推荐像这个例子一样设置4kb

之后正常执行npm run build，可以在dist目录中看到同名的`.js.gz`和`.css.gz`文件，而且压缩效果可观。

{% asset_img result.jpg 生成效果 %}

## nginx的配置

网上很多互相转(chao)载(xi)的文章里面，将动态压缩的配置也加了进来，凡是配置中包含`gzip on;`的都是错误的配置。只要添加两行：`gzip_static on;`开启静态gzip，`gzip_vary on;`使CDN可以通过不同的请求头获取未压缩的文件（若无CDN可不写）

若要全局开启静态gzip，在server字段下配置如下所示（示例中省略无关设置）

```
server {
  gzip_static on;
  gzip_vary on;
}
```

若只在某个路径下开启静态gzip，就在该路径下配置如下所示

```
server {
    location /files/ {
      gzip_static on;
      gzip_vary on;
    }
}
```

完成后输入`sudo nginx -s reload`重载设置，然后通过浏览器的F12菜单可以看到响应头出现`Content-Encoding: gzip`字样，说明成功启用了预压缩gzip

{% asset_img header.jpg 响应头示例 %}
