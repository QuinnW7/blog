+++
date = '2025-10-27T15:00:00+08:00'
draft = true
title = 'vLLM'
+++

<!--more-->


# vLLM

## what
一个开源库，为了提升显存利用率

## why

运行大模型的时候，最大的瓶颈之一是显存，特别是在处理用户请求时，模型需要为每个请求存储一个叫做KV cache的中间状态

- KV cache是什么？
    - 生成文本的时候，模型每生成一个词，需要回顾前面所有的词，为了避免重复计算，模型会将之前词语的key-value缓存起来

传统方法：
- 提前为每个请求预留一块足够大的连续显存空间，以存放可能生成的最大长度的文本


## how

核心技术：PagedAttention

虚拟内存+分片
将内存拆成多个固定大小的块(block)，按需分配，，逻辑连续物理分散

## 效果
- 高吞吐量：比其他主流服务框架（如 Hugging Face TGI）高出数倍甚至数十倍。
- 低延迟：高效的调度和计算内核保证了快速响应。
- 高效的显存管理：显著降低了部署大模型的硬件成本。
- 易于使用：提供了简洁的 Python API，几行代码就能启动一个高性能的 LLM 服务。

## 示例

```python
from vllm import LLM, SamplingParams

# 定义提示（prompts）
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]

# 定义采样参数
sampling_params = SamplingParams(temperature=0.8, top_p=0.95, max_tokens=50)

# 从Hugging Face Hub加载模型并创建LLM引擎
llm = LLM(model="meta-llama/Llama-2-7b-chat-hf")

# 生成文本
outputs = llm.generate(prompts, sampling_params)

# 打印结果
for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt!r}, Generated: {generated_text!r}")
```