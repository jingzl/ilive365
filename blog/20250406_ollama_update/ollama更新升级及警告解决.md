[tu]
## 1. 概述
在大模型业务处理中，需要用到gemma3 和 qwen2.5-VL，当前服务器的ollama版本 0.3.11，无法满足要求，需要更新升级。
## 2. 实施过程
参考官网升级要求：
```python
curl -fsSL https://ollama.com/install.sh | sh
```
不知道啥原因，访问一直超时，无法下载。没有办法只好在本地通过 VPN下载，然后上传到服务器。
### 1）查看本地当前版本，并停止，确保端口不被占用。
```python
# 当前系统中ollama路径
which ollama
# 输出
/usr/local/bin/ollama
 
# 先停止当前版本
sudo systemctl stop ollama
```
### 2）找到当前最新版本 0.6.4
到版本release页面 [Releases · ollama/ollama](https://github.com/ollama/ollama/releases/)，找到合适的平台包 （ollama-linux-amd64.tgz），然后上传到服务器。
### 3）解压
在解压之前，先检查压缩包中的文件内容，以确保不会覆盖重要文件：
```python
tar -tzf ollama-linux-amd64.tgz
# 可以看到输出：
bin/ollama
lib/ollama/cuda_v11/
lib/ollama/cuda_v11/libggml-cuda.so
lib/ollama/cuda_v11/libcublas.so.11
lib/ollama/cuda_v11/libcublas.so.11.5.1.109
lib/ollama/cuda_v11/libcublasLt.so.11.5.1.109
lib/ollama/cuda_v11/libcudart.so.11.3.109
lib/ollama/cuda_v11/libcublasLt.so.11
lib/ollama/cuda_v11/libcudart.so.11.0
lib/ollama/cuda_v12/
lib/ollama/cuda_v12/libggml-cuda.so
lib/ollama/cuda_v12/libcudart.so.12
lib/ollama/cuda_v12/libcudart.so.12.8.90
lib/ollama/cuda_v12/libcublasLt.so.12
lib/ollama/cuda_v12/libcublas.so.12.8.4.1
lib/ollama/cuda_v12/libcublas.so.12
lib/ollama/cuda_v12/libcublasLt.so.12.8.4.1
lib/ollama/libggml-base.so
lib/ollama/libggml-cpu-alderlake.so
lib/ollama/libggml-cpu-haswell.so
lib/ollama/libggml-cpu-icelake.so
lib/ollama/libggml-cpu-sandybridge.so
lib/ollama/libggml-cpu-skylakex.so
```
解压到 /usr 目录，根据上述可以看到，会更新 /usr 目录下的 bin 和 lib 目录。
```python
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```
这样相关新版本ollama都安装完毕。
### 4）更新 service
更新 /etc/systemd/system/ollama.service，主要是 ExecStart 路径：
```python
[Unit]
Description=Ollama Service
After=network-online.target
 
[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=$PATH"
Environment="CUDA_VISIBLE_DEVICES=0,1,2,3"
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_NUM_PARALLEL=4"
Environment="OLLAMA_MAX_LOADED_MODELS=2"
Environment="OLLAMA_KEEP_ALIVE=-1"
 
[Install]
WantedBy=default.target
```
然后执行更新：
```python
sudo systemctl daemon-reload
sudo systemctl enable ollama
sudo systemctl start ollama
```
查看版本：
```python
ollama -v
# 输出
ollama version is 0.6.4
Warning: client version is 0.3.11
```
### 5）解决警告
这个警告的原因，应该是上一个版本的信息残留，直接去 /usr/local/bin/ 路径下，删除 ollama，然后建立软链接：
```python
ln -s /usr/bin/ollama ollama
```
再次执行 ollama -v：
```python
ollama version is 0.6.4
# 正常
```
