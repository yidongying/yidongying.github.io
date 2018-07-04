---
title: 使用hexo+github搭建个人博客(基础篇)
date: 2017-3-22
tags:
categories: 博客搭建
---
## 开始
本文为建站初期小结，查看完整建站教程：<a href = ''>Hexo+nexT主题搭建个人博客</a>

讲述Hexo的安装，nexT主题的下载及其简单配置。

### 安装HEXO
切换到博客所在目录，运行Git Bash，依次执行以下命令：


$ npm install -g hexo-cli
$ hexo init
$ npm install
### 指定博客文件夹的目录如下：


├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
### 运行：

$ hexo g
$ hexo s

在浏览器中输入：http://localhost:4000/即可访问本地博客，如下图所示：

参考: HEXO官方帮助文档

### 下载NexT主题
切换到博客所在目录，运行Git Bash，执行以下命令：

$  git clone https://github.com/iissnan/hexo-theme-next themes/next
打开站点配置文件_config.yml，将其中的theme: landscape改为theme: next，保存修改，执行hexo g,hexo s,在浏览器中查看本地博客如图：



### NexT主题简单配置
根据NexT官方帮助文档，选择scheme为Mist，界面语言为zh-Hans，选择侧栏位置为right，侧栏显示时机为post,添加站点图像，作者昵称，站点描述等基本信息，效果如图：

### 完成

至此，个人博客雏形基本显现，下一步进行优化。
