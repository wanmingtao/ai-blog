---
title: "本地部署大模型：从Ollama到vLLM的完全指南"
date: 2026-05-22T10:00:00+08:00
draft: false
tags: ["本地部署", "Ollama", "vLLM", "量化"]
categories: ["AI实践"]
summary: "不想依赖云端API？本文手把手教你如何在本地部署和运行大语言模型，从入门到生产级部署。"
cover:
  image: ""
  alt: "Local LLM"
  hidden: false
---

## 为什么选择本地部署？

- **隐私保护**：数据不离开本地
- **成本控制**：避免按token计费
- **离线使用**：无需网络连接
- **自定义**：完全控制模型行为

## 方案对比

| 方案 | 易用性 | 性能 | 适用场景 |
|------|--------|------|----------|
| Ollama | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 个人使用 |
| llama.cpp | ⭐⭐⭐ | ⭐⭐⭐⭐ | 开发测试 |
| vLLM | ⭐⭐ | ⭐⭐⭐⭐⭐ | 生产部署 |
| TGI | ⭐⭐⭐ | ⭐⭐⭐⭐ | 生产部署 |

## Ollama：最简单的方案

### 安装与使用

```bash
# 安装
curl -fsSL https://ollama.ai/install.sh | sh

# 运行模型
ollama run llama3:8b
ollama run qwen2:7b
ollama run deepseek-coder:6.7b

# API调用
curl http://localhost:11434/api/generate -d '{
  "model": "llama3:8b",
  "prompt": "解释量子计算",
  "stream": false
}'
```

### Python集成

```python
import requests

def chat_with_ollama(prompt, model="llama3:8b"):
    response = requests.post(
        "http://localhost:11434/api/chat",
        json={
            "model": model,
            "messages": [{"role": "user", "content": prompt}],
            "stream": False
        }
    )
    return response.json()["message"]["content"]
```

## llama.cpp：GGUF量化推理

### 量化模型

```bash
# 下载模型
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# 转换为GGUF格式
python convert_hf_to_gguf.py /path/to/model --outtype q4_K_M

# 运行推理
./main -m model.gguf -p "你好" -n 256
```

### Python绑定

```python
from llama_cpp import Llama

llm = Llama(
    model_path="./models/llama-3-8b-Q4_K_M.gguf",
    n_ctx=4096,
    n_gpu_layers=35  # GPU加速层数
)

output = llm("写一个快速排序算法", max_tokens=512)
print(output["choices"][0]["text"])
```

## vLLM：生产级部署

### 安装与启动

```bash
pip install vllm

# 启动API服务
python -m vllm.entrypoints.openai.api_server     --model meta-llama/Meta-Llama-3-8B-Instruct     --tensor-parallel-size 2     --max-model-len 8192     --gpu-memory-utilization 0.9
```

### 高级配置

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Meta-Llama-3-8B-Instruct",
    tensor_parallel_size=2,
    gpu_memory_utilization=0.9,
    max_num_batched_tokens=8192,
    max_num_seqs=256
)

sampling = SamplingParams(temperature=0.7, top_p=0.9, max_tokens=512)
outputs = llm.generate(["你好"], sampling)
```

## 性能优化

### 量化策略

- **GPTQ**：训练后量化，精度损失小
- **AWQ**：激活感知量化，效果更好
- **GGUF**：CPU友好，兼容性好

### 推理加速

```python
# vLLM的PagedAttention
# 自动管理KV Cache，显著提升吞吐量

# 批量推理
prompts = ["问题1", "问题2", "问题3"]
outputs = llm.generate(prompts, sampling)  # 自动批处理
```

## 硬件建议

| 模型大小 | 最低显存 | 推荐配置 |
|----------|----------|----------|
| 7B | 8GB | RTX 3080/4070 |
| 13B | 16GB | RTX 4080/A4000 |
| 70B | 48GB | A6000/A100 |

## 总结

本地部署大模型已经变得越来越简单。根据你的需求选择合适的方案，享受完全自主的AI体验。
