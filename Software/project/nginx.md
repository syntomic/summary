- 默认配置文件:`/etc/nginx/nginx.conf`
    - 单独文件配置:`/etc/nginx/sites-enabled` or `/etc/nginx/conf.d`
- 常用指令
    - events: 事件设置, 定义连接
    - http: HTTP设置, 包含server和upstream两种块
    - server: 主机设置, 每一个server块表示一个主机(域名), 包含location块
    - location: URL设置, 每一个lcation块定义一个URL模式
    - upstream: 负载均衡设置
- Nginx配置: `/etc/nginx/sites-enabled`目录下的配置文件会被自动插入到全局配置文件(`/etc/nginx/nginx.conf`)的`http`块中
```nginx
server {
    listen 443 ssl;
    server_name _; # 如果你映射了域名,那么可以写在这里
    ssl on;
    ssl_certificate /home/cert/syntomic.top.pem; #pem证书路径
    ssl_certificate_key     /home/cert/syntomic.top.key; #pem证书key路径
    ssl_session_timeout     5m; #会话超时时间
    ssl_ciphers     ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #加密算法
    ssl_protocols   TLSv1 TLSv1.1 TLSv1.2; #SSL协议

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
    proxy_pass http://127.0.0.1:8000; # 转发的地址,即Gunicorn运行的地址
    proxy_redirect      off;

    proxy_set_header    Host               $host;
    proxy_set_header    X-Real-IP          $remote_addr;
    proxy_set_header    X-Forwarded-For    $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto  $scheme;
    }

    location /static { # 处理静态文件夹中的静态文件
        alias /home/zhouh/cleanlog/cleanlog/static/;
        expires 30d; # 设置缓存过期时间
    }
}
```
- 测试语法正确性: `sudo nginx -t`
- 重启Nginx让配置生效: `sudo service nginx restart`