---
attachments: [Clipboard_2022-12-22-16-45-37.png, Clipboard_2022-12-22-16-46-04.png, Clipboard_2022-12-22-16-46-39.png]
tags: [Vscode]
title: vscode docker 配置xdebug3
created: '2022-12-22T08:39:51.029Z'
modified: '2022-12-22T08:52:41.877Z'
---

# vscode docker 配置xdebug3
#### docker-php-ext-xdebug.ini
```
zend_extension=xdebug

xdebug.mode=debug
xdebug.start_with_request=yes
#本机ip，因为docker在mac中不能使用host网络模式，故这里不能用localhost
xdebug.client_host=host.docker.internal
#本机端口
xdebug.client_port=9100
xdebug.idekey = PHPSTORM
```
## VS Code 配置
> VS Cdoe 要提前安装 PHP Debug

一般在 VS Code 里面开启一个 PHP 项目，点击调试工具，它会自动提示生成一个 .vscode 的目录，在目录下，会有一个 launch.json, 如：
![](@attachment/Clipboard_2022-12-22-16-45-37.png)
#### 第一步 添加 Xdebug for Docke 配置
![](@attachment/Clipboard_2022-12-22-16-46-04.png)
会生成：
![](@attachment/Clipboard_2022-12-22-16-46-39.png)
#### 第二步 指定 pathMappings 映射关系
> 也就是指明在 容器里面目录 对应 当前本地的目录
```
"pathMappings": {
  "/var/www/html": "${workspaceRoot}/www"
}
```
#### 在vscode里断点好后。按F5，等待请求
