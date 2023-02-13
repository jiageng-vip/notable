---
tags: [Source]
title: V2Club
created: '2022-05-03T11:48:01.959Z'
modified: '2022-05-03T11:54:01.542Z'
---

# V2Club

```
地址 ： https://join.v2fly.club/#/dashboard

账号 ： 1582363645@qq.com
密码 ： 48658521yyp

账号 ： jiageng_vip@163.com
密码 ： 48658521yyp

账号 ： 260734888@qq.com
密码 ： 48658521yyp
```
##### 命令行开启代理连接国外节点
```
// 安装 V2ray-core
brew tap v2ray/v2ray
brew install v2ray/v2ray/v2ray-core

// 修改 V2ray-core 配置文件
/usr/local/etc/v2ray/config.json
{
  "inbounds": [
    {
      "port": 7892,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "lsb-v1.lanan.world",
            "port": 5041,
            "users": [
              {
                "id": "dd8f12fc-0854-4bc6-ba56-ce06194b51d8",
                "alterId": 2
              }
            ]
          }
        ]
      }
    }
  ]
}
/*****************************************************************************
/* 获取重要数据节点步骤
/*vmess://eyJ2IjoiMiIsInBzIjoiXHVkODNjXHVkZGZhXHVkODNjXHVkZGY4IFx1MzAwY1YycmF5XHUzMDBkXHU3ZjhlXHU1NmZkMiIsImFkZCI6ImxzYi12MS5sYW5hbi53b3JsZCIsInBvcnQiOjUwNDIsImlkIjoiZGQ4ZjEyZmMtMDg1NC00YmM2LWJhNTYtY2UwNjE5NGI1MWQ4IiwiYWlkIjoiMiIsIm5ldCI6InRjcCIsInR5cGUiOiJub25lIiwiaG9zdCI6IiIsInBhdGgiOiIiLCJ0bHMiOiIifQ==
/* base64解码
/* {
/*    "v": "2",
/*    "ps": "🇺🇸 「V2ray」美国2",
/*    "add": "lsb-v1.lanan.world",
/*    "port": 5042,
/*    "id": "dd8f12fc-0854-4bc6-ba56-ce06194b51d8",
/*    "aid": "2",
/*    "net": "tcp",
/*    "type": "none",
/*    "host": "",
/*    "path": "",
/*    "tls": ""
/*}
/*****************************************************************************

// 启动 V2ray-core
brew services start v2ray-core

/******************************************************************************
/* cat /Users/lijiageng/Library/LaunchAgents/homebrew.mxcl.v2ray-core.plist
/* <?xml version="1.0" encoding="UTF-8"?>
/* <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
/* <plist version="1.0">
/*   <dict>
/*     <key>KeepAlive</key>
/*     <true/>
/*     <key>RunAtLoad</key>
/*     <true/>
/*     <key>Label</key>
/*     <string>homebrew.mxcl.v2ray-core</string>
/*     <key>ProgramArguments</key>
/*     <array>
/*       <string>/usr/local/Cellar/v2ray-core/4.31.3/bin/v2ray</string>
/*       <string>-config</string>
/*       <string>/usr/local/etc/v2ray/config.json</string>
/*     </array>
/*   </dict>
/* </plist>
/* /usr/local/Cellar/v2ray-core/4.31.3/bin/v2ray -config /usr/local/etc/v2ray/config.json 如果启动服务失败，运行实际命令，查看服务启动失败原因
/******************************************************************************

// 查看启动状态
brew services list
```
