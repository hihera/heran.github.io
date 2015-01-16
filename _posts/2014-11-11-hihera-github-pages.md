---
layout: post
title: 使用Github Pages创建博客
description: 本博客是基于Github Pages博客系统，在fork Siberiawolf的基础上修改创建。
category: 技术分享
tags: [Github Pages]
---
{% include JB/setup %}
# 使用Github Pages创建博客


##Github Pages 是什么？
[Github](https://github.com/)作为现在最流行的代码仓库，很好的将代码和社区联系在了一起，目前已经得到很多大公司和项目的青睐。为使项目更方便的被人理解，github就设计了[Pages](https://pages.github.com/)功能，允许用户自定义项目首页，用来替代默认的源码列表。所以，github Pages可以被认为是用户编写的、托管在github上的静态网页。

<!--break-->

Github Pages有以下几个优点：

<ul>
    <li>轻量级的博客系统，没有麻烦的配置</li>
    <li>使用标记语言，比如<a href="http://markdown.tw">Markdown</a></li>
    <li>无需自己搭建服务器</li>
    <li>享受git的版本管理功能，不用担心文章遗失</li>
    <li>可以绑定自己的域名</li>
</ul>

当然他也有缺点：

* 使用[Jekyll][]模板系统，相当于静态页发布，适合博客，文档介绍等。
* 动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。
* 基于Git，很多东西需要动手，不像Wordpress有强大的后台。
* 它不适合大型网站，因为没有用到数据库，每运行一次都必须遍历全部的文本文件，网站越大，生成时间越长。

##Jekyll是什么？
[Jekyll](http://jekyllrb.com/)是一个静态站点生成器，它会根据网页源码生成静态文件。它提供了模板、变量、插件等功能，所以实际上可以用来编写整个网站。我们可以在本地编写符合Jekyll规范的网站源码，然后上传到github，由github生成并托管整个网站。

大致就是这些，下面介绍一下该博客搭建的整个过程，以供参考。

##一、准备工作
###安装RubyInstaller
[RailsInstaller 一键安装包](http://railsinstaller.org/en)
安装完成后，cmd中输入ruby -v，出现ruby的版本号即表示安装成功。

    ruby 2.0.0p195 (2013-05-14) [i386-mingw32]

###安装Jekyll
安装ruby成功之后，就可以安装Jekyll了。cmd中执行 gem jekyll install，若提示如下错误，解决方案如下：

    ERROR:  Could not find a valid gem 'jekyll' (>= 0) in any repository
    ERROR:  Possible alternatives: jekyll`

【解决】[Github jekyll](https://github.com/jekyll/jekyll/issues/1409)中有解决方案

    $ gem sources --remove http://rubygems.org/
    $ gem sources -a http://ruby.taobao.org/
    $ gem sources -l
    *** CURRENT SOURCES ***
    http://ruby.taobao.org
    $ gem install rack

然后继续执行`gem install jekyll`即可。此处一定要耐心等待，不要看没有任何反应不耐心就关掉cmd或者强制推出，此刻正在下载jekyll包，下载完成就会自动解压安装。等待片刻，当提示21 gems installed。表示jekyll 安装完毕。

###安装markdown解释器RDiscount，通过命令：
cmd进入E:\RailsInstaller\DevKit目录下，执行命令gem install rdiscount,出现下面命令表示安装成功。

    $gem install rdiscount
    Successfully installed rdiscount-2.1.7.1
    1 gem installed
    ...

###安装Python
安装适合自己系统的版本，[Python官网](http://www.python.org/download/) win7 32位快速下载通道,[Python 2.7.2](http://www.python.org/ftp/python/2.7.2/python-2.7.2.msi)

###安装Pygments语法高亮
首先下载[distribute_setup.py](https://pypi.python.org/pypi/distribute/0.6.49)文件
解压该文件，进入到distribute-0.6.49下，在cmd控制台输入命令:

    python.exe distribute_setup.py

接着安装Pygments，通过上一步安装好的easy_install，进入E:\Python34\Scripts输入命令：

    easy_install.exe pygments

最后配置Pygmentize.exe，在系统变量的PATH路径里，添加其路径，比如在我的方案里，添加：E:\Python34\Scripts

## 二、配置和使用Github
Git是版本管理的未来，他的优点我不再赘述，相关资料很多。推荐这本[Git中文教程][4]。

要使用Git，需要安装它的客户端，推荐在Linux下使用Git，会比较方便。Windows版的下载地址在这里：[http://code.google.com/p/msysgit/downloads/list](http://code.google.com/p/msysgit/downloads/list "Windows版Git客户端")。其他系统的安装也可以参考官方的[安装教程][5]。

下载安装客户端之后，各个系统的配置就类似了，我们使用windows作为例子，Linux和Mac与此类似。具体参考<a href="http://siberiawolf.iteye.com/blog/2035330"></a>
在Windows下，打开Git Bash，其他系统下面则打开终端（Terminal）：
![Git Bash](http://siberiawolf.qiniudn.com/images/githubpages/bootcamp_1_win_gitbash.jpg)

###1、检查SSH keys的设置
首先我们需要检查你电脑上现有的ssh key：

    $ cd ~/.ssh

如果显示“No such file or directory”，跳到第三步，否则继续。

###2、备份和移除原来的ssh key设置：
因为已经存在key文件，所以需要备份旧的数据并删除：

    $ ls
    config	id_rsa	id_rsa.pub	known_hosts
    $ mkdir key_backup
    $ cp id_rsa* key_backup
    $ rm id_rsa*

###3、生成新的SSH Key：
输入下面的代码，就可以生成新的key文件，我们只需要默认设置就好，所以当需要输入文件名的时候，回车就好。

    $ ssh-keygen -t rsa -C "邮件地址@youremail.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>

然后系统会要你输入加密串（[Passphrase][6]）：

    Enter passphrase (empty for no passphrase):<输入加密串>
    Enter same passphrase again:<再次输入加密串>

最后看到这样的界面，就成功设置ssh key了：
![ssh key success](http://siberiawolf.qiniudn.com/images/githubpages/ssh-key-set.png)

###4、添加SSH Key到GitHub：
在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。

用文本编辑工具打开id_rsa.pub文件，如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。

在GitHub的主页上点击设置按钮：
![github account setting](http://siberiawolf.qiniudn.com/images/githubpages/github-account-setting.png)

选择SSH Keys项，把复制的内容粘贴进去，然后点击Add Key按钮即可：
![set ssh keys](http://siberiawolf.qiniudn.com/images/githubpages/bootcamp_1_ssh.jpg)

PS：如果需要配置多个GitHub账号，可以参看这个[多个github帐号的SSH key切换](http://omiga.org/blog/archives/2269)，不过需要提醒一下的是，如果你只是通过这篇文章中所述配置了Host，那么你多个账号下面的提交用户会是一个人，所以需要通过命令

    `git config --global --unset user.email`删除用户账户设置，在每一个repo下面使用
    `git config --local user.email '你的github邮箱@mail.com'` 命令单独设置用户账户信息

###5、测试一下
可以输入下面的命令，看看设置是否成功，`git@github.com`的部分不要修改：

    $ ssh -T git@github.com


如果是下面的反应：

    The authenticity of host 'github.com (207.97.227.239)' can't be established.
    RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
    Are you sure you want to continue connecting (yes/no)?


不要紧张，输入`yes`就好，然后会看到：

    Hi <em>username</em>! You've successfully authenticated, but GitHub does not provide shell access.

###6、设置你的账号信息
现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。

Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的。

    $ git config --global user.name "你的名字"
    $ git config --global user.email "your_email@youremail.com"

###成功了
好了，你已经可以成功连接GitHub了。

## 三、使用GitHub Pages建立博客
与GitHub建立好链接之后，就可以方便的使用它提供的Pages服务，GitHub Pages分两种，一种是你的GitHub用户名建立的`username.github.io`这样的用户&组织页（站），另一种是依附项目的pages。

###User & Organization Pages
想建立个人博客是用的第一种，形如`beiyuu.github.io`这样的可访问的站，每个用户名下面只能建立一个，创建之后点击`Admin`进入项目管理，可以看到是这样的：
![user pages](http://siberiawolf.qiniudn.com/images/githubpages/user-pages.png)

而普通的项目是这样的，即使你也是用的`othername.github.io`：
![other pages](http://siberiawolf.qiniudn.com/images/githubpages/other-pages.png)

以第一种为例：
登陆你的Github账户，创建一个新的版本库，命名为username.github.io,打开git bash命令行拷贝项目：

    $ git clone https://github.com/username/username.github.io
    $ cd username.github.io
    $ echo "Hello World" > index.html
    $ git add --all
    $ git commit -m "Initial commit"
    $ git push origin master


当文件推送完毕之后，你会在Github的仓库中，找到刚添加的index.html文件，里面会有一行Hello World。大概10分钟左右，你就可以访问`username.github.io`就可以看到你上传的页面了，[beiyuu.github.com][7]就是一个例子。

关于第二种项目`pages`，简单提一下，他和用户pages使用的后台程序是同一套，只不过它的目的是项目的帮助文档等跟项目绑定的内容，所以需要在项目的`gh-pages`分支上去提交相应的文件，GitHub会自动帮你生成项目pages。具体的使用帮助可以参考[Github Pages][]的官方文档：

###购买、绑定独立域名

[Godaddy][]作为域名服务商他做的还不赖，而且支持支付宝，免去了在中国申请域名需要备案的麻烦，当然，如果你追求速度的话，中国万网或许是个不错的选择。
如果你不明白如何注册，添加需要绑定的独立域名的服务器，可以参考这里：<a href="http://hugege.com/2009/04/05/godaddy/"></a>
如果你想让`username.github.io`能通过你自己的域名来访问，需要在项目的根目录下新建一个名为`CNAME`的文件，文件内容形如：

    beiyuu.com

注意一定保证CNAME文件是大写，且里边只有域名一行代码，否则访问会失败。

你也可以绑定在二级域名上：

    blog.beiyuu.com

需要提醒的一点是，如果你使用形如`beiyuu.com`这样的一级域名的话，需要在DNS处设置A记录到`192.30.252.153`和`192.30.252.154`（**这个地址可能会有变动，[这里][a-record]查看**），而不是在DNS处设置为CNAME的形式，否则可能会对其他服务（比如email）造成影响。

设置成功后，稍等10分钟，方可生效。

##四、Jekyll模板系统
GitHub Pages为了提供对HTML内容的支持，选择了[Jekyll][]作为模板系统，Jekyll是一个强大的静态模板系统，作为个人博客使用，基本上可以满足要求，也能保持管理的方便，你可以查看[Jekyll官方文档][8]。

###Jekyll基本结构
Jekyll的核心其实就是一个文本的转换引擎，用你最喜欢的标记语言写文档，可以是Markdown、Textile或者HTML等等，再通过`layout`将文档拼装起来，根据你设置的URL规则来展现，这些都是通过严格的配置文件来定义，最终的产出就是web页面。

基本的Jekyll结构如下：

    |-- _config.yml
    |-- _includes
    |-- _layouts
    |   |-- default.html
    |   `-- post.html
    |-- _posts
    |   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
    |   `-- 2009-04-26-barcamp-boston-4-roundup.textile
    |-- _site
    `-- index.html


简单介绍一下他们的作用：
####_config.yml
配置文件，用来定义你想要的效果，设置之后就不用关心了。

####_includes
可以用来存放一些小的可复用的模块，方便通过`{ % include file.ext %}`（去掉前两个{中或者{与%中的空格，下同）灵活的调用。这条命令会调用_includes/file.ext文件。

####_layouts
这是模板文件存放的位置。模板需要通过[YAML front matter][9]来定义，后面会讲到，`{ { content }}`标记用来将数据插入到这些模板中来。

####_posts
你的动态内容，一般来说就是你的博客正文存放的文件夹。他的命名有严格的规定，必须是`2012-02-22-artical-title.MARKUP`这样的形式，MARKUP是你所使用标记语言的文件后缀名，根据_config.yml中设定的链接规则，可以根据你的文件名灵活调整，文章的日期和标记语言后缀与文章的标题的独立的。

####_site
这个是Jekyll生成的最终的文档，不用去关心。最好把他放在你的`.gitignore`文件中忽略它。

####其他文件夹
你可以创建任何的文件夹，在根目录下面也可以创建任何文件，假设你创建了`project`文件夹，下面有一个`github-pages.md`的文件，那么你就可以通过`yoursite.com/project/github-pages`访问的到，如果你是使用一级域名的话。文件后缀可以是`.html`或者`markdown`或者`textile`。这里还有很多的例子：[https://github.com/mojombo/jekyll/wiki/Sites](https://github.com/mojombo/jekyll/wiki/Sites)

###Jekyll的配置
Jekyll的配置写在_config.yml文件中，可配置项有很多，我们不去一一追究了，很多配置虽有用但是一般不需要去关心，[官方配置文档][10]有很详细的说明，确实需要了可以去这里查，我们主要说两个比较重要的东西，一个是`Permalink`，还有就是自定义项。

`Permalink`项用来定义你最终的文章链接是什么形式，他有下面几个变量：

* `year` 文件名中的年份
* `month` 文件名中的月份
* `day` 文件名中的日期
* `title` 文件名中的文章标题
* `categories` 文章的分类，如果文章没有分类，会忽略
* `i-month` 文件名中的除去前缀0的月份
* `i-day` 文件名中的除去前缀0的日期

看看最终的配置效果：

* `permalink: pretty` /2009/04/29/slap-chop/index.html
* `permalink: /:month-:day-:year/:title.html` /04-29-2009/slap-chop.html
* `permalink: /blog/:year/:month/:day/:title` /blog/2009/04/29/slap-chop/index.html

我使用的是：

* `permalink: /:title` /github-pages

自定义项的内容，例如我们定义了`title:BeiYuu的博客`这样一项，那么你就可以在文章中使用`{ { site.title }}`来引用这个变量了，非常方便定义些全局变量。

###YAML Front Matter和模板变量
对于使用YAML定义格式的文章，Jekyll会特别对待，他的格式要求比较严格，必须是这样的形式：

    ---
    layout: post
    title: Blogging Like a Hacker
    ---

前后的`---`不能省略，在这之间，你可以定一些你需要的变量，layout就是调用`_layouts`下面的某一个模板，他还有一些其他的变量可以使用：

* `permalink` 你可以对某一篇文章使用通用设置之外的永久链接
* `published` 可以单独设置某一篇文章是否需要发布
* `category` 设置文章的分类
* `tags` 设置文章的tag

上面的`title`就是自定义的内容，你也可以设置其他的内容，在文章中可以通过`{ { page.title }}`这样的形式调用。

模板变量，我们之前也涉及了不少了，还有其他需要的变量，可以参考官方的文档：[https://github.com/mojombo/jekyll/wiki/template-data](https://github.com/mojombo/jekyll/wiki/template-data "Jekyll Template Data")

###jekyll 本地化测试
    D:>jekyll new myblog
    D:>jekyll serve
启动报错：

    E:/RailsInstaller/Ruby2.0.0/lib/ruby/gems/2.0.0/gems/posix-spawn-0.3.9/lib/posix/spawn.rb:162: warning: cannot close fd before spawn
    ←[31m  Liquid Exception: No such file or directory - /bin/sh in 2012-01-17-test-post.md←[

解决办法：
首先，根据[Jekyll在Windows下面中文编码问题解决方案](http://www.cnblogs.com/aleda/articles/Jekyll-in-Windows-following-Chinese-encoding-problem-solutions.html)这里的解决方案，我全都找了一遍，但是依然没有解决。
而且`convertible.rb`文件中，根本就没有文章中提到的那段添加`utf-8`的代码。
最后这个文章中提到的编码格式、换行符、YAML格式，我这里都没有问题。

接着看到[stackoverflow](http://stackoverflow.com/questions/17364028/jekyll-on-windows-pygments-not-working)解决方案，删除0.6版本的pygments，安装0.5版本。尝试如下：

    gem uninstall pygments.rb --version ">0.6.0"
    gem install pygments.rb --version "=0.5.0"

启动仍然报错，故移除jekyll最新版本重新安装了个较低的版本 gem install jekyll -v 1.5.1，出现如下错误：

    Destination: d:/myblog/_site
    Generating... Error reading file d:/myblog/_layouts/post.html: invalid byte sequence in GBK
    Liquid Exception: Failed to get header. in _posts/2014-11-17-welcome-to-jekyll.markdown
    error: Failed to get header.. Use --trace to view backtrace

然后配置了_config.yml,注释掉了原来的这行代码#markdown: kramdown，重新添加如下两行：

    encoding: utf-8
    markdown: rdiscount

启动项目仍然出现上面的错误，这时参考了[run jekyll --server failed in win7](http://stackoverflow.com/questions/14253116/run-jekyll-server-failed-in-win7")和[Jekyll Liquid Exception Error](http://akenn.org/blog/Liquid-Exception-Jekyll/)两篇文章，觉得可能是Python版本有问题，之前我装的是3.0的最新版本，故卸载该版本，重新安装了[Python 2.7.2](http://www.python.org/ftp/python/2.7.2/python-2.7.2.msi)，配置了环境变量，安装了distribute-0.6.49。

再次启动项目，发现不报错了，浏览器输入<a>http://localhost:4000/</a>项目可以正常访问了。

    cd D:/myblog
    $ jekyll serve
    Configuration file: d:/myblog/_config.yml
    Source: d:/myblog
    Destination: d:/myblog/_site
    Generating... done.
    Server address: http://0.0.0.0:4000
    Server running... press ctrl-c to stop.

## 五、使用Disqus管理评论
模板部分到此就算是配置完毕了，但是Jekyll只是个静态页面的发布系统，想做到关爽场倒是很容易，如果想要评论呢？也很简单。

现在专做评论模块的产品有很多，比如[Disqus][]，还有国产的[多说][]，Disqus对现在各种系统的支持都比较全面，到写博客为止，多说现在仅是WordPress的一个插件，所以我这里暂时也使用不了，多说与国内的社交网络紧密结合，还是有很多亮点的，值得期待一下。我先选择了Disqus。

注册账号什么的就不提了，Disqus支持很多的博客平台，参见下图：
![Disqus sites](http://siberiawolf.qiniudn.com/images/githubpages/disqus-site.jpg)

我们选择最下面的`Universal Code`就好，然后会看到一个介绍页面，把下面这段代码复制到你的模板里面，可以只复制到显示文章的模板中：

    <div id="disqus_thread"></div>
    <script type="text/javascript">
        /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
        var disqus_shortname = 'example'; // required: replace example with your forum shortname 这个地方需要改成你配置的网站名

        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
    <a href="http://disqus.com" class="dsq-brlink">blog comments powered by <span class="logo-disqus">Disqus</span></a>

配置完之后，你也可以做一些异步加载的处理，提高性能，比如我就在最开始页面打开的时候不显示评论，当你想看评论的时候，点击“显示评论”再加载Disqus的模块。代码很简单，你可以参考我的写法。


如果你不喜欢Disqus的样式，你也可以根据他生成的HTML结构，自己改写样式覆盖它的，Disqus现在也提供每个页面的评论数接口，[帮助文档][12]在这里可以看到。

##六、结语
OK，如果你跟着这篇教程操作顺利的话，那么恭喜你，你已经成功搭建了自己的博客，剩下的就是保持热情去写自己的文章吧！

[BeiYuu]:    http://beiyuu.com  "BeiYuu"
[Github]:   http://github.com "Github"
[jQuery]:   https://github.com/jquery/jquery "jQuery@github"
[Twitter]:  https://github.com/twitter/bootstrap "Twitter@github"
[Github Pages]: http://pages.github.com/ "Github Pages"
[Godaddy]:  http://www.godaddy.com/ "Godaddy"
[Jekyll]:   https://github.com/mojombo/jekyll "Jekyll"
[DNSPod]:   https://www.dnspod.cn/ "DNSPod"
[Disqus]: http://disqus.com/
[多说]: http://duoshuo.com/
[1]:    {{ page.url}}  ({{ page.title }})
[2]: http://markdown.tw/    "Markdown语法"
[3]:    http://baike.baidu.com/view/65575.htm "A记录"
[4]: http://progit.org/book/zh/ "Pro Git中文版"
[5]: http://help.github.com/mac-set-up-git/ "Mac下Git安装"
[6]: http://help.github.com/ssh-key-passphrases/
[7]: http://beiyuu.github.com
[8]: https://github.com/mojombo/jekyll/blob/master/README.textile
[9]: https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter
[10]: https://github.com/mojombo/jekyll/wiki/configuration
[11]: https://github.com/beiyuu/beiyuu.github.com
[12]: http://docs.disqus.com/developers/universal/
[13]: http://mihai.bazon.net/projects/javascript-syntax-highlighting-engine
[14]: http://code.google.com/p/google-code-prettify/
[15]: https://github.com/mojombo/jekyll/wiki/Install
[16]: https://rvm.io/rvm/install/
[17]: http://jekyllbootstrap.com/
[18]: http://chxt6896.github.com/blog/2012/02/13/blog-jekyll-native.html
[a-record]: https://help.github.com/articles/my-custom-domain-isn-t-working
