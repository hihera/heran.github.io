---
layout: post
title: Sublime Text 3 使用介绍
description: Sublime Text 被称为性感无比的编辑器，本文主要介绍如何使用其强大的功能。
category: 技术分享
tags: [Sublime Text,编辑器]
---
{% include JB/setup %}
# Sublime Text 3 使用介绍



##Sublime Text 是什么
Sublime Text 是一款跨平台代码编辑器（Code Editor），也是HTML和散文先进的文本编辑器，你不仅可以用它来写代码，还可以用它来写文章做笔记。从最初的Sublime Text 1.0，到现在的 3.0，被无数的开发者所喜爱。

<!--break-->

###为什么选择它？
**跨平台**：可以在 Linux 、OS X 和 Windows 任意操作系统下使用，减少了重复学习的成本，这一点和 Vim 有异曲同工之妙。  
**可扩展**：它支持大部分的程序语言，有超过 1,300 “packages” 可以扩展它的功能，你可以安装自己领域的插件来成倍提高工作效率。  
**效率高**：Sublime Text 拥有强大的特性，像任意跳转、分区编辑，快速项目切换等，学会使用它会大大提高工作效率。  


Sublime Text 编辑器面向无语义的纯文本，不涉及领域逻辑，体积小速度快，非常适合编写单独的配置文件和动态脚本语言（如Shell、Python和Ruby等）学会使用它，绝对会使你的工作事倍功半，下面就来讲讲如何优雅的使用它。

##安装（Installation）
官网：<http://www.sublimetext.com/3>

###添加 Sublime Text 到环境变量
键盘 `Win + R`，运行 `sysdm.cpl` 打开系统属性，切换到“高级”选项卡，选择“环境变量”，在系统变量里找到 path，编辑，增加你 Sublime 的安装路径，例如：

    C:\Sublime Text 3

这样你就可以通过 `Win + R` ，运行cmd，在命令行利用 `subl` 命令直接使用 Sublime Text 。

    > subl filename   "使用sublime text 打开某个文件,filename 为文件名 "
    > subl foldername "使用sublime text 打开某个文件夹，foldername 为文件夹名"
    > subl .          "使用sublime text 打开当前路径下的文件夹"
    > $ git config --global core.editor "subl --new-window --wait" 配置git上默认编辑方式

##安装 Package Control
Package Control 是一个必备的包管理器，前面提到的插件安装就是在这个基础上安装的。
安装教程：<http://sublime.wbond.net/installation>

通过 `View -> Show Console ` 启动控制台，复制粘贴下面命令到最后一行输入框，回车等待安装完成。

    > import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)

安装成功后，使用`Ctrl + Shift + P`，打开命令版，输入 `PCIP` （支持模糊匹配,可以使用文件名的前缀、首字母或是某部分进行匹配）,出现 Package Control，这表名你可以安装任何你想用的插件了。

##插件安装
###常用插件介绍

推荐以下几款个人觉得比较常用的插件，其中所有插件的安装都是先使用`Ctrl + Shift + P`，打开命令面板安装Package Control，然后再在弹出的输入框内输入要安装的插件名称（可以模糊匹配插件名称）。

**ConvertToUTF8**　          支持 GBK等编码的插件，安装它后就会解决中文乱码问题。  
**Bracket Highlighter**         用于匹配括号，引号和html标签。安装好后，自动生效。  
**AllAutocomplete**             从所有打开的文件中匹配查找的内容，是对默认autocomplete的扩展。  
**SublimeCodeIntel**           支持各种语言的自动提示功能。使用参考：<a href="http://sublime.wbond.net/packages/SublimeCodeIntel"></a>  
**ColorPicker**                   调色盘，默认快捷键`Cmd + Shift + C`，冲突的话用上面的方式自行修改。  
**BracketHighlighter**          高亮匹配 [] ,  () ,  {} ,  <tag></tag>等，对于快速定位代码段非常有用。  
**DocBlockr**　　              可以自动生成PHPDoc风格的注释。支持Javascript, PHP, Java,  C等。  
**Emmet(Zen Coding)**       快速生成 HTML 代码段的插件。  
**SideBar Enhancements**  增强侧边栏功能。配置浏览器预览效果参考：<a href="http://dengo.org/archives/923"></a>   
**JsFormat javascript**         javascript代码格式化，按快捷键Ctrl+Alt+F可格式化当前的js文件。  
**Git**                                 将Git整合进你的SublimeText，在SublimeText中运行Git命令。  
**GitGutter**                       它可以在行数的前面显示那一行是被增加，修改还是删除。  
**Alignment**                      等号进行智能对齐，可以将凌乱的代码以等号为准左右对齐。
默认快捷键`Ctrl+Alt+A`，（与QQ截图快捷键冲突，可以在 `Preferences->Key Bindings-User` 中自己修改快捷键，加入下面代码）

    > { "keys": ["f10"], "command": "alignment" }


###Sublime for Ruby

1、添加 Sublime Text 到环境变量和安装 Package Control，开始已完成，跳过此步。  
2、安装 Soda theme 和 RailsCasts color scheme 主题风格

* `Ctrl + Shift + P` 输入 `PCIP` 找到 “Package Control: Install Package” 回车
* 输入`RailsCasts Colour Scheme` 可以模糊匹配
* 同上操作安装`Theme - Soda`
* 安装完成后在 `Preferences->Settings -User`里边加上这句话激活主题 

    > `"theme": "Soda Light 3.sublime-theme"`

3、安装 Ruby Debugger，具体安装步骤参考 <a href="http://sublime.wbond.net/packages/Ruby%20Debugger"></a>

### Sublime for Markdown

1、安装 Markdown Preview

