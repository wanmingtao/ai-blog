---
title: "大模型微调实战：从LoRA到全参数微调"
date: 2026-05-30T10:00:00+08:00
draft: false
tags: ["微调", "LoRA", "Fine-tuning", "训练"]
categories: ["AI实践"]
summary: "微调是让通用大模型适应特定任务的关键技术。本文详解LoRA、QLoRA等主流微调方法的原理与实践。"
cover:
  image: ""
  alt: "Fine-tuning Guide"
  hidden: false
---

## 为什么要微调？

预训练大模型虽然能力强大，但在特定领域可能表现不佳。微调可以：

- 提升特定任务的准确率
- 让模型输出符合特定风格
- 注入领域专业知识
- 降低推理成本（更小的专用模型）

## 微调方法对比

### 全参数微调

更新模型所有参数，效果最好但成本最高。

```python
from transformers import AutoModelForCausalLM, TrainingArguments

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B")

training_args = TrainingArguments(
    output_dir="./output",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=8,
    learning_rate=2e-5,
    warmup_steps=100,
    logging_steps=10,
    save_steps=500,
    fp16=True,
)
```

### LoRA（低秩适应）

只训练少量新增参数，效率极高。

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=16,                    # 秩
    lora_alpha=32,           # 缩放因子
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable params: 4,194,304 || all params: 8,000,000,000 || trainable%: 0.05%
```

### QLoRA

在4bit量化基础上应用LoRA，进一步降低显存需求。

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3-8B",
    quantization_config=bnb_config,
    device_map="auto"
)
```

## 数据准备

高质量的训练数据是微调成功的关键。

```python
# 数据格式示例
training_data = [
    {
        "instruction": "将以下英文翻译成中文",
        "input": "Hello, how are you?",
        "output": "你好，你好吗？"
    },
    {
        "instruction": "总结以下文章的核心观点",
        "input": "人工智能正在改变...",
        "output": "本文讨论了AI的三大趋势..."
    }
]
```

### 数据质量检查清单

- [ ] 数据格式统一
- [ ] 无重复样本
- [ ] 标注质量验证
- [ ] 数据分布均衡
- [ ] 敏感信息过滤

## 训练最佳实践

1. **学习率**：通常1e-5到5e-5之间
2. **批次大小**：配合梯度累积，有效批次32-128
3. **训练轮次**：1-3轮，避免过拟合
4. **评估策略**：定期在验证集上评估
5. **早停机制**：监控loss变化，及时停止

## 评估与部署

```python
# 模型评估
from evaluate import load

bleu = load("bleu")
results = bleu.compute(predictions=preds, references=refs)
print(f"BLEU Score: {results['bleu']:.4f}")
```

## 总结

微调是将大模型落地应用的关键环节。选择合适的微调方法，准备高质量数据，遵循最佳实践，你就能训练出优秀的专用模型。
