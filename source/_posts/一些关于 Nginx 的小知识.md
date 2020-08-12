---

title: 一些关于 Nginx 的小知识

category: Nginx

date: 2020-08-10

author: 林鸿鹄

---

```
curl --silent --location https://rpm.nodesource.com/setup_7.x | bash -
```

- curl 获取页面
- silent 不要太多console
- location http 转发
- bash 执行页面的 bash 文件

```
npm install -g pm2 http-server
```

### PM2
- 内建 node 集群
- 后台运行
- 停止不稳定的进程

### http-server
- 可以本地开启http服务器来测试
- p 要使用的端口（默认为8080）
- a 要使用的地址（默认为0.0.0.0）


### Nginx 建造
- Https 搭建
- Loading Balancing 负载均衡
- 服务静态文件 static file


### 网络接口
- IP Address 188.166.227.208
- ifconfig （interface config）
- nestat -tln（network statistic）
 - t tcp
 - listen sockets
 - n with number on port and host

 
### Nginx 配置
- linux 指令大全： https://man.linuxde.net/systemctl
- systemctl 系统服务管理器指令

开起 nginx

```
systemctl start nginx
```

重启 nginx

```
systemctl restart nginx
```


当服务器开启再开启 nginx
/Users/raylin/Documents/VUE instance .md
```
systemctl enable nginx
```

nginx 文件

```
vim /etc/nginx/nginxconf
```

nginxconf 文件详情

```
server{
	listen 80
	listen
	root
	server_name rayreact.com

	location / {
		proxy_pass "http://localhost:8080"
	}
}
```

### Proxying WebSockets with Nginx

- websocket hop-by-hop 

```
server{
	location / {
		proxy_pass "http://localhost:8080/"
		
	}
	
	location /socket.io/ {
		proxy_http_version 1.1
		proxy_set_header Upgrade $http_upgrade
		proxy_set_header Connection "upgrade"
		proxy_pass "http://localhost:8080/socket.io"
	}
}
```

```
proxy_http_version 1.1
```

是告诉nginx使用HTTP/1.1通信协议，这是websoket必须要使用的协议。

```
proxy_set_header Upgrade $http_upgrade
proxy_set_header Connection "upgrade"
```

第二行和第三行告诉nginx，当它想要使用WebSocket时，响应http升级请求。

### 客户端 IP Nginx 反向代理

```
server {
	proxy_set_header Host $http_host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forward_for;
}
```

信息来自 

- https://www.youtube.com/watch?v=4p1Zc8F29Lk&list=PLQlWzK5tU-gDyxC1JTpyC2avvJlt3hrIh&index=12
