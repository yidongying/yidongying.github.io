---
title: 使用hexo+github搭建个人博客(进阶篇)
date: 2017-3-23
tags:
categories: 博客搭建
---
1. Hexo简介
Hexo 是一款基于 Node.js 的静态博客框架。Hexo 使用 Markdown 解析文章，用户在本地安装Hexo并进行写作，通过一条命令，Hexo即可利用靓丽的主题自动生成静态网页。
参考：[Hexo-Github地址](https://github.com/hexojs/hexo)        [Hexo帮助文档](https://hexo.io/zh-cn/docs/)
![这里写图片描述](https://img-blog.csdn.net/20180704101848462?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MjEwNTIz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2. 博客环境搭建
2.1 安装Git
Windows平台：以 Win7 64位机为例

到[官网](https://git-scm.com/download)下载 Git，一路默认选项安装。本文使用的是Git-2.8.1-64-bit. 

Linux平台

2.2 安装Node.js
Windows平台：以 Win7 64位机为例

到官网下载 Node.js，一路默认选项安装。本文使用的是node-v4.4.2-x64，需要的用户可以点此下载 。

Linux平台

2.3 安装Hexo
Git 和 Node.js 都安装好后,首先创建一个用于存放博客文件的文件夹，如 blog，然后进入 blog 文件夹，下面开始安装并使用 Hexo。

安装并初始化Hexo

右键选择 git bash here ，弹出Git Bash窗口；执行命令：

```
$ npm install -g hexo-cli
$ hexo init
```

安装完成后，指定文件夹的目录如下：
```
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

其中**_config.yml**文件用于存放网站的配置信息，你可以在此配置大部分的参数；**scaffolds**是存放模板的文件夹，当新建文章时，Hexo会根据**scaffold**来建立文件；**source**是资源文件夹，用于存放用户资源，**themes**是主题文件夹，存放博客主题，Hexo 会根据主题来生成静态页面。

**生成静态博客文件**

在Git Bash终端执行命令：

```
$ hexo g
$ hexo s
```

Hexo将source文件夹中的Markdown 和 HTML 文件会被解析并放到public文件夹中，public文件夹用于存放静态博客文件，相当于网站根目录。
至此博客雏形基本完成，在浏览器中访问http://localhost:4000/，如图所示：

![002](https://img-blog.csdn.net/20180704100855165?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MjEwNTIz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.4 使用nexT主题
**下载nexT主题**

在Git Bash终端执行以下命令：


```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

解压所下载的压缩包至站点的 themes 目录下， 并将解压后的文件夹名称更改为 next 。本文使用hexo-theme-next-5.0.1，需要的用户可以点此下载 。

**启用nexT主题**

打开站点配置文件 _config.yml，找到 theme 字段，并将其值更改为 next。

```
theme: next
```

在Git Bash终端执行命令hexo s，在浏览器中访问http://localhost:4000/，当你看到站点的外观与下图所示类似时即说明你已成功安装 NexT 主题。这是 NexT 默认的 Scheme —— Muse。

![003](https://img-blog.csdn.net/20180704101136590?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MjEwNTIz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
本博客使用的是NexT.Pisces主题，修改主题配置文件 _config.yml的 Schemes 字段的值为：

```
scheme: Pisces
```

博客预览如图：

004

3. NexT主题配置
3.1 主题基本设定
参照[NexT使用文档](http://theme-next.iissnan.com/getting-started.html#theme-settings)，设置界面语言、菜单、侧栏、头像、作者昵称和站点描述。由于该使用文档描述非常详细，本文不再赘述。此处需要注意，添加新的菜单项时，需要手动创建该页面才能正常访问，下面以分类页面为例讲述创建新页面的方法：

**创建分类页面**

在Git Bash终端执行命令:

```
$ hexo new page categories
```

**编辑分类页面**

添加页面类型字段，将其值设置为 "categories"，主题将自动为这个页面显示所有分类，如果有启用多说 或者 Disqus 评论，默认页面也会带有评论。需要关闭的话，请添加字段 comments 并将值设置为 false。
```
title: 分类
date: 2014-12-22 12:39:04
type: "categories"
comments: false
```
创建标签页的方法同上，只需要将type字段设置为"tags"即可。

3.2 添加侧栏社交链接和友链
添加侧栏社交链接

在主题配置文件 _config.yml中Sidebar Settings部分添加字段：

```
# Social Links
social:
  GitHub: https://github.com/wuxubj
  Weibo: http://weibo.com/wuxubj
```

本博客将侧栏社交链接设置居中显示，修改themes\next\source\css\_common\components\sidebar\sidebar-author-links.styl文件，添加如下样式：
```
.links-of-author-item {
  text-align: center;
  }
```

添加侧栏友情链接

在主题配置文件 _config.yml中Sidebar Settings部分添加字段：


```
# Blogrolls
links_title: 友情链接
links_layout: inline
links_icon: link  # 设置图标
links:
  鱼汐笔记: https://yidongying.github.io/
```

本博客侧栏友情链接使用了与侧栏社交链接相同的css样式，但文本左对齐。实现方法为：
修改themes\next\layout\_macro\sidebar.swig，将如下内容

```
<ul class="links-of-blogroll-list">
  {% for name, link in theme.links %}
    <li class="links-of-blogroll-item">
      <a href="{{ link }}" title="{{ name }}" target="_blank">
        {{ name }}
      </a>
    </li>
  {% endfor %}
</ul>
```

修改为：

```
{% for name, link in theme.links %}
  <span class="links-of-author-item" style="text-align:left">
    <a href="{{ link }}" title="{{ name }}" target="_blank">
      {{ name }}
    </a>
  </span>
{% endfor %}
```

3.3 设置阅读次数统计
需要在leanCloud注册一个账号, 前往 https://leancloud.cn/dashboard , 注册账号是为了得到一个appId 和appKey, 如下图所示:
![这里写图片描述](https://img-blog.csdn.net/20180704103006573?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MjEwNTIz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
具体的操作就不多说了, 请查看 [这里的教程](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
复制AppID以及AppKey并在NexT主题的_config.yml文件中我们相应的位置填入即可，正确配置之后文件内容像这个样子:

```
leancloud_visitors:
  enable: true
  app_id: joaeuuc4hsqudUUwx4gIvGF6-gzGzoHsz
  app_key: E9UJsJpw1omCHuS22PdSpKoh
```
3.4 添加cnzz站长统计
**添加站长统计**

到[友盟+](https://err.taobao.com/error1.html?c=404&u=https://www.taobao.com/markets/umplus/www/signup?spm=0.0.0.0.ma5nae&r=http://www.wuxubj.cn/2016/08/Hexo-nexT-build-personal-blog/)注册账户，并添加自己的网站域名，获取到一个站点ID，这个ID可以在地址栏里，或者自动生成的脚本里面找到。
在主题配置文件 _config.yml中添加如下字段：


```
# CNZZ count
cnzz_siteid: 1259784696
```

**注意把字段cnzz_siteid的值修改为你自己的站点ID。**
修改themes\next\layout\_layout.swig文件，添加如下内容，用于生成cnzz统计代码：


```
{% include '_scripts/third-party/analytics/cnzz-analytics.swig' %}
```

至此cnzz站长统计功能已经添加。由于默认默认不显示“站长统计”字样，所以从页面外观看不到任何变化。

**页脚添加“站长统计”链接**

修改\themes\next\layout\_partials\footer.swig文件，在<span class="author" itemprop="copyrightHolder">{{ config.author }}</span>后面添加如下代码：


```
{% if theme.cnzz_siteid %}
   <span style="margin-left:8px;">
   <script src="http://s6.cnzz.com/stat.php?id={{ theme.cnzz_siteid }}&web_id={{ theme.cnzz_siteid }}" type="text/javascript"></script>
   </span>
{% endif %}
```

4.网站发布
**4.1 云主机**
学生党推荐参加腾讯云云+校园优惠活动，云主机+CN域名只需1元/月。
工作党建议花钱购买云主机，个人博客选择最便宜的就行，一年几百元人民币。
**4.2 Git托管的Pages服务**
常用的有GitHub pages和Coding Pages。
GitHub pages 的使用教程参见：GitHub Pages + Hexo搭建博客 Hexo 3.1.1 静态博客搭建指南
Coding Pages 的使用教程参见：将hexo博客同时托管到github和coding

我刚开始建站的时候使用的是GitHub pages，后来也部署到了Coding，但访问速度都不咋令人满意。最后我选择了腾讯云主机，顿时感觉访问速度飞快。

5 NexT主题美化
**5.1 修改导航栏图标**
NexT 使用的是 [Font Awesome](https://fontawesome.com/?from=io) 提供的图标， [Font Awesome](https://fontawesome.com/?from=io) 提供了 600+ 的图标，可以满足绝大的多数的场景，同时无须担心在 Retina 屏幕下 图标模糊的问题。对应的文件在themes\next\source\vendors\font-awesome中。
在http://fontawesome.dashgame.com/中有图标与其名称的对应，用户可根据需要修改图标。我的menu_icons配置为：

```
menu_icons:
  enable: true
  #KeyMapsToMenuItemKey: NameOfTheIconFromFontAwesome
  home: home
  about: user
  categories: th
  tags: tags
  archives: calendar-check-o
```

**5.2 修改文章内链接文本样式**
将链接文本设置为蓝色，鼠标划过时文字颜色加深，并显示下划线。
修改文件themes\next\source\css\_common\components\post\post.styl，添加如下css样式：

```
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  &:hover {
    color: #0477ab;
    text-decoration: underline;
  }
}
```

选择.post-body是为了不影响标题，选择p是为了不影响首页“阅读全文”的显示样式。



**5.3 文章末尾添加本文结束标记**

**新建 passage-end-tag.swig 文件**
在路径\themes\next\layout\_macro中添加passage-end-tag.swig文件，其内容为：

```
{% if theme.passage_end_tag.enabled %}
<div style="text-align:center;color: #ccc;font-size:14px;">
------ 本文结束 ------</div>
{% endif %}
```
**修改 post.swig 文件**
在\themes\next\layout\_macro\post.swig中，post-body之后，post-footer之前添加如下代码：
```
<div>
  {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
  {% endif %}
</div>
```
**在主题配置文件中添加字段**

在主题配置文件 _config.yml中添加以下字段开启此功能：

```
# 文章末尾添加“本文结束”标记
passage_end_tag:
  enabled: true
```

完成以上设置之后，在每篇文章之后都会添加“本文结束”标记。
该功能简易添加方法参见：[Issues of hexo-theme-next](https://github.com/iissnan/hexo-theme-next/issues/1039)
5.4 文章末尾添加网站二维码
**利用 NexT 主题自带的wechat_subscriber功能在文章末尾添加网站二维码。**
首先生成你网站的二维码，放到网站根目录下的images文件夹中，然后修改主题配置文件 _config.yml，添加如下内容：
```
# Wechat Subscriber
wechat_subscriber:
  enabled: true
  qcode: /images/wuxubj.png
  description: 扫一扫，用手机访问本站
```

5.5 其他美化
1.标签云页面鼠标划过字体加粗
2.文章末尾标签鼠标划过变蓝色
3.调换文章末尾上一篇和下一篇链接显示位置（左右互换）
4.优化文章末尾上一篇和下一篇链接显示效果完成以上设置之后，在每篇文章之后都会添加网站二维码。


6.SEO推广
**6.1 生成sitemap**
Sitemap用于通知搜索引擎网站上有哪些可供抓取的网页，以便搜索引擎可以更加智能地抓取网站。
执行以下命令，安装插件hexo-generator-sitemap，用于生成sitemap：


```
$ npm install hexo-generator-sitemap --save
```

在站点配置文件 _config.yml中添加如下字段：


```
sitemap:
path: sitemap.xml
```

执行hexo g，就会在网站根目录生成 sitemap.xml 。

**6.2 开启百度自动推送**
在主题配置文件 _config.yml中添加如下字段：
```
baidu_push: true
```
6.3 使用各大搜索引擎站长工具
	在搜索引擎搜索框输入site:your.domain可以查看域名是否被该搜索引擎收录，用户可以使用各大搜索引擎站长工具提交个人博客网址。

7.更换配置
	这里记录一个我遇到的麻烦, 更换电脑之后,重新更新博客,遇到了一系列的问题,我的博客是备份在了github上面,当我拉下代码发现_.config.html文件不存在, 而使用hexo的都知道,hexo主要是需要配置文件, 幸好原来的电脑还保存着一份配置文件, 于是新建了一个文件夹, 然后hexo init  , hexo g ,hexo s,打开localhost:4000就可以看到生成了一个博客, 这时,替换配置文件, 并把拉下来的代码放置到source文件夹中 .之后又出现了git的问题, 很明显, ssh key肯定是不对的, 那么就需要进行设置了, 这个可以参考我的[这篇文章](https://www.jianshu.com/p/301afa16f471)
	这里提醒一下各位,如果是刚开始建博客, 最好设置两个分支,dev/master用来放置静态网站页面文件,hexo分支用来放置配置文件.具体可以参考[网上的资源](https://blog.csdn.net/Rainy_X/article/details/79829181)

8.相关资源
[我的站点文件备份](https://github.com/yidongying/yidongying.github.io)
[优化之后的NexT主题下载](https://github.com/wuxubj/hexo-theme-next-wuxubj/releases)
[markdownpad2](http://markdownpad.com/)
[Notepad++ v6.9.2](https://notepad-plus-plus.org/download/v6.9.2.html)
[Git-2.8.1-64-bit](https://git-scm.com/)
[node-v4.4.2-x64](https://nodejs.org/zh-cn/)

9.参考文档
(1)[hexo官方文档](https://hexo.io/zh-cn/docs/)
(2) [next主题官方文档](http://theme-next.iissnan.com/)
(3) [第三方服务集成](http://theme-next.iissnan.com/third-party-services.html#algolia-search)
(4)[next主题特性配置](https://github.com/iissnan/hexo-theme-next/wiki/%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E5%8F%82%E8%80%83)
(5)[电脑更换后的问题](https://blog.csdn.net/wxl1555/article/details/79293159)