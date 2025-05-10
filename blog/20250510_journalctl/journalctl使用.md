图

journalctl可以查看很多服务的日志，比如 docker，ollama等。

### 1. 查看 xx 服务的最新日志（实时滚动）
```python
sudo journalctl -u docker -f
-f 参数表示 跟随（follow），会持续输出最新日志（类似 tail -f）。
```
### 2. 查看最近一天的日志
```python
sudo journalctl -u docker --since "1 day ago"
--since "1 day ago" 表示 从1天前到现在 的日志。
　　如果想指定更精确的时间范围，可以使用：
sudo journalctl -u docker --since "2025-03-01 00:00:00" --until "2025-03-02 12:00:00"
```
### 3. 仅查看最新的若干条日志（如最近 100 行）
```python
sudo journalctl -u docker -n 100
-n 100 表示显示最新的 100 条日志。
```
### 4. 按时间倒序排列（最新日志在前）
```python
sudo journalctl -u docker --reverse
--reverse 参数可以让日志按 从新到旧 显示（默认是旧→新）。
```
### 5. 结合时间范围和日志级别（如仅查看错误）
```python
sudo journalctl -u docker --since "1 day ago" -p err
-p err 表示仅显示 错误级别（ERROR） 的日志，还支持：
emerg (0), alert (1), crit (2), err (3)
warning (4), notice (5), info (6), debug (7)
```
### 6. 导出日志到文件
```python
sudo journalctl -u docker --since "1 day ago" > docker_logs.txt
```

---
