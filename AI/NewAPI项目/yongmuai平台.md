# 旧版本的bug

环境：Claude Code Router + yongmuai聚合平台令牌 + config.json配置

原始的config.json配置如下：
```json
{

  "PORT": 3456,
  "Providers": [
     {
      "name": "yongmuai",
      "api_base_url": "https://yongmuai.com/v1/chat/completions",
      "api_key": "sk-xxxxxxxx",
      "models": ["claude-sonnet-4-6", "claude-opus-4-6"]
    }

  ],

  "Router": {
    "default": "yongmuai,claude-sonnet-4-6",    
    "think": "yongmuai,claude-opus-4-6" ,      
    "background":"yongmuai,claude-sonnet-4-6"

  }

}
```

在claude code中运行命令时，报错如下：
```json
 API Error: 400 Error from provider(yongmuai,claude-opus-4-6: 400): {"error":{"message":"messages:
      text content blocks must be non-empty (request id:
     202605120316148196550128268d9d6laJBWqbI)","type":"upstream_error","param":"","code":null}}Error:
     Error from provider(yongmuai,claude-opus-4-6: 400): {"error":{"message":"messages: text content
     blocks must be non-empty (request id:
     202605120316148196550128268d9d6laJBWqbI)","type":"upstream_error","param":"","code":null}}
         at qn (C:\Users\Halcyon\AppData\Roaming\npm\node_modules\@musistudio\claude-code-router\dist\
     cli.js:582:7451)
         at fD (C:\Users\Halcyon\AppData\Roaming\npm\node_modules\@musistudio\claude-code-router\dist\
     cli.js:582:11338)
         at process.processTicksAndRejections (node:internal/process/task_queues:104:5)
         at async cN (C:\Users\Halcyon\AppData\Roaming\npm\node_modules\@musistudio\claude-code-router
     \dist\cli.js:582:8659)
```

## 错误分析
错误信息：`messages: text content blocks must be non-empty`

这是 Anthropic Claude API 的一个严格校验规则：消息中的 text content block 不能为空字符串。
这正好印证了"格式转换冲突"问题：

```json
Claude Code → CCR (转换) → NewAPI (再次转换) → 上游 Claude API
```

  双重转换链路中的问题：
  ```json
  1. CCR 将 Claude Code 的请求格式做一次转换
  2. NewAPI 收到后又做一次格式处理
  3. NewAPI 在 v0.12.11 的 changelog 中明确提到：
  ▎ "OpenAI 到 Claude 消息转换时空字符串内容处理增强"
  
  4. 这说明 NewAPI 曾经存在空字符串 content block 的 bug，v0.12.11 才修复
  ```

  可能的原因

```json
  
NewAPI 版本过旧         │ NewAPI 版本低于 v0.12.11，空字符串 bug 未修复 
CCR 产生了空 text block │ CCR 转换时生成了 {"type":"text","text":""}         
双重转换叠加            │ CCR 和 NewAPI 各自的格式转换互相干扰，产生空内容块 
```

修改后的config.json配置:
```json
{
  "PORT": 3456,
  "Providers": [
     {
      "name": "yongmuai",
      "api_base_url": "https://yongmuai.com/v1/chat/completions",
      "api_key": "sk-xxxxxxxx",
      "models": ["claude-sonnet-4-6", "claude-opus-4-6"],
      "trandsformer":{
        "use":["openai"]
      },
       "settings": {
        "stripEmptyContentBlocks": true,
        "filterEmptyText": true
      }
    }
  ],
  "Router": {
    "default": "yongmuai,claude-sonnet-4-6",    
    "think": "yongmuai,claude-opus-4-6" ,      
    "background":"yongmuai,claude-sonnet-4-6"
  }
}
```

