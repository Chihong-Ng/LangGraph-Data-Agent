# LangGraph Cli 构建智能数据分析助手
使用LangGraph的开发生态，来创建一个同时具备数据库查询和Python代码解释器功能的智能数据分析助手Data Agent。
## 使用方法
### 创建项目文件夹，复制graph.py和requirements.txt，安装依赖
```
pip install -r requirements.txt
```
### 编辑.env文件
填入相应的api key，并根据本地安装的MySQL进行设置
### 创建langgraph.json文件，用于LangGraph Cli读取配置
```josn
{
  "dependencies": ["./"],
  "graphs": {
    "data_agent": "./graph.py:graph"
  },
  "env": ".env"
}
```
这里将当前的Agent取名为data_agent
### 开启后端服务
配置并开启MySQL服务，然后在终端运行以下命令开启后端服务：
```bash
langgraph dev
```
### 开启前端服务
利用LangChain官方提供的agent-chant-ui开启前端服务（需要安装Node.js）：
```bash
git clone https://github.com/langchain-ai/agent-chat-ui.git
cd agent-chat-ui
pnpm install
pnpm dev
```
然后登录并进行测试，这里需要写清楚Agent名称：
<img width="1605" height="1174" alt="image" src="https://github.com/user-attachments/assets/1da0b4da-3a36-431d-ac76-273db9d52629" />

这样就可以进行对话了
<img width="2544" height="1360" alt="image" src="https://github.com/user-attachments/assets/a4807477-3073-43c1-baba-cc1e0a49b41e" />
<img width="2537" height="1361" alt="image" src="https://github.com/user-attachments/assets/a6104706-1627-45cd-bb74-e49f38a5e83b" />


### 
