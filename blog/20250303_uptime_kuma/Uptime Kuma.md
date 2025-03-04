# 开源监控工具：Uptime Kuma

## 1. 概述
**Github**：[louislam/uptime-kuma: A fancy self-hosted monitoring tool](https://github.com/louislam/uptime-kuma)

Uptime Kuma is an easy-to-use self-hosted monitoring tool.

Uptime Kuma 是一款开源的监控工具，可以帮助你实时监测网站或服务的状态，并在发生故障时及时通知。它支持多种监控方式（如 HTTP、Ping、TCP 等），且操作简单，适合个人或团队自托管使用。

**特点**：
- Monitoring uptime for HTTP(s) / TCP / HTTP(s) Keyword / HTTP(s) Json Query / Ping / DNS Record / Push / Steam Game Server / Docker Containers
- Fancy, Reactive, Fast UI/UX
- Notifications via Telegram, Discord, Gotify, Slack, Pushover, Email (SMTP), and [90+ notification services, click here for the full list](https://github.com/louislam/uptime-kuma/tree/master/src/components/notifications)
- Proxy support

## 2. 安装部署
**前提**：需要安装好Docker，可以参考《[Docker的安装及使用摘要-CSDN博客](https://blog.csdn.net/james506/article/details/140154707)》。

### 2.1 Docker部署
```python
docker run -d --restart=always -p 8000:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
````

### 2.2 Docker Compose部署

1）新建 uptime-kuma 目录，并新建 docker-compose.yaml 文件，内容如下：
```python
version: '3.3'
  
services:
  uptime-kuma:
    image: louislam/uptime-kuma
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - ./uptime-kuma:/app/data
    ports:
      - 8000:3001
```
部署在 ECS服务器上，并打开了 8000 端口。

2）启动
```python
docker-compose up -d
```

3）访问
通过  IP + 端口 访问。
第一次启动时，需要设置管理员账户。
配置默认进入页面为状态页后，如果需要登录，则访问：http://IP:端口/dashboard，进行登录。

4）添加监控项
根据需要选择合适的类型进行监控项的添加，可以设置分组，设置标签，并且可以设置HTTPS证书的检测。

[图]

5）设置邮件通知
注意点如下：
```python
端口：465
安全性：TLS
忽略TLS错误
 
# 注意：如果uptime kuma部署在阿里云的ECS服务器上，只能使用465端口，无法使用25端口
# 阿里云默认会 禁止基于 25 端口发信
465（ SMTP SSL 认证端口 ）
587 （ SMTP 非 SSL 认证端口 ）
```
### 2.3 源码部署
参考 github官网，下载最新代码，并提前安装 node环境，通过 pm2 进行服务管理。

## 3. 界面效果
图

## 4. 小结
相比 NetData 和 Checkmate 监测工具，uptime kuma 界面明快简洁，功能易用，对服务器压力小。

---

