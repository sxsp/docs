# Codex 配置

### 1. VScode 下安装插件

在 VScode 中安装 **Codex – OpenAI’s coding agent** 插件

### 2. 修改配置文件

修改 **C:\Users\xxx\\.codex\config.toml** 文件：

```
disable_response_storage = true
model = "gpt-5.3-codex"
model_provider = "packycode"
model_reasoning_effort = "xhigh"
model_verbosity = "high"

[features]
web_search_request = true

[model_providers.packycode]
base_url = "https://www.packyapi.com/v1"
name = "packycode"
requires_openai_auth = true
wire_api = "responses"

[windows]
sandbox = "elevated"
```

修改 **C:\Users\xxx\\.codex\auth.json** 文件：

```
{
  "auth_mode": "apikey",
  "OPENAI_API_KEY": "sk-xxx"
}
```

