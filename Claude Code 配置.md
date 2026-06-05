# Claude Code 配置

### 0. 安装 Node.js 和 Git

### 1. 安装 Claude Code

```
npm install -g @anthropic-ai/claude-code
```

​	检查

```
claude --version
```



### 2. 跳过地区检查

​	在  **C:\Users\xxx\\.claude.json** 中根节点下添加一行：

```
"hasCompletedOnboarding": true,
```



### 3. 配置第三方 API（以Deepseek为例）

​	添加  **C:\Users\xxx\\.claude\settings.json** 文件：

```
{
"env": {
        "ANTHROPIC_AUTH_TOKEN": "sk-xxx",
        "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
        "API_TIMEOUT_MS": "3000000",
        "ANTHROPIC_MODEL": "deepseek-v4-pro[1m]"
    }
}
```



### 4. 接入项目

​	进入项目目录下，运行 ：

```
claude
```

​	顺序选择：

```
 1. Yes, I trust this folder
```