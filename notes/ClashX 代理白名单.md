---
tags: [ClashX]
title: ClashX 代理白名单
created: '2022-06-21T02:19:03.161Z'
modified: '2022-06-21T02:23:31.392Z'
---

# ClashX 代理白名单

## ClashX（Mac）Git地址

- 编辑~/.config/clash/proxyIgnoreList.plist(如果没有该文件，则新建)
- 参考Demo文件加入自定义的白名单域名

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
	<string>192.168.0.0/16</string>
	<string>10.0.0.0/8</string>
	<string>172.16.0.0/12</string>
	<string>127.0.0.1</string>
	<string>localhost</string>
	<string>*.local</string>
	<string>*.crashlytics.com</string>
	<!-- 上面的不能删掉 -->
	...加入新的域名，如：<string>*.baidu.com</string>
</array>
</plist>
```

- 重启ClashX

## Clash For Windows

绕过系统代理
Clash for Windows在v 0.4.5 版本后可以自定义系统代理需要绕过的域名或IP
部分应用检测到系统代理会拒绝响应（例如网易云音乐uwp），此功能用于解决此类问题

- 打开config.yaml代理配置文件
```
port: 8888
socks-port: 8889
redir-port: 0
allow-lan: true
mode: Rule
log-level: info
external-controller: '0.0.0.0:6170'
secret: ''
Proxy:
  ...
Proxy Group:
  ...
Rule:
  ...
cfw-bypass:
  ... # 原有字段不用删除
  - 'music.163.com' # 网易云域名1
  - '*.music.126.net' # 网易云域名2
```

> cfw-bypass类型为数组，item为需要绕过的域名或节点，支持通配符*
> 最后一行对应系统中“请勿将代理服务器用于本地(Intranet)地址”选项，请确保此项在最底部
