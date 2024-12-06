---
date: "2024-12-06"
title: "LLM_batch_reference"
---
[toc]

# Llama-3.1-70b的部署和批量推理（离线推理）

offline inferencem, or batch inference

### top_p vs top_k

> `top_p`是指在生成文本时，选择概率最高的 `p`个token作为候选集。例如，如果 `top_p`为0.9，则意味着在生成文本时，选择概率最高的90%的token作为候选集。
>
> `top_p`的作用是：
>
> * 限制生成的文本的多样性：通过选择概率最高的token，`top_p`可以限制生成的文本的多样性，使得生成的文本更加集中和可预测。
> * 提高生成的文本的质量：通过选择概率最高的token，`top_p`可以提高生成的文本的质量，使得生成的文本更加流畅和自然。
>
> `top_k`是指在生成文本时，选择前 `k`个概率最高的token作为候选集。例如，如果 `top_k`为100，则意味着在生成文本时，选择前100个概率最高的token作为候选集。
>
> `top_k`的作用是：
>
> * 提高生成的文本的多样性：通过选择前 `k`个概率最高的token，`top_k`可以提高生成的文本的多样性，使得生成的文本更加丰富和多样。
> * 降低生成的文本的质量：通过选择前 `k`个概率最高的token，`top_k`可以降低生成的文本的质量，使得生成的文本更加随机和不确定。
>
> `top_p`和 `top_k`是两个相关但不同的参数。`top_p`限制了生成的文本的多样性，而 `top_k`提高了生成的文本的多样性。通常情况下，`top_p`和 `top_k`会被同时使用，以便在生成的文本的质量和多样性之间找到一个平衡。
>
> 例如，如果你想生成一个高质量的文本，你可以设置 `top_p`为0.9和 `top_k`为100。这意味着在生成avour时，选择概率最高的90%的token作为候选集，并从候选集中选择前100个概率最高的token作为生成的文本。

## Huggingface model

```powershell
huggingface-cli login
huggingface-cli download -h
huggingface-cli download meta-llama/Llama-3.1-70B-Instruct
```

## Vllm

### CUDA和Pytorch

> [显卡驱动CUDA 和 pytorch CUDA 之间的区别_cuda版本和torch.cuda一样吗-CSDN博客](https://blog.csdn.net/null_one/article/details/129412159)

区别了nvcc nvidia-smi torch.\_\_version\_\_

### conda和pip

> [conda install和pip install区别 - lmqljt - 博客园](https://www.cnblogs.com/Li-JT/p/14024034.html)

pip 包含build conda一般是可执行

1. 安装环境 此处安装的是老版vllm 支持cuda11.8

```powershell
# conda 虚拟环境
conda create myenv

# 先安装pytorch
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

# Install vLLM with CUDA 11.8.
export VLLM_VERSION=0.6.1.post1
export PYTHON_VERSION=310
pip install https://github.com/vllm-project/vllm/releases/download/v${VLLM_VERSION}/vllm-${VLLM_VERSION}+cu118-cp${PYTHON_VERSION}-cp${PYTHON_VERSION}-manylinux1_x86_64.whl --extra-index-url https://download.pytorch.org/whl/cu11
```

2. 代码
   注意这里的要指定 `CUDA_VISIBLE_DEVICES`， 而且在代码内支持着一种方式

> [【杂记】vLLM如何指定GPU单卡离线推理_vllm指定gpu-CSDN博客](https://blog.csdn.net/m0_65814643/article/details/143882882)

```python
from vllm import LLM, SamplingParams
import os
os.environ["CUDA_DEVICE_ORDER"] = "PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"] = "2"

# Sample prompts.
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]
# Create a sampling params object.
sampling_params = SamplingParams(temperature=0.8, top_p=0.95)

# Create an LLM.
llm = LLM(model="facebook/opt-125m")
# Generate texts from the prompts. The output is a list of RequestOutput objects
# that contain the prompt, generated text, and other information.
outputs = llm.generate(prompts, sampling_params)
# Print the outputs.
for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
```

## Llama factory

webui board
