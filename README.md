# 基于魔搭平台的开源大语言模型本地部署与横向对比测试

### 项目简介

本项目在魔搭（ModelScope）平台提供的免费 CPU 云计算资源上，从零搭建了 Python 运行环境，并成功本地部署了三个主流的开源轻量级大语言模型：通义千问（Qwen-7B-Chat）、智谱（ChatGLM3-6B）以及百川（Baichuan2-7B-Chat）。通过同一套无显式对话模板的中文高难度语境测试题，对三个模型在指令遵循、多义词消歧、逻辑指代等维度进行了横向对比。

------

### 环境配置与依赖安装

由于新开实例环境较为干净，且默认未预装环境管理工具，本项目首先手动下载并配置了 Miniconda。为了提高在云端环境下的下载效率，所有 Python 依赖库均采用了阿里云国内镜像源（mirrors.aliyun.com）进行加速，避免因大文件下载超时导致的报错。

1. #### 下载并安装 Miniconda

   ```
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/conda
   echo 'export PATH="/opt/conda/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

2. #### 同意 Anaconda 条款并创建 Python 3.10 虚拟环境

   ```
   conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
   conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
   conda create -n qwen_env python=3.10 -y
   source /opt/conda/etc/profile.d/conda.sh
   conda activate qwen_env
   ```

3. #### 使用阿里云国内源加速，安装纯 CPU 版本的 PyTorch 核心计算库

   ```
   pip install \
   torch==2.3.0+cpu \
   torchvision==0.18.0+cpu \
   --index-url https://download.pytorch.org/whl/cpu \
   -i https://mirrors.aliyun.com/pypi/simple/
   ```

4. #### 配合阿里云国内源，安装核心基础依赖包

   ```
   pip install \
   "intel-extension-for-transformers==1.4.2" \
   "neural-compressor==2.5" \
   "transformers==4.33.3" \
   "modelscope==1.9.5" \
   "pydantic==1.10.13" \
   "sentencepiece" \
   "tiktoken" \
   "einops" \
   "transformers_stream_generator" \
   "uvicorn" \
   "fastapi" \
   "yacs" \
   "setuptools_scm" \
   -i https://mirrors.aliyun.com/pypi/simple/
   ```

5. #### 安装 fschat 并启用 PEP517 构建规范

   ```
   pip install fschat --use-pep517 -i https://mirrors.aliyun.com/pypi/simple/
   ```

   ------


### 模型获取方法

环境配置完成后，进入数据工作区 `/mnt/workspace` 目录。由于大模型权重文件通常有十几个G，为防止云端硬盘存储不足，本项目采取了下载一个、测试一个、清理空间、再换下一个的下载策略。

下载各模型所使用的 Git 命令如下：

```
cd /mnt/workspace

# 拉取通义千问模型
git clone https://www.modelscope.cn/qwen/Qwen-7B-Chat.git

# 拉取智谱 ChatGLM3 模型
git clone https://www.modelscope.cn/ZhipuAI/chatglm3-6b.git

# 拉取百川2模型
git clone https://www.modelscope.cn/baichuan-inc/Baichuan2-7B-Chat.git
```

------

### 测试脚本运行说明

本项目编写了通用的推理测试脚本 `run_qwen_cpu.py`。在切换不同的测试模型时，只需双击打开该脚本，修改 `model_name` 路径至对应模型的本地解压目录，并修改 `prompt` 变量为你想要测试的代码。

每次修改保存后，在已激活 `(qwen_env)` 环境的终端中运行以下命令即可看到模型的打字机输出结果：

```
python run_qwen_cpu.py
```

------

### 项目文件清单

- `run_qwen_cpu.py`：纯 CPU 环境下调用 Transformers 库加载模型并进行流式文本生成的 Python 核心推理脚本。

- `README.md`：本项目环境配置、模型拉取、运行指南及说明文档。

