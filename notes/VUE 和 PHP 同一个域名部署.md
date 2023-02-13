---
tags: [Nginx]
title: VUE 和 PHP 同一个域名部署
created: '2022-04-29T16:42:33.248Z'
modified: '2022-04-29T16:43:34.837Z'
---

# VUE 和 PHP 同一个域名部署

```
server {
    listen 80;
		root /data01/www/home.console.insurforce.cn/dist;

		// 默认是前端的 VUE 返回
		location / {
        try_files $uri /index.html;
    }

		// VUE 里面把所有请求发到 /api/ 开头的地址
		location ~ ^/api/ {
        try_files $uri $uri/ /index.php?$query_string;
    }

		// PHP 处理
		location ~ \.php {
        root /data01/www/api.console.insurforce.cn/public;
        gzip off;

        # keep alive
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_send_timeout 300;
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect off;

        fastcgi_pass  127.0.0.1:9000;
        fastcgi_keep_conn on;
        fastcgi_index index.php;

        fastcgi_split_path_info       ^(.+\.php)(/.+)$;
        fastcgi_param PATH_INFO       $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;

				// PHP 里面修改 REQUEST_URI，避免 PHP 继续读取 /api/login 开头的报错
        if ($request_uri ~ ^/api/(.+)$) {
            set $new_request_uri $1;
        }
        fastcgi_param REQUEST_URI $new_request_uri;
    }
```



