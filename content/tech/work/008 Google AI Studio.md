+++
date = '2025-06-18T14:50:33+08:00'
draft = false
title = 'Google AI Studio'
+++

https://aistudio.google.com/prompts/new_chat

<!--more-->

# 背景
Google AI Studio 是一个基于浏览器的免费在线工具，旨在帮助开发者快速、轻松地进行生成式 AI 模型的原型设计和实验。你可以把它想象成一个“AI 模型的在线实验室”或“创意沙盒”。它的核心是让你能够直接与 Google 最先进的 Gemini 系列模型 进行交互。
- 降低门槛： 在 AI Studio 出现之前，要测试一个大型语言模型（LLM），开发者通常需要配置复杂的开发环境、管理 API 密钥、编写不少的样板代码。AI Studio 将这一切简化，用户只需打开浏览器就能开始。
- 推广 Gemini 生态： Google 希望吸引更多开发者使用其强大的 Gemini 模型。提供一个免费、易用的工具是展示模型能力、建立开发者生态的最佳方式。它直接对标 OpenAI 的 Playground。
- 加速原型验证： 许多 AI 应用的成功与否取决于“提示工程”（Prompt Engineering）。AI Studio 提供了一个理想的环境，让开发者可以快速迭代、测试和优化他们的提示（Prompt），从而验证一个想法是否可行，然后再投入实际的编码工作。
- 连接生产环境： AI Studio 不仅仅是一个玩具。当你在其中调试好一个满意的原型后，可以一键生成 Python、Node.js、Go 等多种语言的 API 调用代码，无缝地将原型集成到你的应用程序中。它也是通往更强大的企业级平台 Vertex AI 的桥梁。

# 用例图

```mermaid
 graph TD
    A["开发者 / 用户"] --> B(("设计与测试提示"))
    A --> C(("管理API密钥"))
    A --> D(("导出到生产环境"))
    subgraph core["核心系统: Google AI Studio"]
        B --> B1["创建自由格式提示"]
        B --> B2["创建结构化提示"]
        B --> B3["创建聊天提示"]
        B --> B4["运行提示并查看结果"]
        B --> B5["调整模型参数"]

        B1 -.-> B1_1["上传图片 - 多模态"]

        C --> C1["获取免费API密钥"]

        D --> D1["一键生成API代码"]
        D --> D2["迁移到Vertex AI"]
    end
```
# 静态建模
```mermaid
classDiagram
    direction LR

    class User {
      +interactsWith(AIStudioUI)
    }

    class AIStudioUI {
      -currentPrompt: Prompt
      -apiKey: string
      -apiClient: GenerativeAI_API_Client
      -codeGenerator: CodeGenerator
      +newPrompt(type: PromptType)
      +runPrompt()
      +displayResponse(response: ModelResponse)
      +displayError(error: Error)
      +generateCodeForPrompt(language: string)
    }

    class Prompt {
      <<abstract>>
      #id: string
      #modelParameters: ModelParameters
      +getFormattedContentForAPI(): object
    }

    class FreeformPrompt {
      -parts: ContentPart[]
      +addText(text: string)
      +addImage(imageData: blob)
      +getFormattedContentForAPI(): object
    }

    class StructuredPrompt {
      -examples: Example[]
      -finalInput: string
      +addExample(input: string, output: string)
      +setFinalInput(text: string)
      +getFormattedContentForAPI(): object
    }

    class ChatPrompt {
      -history: ChatMessage[]
      +addMessage(role: string, text: string)
      +getFormattedContentForAPI(): object
    }

    class ContentPart {
      <<abstract>>
    }
    class TextPart { +text: string }
    class ImagePart { +inlineData: object }

    class Example {
      +input: string
      +output: string
    }

    class ChatMessage {
      +role: string
      +parts: ContentPart[]
    }

    class ModelParameters {
      +temperature: float
      +topK: int
      +topP: float
    }

    class CodeGenerator {
      +generate(prompt: Prompt, language: string): string
    }

    class GenerativeAI_API_Client {
      -baseUrl: string
      +generateContent(payload: object, apiKey: string): Promise<ModelResponse>
    }

    class ModelResponse {
      +responseText: string
      +finishReason: string
      +error: object
    }

    User --> AIStudioUI

    AIStudioUI o-- "1" Prompt : "manages"
    AIStudioUI o-- "1" ModelParameters : "configures"
    AIStudioUI ..> GenerativeAI_API_Client : "uses"
    AIStudioUI ..> CodeGenerator : "uses"

    Prompt <|-- FreeformPrompt
    Prompt <|-- StructuredPrompt
    Prompt <|-- ChatPrompt

    Prompt o-- "1" ModelParameters

    FreeformPrompt *-- "0..*" ContentPart
    StructuredPrompt *-- "0..*" Example
    ChatPrompt *-- "0..*" ChatMessage

    ChatMessage o-- "1..*" ContentPart
    ContentPart <|-- TextPart
    ContentPart <|-- ImagePart

```

# 系统架构
# 时序图
# 状态机
# 对外API
AI Studio 本身不提供 API，但它使用的 Google Generative AI API (Gemini API) 是完全公开给开发者使用的。这正是 AI Studio "Get Code" 功能所生成代码的目标。
- 核心 Endpoint: generateContent
- HTTP Method: POST
- URL: https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent
{model}: 是一个占位符，例如 gemini-pro (文本) 或 gemini-pro-vision (文本和图片)。
Authentication: 通过在请求头中加入 x-goog-api-key: YOUR_API_KEY 或使用 OAuth 实现。

stateDiagram-v2
[*] --> Drafting: 新建或打开提示

    state Drafting {
        description: 用户正在编辑提示、调整参数
    }
    state Running {
        description: 请求已发送，等待后端响应
    }
    state DisplayingResult {
        description: 成功获取并显示结果
    }
    state Error {
        description: API调用失败或返回错误
    }

    Drafting --> Running: 用户点击 "Run"
    Running --> DisplayingResult: API 返回成功 (200 OK)
    Running --> Error: API 返回错误 (e.g., 4xx, 5xx)

    DisplayingResult --> Drafting: 用户修改提示或参数
    Error --> Drafting: 用户编辑提示以修复问题




