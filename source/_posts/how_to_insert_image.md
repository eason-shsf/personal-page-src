---
title: 如何在hexo博客中插入图片
date: 2018-08-06 17:35:35
tags:
header-img: "/img/header.jpg"
---
*  首先在_config.yml文件中插入post_asset_folder: true
*  然后通过命令hexo new theName生成新的博客，此时生成的博客可以自带一个同名文件夹里面放入的图片，可以在md文档中当做当前路径的图片引用。  
   
*  配置为\!\[\]\(image.jpg\) 的图片显示如下:
![](image.jpg)  