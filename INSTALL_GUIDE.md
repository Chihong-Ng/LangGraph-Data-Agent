# 数据代理系统安装指南

本指南将帮助您设置数据代理系统的开机自启动功能。

## 系统要求

- Linux系统（支持systemd）
- Python 3.11或更高版本
- Node.js 16或更高版本

## 项目结构

- `/home/data-agent/backend/` - 后端Python项目
- `/home/data-agent/frontend/` - 前端Vue项目
- `/home/data-agent/ace_env/` - Python虚拟环境

## 开机自启动设置

### 1. 设置后端服务

1. 将后端服务文件复制到systemd目录：

```bash
sudo cp /home/data-agent/backend/data-agent-backend.service /etc/systemd/system/
```

2. 重新加载systemd配置：

```bash
sudo systemctl daemon-reload
```

3. 启用后端服务开机自启：

```bash
sudo systemctl enable data-agent-backend.service
```

4. 启动后端服务：

```bash
sudo systemctl start data-agent-backend.service
```

### 2. 设置前端服务

1. 确保前端启动脚本具有执行权限：

```bash
chmod +x /home/data-agent/frontend/start_frontend.sh
```

2. 将前端服务文件复制到systemd目录：

```bash
sudo cp /home/data-agent/frontend/data-agent-frontend.service /etc/systemd/system/
```

3. 重新加载systemd配置：

```bash
sudo systemctl daemon-reload
```

4. 启用前端服务开机自启：

```bash
sudo systemctl enable data-agent-frontend.service
```

5. 启动前端服务：

```bash
sudo systemctl start data-agent-frontend.service
```

## 检查服务状态

- 检查后端服务状态：
  ```bash
  sudo systemctl status data-agent-backend.service
  ```

- 检查前端服务状态：
  ```bash
  sudo systemctl status data-agent-frontend.service
  ```

## 查看服务日志

- 查看后端服务日志：
  ```bash
  sudo journalctl -u data-agent-backend.service
  ```

- 查看前端服务日志：
  ```bash
  sudo journalctl -u data-agent-frontend.service
  ```

## 手动启动服务（不使用systemd）

如果您不想使用systemd，也可以手动启动服务：

### 后端服务

```bash
cd /home/data-agent/backend
source /home/data-agent/ace_env/bin/activate
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

### 前端服务

```bash
cd /home/data-agent/frontend
source /home/data-agent/ace_env/bin/activate
npm install
npm run dev
```

## 访问系统

服务启动后，可以通过以下地址访问系统：

- 前端：http://服务器IP:5173/
- 后端API：http://服务器IP:8000/

## 故障排除

- 如果服务启动失败，请检查日志以获取详细错误信息
- 确保所有依赖都已正确安装
- 检查端口8000和5173是否已被其他进程占用
- 确保文件路径和权限设置正确