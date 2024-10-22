# bitnet.cpp
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT) 
![version](https://img.shields.io/badge/version-1.0-blue) 

bitnet.cpp 是官方的 1 位大型语言模型（例如 BitNet b1.58）的推理框架。它提供了一套优化的核心，支持在 CPU 上进行 **快速** 和 **无损** 的 1.58 位模型推理（NPU 和 GPU 支持即将推出）。

bitnet.cpp 的首次发布是为了支持在 CPU 上进行推理。bitnet.cpp 在 ARM CPU 上实现了 **1.37x** 到 **5.07x** 的加速，更大的模型获得了更大的性能提升。此外，它还减少了 **55.4%** 到 **70.0%** 的能源消耗，进一步提高了整体效率。在 x86 CPU 上，加速范围从 **2.37x** 到 **6.17x**，能源减少在 **71.9%** 到 **82.2%** 之间。此外，bitnet.cpp 可以在单个 CPU 上运行一个 100B BitNet b1.58 模型，实现与人类阅读速度（每秒 5-7 个标记）相当的速率，显著增强了在本地设备上运行大型语言模型的潜力。更多细节将很快提供。

<img src="./assets/m2_performance.jpg" alt="m2_performance" width="800"/>
<img src="./assets/intel_performance.jpg" alt="m2_performance" width="800"/>

> 测试的模型是在研究环境中使用的虚拟设置，以展示 bitnet.cpp 的推理性能。

## 演示

bitnet.cpp 在 Apple M2 上运行 BitNet b1.58 3B 模型的演示：

https://github.com/user-attachments/assets/7f46b736-edec-4828-b809-4be780a3e5b1 

## 时间线

- 2024年10月17日 bitnet.cpp 1.0 发布。
- 2024年2月27日 [1位大型语言模型的时代：所有大型语言模型都在 1.58 位](https://arxiv.org/abs/2402.17764) 
- 2023年10月17日 [BitNet：为大型语言模型扩展 1位 Transformer](https://arxiv.org/abs/2310.11453) 

## 支持的模型
❗️**我们使用在 [Hugging Face](https://huggingface.co/) 上可用的现有 1 位大型语言模型来展示 bitnet.cpp 的推理能力。这些模型不是由微软训练或发布的。我们希望 bitnet.cpp 的发布将激发在大规模设置中就模型大小和训练标记方面发展 1 位大型语言模型。**

<table>
    </tr>
    <tr>
        <th rowspan="2">模型</th>
        <th rowspan="2">参数</th>
        <th rowspan="2">CPU</th>
        <th colspan="3">核心</th>
    </tr>
    <tr>
        <th>I2_S</th>
        <th>TL1</th>
        <th>TL2</th>
    </tr>
    <tr>
        <td rowspan="2"><a href="https://huggingface.co/1bitLLM/bitnet_b1_58-large">bitnet_b1_58-large</a></td> 
        <td rowspan="2">0.7B</td>
        <td>x86</td>
        <td>&#10004;</td>
        <td>&#10008;</td>
        <td>&#10004;</td>
    </tr>
    <tr>
        <td>ARM</td>
        <td>&#10004;</td>
        <td>&#10004;</td>
        <td>&#10008;</td>
    </tr>
    <tr>
        <td rowspan="2"><a href="https://huggingface.co/1bitLLM/bitnet_b1_58-3B">bitnet_b1_58-3B</a></td> 
        <td rowspan="2">3.3B</td>
        <td>x86</td>
        <td>&#10008;</td>
        <td>&#10008;</td>
        <td>&#10004;</td>
    </tr>
    <tr>
        <td>ARM</td>
        <td>&#10008;</td>
        <td>&#10004;</td>
        <td>&#10008;</td>
    </tr>
    <tr>
        <td rowspan="2"><a href="https://huggingface.co/HF1BitLLM/Llama3-8B-1.58-100B-tokens">Llama3-8B-1.58-100B-tokens</a></td> 
        <td rowspan="2">8.0B</td>
        <td>x86</td>
        <td>&#10004;</td>
        <td>&#10008;</td>
        <td>&#10004;</td>
    </tr>
    <tr>
        <td>ARM</td>
        <td>&#10004;</td>
        <td>&#10004;</td>
        <td>&#10008;</td>
    </tr>
</table>



## 安装

### 要求
- python>=3.9
- cmake>=3.22
- clang>=18
    - 对于 Windows 用户，请安装 [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/)。在安装程序中，至少切换以下选项（这也自动安装了所需的额外工具，如 CMake）：
        -  Desktop-development with C++
        -  C++-CMake Tools for Windows
        -  Git for Windows
        -  C++-Clang Compiler for Windows
        -  MS-Build Support for LLVM-Toolset (clang)
    - 对于 Debian/Ubuntu 用户，您可以使用 [Automatic installation script](https://apt.llvm.org/) 下载

        ` bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)&#34;` 
- conda （强烈推荐）

### 从源代码构建

> [!IMPORTANT]
> 如果您使用的是 Windows，请记住在执行以下命令时始终使用 Developer Command Prompt / PowerShell for VS2022

1. 克隆仓库
```bash
git clone --recursive https://github.com/microsoft/BitNet.git 
cd BitNet
```

2. 安装依赖项
```bash
# （推荐）创建一个新的 conda 环境
conda create -n bitnet-cpp python=3.9
conda activate bitnet-cpp

pip install -r requirements.txt
```

3. 构建项目
```bash
# 从 Hugging Face 下载模型，将其转换为量化的 gguf 格式，并构建项目
python setup_env.py --hf-repo HF1BitLLM/Llama3-8B-1.58-100B-tokens -q i2_s

# 或者您可以手动下载模型并使用本地路径运行
huggingface-cli download HF1BitLLM/Llama3-8B-1.58-100B-tokens --local-dir models/Llama3-8B-1.58-100B-tokens
python setup_env.py -md models/Llama3-8B-1.58-100B-tokens -q i2_s
```

```
usage: setup_env.py [-h] [--hf-repo {1bitLLM/bitnet_b1_58-large,1bitLLM/bitnet_b1_58-3B,HF1BitLLM/Llama3-8B-1.58-100B-tokens}] [--model-dir MODEL_DIR] [--log-dir LOG_DIR] [--quant-type {i2_s,tl1}] [--quant-embd]
                    [--use-pretuned]

Setup the environment for running inference

optional arguments:
  -h, --help            show this help message and exit
  --hf-repo {1bitLLM/bitnet_b1_58-large,1bitLLM/bitnet_b1_58-3B,HF1BitLLM/Llama3-8B-1.58-100B-tokens}, -hr {1bitLLM/bitnet_b1_58-large,1bitLLM/bitnet_b1_58-3B,HF1BitLLM/Llama3-8B-1.58-100B-tokens}
                        Model used for inference
  --model-dir MODEL_DIR, -md MODEL_DIR
                        Directory to save/load the model
  --log-dir LOG_DIR, -ld LOG_DIR
                        Directory to save the logging info
  --quant-type {i2_s,tl1}, -q {i2_s,tl1}
                        Quantization type
  --quant-embd          Quantize the embeddings to f16
  --use-pretuned, -p    Use the pretuned kernel parameters
```
</pre>

## 使用方法

### 基本使用

```bash
    # 使用量化模型运行推理
    python run_inference.py -m models/Llama3-8B-1.58-100B-tokens/ggml-model-i2_s.gguf -p "Daniel went back to the the the garden. Mary travelled to the kitchen. Sandra journeyed to the kitchen. Sandra went to the hallway. John went to the bedroom. Mary went back to the garden. Where is Mary?\nAnswer:" -n 6 -temp 0

    # 输出：
    # Daniel went back to the the the garden. Mary travelled to the kitchen. Sandra journeyed to the kitchen. Sandra went to the hallway. John went to the bedroom. Mary went back to the garden. Where is Mary?
    # 答案：Mary is in the garden.
```
</pre>

```
    usage: run_inference.py [-h] [-m MODEL] [-n N_PREDICT] -p PROMPT [-t THREADS] [-c CTX_SIZE] [-temp TEMPERATURE]
    
    Run inference
    
    optional arguments:
      -h, --help            show this help message and exit
      -m MODEL, --model MODEL
                            Path to model file
      -n N_PREDICT, --n-predict N_PREDICT
                            Number of tokens to predict when generating text
      -p PROMPT, --prompt PROMPT
                            Prompt to generate text from
      -t THREADS, --threads THREADS
                            Number of threads to use
      -c CTX_SIZE, --ctx-size CTX_SIZE
                            Size of the prompt context
      -temp TEMPERATURE, --temperature TEMPERATURE
                            Temperature, a hyperparameter that controls the randomness of the generated text
```
</pre>

### 基准测试
我们提供了运行推理基准测试的脚本，以提供模型。

```
    usage: e2e_benchmark.py -m MODEL [-n N_TOKEN] [-p N_PROMPT] [-t THREADS]  
        
    Setup the environment for running the inference  
      
    required arguments:  
      -m MODEL, --model MODEL  
                            Path to the model file. 
       
    optional arguments:  
      -h, --help  
                            Show this help message and exit. 
      -n N_TOKEN, --n-token N_TOKEN  
                            Number of generated tokens. 
      -p N_PROMPT, --n-prompt N_PROMPT  
                            Prompt to generate text from. 
      -t THREADS, --threads THREADS  
                            Number of threads to use. 
``` 
    
以下是每个参数的简要说明：  
    
- `-m`, `--model`: 模型文件的路径。这是一个在运行脚本时必须提供的必需参数。  
- `-n`, `--n-token`: 在推理期间生成的标记数量。这是一个可选参数，默认值为 128。  
- `-p`, `--n-prompt`: 用于生成文本的提示标记数量。这是一个可选参数，默认值为 512。  
- `-t`, `--threads`: 运行推理时使用的线程数量。这是一个可选参数，默认值为 2。  
- `-h`, `--help`: 显示帮助信息并退出。使用这个参数可以显示使用信息。  
    
例如：  
    
```sh  
python utils/e2e_benchmark.py -m /path/to/model -n 200 -p 256 -t 4  
```  
    
此命令将使用位于 `/path/to/model` 的模型运行推理基准测试，从 256 个标记的提示中生成 200 个标记，使用 4 个线程。  

对于任何公共模型不支持的模型布局，我们提供了脚本来生成具有给定模型布局的虚拟模型，并在您的机器上运行基准测试：

```bash
python utils/generate-dummy-bitnet-model.py models/bitnet_b1_58-large --outfile models/dummy-bitnet-125m.tl1.gguf --outtype tl1 --model-size 125M

# 使用生成的模型运行基准测试，使用 -m 指定模型路径，-p 指定处理的提示，-n 指定要生成的标记数量
python utils/e2e_benchmark.py -m models/dummy-bitnet-125m.tl1.gguf -p 512 -n 128
```

## 致谢

本项目基于 [llama.cpp](https://github.com/ggerganov/llama.cpp) 框架。我们要感谢所有作者对开源社区的贡献。我们还要感谢 [T-MAC](https://github.com/microsoft/T-MAC/) 团队对低位大型语言模型推理的 LUT 方法的有益讨论。
