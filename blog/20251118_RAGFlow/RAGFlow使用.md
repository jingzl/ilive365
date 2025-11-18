# 1. 概述
## 1.1 官网
github：[infiniflow/ragflow: RAGFlow is a leading open-source Retrieval-Augmented Generation (RAG) engine that fuses cutting-edge RAG with Agent capabilities to create a superior context layer for LLMs](https://github.com/infiniflow/ragflow)
## 1.2 功能特性
RAGFlow 是一个基于深度文档理解的开源 RAG（Retrieval-Augmented Generation，检索增强生成）引擎，专为个人和企业构建知识库问答系统而设计。其核心优势在于：
- 深度文档理解：不仅能处理文本，还能解析 Word、PPT、Excel、PDF（含扫描件）、图片、网页等多种格式，通过 OCR、表结构识别(TSR)和文档布局识别(DLR) 等技术提取有用信息
- 有据可查的回答：在输出答案时提供引用来源，可点击溯源到原始文档的具体位置，显著降低 AI 幻觉问题
- 可视化知识分块：将大文档切分为有逻辑的知识片段，用户可查看并手动调整分块效果
- 灵活的模型集成：支持 OpenAI、DeepSeek、百度文心一言、Ollama 等多种大语言模型和向量模型
- RAGFlow适用于企业搭建知识库

# 2. 部署实施
## 2.1 系统配置要求
**最低配置（官方推荐）**：
- CPU：至少 4 核，推荐 8 核以上
- 内存：至少 16 GB，推荐 32 GB 以上
- 硬盘：至少 50 GB，建议使用固态硬盘并预留充足空间
- GPU：非必需，但文档解析和 Embedding 时会消耗大量资源，有 GPU 可显著提速

**软件依赖：**
- Docker (>= 24.0.0)
- Docker Compose (>= v2.26.1)
## 2.2 系统参数调整
确保 vm.max_map_count 不小于 262144。
```
# 如需确认 vm.max_map_count 的大小：
$ sysctl vm.max_map_count
 
# 如果 vm.max_map_count 的值小于 262144，可以进行重置：
# 这里我们设为 262144:
$ sudo sysctl -w vm.max_map_count=262144
 
# 改动会在下次系统重启时被重置。如果希望做永久改动，还需要在 /etc/sysctl.conf 文件里把 vm.max_map_count 的值再相应更新一遍：
vm.max_map_count=262144
```
## 2.3 启动服务器
**1）克隆仓库**
```
$ git clone https://github.com/infiniflow/ragflow.git
```
**2）启动**
进入 docker 文件夹，利用提前编译好的 Docker 镜像启动服务器：目前官方提供的所有 Docker 镜像均基于 x86 架构构建，并不提供基于 ARM64 的 Docker 镜像。 如果你的操作系统是 ARM64 架构，请参考这篇文档（[Build RAGFlow Docker image | RAGFlow](https://ragflow.io/docs/dev/build_docker_image)）自行构建 Docker 镜像。
运行以下命令会自动下载 RAGFlow Docker 镜像 v0.22.0。请参考下表查看不同 Docker 发行版的描述。如需下载不同于 v0.22.0 的 Docker 镜像，请在运行 docker compose 启动服务之前先更新 docker/.env 文件内的 RAGFLOW_IMAGE 变量。
```
$ cd ragflow/docker

# 可选：使用稳定版本标签（查看发布：https://github.com/infiniflow/ragflow/releases），例如：git checkout v0.22.0
# 这一步确保代码中的 entrypoint.sh 文件与 Docker 镜像的版本保持一致。

# Use CPU for DeepDoc tasks:
$ docker compose -f docker-compose.yml up -d

# To use GPU to accelerate DeepDoc tasks:
# sed -i '1i DEVICE=gpu' .env
# docker compose -f docker-compose.yml up -d
```
注意：在 v0.22.0 之前的版本，会同时提供包含 embedding 模型的镜像和不含 embedding 模型的 slim 镜像。
从 v0.22.0 开始，只发布 slim 版本，并且不再在镜像标签后附加 -slim 后缀。

如果遇到 Docker 镜像拉不下来的问题，可以在 docker/.env 文件内根据变量 RAGFLOW_IMAGE 的注释提示选择华为云或者阿里云的相应镜像。
- 华为云镜像名：swr.cn-north-4.myhuaweicloud.com/infiniflow/ragflow
- 阿里云镜像名：registry.cn-hangzhou.aliyuncs.com/infiniflow/ragflow

服务器启动后，确认状态：
```
docker logs -f xxx-ragflow-cpu-1
# 输出
...
2025-11-17 10:46:12,574 INFO     22 
        ____   ___    ______ ______ __
       / __ \ /   |  / ____// ____// /____  _      __
      / /_/ // /| | / / __ / /_   / // __ \| | /| / /
     / _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ /
    /_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/

    
2025-11-17 10:46:12,574 INFO     22 RAGFlow version: v0.22.0
2025-11-17 10:46:12,574 INFO     22 project base: /ragflow
...
```
**3）访问**
通过http://IP 访问，默认端口80，可以自行在 .env 文件中调整。
图1
第一次访问，需要注册。
登录进去后，可以调整为 中文界面。
图2
**4）配置**
在右上角的设置中，可以进行数据源，大模型及MCP等配置。
点击右上角头像，获得RAGFlow的API KEY和API服务器地址。
图3
如果需要关闭注册，在 .env 文件中调整即可：
```
REGISTER_ENABLED=0
```
然后重启容器。可以看到，登录页面没有注册入口了。
# 3. 可能出现的问题
**1）镜像下载中报错**
error pulling image configuration: download failed after attempts=1: toomanyrequests: too many requests
解决方案：一方面更新docker的国内最新镜像源；另一方面把 .env 中的镜像地址调整为 华为源。
**2）启动报错**
Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "./entrypoint.sh": permission denied: unknown
原因分析：因为拷贝的仓库中的docker文件夹到服务器，entrypoint.sh 的权限不够
解决方案：chmod +x entrypoint.sh，然后重启容器即可。


