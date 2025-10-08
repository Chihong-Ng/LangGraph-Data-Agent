# Data Agent 智能数据分析助手

基于LangGraph的开发生态，创建的一个同时具备数据库查询和Python代码解释器功能的智能数据分析助手。

## 项目简介

Data Agent 是一个智能数据分析平台，它能够根据用户的自然语言提问，自动进行数据库查询、数据处理和可视化分析。该项目结合了大语言模型的推理能力与数据处理工具，为用户提供直观、高效的数据洞察体验。

## 功能特性

- **自然语言交互**：通过对话方式提问，无需编写SQL或Python代码
- **数据库查询**：自动将自然语言转换为SQL查询语句并执行
- **数据可视化**：自动生成图表展示分析结果
- **代码执行**：支持Python代码执行，进行复杂的数据处理
- **会话管理**：支持多轮对话，保持上下文连贯性
- **数据提取**：从查询结果中提取关键信息
- **搜索工具**：集成搜索引擎，补充外部知识

## 技术栈

### 后端
- Python 3.11+
- LangGraph
- LangChain
- FastAPI
- MySQL
- Pandas
- Matplotlib/Seaborn

### 前端
- Vue 3
- TypeScript
- Vite

## 安装指南

### 1. 克隆项目

```bash
git clone https://github.com/Chihong-Ng/data-agent.git
cd data-agent
```

### 2. 设置Python虚拟环境

```bash
# 创建虚拟环境
python -m venv ace_env

# 激活虚拟环境
# Linux/MacOS
source ace_env/bin/activate
# Windows
# ace_env\Scripts\activate
```

### 3. 安装后端依赖

```bash
cd backend
pip install -r requirements.txt
```

### 4. 配置环境变量

在backend目录下创建`.env`文件，并填入以下内容：

```env
# 大模型API配置
DEEPSEEK_API_KEY=your_deepseek_api_key

# 数据库配置
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=telco_data

# Tavily搜索API配置（可选）
TAVILY_API_KEY=your_tavily_api_key
```

### 5. 安装前端依赖

```bash
cd ../frontend
npm install
```

## 使用说明

### 启动后端服务

```bash
cd ../backend
uvicorn main:app --reload
```

后端服务将运行在 http://localhost:8000

### 启动前端服务

```bash
cd ../frontend
npm run dev
```

前端服务将运行在 http://localhost:5173

### 会话管理

- 系统自动为每个用户创建会话ID，并存储在浏览器localStorage中
- 支持多轮对话，大模型可以记住上下文
- 点击页面顶部的清除按钮可重置对话历史

## 项目结构

```
data-agent/
├── backend/             # 后端代码
│   ├── main.py          # 主程序入口
│   ├── requirements.txt # Python依赖
│   ├── .env             # 环境变量配置
│   └── telco_data.csv   # 示例数据集
├── frontend/            # 前端Vue项目
│   ├── src/             # 前端源码
│   │   ├── App.vue      # 主应用组件
│   │   ├── main.ts      # 入口文件
│   │   └── style.css    # 样式文件
│   ├── package.json     # npm依赖配置
│   └── vite.config.ts   # Vite配置
├── ace_env/             # Python虚拟环境
├── INSTALL_GUIDE.md     # 安装指南
└── README.md            # 项目说明文档
```

## 开机自启动配置

项目包含了systemd服务文件模板，可用于配置系统开机自启动：

1. 后端服务配置：`backend/data-agent-backend.service`
2. 前端服务配置：`frontend/data-agent-frontend.service`
3. 前端启动脚本：`frontend/start_frontend.sh`

详细配置步骤请参考`INSTALL_GUIDE.md`文件。

## 数据说明

项目包含电信客户数据集（`telco_data.csv`），包含客户的基本信息、服务使用情况和账单信息等字段，用于演示数据分析功能。

## 注意事项

1. 请确保已正确配置环境变量，特别是API密钥和数据库连接信息
2. 首次运行可能需要安装额外的系统依赖
3. 如有端口冲突，请修改配置文件中的端口号
4. 服务启动后，请通过前端界面访问系统，不要直接调用后端API

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request来改进此项目。