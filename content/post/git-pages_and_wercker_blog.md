+++
date = "2016-05-14T20:35:23+08:00"
description = "本篇文章是教大家用git-pages和wercker来写博客"
draft = false
tags = ["git-pages","wercker"]
title = "以github-pages和wercker来写博客"
topics = ["blog"]

+++

#### 本篇文章旨在引领人们享受写作的乐趣，而不是“工欲善其事，必先利其器”


### 背景以及准备工作

相信每一个好的技术人员都有记录思想，工作经验，学习笔记等等多好习惯，而一个运行良好的blog不可或缺。所以这次我们以github pages来做一个blog。

github是人们常用的版本控制工具，相信每个程序员都听说过。啥？你不知道，你一定是火星来的。:)-
hugo是一个前docker重量级员工跳出来搞的一个静态网页生成器，使用golang写的。
wercker是一个以docker为基础的持续性代码部署云服务。
好的，下面，我们开始吧:

### 安装hugo，创建github项目分支，创建wercker帐号

由于我用的是centos_64位，所以在hugo的github上下载[hugo_0.15_linux_amd64.tar.gz](https://github.com/spf13/hugo/releases/download/v0.15/hugo_0.15_linux_amd64.tar.gz)这个软件包。
解压：
```
$ tar -xzvf hugo_0.15_linux_amd64.tar.gz
```
得到一个可执行的二进制go程序，改个短名字，如hugo，并把它放到/opt下，加入系统变量中
编辑 /etc/profile
```
$ mv hugo /opt
$ vi /etc/profile
$ export HUGOROOT=/opt
$ export PATH=$PATH::$HUGOROOT
```
然后就可以使用hugo来生成网站了,mysite是网站的路径

```
$ hugo new site mysite
```

然后进入该路径

```
$ cd mysite
```

在该目录下你可以看到以下几个目录和``config.toml``文件

```
+ archetypes/ 
+ content/
+ layouts/
+ static/
  config.toml
```
``config.toml``是网站的配置文件，包括``baseurl``, ``title``, ``copyright``等等网站参数。

这几个文件夹的作用分别是：

* archetypes：包括内容类型，在创建新内容时自动生成内容的配置
* content：包括网站内容，全部使用markdown格式
* layouts：包括了网站的模版，决定内容如何呈现
* static：包括了css, js, fonts, media等，决定网站的外观

Hugo提供了一些完整的主题可以使用，下载这些主题：

```
$ git clone --recursive https://github.com/spf13/hugoThemes themes
```

此时现成的主题存放在``themes/``文件夹中。

现在我们先熟悉一下Hugo，创建新页面：

```
$ hugo new about.md
```

进入``content/``文件夹可以看到，此时多了一个markdown格式的文件``about.md``，打开文件可以看到时间和文件名等信息已经自动加到文件开头，包括创建时间，页面名，是否为草稿等。


我在页面中加入了一些内容，然后运行Hugo:

```
$ hugo server -t hyde --buildDrafts
```

``-t``参数的意思是使用hyde主题渲染我们的页面，注意到``about.md``目前是作为草稿，即``draft``参数设置为``true``，运行Hugo时要加上``--buildDrafts``参数才会生成被标记为草稿的页面。
在浏览器输入localhost:1313，就可以看到我们刚刚创建的页面。

{{% fluid_img class="pure-u-1-1" src="/img/post/hugo-server-1.png" alt="name?" %}}


注意观察当前目录下多了一个文件夹``public/``，这里面是Hugo生成的整个静态网站，如果使用Github pages来作为博客的Host，你只需要将``public/``里的文件上传就可以，这相当于是Hugo的输出。


### 主题选择

进入``themes/hyde``文件夹，可以看到熟悉的文件夹名，和主题相关的文件主要是在``layouts/``和``static/``这两个文件内，选择好一个主题后，可以将``themes/``中的文件夹直接复制到``mysite/``目录下，覆盖原来的``layouts/``, ``static/``文件夹，此时直接使用\$Hugo server就可以看到主题效果，修改主题也可以直接修改其中的css, js, html等文件。

我的博客模版是在Hugo作者spf13的[博客](http://spf13.com)基础上修改的。第一步，先去他的博客网站源码[主页](https://github.com/spf13/spf13.com)把整个项目clone下来

```
$ git clone git@github.com:spf13/spf13.com.git
```

把项目中的``static/``和``layouts/``文件复制到自己网站的目录下替换原来的文件夹。再次运行Hugo:

```
$ hugo server --buildDrafts -w
```

这次没有选择主题，如果选择了主题会将当前的主题覆盖掉。参数``-w``意味监视watch，此时如果修改了网站内的信息，会直接显示在浏览器的页面上，不需要重新运行\$hugo server，方便我们进行修改。这是采用了spf13主题的页面：

{{% fluid_img class="pure-u-1-1" src="/img/post/hugo-server-2.png" alt="name?" %}}

我们尝试在他的主题基础上修改，找到``/layouts/partials/subheader.html``文件:

```html
<header id="header">
    <figure>
      <a href="/" border=0 id="logolink"><div class="icon-spf13-3" id="logo"> </div></a>
    </figure>
    <div id="byline">by Steve Francia</div>
    <nav id="nav">
    {{ partial "nav.html" . }}
    {{ partial "social.html" . }}
    </nav>
</header>
```

将by Steve Francia换成by myname，再次回到浏览器，可以看到左边侧栏已经发生变化了，你可以根据自己的需要修改对应的文件，当然得懂一点css, html。

{{% fluid_img class="pure-u-1-1" src="/img/post/hugo-server-change.png" alt="name?" %}}


### 评论功能

个人博客当然不能没有评论，Hugo默认支持[Disqus](https://disqus.com/)的评论，需要在模版中添加以下代码：

```
{{ template "_internal/disqus.html" . }}
```

spf13在``/layouts/partials/disqus.html``中已经添加好了。

只需要去Disqus注册一个账号，然后在``config.toml``里加上：

```
disqusShortname = "yourdisqusShortname"
```

注意``-w``参数是不能监测``config.toml``里参数变化的，因此需要重新运行Hugo，进入localhost:1313/about，可以看到评论功能。

{{% fluid_img class="pure-u-1-1" src="/img/post/comments.png" alt="name?" %}}

### 代码高亮

作为码农，代码高亮对于写博客来说当然必不可少。有两种方法：第一种是在生成页面时就生成好代码高亮过的页面；第二种是使用js，用户加载页面时浏览器再进行渲染。

第一种方法需要使用[Pygments](http://pygments.org/)，一个python写的工具。

安装Pygments：

```shell
$ pip install Pygments
```

没有pip的先下载 https://bootstrap.pypa.io/get-pip.py ，然后安装pip：


```shell
$ python get-pip.py
```

Pygments的调用采用shortcodes实现，spf13里也写好了，在``/layouts/shortcode/highlight.html``里


```
{{ $lang := index .Params 0 }}
{{ highlight .Inner $lang }}
```

要使代码高亮，在你的代码外面加上：

```
{{ % highlight python %}}
your code here.
{{ % /highlight %}}
```

这里为了避免以上两行被识别为代码高亮的标识，在``{{``和``%``之间多加了一个空格，实际使用的时候需要把空格去掉。

第二种方法比较简单，在``layouts/partials/header_includes.html``中加上：


```
<script src="https://yandex.st/highlightjs/8.0/highlight.min.js"></script>
<link rel="stylesheet" href="https://yandex.st/highlightjs/8.0/styles/default.min.css">
<script>hljs.initHighlightingOnLoad();</script>
```


这里使用了[Yandex](http://yandex.ru/)的[Highlight.js](http://highlightjs.org/)。

其他的可以实现代码高亮的js库还有：

* [Highlight.js](http://highlightjs.org/)
* [Rainbow](http://craig.is/making/rainbows)
* [Syntax Highlighter](http://alexgorbatchev.com/SyntaxHighlighter/)
* [Google Prettify](https://code.google.com/p/google-code-prettify/)

### 插入图片

图片文件放在``static/media``文件中，插入图片：

```
{{ % img src="/media/example.jpg" alt="example" %}}
```

注意这里的``{{``和``%``之间也加上了空格，避免这行代码起作用，实际使用也需要把空格去掉。

### 使用Mathjax

在需要渲染公式的页面加入以下代码，比如``layouts/_default/single.html``文件，这个文件是对于所有post进行页面生成的模版，如果你希望所有页面都对公式渲染的话，可以加入``layouts/partials/footer.html``文件里，保证所有生成的页面都有这几行代码。

```html
<script type="text/javascript"
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```
Mathjax和Markdown会有冲突问题，[这里](http://doswa.com/2011/07/20/mathjax-in-markdown.html)提供了解决方案。

### 用github pages作为网站的Host

Github pages分为两种：一种是项目主页，每个项目都可以有一个；另一种是用户主页，一个用户只能有一个。

因为用户主页只能有一个，所以建议使用项目主页托管，不过我这里采用了用户主页，反正我也只用一个博客，使用个人主页作为Host也相对更简单一点。

我们需要创建两个单独的repo，一个用于放Hugo的输入文件，即除了``public/``文件夹之外的所有文件，另一个放我们生成的静态网站，也就是``public/``的内容。

步骤如下：

1. 在Github上创建repo ``<your-project>-hugo``，托管Hugo的输入文件。
2. 创建repo ``<username>.github.io``，用于托管``public/``文件夹，注意这里的repo名字一定要用自己的用户名，才会被当作是个人主页。
3. clone your-project
```
$ git clone <<your-project>-hugo-url>
```
4. 进入your-project 目录
```
$ cd <your-project>-hugo
```
5. 删掉public目录（这个目录每次运行Hugo都会再次生成，不用担心）
```
$ rm -rf public
```
6. 把public/目录添加为submodule 与<username>.github.io同步
```
$ git submodule add git@github.com:<username>/<username>.github.io.git public
```
7. 添加.gitignore文件，文件中写``public/``，在同步``<your-project>-hugo``时会忽略public文件夹
8. 下面是写好的一个script ``deploy.sh``，拷贝过去直接就能用，记得chmod +x deploy.sh加上运行权限。

```bash
#!/bin/bash
echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi

# Push Hugo content 
git add -A
git commit -m "$msg"
git push origin master


# Build the project. 
hugo # if using a theme, replace by `hugo -t <yourtheme>`

# Go To Public folder
cd public
# Add changes to git.
git add -A

# Commit changes.

git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back
cd ..

```

等一小会儿（10分钟左右），你就能在http://username.github.io/ 这个页面看到你的网站了！每次更新网站或者写了新文章，只需要运行./deploy.sh 发布就搞定了，简单吧？

Github pages还支持域名绑定，三个步骤：

1. 在``<username>.github.io`` repo的跟目录下添加``CNAME``文件，文件里写上你的域名，不用加http://的开头。
2. 记下http://username.github.io/ 的ip地址。
```
$ ping username.github.io
```
3. 在你的域名管理中加上两条A记录，分别是www和@，记录指向http://username.github.io/ 的ip地址，也需要等一小会儿生效。

### 更改字体服务商

我的博客模版里用的字体是从googleapis里获取的，国内访问会下载失败，把字体库改成360的。
找到``layouts/partials/head_includes.html``文件：

```html
<link href='http://fonts.googleapis.com/css?family=Fjalla+One|Open+Sans:300' rel='stylesheet' type='text/css'>
```

将其中的googleapis替换为useso就行了。


#### 参考文章：

+ [nanshu'blog](http://nanshu.wang)

+ [coderzh blog](http://blog.coderzh.com/)

+ [gohugo](http://gohugo.io) 和 [wecker](http://wercker.com)的官方blog