* `Ctrl + Shift + P` 输入 `PCIP` 找到 “Package Control: Install Package” 回车
* 输入`Monokai Preview` 可模糊匹配。支持在浏览器中预览和直接转化为 Html。

2、安装 Markdown 语法高亮主题。

* 同上安装 `Monokai Extended`，然后通过`Preferences->Color Scheme->Monokai Extended`启用。

3、为 Markdown 文件中其他语法添加高亮效果

* 同样安装`Markdown Extended`，打开一个markdown文件在右下角的语言栏选择 Markdown Extended激活这种语言高亮。

##常用快捷键

`Ctrl + Shift + P`打开Package Control，上面已经用过了   
`Ctrl + Shift + D` 复制光标所在整行，插入在该行之前  
`ctrl + shift + F` 在文件夹内查找，与普通编辑器不同的地方是sublime允许添加多个文件夹查找  
`Ctrl + Shift + K` 删除整行  
`Ctrl + Shift + L` 鼠标选中多行（按下快捷键），即可同时编辑这些行  
`Ctrl + Shift + M` 选择括号内的内容（按住-继续选择父括号）  
`Ctrl + Shift + [`  折叠代码  
`Ctrl + Shift + ]` 展开代码  
`Ctrl + Shift + Enter` 光标前插入行  
`Ctrl + PageDown` 光标后插入行  
`Ctrl  + Shift  + V` 格式化粘贴  
`Ctrl + PageUp` 文件按开启的前后顺序切换  
`Ctrl + J` 合并行（已选择需要合并的多行时）  
`Ctrl + L` 选择整行（按住-继续选择下行）  
`Ctrl + K Backspace` 从光标处删除至行首  
`Ctrl + KB` 开启/关闭侧边栏  
`Ctrl + KK` 从光标处删除至行尾  
`Ctrl + KU` 改为大写  
`Ctrl + KL` 改为小写  
`Ctrl + K0` 展开所有  
`Ctrl + Tab` 当前窗口中的标签页切换  
`Ctrl + F2` 设置书签  
`Shift+ F2` 上一个书签  
`Ctrl + /` 注释整行（如已选择内容，同“Ctrl+Shift+/”效果）  
`Ctrl + 鼠标左键` 可以同时选择要编辑的多处文本  
`Shift + 鼠标右键`（或使用鼠标中键）可以用鼠标进行竖向多行选择  
`Shift + Tab` 去除缩进  
`Alt + Shift + 1~9`（非小键盘）屏幕显示相等数字的小窗口  
`Alt + F3` 选中文本按下快捷键，即可一次性选择全部的相同文本进行同时编辑  
`F6` 检测语法错误  
`F9` 行排序(按a-z)  
`Ctrl + P` （Go To Anything）根据文件名打开文件,可以模糊匹配，可以输入路径文件名称  
`Ctrl + R` 找到了需要的文件后，想要在文件中找某个函数，可以输入 `@函数名`  
`Ctrl + G` 输入`:数字` 回车跳到对应行；输入`#关键字`，可找到对应的标识  
`Ctrl + Enter` 在当前行下面新增加一行然后跳至该行  
`Ctrl + Shift + Enter` 在当前行上面增加一行并跳至该行  
`Ctrl + ←/→` 进行逐词移动  
`Ctrl + Shift + ←/→` 进行逐词选择  
`Ctrl + ←/→`进行逐词移动  
`Ctrl + Shift + ←/→`进行逐词选择  
`Ctrl + Shift+F` 在当前打开的项目中，根据你的输入进行全项目查找  
`Ctrl + D` 多重选择，高亮当前选中词，再次 `Ctrl + D`选择该词出现的下一个位置  
`Ctrl + K` 跳过  
`Ctrl + U` 回退  
`Ctrl + Shift + L` 可以将当前选中区域打散，然后进行同时编辑  
`Ctrl + D`选中关键字，`F3`跳到其下一个出现位置编辑`Shift + F3` 跳到其上一个出现位置  
`Alt + F3` 选中所有出现选中的内容，可以进行全部替换  
`Ctrl + H` 可以进行标准查找替换  
`Ctrl + Shift + H` 替换当前关键字  
`Ctrl + Alt + Enter`替换所有匹配关键字  
`Ctrl + N` 在当前窗口创建一个新标签  
`Ctrl + W` 关闭当前标签  
`Ctrl + Shift + T`恢复刚刚关闭的标签  
`Ctrl + 数字键` 跳转到指定屏  
`Ctrl + Shift + 数字键` 将当前屏移动到指定屏  
`F11` 切换普通全屏,在开启全屏前可以关闭菜单栏（"Toggle Menu"）  
`Shift + F11` 切换无干扰全屏  
`Ctrl + [` 向左缩进  
`Ctrl + ]` 向右缩进  
`Ctrl + Shift + V` 可以以当前缩进粘贴代码  
`Ctrl + M` 可以快速的在起始括号和结尾括号间切换  
`Ctrl + Shift + M` 则可以快速选择括号间的内容  
`Ctrl + F` 可以进行查找,最下面，会提示总共几个，匹配上了几个。在 “Find What” 输入框里边，按住`Enter` 查找下一个，`Shift + Enter` 查找上一个，`Alt + Enter` 查找所有  

## 链接

* 官方文档：<http://www.sublimetext.com/docs/3/>
* 官方论坛：<http://www.sublimetext.com/forum/>
* 全程指南：<http://zh.lucida.me/blog/sublime-text-complete-guide/>
* 插件支持：<http://blog.jobbole.com/58725/>
* 其他语言：<http://opensourcehacker.com/2014/03/10/sublime-text-3-for-python-javascript-and-web-developers/>

