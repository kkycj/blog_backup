---
layout: post
title: "Ubuntu18.04安装并配置PaddlePaddle"
categories:
  - Linux
tags:
  - Ubuntu
  - PaddlePaddle
  - 环境配置
---
记录一次在Ubuntu18.04系统安装PaddlePaddle的过程经历。

# Ubuntu18.04 安装PaddlePaddle


## 环境配置

1.首先确认依赖项的安装，根据Paddle官方页面给出的信息，Ubuntu18.04只支持CUDA10.0/10.1（不支持CUDA10.2）

在NVIDIA开发者页面下载[CUDA10.1](https://developer.nvidia.com/cuda-toolkit-archive)

- 选择最新的CUDA10.1 update2

- 安装平台选择Linux，x86_64，Ubuntu，18.04，runfile

- 在终端输入以下指令

  ```
  wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
  sudo sh cuda_10.1.243_418.87.00_linux.run
  ```

  下载会花一些时间

- 等下载完后运行.run文件，打开安装界面

- 如果已经安装过NVIDIA driver并且版本在418.39以上（参考https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#abstract），可以不选择driver的安装，其他都勾选上

- 上一步中，如果没有安装driver，安装完成会提示没有安装CUDA toookit中的driver，但是没有关系，通过`nvcc -V`命令或者`nvidia-smi`命令都可以看到CUDA和driver已经安装好的版本号

2.下一步安装[cudnn]( https://developer.nvidia.com/rdp/cudnn-download)，PaddlePaddle要求CUDA10.1的cudnn对应为7.3+，这里选择安装最新的cudnn7.6.5，注意要CUDA10.1的版本

- 这里有两种选择方式，一种是旧版的tgz压缩包安装，另一种是新版的deb包安装

  - 选择[cuDNN Library for Linux](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.1_20191031/cudnn-10.1-linux-x64-v7.6.5.32.tgz)下载压缩包

	进入下载路径解压，并进入文件夹

	```
	tar -zxvf cudnn-10.1-linux-x64-v7.6.5.32.tgz
	```

    将下列文件拷贝到CUDA Toolkit的路径下

	```
	sudo cp cuda/include/cudnn.h /usr/local/cuda/include
	sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
	sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
	```

  - 或者选择[cuDNN Runtime Library for Ubuntu18.04 (Deb)](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.1_20191031/Ubuntu18_04-x64/libcudnn7_7.6.5.32-1%2Bcuda10.1_amd64.deb)下载deb包

	进入下载路径，安装deb包

	```
	sudo dpkg -i libcudnn7_7.6.5.32-1+cuda10.1_amd64.deb
	```

	验证cudnn是否安装成功

如果上一步用的是tgz安装，查看cudnn版本：

	```
	cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
	```

如果上一步用的是deb安装，查看cudnn版本：

	```
	dpkg -l | grep cudnn
	```


到此为止PaddlePaddle所需要的依赖项都安装完成，重启Linux使安装生效（否则安装完PaddlePaddle可能会提示没有GPU）

回到PaddlePaddle的官方页面选择安装方式，选用Ubuntu+pip+python3+CUDA 10

- 执行pip3安装PaddlePaddle

	```
	python3 -m pip install paddlepaddle-gpu==1.7.1.post107 -i https://mirror.baidu.com/pypi/simple
	```

	如果人在海外可以不加源地址，不指定paddlepaddle版本号，速度也很快

	```
	python3 -m pip install paddlepaddle-gpu
	```

- 安装完成后进入python3界面验证是否安装完成

	```
	>>> import paddle.fluid
	>>> paddle.fluid.install_check.run_check()
	```

    如果出现`Your Paddle Fluid is installed successfully!`，则安装成功

    如果在import时提示`Compiled with WITH_GPU, but no GPU found in runtime.`，很有可能是安装完CUDA和cudnn后没有重启，重启过后即可成功验证

## PaddleSeg配置实践

### 安装下载

- PaddlePaddle安装 https://www.paddlepaddle.org.cn/install/quick

- PaddleSeg库文件 https://github.com/PaddlePaddle/PaddleSeg

	```
	git clone https://github.com/PaddlePaddle/PaddleSeg
	```

	用百度的AI Studio克隆仓库速度比较慢，本地克隆很快

- PaddleSeg依赖

	通过以下命令安装python包依赖，因为本地环境同时存在python2和3两个版本，之前安装PaddlePaddle是通过python3安装，所以这里用pip3安装依赖（基本在PaddlePaddle安装的时候已经装过了）

	```
	cd PaddleSeg
	pip3 install -r requirements.txt
	```

	※ 这里安装的时候PyYAML出现如下提示

	```
	ERROR: Cannot uninstall 'PyYAML'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall
	```

	解决方案：

	```
	sudo -H pip3 install --ignore-installed PyYAML
	```


#### 测试运行

- 这里直接使用PaddleSeg提供的预训练模型进行测试
  https://github.com/PaddlePaddle/PaddleSeg/blob/release/v0.4.0/docs/model_zoo.md

  ```
  cd PaddleSeg/prertrained_model
  python3 download_model.py fast_scnn_cityscapes
  ```

  fast_scnn_cityscapes可以换成其他模型，在model zoo中进行选择

- 下载cityscapes数据集

  ```
  cd PaddleSeg/dataset
  python3 download_cityscapes.py
  ```

- 导出预训练模型

  ```
  cd PaddleSeg/pdseg
  python3 export_model.py --cfg configs/cityscape_fast_scnn.yaml TEST.TEST_MODEL pretrained_model/fast_scnn_cityscapes
  ```

- Python预测部署
  https://github.com/PaddlePaddle/PaddleSeg/tree/release/v0.4.0/deploy/python

  在终端输入以下命令：

  ```
  cd PaddleSeg
  python3 deploy/python/infer.py --conf=freeze_model/deploy.yaml --input_dir=/path/to/dataset --use_pr=False
  ```

  最后一个参数是是否使用优化，详见https://github.com/PaddlePaddle/PaddleSeg/tree/release/v0.4.0/deploy/python

- 模型只接受.jpg和.jpeg后缀的图片，数据集如果是.png格式需要将图片的后缀都改成.jpg
  在终端输入以下命令

  ```
  ls -1 *.png | xargs -n 1 bash -c 'convert "$0" "${0%.png}.jpg"'
  ```

  命令选项的说明：

  ```
  -1 -告诉 ls 每行列出一个图像名称的选项标识
  -n – 指定最多参数个数，例子中为 1
  -c – 指示 bash 运行给定的命令
  ${0%.png}.jpg – 设置新转换的图像文件的名字，% 符号用来删除源文件的扩展名
  ```

  （Ai Studio不能使用convert，不能转换）
