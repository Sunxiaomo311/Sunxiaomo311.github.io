---
title: 【hexo插件】利用hexo-calendar实现仿GitHub的活动日历功能
comments: true
excerpt: 利用hexo-calendar实现仿GitHub的活动日历功能,并进行个性化改动
date: 2020-08-11 23:25:27
tags:
  - hexo插件
categories:
  - 编程
  - 前端
---

![](7.png)

>在个人网站首页添加类似github中的活动日历功能，实现步骤如下:

<br/>

### 1. 在博客根目录引入[hexo-calendar](https://github.com/HCLonely/hexo-calendar/blob/master/README_CN.md)插件

```
npm i hexo-calendar -S
```

*注意：引入前请注意插件作者给出的前提条件：*
![](10.png)

<br/>

### 2. 在首页文章md文件中添加如下标签代码：

```
{% calendar %}
{"monthLang": "cn", "dayLang": "cn", "title": "活动日历"}
{% endcalendar %}
```
<br/>

### 3. 启动hexo服务，在本地查看效果

```
hexo clean && hexo s
```
访问网页后，发现活动日历已经能正常显示了

![](3.png)
但以强迫症的眼光看，这个显示效果有点不尽人意。
1. 只显示了十个月的数据。这个看了作者给出的插件文档，默认是显示40周的数据，需要更改标签中的weeks属性。
2. 有活跃度的日期单元格颜色与主题不搭配，希望改成自定的颜色。

所以接着修改细节问题：

<br/>

### 4. 更改标签中的属性值
根据文档提示：

![](8.png)

修改标签weeks属性为一年52周（注意：严格遵守json语法格式）：
```
{"monthLang": "cn", "dayLang": "cn", "title": "活动日历"，"weeks":52}
```

修改完成后又发现一个问题，增加了两个月的单元格后，插件大小显示的有问题，还需要更改宽度。

![](4.png)

宽度使用width属性更改。实际上宽度修改到足够包裹插件单元格后，又可能会被外层div遮挡。所以最后决定通过用更改每个单元格大小的方式来使插件总体积缩小，完整显示一年的活跃度：

1. 打开博客根目录下的node_modules/hexo-calender/index.js文件。
2. 找到  `cellSize: [13, 13]` 一行,更改为`cellSize: [11, 11]`。
![](6.png)
3. 更改标签代码宽高属性。
```
{"width":"640", "height":"140", "weeks":52, "monthLang": "cn", "dayLang": "cn", "title": "活动日历"}
```

<br/>

### 5. 更改活动单元格颜色

同样也是打开博客根目录下的node_modules/hexo-calender/index.js文件，更改`color: ["#ebedf0", "#c6e48b", "#7bc96f", "#239a3b", "#196127"]`一行中的后四个颜色值。

我的网站主题色是蓝色系，所以用f12打开网页开发者工具，用取色器获取背景图片中不同层次的蓝色。
![](1.png)

全部修改完成后，刷新重启服务，显示效果如下：
![](2.png)

**完美 😂**

<br/>

### 6. 上传部署在GitHub的博客网站

有两点需要注意的，作者已经给出提示了：

>注意：此插件会和hexo g命令冲突，请使用hexo ge或hexo generate替代hexo g命令！

以及

>如果你使用了Travis CI, Github Action之类的自动部署，那么你需要在推送源码之前使用hexo gc -w=40命令生成一个calendar.json文件。-w=40代表显示 40 周之前至今的活动记录。

我改动了weeks的值为52，所以最后推送更新的指令是：

```
hexo gc -w=52

hexo generate -d
```
