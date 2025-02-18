---
title: 自己魔改主题
date: 2025-02-13 21:22:23
categories: Hexo
tags: 
    - Hexo
description: 本站样式的魔改记录
---

## 2025.2.13

照着博客作者的教学视频一点点修改本站的样式，大佬编写的主题样式精美，录制的教程详细丰富。更重要的是，还提供了自己魔改主题的方法。

为了管理博客站的文章，我先后创建了分类(categories)和标签(tags)两个文件夹，并把它们加入到菜单目录栏中。但是从菜单栏点击进去，两者只简单地列在了一个div里，如下图:

![](/images/articles/categories_20250213.png)

经过一番仔细甄选（考虑到自己不忍直视的前端水平），决定将分类界面修改成[本站主题作者预设的图片预览卡片](https://hexo-theme-async.imalun.com/demosite/customize_page/)的样式。

```yml
# _config.async.yml
layout:
   page_category: async/page_category.ejs
```
在原本的col-lg-12盒子下面加了一个盒子，用来为每个类型分配一个图片盒子；又将图片的宽度调整到80%，不然图片会横向铺满，略显丑陋

```ejs
<!-- layout/page_category.ejs -->

<% if (site.categories.length){ %>
      <% site.categories.sort('-posts.length').each(function(item){ %>
       <div class="col-lg-4">
         <a data-fancybox="portfolio" href="<%- config.root %><%- item.path %>" class="trm-portfolio-item trm-scroll-animation">
           <div class="trm-cover-frame">
             <img class="trm-cover" style="left:50%;width:80%;height:80%;transform: translate(-50%);" src="/images/categories/<%= item.name %>.svg" alt="item">
           </div>
           <div class="trm-item-description">
             <h6><%= item.name %></h6>
             <div class="trm-zoom-icon"><i class="fas fa-search-plus"></i></div>
           </div>
         </a>
       </div>
       <% }); %>
   <% } %> 
```
改完以后的样子 
![](/images/articles/categories_20250213_after.png)