---
attachments: [Clipboard_2022-05-03-18-07-10.png, Clipboard_2022-05-03-18-08-13.png, Clipboard_2022-05-03-18-08-19.png, Clipboard_2022-05-03-18-08-41.png, Clipboard_2022-05-03-18-12-27.png, Clipboard_2022-05-03-18-14-58.png, Clipboard_2022-05-03-18-15-58.png, Clipboard_2022-05-03-18-16-43.png, Clipboard_2022-05-03-18-17-10.png, Clipboard_2022-05-03-18-17-56.png]
tags: [MacOs]
title: Mac 终端工具iTerm2安装
created: '2022-05-03T10:03:49.530Z'
modified: '2022-05-03T10:21:18.878Z'
---

# Mac 终端工具iTerm2安装

### iTerm2简介
Mac OS自带的终端，用起来虽然有些不太方便，界面也不够友好,iTerm2是一款相对比较好用的终端工具.iTerm2常用操作包括主题选择、声明高亮、自动填充建议、隐藏用户名和主机名、分屏效果等.
先来看效果图:
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-07-10.png" width="80%">
</center>

### 下载及安装
iTerm2下载地址：https://www.iterm2.com/downloads.html
下载的是压缩文件，解压后直接双击执行程序文件，或者直接将它拖到 Applications 目录下。
也可以直接使用Homebrew进行安装
```
$ brew cask install iterm2
```
### 配置 iTerm2 主题
iTerm2 最常用的主题是 Solarized Dark theme，下载地址：http://ethanschoonover.com/solarized
下载的是压缩文件，解压，然后打开 iTerm2，按Command +键，打开 Preferences 配置界面，然后Profiles -> Colors -> Color Presets,在下拉列表中选择 Import，选择刚才解压的solarized->iterm2-colors-solarized->Solarized Dark.itermcolors文件.导入成功后,在 Color Presets下选择 Solarized Dark 主题，就可以了。
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-08-13.png" width="80%">
</center>
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-08-19.png" width="80%">
</center>

### 设置 iTerm2 背景图片
打开 iTerm2，按Command +键，打开 Preferences 配置界面Profiles -> Window->Background mage,选择一张自己喜欢的背景图.
<a href="@attachment/Clipboard_2022-05-03-18-12-27.png" target="_blank">背景图下载一</a>
### 配置 Oh My Zsh
Oh My Zsh 是对主题的进一步扩展,下载地址https://github.com/robbyrussell/oh-my-zsh
- 一键安装：
```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
- 安装好之后，需要把 Zsh 设置为当前用户的默认 Shell（这样新建标签的时候才会使用 Zsh）：
```
$ chsh -s /bin/zsh
```
- 然后，将主题修改为ZSH_THEME="agnoster"。
```
$ vim ~/.zshrc
```
输入i进入编辑模式,将ZSH_THEME=""编辑为 ZSH_THEME="agnoster"
按下esc键,退出编辑,:wq保存退出:
修改成功!
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-14-58.png" width="80%">
</center>
agnoster是比较常用的 zsh 主题之一，你可以挑选你喜欢的主题，zsh 主题列表：https://github.com/robbyrussell/oh-my-zsh/wiki/themes
### 配置 Meslo 字体
使用上面的主题，需要 Meslo 字体支持，要不然会出现乱码的情况，字体下载地址：Meslo LG M Regular for Powerline.ttf
下载好之后，直接在 Mac OS 中安装即可。
然后打开 iTerm2，按Command + ,键，打开 Preferences 配置界面，然后Profiles -> Text -> Font -> Chanage Font，选择 Meslo LG M Regular for Powerline 字体。
### 声明高亮
特殊命令和错误命令，会有高亮显示。
使用 Homebrew 安装：
```
$ brew install zsh-syntax-highlighting
```
安装成功之后，编辑vim ~/.zshrc文件，在最后一行增加下面配置：
```
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-15-58.png" width="80%">
</center>

### 自动建议填充
这个功能是非常实用的，可以方便我们快速的敲命令。
配置步骤，先克隆zsh-autosuggestions项目，到指定目录：
```
$ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```
然后编辑vim ~/.zshrc文件，找到plugins配置，增加zsh-autosuggestions插件。
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-16-43.png" width="80%">
</center>

### iTerm2 快速隐藏和显示窗体:
打开 iTerm2，按Command + ,键，打开 Preferences 配置界面，然后Profiles → Keys →Hotkey，自定义一个快捷键就可以了。
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-18-17-10.png" width="80%">
</center>

### iTerm2 隐藏用户名和主机名
有时候我们的用户名和主机名太长，终端显示的时候会很不好看，我们可以手动去除。
```
编辑vim ~/.zshrc文件，增加DEFAULT_USER="SKong"配置.
```
但是: 我的电脑没生效,至今未找到原因~~~
补充:已经找到原因,这个地方应该设置为: DEFAULT_USER="你电脑的用户名"
比如:我的电脑用户名为pactepacterara,则DEFAULT_USER="pactepacterara"
感谢网友码渣!
我们可以通过whoami命令，查看当前用户
![](@attachment/Clipboard_2022-05-03-18-17-56.png)
#### iTerm2 快捷命令
```
command + t 新建标签
command + w 关闭标签
command + 数字 command + 左右方向键    切换标签
command + enter 切换全屏
command + f 查找
command + d 水平分屏
command + shift + d 垂直分屏
command + option + 方向键 command + [ 或 command + ]    切换屏幕
command + ; 查看历史命令
command + shift + h 查看剪贴板历史
ctrl + u    清除当前行
ctrl + l    清屏
ctrl + a    到行首
ctrl + e    到行尾
ctrl + f/b  前进后退
ctrl + p    上一条命令
ctrl + r    搜索命令历史 
```
