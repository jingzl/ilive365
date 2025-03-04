# 开源监控工具：Checkmate

开源监控工具，用于服务器硬件状态监控，网站与接口监控，Docker容器监控等。

## 概述

**GitHub**：[Checkmate](https://github.com/bluewave-labs/Checkmate).

**官网文档**：[Welcome to Checkmate | Checkmate](https://docs.checkmate.so/)。

**功能特点：**：
> 服务器监控
- CPU使用率：区分用户态、系统态的使用情况，还能看到负载趋势
- 内存使用：包括物理内存和虚拟内存的使用量、剩余量、使用率
- 磁盘空间：监控各分区的使用情况，提前预警空间不足
- 系统负载：了解1分钟、5分钟、15分钟的平均负载
- 网络流量：监控网卡的出入带宽使用情况
- 进程信息：查看占用资源多的进程，便于定位问题

> 网站与接口监控
- 站点可用性：定期访问网站，验证返回码是否正常
- 响应时间：记录每次请求的耗时，绘制趋势图
- 内容验证：检查页面内容是否符合预期
- API监控：对重要接口进行定期调用测试
- SSL证书：检查证书是否临近过期
- 端口监控：确保关键端口服务正常运行

> Docker容器监控
- 容器状态：运行、停止、退出等状态变化
- 资源占用：CPU、内存、网络等资源使用情况
- 日志查看：实时查看容器的标准输出日志
- 镜像管理：容器使用的镜像版本信息

> 告警通知
- 邮件通知：最常用的告警方式
- Discord/Slack：适合团队协作的即时通知
- Webhook：可以对接到自己的系统
- 告警级别：区分紧急和普通告警
- 故障分析：记录告警历史，便于复盘

## 部署实施
**参考**：[Installing Checkmate | Checkmate](https://docs.checkmate.so/users-guide/quickstart)
### 1. 在本地安装
1）Download [Docker compose file](https://github.com/bluewave-labs/bluewave-uptime/raw/refs/heads/master/Docker/dist/docker-compose.yaml)
2）Run docker compose up to start the application
3）Now the application is running at http://localhost
Optional Config:
If you want to monitor Docker containers, uncomment this line in docker-compose.yaml:
```python
# volumes:
  # - /var/run/docker.sock:/var/run/docker.sock:ro
```
This gives the app access to your docker daemon via unix socket, please be aware of what you are doing.
安装的过程有点长，耐心等待。
安装完毕后，可以看到服务运行起来，可以使用 http://localhost 访问。


### 2. 在服务器安装
如果需要监控的是服务器，并且需要在外网查看状态，那么需要修改配置。
修改docker-compose.yaml 文件中的 UPTIME_APP_API_BASE_URL，如下：
```python
# 默认配置
UPTIME_APP_API_BASE_URL: "http://localhost:5000/api/v1" 
# 调整为远程服务器地址
UPTIME_APP_API_BASE_URL: "http://x.x.x.x:5000/api/v1"
```
记得打开防火墙对应的端口。

### 3. 系统监控Agent
**GitHub**：[bluewave-labs/capture: An open source hardware monitoring agent for Checkmate](https://github.com/bluewave-labs/capture)
**参考**：[Server monitoring agent | Checkmate](https://docs.checkmate.so/users-guide/server-monitoring-agent) 

如果要进行服务器设备的CPU、内存等硬件状态监控，需要安装 Capture，是一个checkmate的Agent。目前只能在linux系统使用。

docker方式安装：
```python
docker run -v /etc/os-release:/etc/os-release:ro \
    -p 59232:59232 \
    -e API_SECRET=REPLACE_WITH_YOUR_SECRET \
    -d \
    capture:latest
 
# REPLACE_WITH_YOUR_SECRET 需要设置 
```
If you don't want to pull the image, you can build and run it locally.
```python
docker buildx build -f Dockerfile -t capture .
 
docker run -v /etc/os-release:/etc/os-release:ro \
    -p 59232:59232 \
    -e API_SECRET=REPLACE_WITH_YOUR_SECRET \
    -d \
    capture:latest
```
这样就可以在 checkmate 前端进行添加创建了，创建后效果如下：


### 4. 其他配置
- Pagespeed monitors 
配置后一直显示 inactive，上网搜索可能的原因是服务器无法访问 Google PageSpeed Insights API，导致无法获取数据，这点在进行部署时，可能要处理一下。

- Maintenance
可以参考页面要求设置。

- 账号设置
可以邀请添加其他账号。

---






2025.~