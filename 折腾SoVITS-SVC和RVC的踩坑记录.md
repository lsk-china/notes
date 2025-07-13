# 折腾SoVITS-SVC和RVC的踩坑记录

Linux 环境但是用整合包....不过好像也有一些win可以参考的？

## 一. 环境搭建

1. 别用系统的pip,整个conda用！

2. docker是好文明！	

3. 项目基本上都是23/24年的老项目，里面的requirements没有版本号依赖是不准确的。有整合包的话，可以直接先提取下整合包里面附赠的环境的信息：

  ```bash
  wine workenv/python.exe -m pip freeze >> requirements_workenv.txt
  ```

 4. conda env的环境，python尽量老一点 或者和整合包的版本一致。python可以用3.8.10 / 3.9.10

    pip需要降级到24.0，否则个别软件包无法安装。

    ```bash
    conda create -p /path/to/ur/env python=3.8.9 
    conda activate /path/to/ur/env
    pip install pip==24.0
    ```

 5. torch版本和cuda版本，显卡匹配

    4090显卡可能出现：

    ```
    RuntimeError: cuFFT error: CUFFT_INTERNAL_ERROR
    ```

    租借的机器可以换个30xx的显卡。

    Reference: https://github.com/pytorch/pytorch/issues/88038

 6. 安装torch版本如果带有cuxxx后缀，需要加上pytorch的索引：

    ```bash
    pip install torch==xx torchaudio==xx torchvision==xx --extra-index-url https://download.pytorch.org/whl/cu117
    ```

    `--extra-index-url`表示额外的索引url,不可以用`-i`否则其他的包没法下载。

    torch官方镜像死慢的，国内机器可用`--proxy http://host:port`挂代理。

 7. 安装PyAudio报错找不到portaudio.h：

    用系统包管理/conda安装portaudio库 （debian系自行加dev包）

    用conda的话编辑`~/.pydistutils.cfg`:

    ```
    [build_ext]
    include_dirs=/path/to/ur/env/include/
    library_dirs=/path/to/ur/env/lib/
    ```

    https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI/来手动指定portaudio安装的路径。

    Reference: https://stackoverflow.com/questions/48690984/portaudio-h-no-such-file-or-directory

7. 直接`-r requirements.txt`装依赖可能引起的，由于软件包版本过新的问题：
   + gradio webapp无法启动
   + torch无法加载模型，有`... Since pytorch 2.6.0 ...`、`weights_only`等关键词的错误（忘记记录了 以后遇到在补上）
   + ... 

## 二. 数据准备

1. 数据集别太短，否则报错。

   ```
   ...Kernel size: (2). Kernel size can't be greater than actual input size
   ```

   Reference: https://github.com/facebookresearch/fairseq/issues/2953

   ffmpeg速查音频时长（但是好像没啥用？）

   ```bash
   for file in ./*; do ffprobe $file 2>&1 | grep Duration; done
   ```

   P.S. 论去正确项目翻issues的重要性……这玩意我翻一上午了属于是

2. 整合包自带的config、log、filelists里面的东西可以删一删

   svc注意目录结构：`raw_dataset/<singer_name>/*.wav`,否则预处理可能不认文件

## 三. 训练

1. 内存不大数据太多的话，别开缓存所有训练数据到内存。
2. 训练次数太多会过拟合，正确的做法是一边训练一边推理，直到找到最优模型。（感谢[这位up](https://space.bilibili.com/481226451)的经验）