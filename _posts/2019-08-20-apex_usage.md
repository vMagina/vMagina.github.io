---
layout: post
title:  "Apex混合精度加速"
date:   2019-08-20 16:35:00 
tags:   工具
---

## Apex混合精度加速
### Pytorch

<h3>简介：</h3>  

> Nvidia提供了一个混合精度工具apex，可以加速pytorch的训练效率（空间和时间上）。号称可以这不降低模型性能的情况下，将训练速度提升<b style='color:red'>2～4倍</b>，训练显存开销减少为原来的一半。开源地址如下：https://github.com/NVIDIA/apex，[论文在此](https://arxiv.org/pdf/1710.03740.pdf)。  
> 
> 目前该工具的版本为0.1版本，工具中主要有四个功能：`amp`，`parallel`，`optimizers`和`normalization`。  
> 1. 其中，第一个工具`amp`用于混合精度计算。pytorch中默认使用的是float32进行存储和数学计算，即单精度。而以float16作为计算类型的训练则称为半精度。半精度相比单精度而言，计算速度更快，空间开销更小，但是存在精度不够导致溢出的问题，单精度的运算稳定性则更好。因此，apex通过混合精度的方式来在保证训练精度的情况下减少空间开销，提高训练效率。  
> 2. `parallel`中的`DistributedDataParallel`是对pytorch官方分布式的包装复现，对NVIDIA的NCCL通信库进行了更好地优化。parallel中的SyncBatchNorm则是用来解决GPU间BN进行同步信息的作用（由于目前的框架上基本BN层的均值和方差都是通过singel GPU上的batch来计算的，无疑上缩小了mini-batch的大小。一般情况下，每个GPU上的batch的大小都是足够大的。但是，当受显存限制导致每个GPU上mini-batch较小或者特殊需求下，就需要BN层之间进行通信同步。GPU间大量的通信会导致训练速度的急剧下降，因此可通过该方法进行解决），同时还可以提升某些收敛精度。  
> 3. `optimizers`中目前仅提供了Adam的复现版本（讲道理还是SGD用的多，毕竟都怕Adam的过拟合啊┑(￣Д ￣)┍）。  
> 4. `normalization`则是BN层的复现版本，使用起来可能有坑。  

<h3>安装巨坑之路：</h3>

> 在安装apex的时候，本以为依据官方的Quick Start分分钟完成，然鹅遇坑无数。  

～～～～～～～～～～～～～～～～ _(:з」∠)_ ～～～～～～～～～～～～～～～～
> 为了获取最佳性能以及全部的功能体验，选择支持CUDA和C++扩展的方式进行安装：  
> `git clone https://github.com/NVIDIA/apex`  
> `cd apex`  
> `pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./`  
> 为了方便，本文采用了conda的方式，其中python=3.6，cuda=9.0，pytorch=1.0。  
> <strong style='color:orange'>注意</strong>⚠️  
> 在安装前，必须保证conda安装的pytorch的cuda版本要与机器上安装的cuda版本保持一致。同时，必须要gcc<6.0。  
> 安装时遇到了个大问题，折腾了半天：`ERROR: compiler_compat/ld: cannot find -lpthread -lc`，找了半天发现好像是Anaconda的问题，可以在`apex`文件夹下的`setup.py`文件中的`ext_modules`中添加如下即可：  
> `extra_link_args=['-L/usr/lib/x86_64-linux-gnu/']`  
> 耐心等待片刻，便可以享用apex啦。  
<h3>三行快速使用：</h3>

> 通过一下三行快速使用混合精度计算：  
> ```
> # Added after model and optimizer construction
> model, optimizer = amp.initialize(model, optimizer, flags...)
> ...
> # loss.backward() changed to:
> with amp.scale_loss(loss, optimizer) as scaled_loss:
>   scaled_loss.backward()
> ```
> 在使用`amp.initialize`前需要将model放在GPU上，通过`.cuda()`或者`to(device)`的方式。并且，在此之前不能调用任何数据并行函数。  
<p>

> `amp.initialize`中最关键的参数为`opt_level`，一共有四种设置方式：`O1`，`O2`，`O3`和`O4`。其中`O0`和`O3`分别是FP32和FP16的纯精度方式。这里引用官方的说法介绍`O1`和`O2`。
> `O1`("conservative mixed precision")patches Torch functions to cast inputs according to a whitelist-blacklist model. FP16-friendly (Tensor Core) ops like gemms and convolutions run in FP16, while ops that benefit from FP32, like batchnorm and softmax, run in FP32. Also, dynamic loss scaling is used by default.   
> `O2`("fast mixed precision")casts the model to FP16, keeps batchnorms in FP32, maintains master weights in FP32, and implements dynamic loss scaling by default. (Unlike --opt-level O1, --opt-level O2 does not patch Torch functions.)   
> 整个函数的参数虽然很多：  
> ``` 
> def initialize(
>     models,
>     optimizers=None,
>     enabled=True,
>     opt_level=None,
>     cast_model_type=None,
>     patch_torch_functions=None,
>     keep_batchnorm_fp32=None,
>     master_weights=None,
>     loss_scale=None,
>     cast_model_outputs=None,
>     num_losses=1,
>     verbosity=1,
>     ):
> ```
> 但是，其实基本设置`opt_level`即可，有些条件下会使其他参数无效。该参数的具体效果如下：  

| 设置 | O0 | O1 | O2 | O3|  
| :------: | :------: | :------: | :------: | :------: |
|cast_model_type|torch.float32|None|torch.float16|torch.float16|
|patch_torch_functions|False|True|False|False|
|keep_batchnorm_fp32|None|None|True|False|
|master_weights|False|None|True|False|
|loss_scale|1.0|"dynamic"|"dynamic"|1.0|
> 总体来说，使用了`O1`和`O2`方式，可以将显存开销真真实实地降低了接近一般，但是速度嘛好像没有变化。
> 当然，使用的时候你的GPU必须支持FP16计算方式，不然也没法加速。
<h3>Todo：</h3>

> 实验效果对比   
> 添加apex所有功能的使用方式及实验效果   
> ......