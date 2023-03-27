---
title: AI
date: 2023-03-27 15:47:00
tags:
    - STABLE-DIFFUSION
categories: STABLE-DIFFUSION
---

# 什么是**Stable Diffusion**

[`Stable Diffusion`](https://github.com/CompVis/stable-diffusion)是一个文本到图像的潜在扩散模型，由CompVis、Stability AI和LAION的研究人员和工程师创建。它使用来自LAION-5B数据库子集的512x512图像进行训练。使用这个模型，可以生成包括人脸在内的任何图像，因为有开源的预训练模型，所以我们也可以在自己的机器上运行它。

具体可以看[知乎科普文章](https://zhuanlan.zhihu.com/p/610094594)

## 系统配置

这边以windows为例子，因为一般windows安装n卡去跑，我这边是4090的显卡，这个教程默认认为你是懂一点程序的程序员，不是给你傻瓜式的安装那种，git这些操作你都要会，我不会详细说。

### 安装cuda

CUDA（Compute Unified Device Architecture），是显卡厂商NVIDIA推出的运算平台。 CUDA™是一种由NVIDIA推出的通用并行计算架构，该架构使GPU能够解决复杂的计算问题。 它包含了CUDA指令集架构（ISA）以及GPU内部的并行计算引擎。 开发人员可以使用C语言来为CUDA™架构编写程序，所编写出的程序可以在支持CUDA™的处理器上以超高性能运行。CUDA3.0已经开始支持C++和FORTRAN。

可以用cmd查看对应的版本下载。

```bash
# 查看cuda版本
nvidia-smi
# 例如我的输出如下
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 528.02       Driver Version: 528.02       CUDA Version: 12.0     |
```

去[navida官网](https://developer.nvidia.com/cuda-toolkit-archive)下载对应版本的

### 安装conda

[conda](https://www.quanxiaoha.com/conda/windows-install-conda.html)可以对你当前的操作系统进行多个不同环境的管理，使用教程可以看[这里](https://zhuanlan.zhihu.com/p/44398592)，需要配置一下清华大学的源，会快点

### 配置环境

如果git太慢，弄个翻墙设置一下代理

```bash
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
# 还原代码
set http_proxy=
set https_proxy=
```

`stable-diffusion`这个git只是一个训练模型，可以直接下载训练好的。执行命令设置相关的环境，

```bash
# 创建一个配置
conda create --name stable-diffusion-webui python=3.10.6
# 启用配置
conda activate stable-diffusion-webui
# 升级pip
python -m pip install --upgrade pip
# 设置国内的源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 安装stable-diffusion-webui

[git地址](https://github.com/AUTOMATIC1111/stable-diffusion-webui)

接下来介绍一下，`stable-diffusion-webui`这个项目，是基于`stable diffusion` 项目的可视化操作项目。通过可视化的网页操作，更方便调试`prompt`，及各种参数。同时也附加了很多功能，比如`img2img`功能，`extra`放大图片功能等等。

`stable-diffusion-webui` 的功能很多，主要有如下 2 个：

文生图（`text2img`）：根据提示词（`Prompt`）的描述生成相应的图片。
图生图（`img2img`）：将一张图片根据提示词（`Prompt`）描述的特点生成另一张新的图片。

#### 网络不好看这里

首次启动会自动下载一些 Python 依赖库等等（具体哪些库请看工程下的`requirements.txt`），以及项目需要用到的配置和模型文件（比如：`v1-5-pruned-emaonly.safetensors`，3.97G，会有点慢），建议提前下好放在项目的`models\Stable-diffusion`目录下。[下载地址](https://huggingface.co/runwayml/stable-diffusion-v1-5/tree/main)

下载``torch-1.13.1+cu117-cp310-cp310-win_amd64.whl``，[下载地址](https://download.pytorch.org/whl/cu117/torch-1.13.1%2Bcu117-cp310-cp310-win_amd64.whl)，安装命令
```
# 设置一下国内的源，这边是conda的环境对应的python路径
C:\tools\conda\envs\stable-diffusion-webui\Scripts\python.exe -m pip config set global.index-url https://pypi.douban.com/simple/
C:\tools\conda\envs\stable-diffusion-webui\Scripts\python.exe -m pip install --upgrade pip
C:\tools\conda\envs\stable-diffusion-webui\Scripts\python.exe -m pip install C:\Users\dinghuang\Desktop\torch-1.13.1+cu117-cp310-cp310-win_amd64.whl
```
[`GFPGAN`](https://github.com/TencentARC/GFPGAN)是腾讯旗下的一个开源项目，可以用于修复和绘制人脸，减少`stable diffusion`人脸的绘制扭曲变形问题。
```
# 安装gfpgan
C:\tools\conda\envs\stable-diffusion-webui\Scripts\python.exe -m pip install gfpgan
```

## 使用

修改webui-user.bat文件，如下：
```bash
@echo off

set PYTHON=
set GIT=
# 这边是设置虚拟路径，上面我们用conda设置了一个环境，这边就填写这个环境，python需要用到，可以cmd python查看conda activate 的python环境地址
set VENV_DIR=C:\tools\conda\envs\stable-diffusion-webui
set COMMANDLINE_ARGS=--xformers
call webui.bat
```

到`stable-diffusion-webui`根目录，执行命令

```bash
webui-user.bat
```

- 如果提示git拉取失败了，参照上面设置一下代理。
- 如果提示``ValueError: When localhost is not accessible, a shareable link must be created. Please set share=True.``，把代理还原掉

[更多启动参数](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings)



| 参数                              | 说明                                                                                           |
| :------------------------------ | :------------------------------------------------------------------------------------------- |
| –listen                         | 默认启动绑定的 ip 是`127.0.0.1`，只能是你自己电脑可以访问 webui，如果你想让同个局域网的人都可以访问的话，可以配置该参数（会自动绑定`0.0.0.0`ip）。 |
| –port xxx                       | 默认端口是`7860`，如果想换个端口，可以配置该参数，例如：`--port 8888`。                                               |
| –gradio-auth username\:password | 如果你希望给 webui 设置登录密码，可以配置该参数，例如：`--gradio-auth GitLqr:123456`。                                |
| –use-cpu                        | 默认使用 GPU 算力（需要 Nvidia 显卡），如果没显卡，可以配置该参数，改用 CPU 算力。                                           |
| –medvram                        | 为低显存（比如：4G）启用模型优化，会牺牲一点速度。                                                                   |
| –lowvram                        | 为极低显存（比如：2G）启用模型优化，会牺牲很多速度。                                                                  |
| –autolaunch                     | 启动时自动打开浏览器访问 webui。                                                                          |


启动成功后打开网站`http://127.0.0.1:7860/`，我这边随便写了一个`Prompt`：``a cute cat, cyberpunk art``，如图所示：

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20230313004839.png)

### 文生图

根据提示词`Prompt`的描述生成相应的图片。

在开始使用文生图之前，有必要了解以下几个参数的含义：

| 参数              | 说明                                                                      |
| :-------------- | :---------------------------------------------------------------------- |
| Prompt          | 提示词（正向）                                                                 |
| Negative prompt | 消极的提示词（反向）                                                              |
| Width & Height  | 要生成的图片尺寸。尺寸越大，越耗性能，耗时越久。                                                |
| CFG scale       | AI 对描述参数（Prompt）的倾向程度。值越小生成的图片越偏离你的描述，但越符合逻辑；值越大则生成的图片越符合你的描述，但可能不符合逻辑。 |
| Sampling method | 采样方法。有很多种，但只是采样算法上有差别，没有好坏之分，选用适合的即可。                                   |
| Sampling steps  | 采样步长。太小的话采样的随机性会很高，太大的话采样的效率会很低，拒绝概率高(可以理解为没有采样到,采样的结果被舍弃了)。            |
| Seed            | 随机数种子。生成每张图片时的随机种子，这个种子是用来作为确定扩散初始状态的基础。不懂的话，用随机的即可。                    |

### 模型使用

模型一般都是在[`civitai`](https://civitai.com/)上面下载，挑选合适的模型查看，前面那个文生图的例子，使用的就是这个 Deliberate 模型，直接点击 “Download Latest” 即可下载该模型文件。

> 模型文件有 2 种格式，分别是 `.ckpt（Model PickleTensor）` 和 `.safetensors（Model SafeTensor）`，据说 `.safetensors` 更安全，这两种格式 `stable-diffusion-webui` 都支持，随意下载一种即可。

将下载好的模型文件放到 `stable-diffusion-webui\models\Stable-diffusion` 目录下，放置好模型文件之后，需要重启一下 `stable-diffusion-webui`（执行 `webui-user.bat`）才能识别到。这些模型文件一般会附带一组效果图，点击任意一张，就可以看到生成该效果图的一些参数配置：把这些参数配置到 `stable-diffusion-webui` 中，点击 `Generate` 就可以生成类似效果的图片了。

### 插件使用
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/WechatIMG275.png)
如图所示，两种方式，第一种是通过默认给的git地址去加载，勾选install就好了，成功后点击重启应用即可，第二种是手动输入地址

#### controlNet

ControlNet 在 Adding Conditional Control to Text-to-Image Diffusion Models 一文中提被出，作者是 Lvmin Zhang 和 Maneesh Agrawala。它引入了一个框架，支持在扩散模型 (如 Stable Diffusion) 上附加额外的多种空间语义条件来控制生成过程。

训练 ControlNet 包括以下步骤:

克隆扩散模型的预训练参数 (文中称为 可训练副本, trainable copy。如 Stable Diffusion 的 latent UNet 部分)，同时保留原本的预训练参数 (文中称为 锁定副本, locked copy)。这样可以实现: a) 让锁定副本保留从大型数据集中学到的丰富知识；b) 让可训练副本学习特定任务的知识。
可训练副本和锁定副本的参数通过 “零卷积” 层 (详见 此处) 连接。“零卷积” 层是 ControlNet 框架的一部分，会在特定任务中优化参数。这是一种训练技巧，可以在新任务条件训练时保留已冻结模型已经学到的语义信息。


##### 安装
选择install from URL，输入地址 ``https://github.com/Mikubill/sd-webui-controlnet``，然后点击install，重启应用。接着下载相关的模型，到地址``https://huggingface.co/lllyasviel/ControlNet/tree/main/models``下载模型，然后把模型拷贝到目录``extensions/sd-webui-controlnet/models``，就可以使用了。


# 教程学习

[bilibili教程合集](https://www.bilibili.com/video/BV1Fv4y1e7Wm/?spm_id_from=333.999.0.0&vd_source=34926089f37a6eac61e104228717ddb1)
[controlNet使用](https://www.bilibili.com/video/BV1iT411r7xb/?spm_id_from=333.999.0.0&vd_source=34926089f37a6eac61e104228717ddb1)
[人物模型训练1](https://www.bilibili.com/video/BV1y84y1K7Rw/?spm_id_from=333.337.search-card.all.click&vd_source=34926089f37a6eac61e104228717ddb1)
[人物模型训练2](https://www.bilibili.com/video/BV1fs4y1x7p2/?spm_id_from=333.788.recommend_more_video.0&vd_source=34926089f37a6eac61e104228717ddb1)
[人物模型训练3](https://www.bilibili.com/video/BV1Kj411V78D/?spm_id_from=333.337.search-card.all.click&vd_source=34926089f37a6eac61e104228717ddb1)
[人物模型训练4](https://www.bilibili.com/video/BV16d4y1c7dh/?spm_id_from=333.337.search-card.all.click&vd_source=34926089f37a6eac61e104228717ddb1)
https://www.bilibili.com/video/BV1fD4y1C7nj/?spm_id_from=333.337.search-card.all.click&vd_source=34926089f37a6eac61e104228717ddb1

## prompt学习

[【AI绘画】全网 Stable Diffusion Prompt运用技巧（自用）](https://www.bilibili.com/read/cv19903784)

[prompthero](https://prompthero.com/stable-diffusion-prompts)

[手抄本](https://docs.google.com/spreadsheets/d/14Gg1kIGWdZGXyCC8AgYVT0lqI6IivLzZOdIT3QMWwVI/edit#gid=1772555973)

# 进阶使用

## API 接口服务
参数 `--api`，可以在启动 `stable-diffusion-webui` 的同时，启动一个接口服务，在 `COMMANDLINE_ARGS` 后面追加上 `--api`
```bash
@echo off

set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS=--listen --port 8888 --gradio-auth GitLqr:123456 --autolaunch --api

call webui.bat
```

重启后在 url 后面加上 /docs 即可看到 api 请求说明文档：这样我们就可以通过编写程序的方式，使用文生图、图生图等功能了，关于接口传参格式等要求，[官方说明文档]https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/API#api-guide-by-kilvoctu)。