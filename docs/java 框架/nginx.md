# nginx

```bash
## 查看nginx是否启动
ps aux | grep nginx
## 查看客户端请求历史
tail -f /var/log/nginx/access.log
## 查看nginx错误日志
tail -f /var/log/nginx/error.log
## 查看 nginx 代理转发配置文件
vim /etc/nginx/sites-available/default
## 校验转发配置的合法性
nginx -t
## 配置热加载
nginx -s reload
```

代理配置示例

```json
server{
        listen 80; # 监听端口
        server_name dpbirder.dev.jd.local;
        location / { # 匹配路径
                proxy_pass http://127.0.0.1:1601/datacenter/api$uri; # 代理转发路径
                proxy_set_header Host $host;
                proxy_set_header X_Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_connect_timeout 10s;
                proxy_send_timeout 10s;
                proxy_read_timeout 10s;
        }
}
```
