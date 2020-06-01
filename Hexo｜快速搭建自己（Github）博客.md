---
title: Hexo｜快速搭建自己（Github）博客
date: 2019-11-02 14:12:52
tag: 博客
category: web
---
<!-- TOC -->

- [一、准备工作](#一准备工作)
- [二、申请github博客](#二申请github博客)
  - [1.创建github仓库](#1创建github仓库)
- [三、安装Hexo](#三安装hexo)
  - [1. 初始化](#1-初始化)
  - [2. 新建文章](#2-新建文章)
  - [3. 开启服务](#3-开启服务)
- [四、配置Hexo与Github pages进行关联](#四配置hexo与github-pages进行关联)
  - [1. 配置关联](#1-配置关联)
  - [2. 发布](#2-发布)
- [五、更换NexT主题](#五更换next主题)
  - [1. 下载主题](#1-下载主题)
  - [2. 设置主题](#2-设置主题)
  - [3. 配置主题](#3-配置主题)
    - [1. 标签云页面](#1-标签云页面)
    - [2. 分类页面](#2-分类页面)
    - [3. About页面](#3-about页面)
    - [4. 腾讯公益404页面](#4-腾讯公益404页面)
    - [5. 代码高亮主题](#5-代码高亮主题)
    - [6. 站点建立时间](#6-站点建立时间)
    - [7. 数学公式显示](#7-数学公式显示)
    - [8. 侧边栏社交链接](#8-侧边栏社交链接)
    - [9. 首页显示文章摘录](#9-首页显示文章摘录)
    - [10. 设置 favicon](#10-设置-favicon)
    - [11. 配置不蒜子计数](#11-配置不蒜子计数)
- [六、问题](#六问题)
  - [1.hexo引用本地图片无法显示](#1hexo引用本地图片无法显示)
    - [插件安装与配置](#插件安装与配置)

<!-- /TOC -->

# 一、准备工作

* 安装[nodejs](https://nodejs.org/en/)
* 安装[git](https://git-scm.com/)
* 注册[github](https://github.com/)账号

# 二、申请github博客

## 1.创建github仓库
创建一个新的github仓库，仓库名为`username.github.io`,其他选项可以默认。

![](Hexo｜快速搭建自己（Github）博客/new_repository.png)

创建按仓库成功后，进入设置页面GitHub Pages 选项，出现如下界面则表示博客页面创建成功。

![](Hexo｜快速搭建自己（Github）博客/github_pages.png)

# 三、安装Hexo

```bash
$ npm install hexo-cli -g
```

## 1. 初始化
```bash
$ hexo  init blog
$ cd blog
$ npm install
```

## 2. 新建文章
```bash
$ hexo new "Hello-Hexo"
```
会在根目录下source/_posts/ 生成Hello-Hexo.md文件，使用markdown编辑器非常方便。

## 3. 开启服务
```bash
$ hexo server
```
会看到提示`INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.` 代表博客已经正常运行，在浏览器打开`http://0.0.0.0:4000/` 即可预览博客的初始状态。

了解更多Hexo的信息和其他步骤请参考官网的[这篇文档](https://hexo.io/zh-cn/docs/setup.html) 。

# 四、配置Hexo与Github pages进行关联

## 1. 配置关联
进入`Blog`目录，然后以文本编辑器打开_config.yml文件，修改deploy配置如下，rego为CKCat.github.io.git仓库提交代码的git地址。
```yml
deploy:
  type: git
  repo: https://github.com/CKCat/CKCat.github.io.git
  branch: master
```
## 2. 发布
当然这只是本地跑起来了，而你的github pages服务器上并没有，所以你就需要在你的站点里使用终端命令进行发布：
```bash
hexo clean  # 清除缓存
hexo g      # 生成本地发布文件夹
hexo d      # 发布到github pages上
```
更多的hexo命令操作请参考[Hexo官方文档](https://hexo.io/zh-cn/docs/commands.html)即可。

# 五、更换NexT主题

## 1. 下载主题
在Blog目录运行下列命令：
```bash
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

## 2. 设置主题
克隆完成以后打开站点配置 `_config.yml`，找到theme字段，值改为`next`。
运行`hexo server --debug`，访问`http://0.0.0.0:4000/` 查看主题是否生效。

## 3. 配置主题
常用的配置可以直接按照主题的官方[参考手册](http://theme-next.iissnan.com/)配置。
下面介绍一下常用配置：
### 1. 标签云页面

1. 新建一个页面，命名为tags。命令如下：
    ```bash
    $ hexo new page "tags"
    ```

2. 编辑刚新建的页面，将页面的类型设置为tags，主题将自动为这个页面显示标签云。页面内容如下：
    ```bash
    ---

    title: 标签
    date: 2019-11-01 11:46:23
    type: "tags"
    ---
    ```
    注意：如果有启用多说 或者 Disqus 评论，默认页面也会带有评论。需要关闭的话，请添加字段 comments 并将值设置为 false，如：
    ```bash
    title: 标签
    date: 2019-11-01 11:46:23
    type: "tags"
    comments: false
    ---
    ```

3. 在菜单中添加链接。编辑 **主题配置文件** ，添加tags到 menu中，如下:
    ```bash
    menu: 
    home: / 
    archives: /archives 
    tags: /tags
    ```

### 2. 分类页面

1. 新建一个页面，命名为categories。命令如下：
    ```bash
    $ hexo new page categories
    ```

2. 编辑刚新建的页面，将页面的类型设置为categories，主题将自动为这个页面显示所有分类。
    ```bash
    ---
    title: 分类
    date: 2019-11-01 12:03:27
    type: "categories"
    ---
    ```

    注意：如果有启用多说 或者 Disqus 评论，默认页面也会带有评论。需要关闭的话，请添加字段comments并将值设置为false，如：

    ```bash
    ---
    title: 分类
    date: 2019-11-01 12:03:27
    type: "categories"
    comments: false
    ---
    ```

3. 在菜单中添加链接。编辑 **主题配置文件** ，将menu中的categories: /categories注释去掉，如下:
    ```
    menu: 
    home: / 
    categories: /categories 
    archives: /archives 
    tags: /tags
    ```

### 3. About页面

1. 新建一个 about页面：
    ```bash
    $ hexo new page "about"
    ```
2. 菜单显示about链接，在 **主题配置文件** 设置中将menu中about
前面的注释去掉即可。
    ```
    menu: 
    home: / 
    archives: /archives 
    tags: /tags 
    about: /about
    ```

3. 友情链接编辑 **站点配置文件** 添加：
    ```
    # title, chinese available
    links_title: Links
    # links
    links: 
    MacTalk: http://macshuo.com/
    ```

### 4. 腾讯公益404页面

简体中文增加腾讯公益404页面，寻找丢失儿童，让大家一起关注此项公益事业！效果如下http://www.ixirong.com/404.html

使用方法，新建 404.html 页面，放到主题的source目录下，内容如下：
```html
<!DOCTYPE HTML><html><head>
 <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
 <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
 <meta name="robots" content="all" />
 <meta name="robots" content="index,follow"/>
</head>
<body>

<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="your site url " homePageName="回到我的主页"></script>
</body>
</html>
```

### 5. 代码高亮主题

NexT 使用 [Tomorrow Theme0](https://github.com/chriskempson/tomorrow-theme) 作为代码高亮，共有5款主题供你选择。
默认使用的是白色的 normal。

### 6. 站点建立时间

这个时间将在站点的底部显示，例如`© 2013 - 2019`编辑 **站点配置文件**，新增字段since。
```
since: 2013
```

### 7. 数学公式显示

NexT 借助于 MathJax 来显示数学公式，此选项默认关闭。
编辑 **主题配置文件**，将mathjax设定为true即可。
```
# MathJax Support
mathjax: true
```
ProTip: [使用七牛 CDN 来加速 MathJax 脚本的加载](https://github.com/iissnan/hexo-theme-next/pull/363)

### 8. 侧边栏社交链接

编辑 **站点配置文件** ，新增字段social，然后添加社交站点名称与地址即可。例如：
```
#Social linkssocial: 
  github: https://github.com/your-user-name 
  twitter: https://twitter.com/your-user-name 
  weibo: http://weibo.com/your-user-name 
  douban: http://douban.com/people/your-user-name 
  zhihu: http://www.zhihu.com/people/your-user-name 
  # 等等
```

### 9. 首页显示文章摘录
NexT 支持三种方式来控制首页文章的显示方式。若需要在首页显示摘录，可以通过以下方法：

在文章中使用`<!-- more -->`手动进行截断
在文章的 [front-matter](https://hexo.io/docs/front-matter.html) 中添加description，并提供文章摘录
自动形成摘要，在 主题配置文件 中添加：
```
auto_excerpt: 
enable: true 
length: 150
```
默认截取的长度为150字符，可以根据需要自行设定。

### 10. 设置 favicon

将`favicon`放置到站点的 `source`目录下即可。


### 11. 配置不蒜子计数
1. 安装脚本（必选）
要使用不蒜子必须在页面中引入busuanzi.js，代码如下：
```
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

本人使用的是next主题，所以在themes/next/layout/_partial/footer.swig中添加上述脚本，也可以把脚本添加在header中，如果使用的是其他主题，大致也是一样的，不过可能后缀是ejs，没有影响。

2. 配置方式使用不蒜子计数
第一种引入脚本的方法适用于任何类型的个人站点，如果你使用的主题是NexT，那么你可以很方便的进行不蒜子的访客统计设置，仅仅只需要一步：

打开主题的配置文件`/theme/next/_config.yml`，配置如下：
```
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i> 访客数
  site_uv_footer: 人
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i> 总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i>
  page_pv_footer: 次
```
> site_uv表示是否显示整个网站的UV数
> 
> site_pv表示是否显示整个网站的PV数
> 
> page_pv表示是否显示每个页面的PV数


# 六、问题

## 1.hexo引用本地图片无法显示
### 插件安装与配置
首先我们需要安装一个图片路径转换的插件，这个插件名字是hexo-asset-image
```bash
$ npm install https://github.com/CodeFalling/hexo-asset-image --save
```
但是这个插件的内容需要修改【不然可能会出Bug】

打开`/node_modules/hexo-asset-image/index.js`，将内容更换为下面的代码
```javascript
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    	var link = data.permalink;
	if(version.length > 0 && Number(version[0]) == 3)
	   var beginPos = getPosition(link, '/', 1) + 1;
	else
	   var beginPos = getPosition(link, '/', 3) + 1;
	// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
	var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
		if ($(this).attr('src')){
			// For windows style path, we replace '\' to '/'.
			var src = $(this).attr('src').replace('\\', '/');
			if(!/http[s]*.*|\/\/.*/.test(src) &&
			   !/^\s*\//.test(src)) {
			  // For "about" page, the first part of "src" can't be removed.
			  // In addition, to support multi-level local directory.
			  var linkArray = link.split('/').filter(function(elem){
				return elem != '';
			  });
			  var srcArray = src.split('/').filter(function(elem){
				return elem != '' && elem != '.';
			  });
			  if(srcArray.length > 1)
				srcArray.shift();
			  src = srcArray.join('/');
			  $(this).attr('src', config.root + link + src);
			  console.info&&console.info("update link as:-->"+config.root + link + src);
			}
		}else{
			console.info&&console.info("no src attr, skipped...");
			console.info&&console.info($(this));
		}
      });
      data[key] = $.html();
    }
  }
});
```

打开_config.yml文件，修改下述内容
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://ckcat.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing index.html from permalinks


post_asset_folder: true
```
参考[hexo引用本地图片无法显示](https://850552586.github.io/2018/11/15/hexo%E5%BC%95%E7%94%A8%E6%9C%AC%E5%9C%B0%E5%9B%BE%E7%89%87%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA/)






