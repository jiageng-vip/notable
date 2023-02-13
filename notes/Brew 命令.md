---
tags: [Brew]
title: Brew 命令
created: '2022-05-03T08:56:49.050Z'
modified: '2022-05-03T11:44:49.842Z'
---

# Brew 命令

Homebrew是一款超级好用的包管理工具，可以实现快速搭建各种开发环境。
- 如果没有安装brew，可以执行一下命令进行安装。
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
- 安装Nginx
```
brew install nginx
```
配置文件：/usr/local/etc/nginx/
启动：brew services start nginx
停止：brew services stop nginx
重启：brew services restart nginx
- 安装PHP([version]代版本，如56)
```
brew install php-[version]
```
配置文件：/usr/local/etc/php/
启动PHP-FPM：sudo php-fpm -D 或者 brew services start php72(php版本号)
停止PHP-FPM：sudo kill -INT cat /usr/local/var/run/php-fpm.pid
重启PHP-FPM：sudo kill -USR2 cat /usr/local/var/run/php-fpm.pid
- 安装Mysql
```
brew install mysql
```
配置文件：/usr/local/etc/my.cnf
启动：brew services start mysql
停止：brew services stop mysql
重启：brew services restart mysql
```
命令	说明
brew search [TEXT|/REGEX/]	搜索软件
brew (info|home|options) [FORMULA...]	查询软件信息
brew install FORMULA...	安装软件
brew update	更新brew和所有软件包
brew upgrade [FORMULA...]	更新软件包
brew uninstall FORMULA...	卸载软件包
brew list [FORMULA...]	罗列所有已安装的软件包
brew outdated	查看哪些软件包需要更新
brew deps FORMULA	显示包依赖
man brew	查询brew命令的使用手册
brew help [COMMAND	查询brew子命令的使用帮助，如brew help search
brew config	查看brew的全局配置
brew doctor	检查系统的潜在问题
brew install -vd FORMULA	安装软件包打印详细信息，并开启debug功能。
brew create [URL [--no-fetch]]	创建软件包
brew edit [FORMULA...]	编辑软件包源码
```
