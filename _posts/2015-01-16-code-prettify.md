---
layout: post
title: 使用Prettify渲染代码高亮
description: Jekyll通过Pygments可以直接处理代码高亮，但是打破了markdown的格式，而使用google-code-prettify却小巧方便。
category: 技术分享
tags: [Prettify]
---
{% include JB/setup %}
# 使用Prettify渲染代码高亮

Prettify使用
1.下载代码

直接到[google-code-prettify](http://code.google.com/p/google-code-prettify/)官网下载代码，然后将它们放到项目下

2.引入css和js
<!--break-->

官网上说可以直接包含run_prettify.js的方式，这个会导入远程库，也可以自己导入，如下

    <link rel="stylesheet" href="/public/js/prettify/prettify.css">
    <script src="/public/js/prettify/prettify.js"></script>
    <script type="text/javascript">
      $(function(){
        $("pre").addClass("prettyprint linenums");
        prettyPrint();
      });
    </script>

这里导入了css和js后，就可以直接用markdown的tab的方式来导入代码段了

3.更换主题

默认主题不是很好看，只要更换prettify.css即可更换样式，可以到这里[下载](http://google-code-prettify.googlecode.com/svn/trunk/styles/index.html)自己喜欢的主题css即可

