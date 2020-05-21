---
layout: post
title: "Ubuntu18.04安装配置BiseNetv2-Tensorflow 踩坑记录"
categories:
  - Linux
tag:
  - Ubuntu
  - Ubuntu 18.04
  - semantic segmentation
  - 语义分割

---

尝试了一段时间PaddleSeg，被kitti和cityscapes的数据集格式搞到头晕，我又是DL的纯小白，整个配置文件都云里雾里。正好最近在相关的公众号上看到有新的实时语义分割模型，换个口味试试。

# Ubuntu18.04 安装配置BisNetv2-Tensorflow踩坑记录

[参考](https://github.com/MaybeShewill-CV/bisenetv2-tensorflow)：Unofficial tensorflow implementation of real-time scene image segmentation model "BiSeNet V2: Bilateral Network with Guided Aggregation for Real-time Semantic  Segmentation"

## 准备工作

Github推荐的测试环境：

- Ubuntu 16.04(x64)
- python 3.5
- cuda-9.0
- cudnn-7.0
- GTX-1070
- tensorflow-gpu 1.12.0

我自己这边的为了PaddleSeg配置的环境：

- Ubuntu 18.04(x64)
- python 3.6.9
- cuda-10.1
- cudnn-7.6.5
- GTX-1070

首先在这个库的requirements列表里tensorrt这个tensorflow的组件对依赖项的版本有比较苛刻的要求。requirements里要求7.0.0.11版本，参考TensorRT[官方文档](https://docs.nvidia.com/deeplearning/tensorrt/archives/tensorrt-700/tensorrt-install-guide/index.html)，需要以下依赖：

- 9.0,10.0,10.2的CUDA Toolkit
- TensorFlow 1.14.0（以上）

之前安装的CUDA10.1并不支持tensorrt，因此在使用pip安装时，会提示没有适合的版本可以安装。



## 更新CUDA和cudnn

因为PaddleSeg要求10.1版本（实测其实PaddleSeg更新后也支持10.2），为了保留CUDA10.1，安装多版本的CUDA。

1. 如何查看当前CUDA以及cudnn版本

    CUDA版本：

    ```
    cat /usr/local/cuda/version.txt
    或者
    nvcc --version
    ```

    cudnn版本：

    ```
    cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
    ```

    注意：使用`nvcc --version`后显示的CUDA版本和`nvidia-smi`显示的CUDA可能是不同的，是因为CUDA有runtime API和driver API两种API，`nvcc`显示的版本是runtime API，而`nvidia-smi`显示的是driver API。（[参考文章](https://blog.csdn.net/ljp1919/article/details/102640512)）

2. 先卸载CUDA10.1的NVIDIA driver。

    ```
    sudo apt-get --purge remove "*nvidia*"
    ```

    卸载后禁用nouveau驱动，参考[NVIDIA官方文档](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-nouveau-ubuntu)

    - 新建conf文件，`/etc/modprobe.d/blacklist-nouveau.conf`，内容如下：

        ```
        blacklist nouveau
        options nouveau modeset=0
        ```

    - 重新生成内核initramfs信息：

        ```
        sudo update-initramfs -u
        ```

    全部完成后，重启系统。此时系统是无显驱的状态，分辨率会变得很低。

    > 3. 安装CUDA 10.2的新驱动。
    >
    > ```
    > wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
    > sudo sh cuda_10.2.89_440.33.01_linux.run
    > ```
    >
    > 因为卸载了驱动，所以选项全选，驱动和CUDA Toolkit全都安装。
    >
    > 4. 安装cudnn
    >
    >     cudnn[文件下载](https://developer.nvidia.com/rdp/cudnn-download)，选择压缩包文件，解压到cuda文件夹下：
    >
    >     ```
    >     sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
    >     sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
    >     sudo chmod a+r /usr/local/cuda/include/cudnn.h
    >     sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
    >     ```
    >
    >     注：如果多版本共存的情况下，可以把各个版本相对应的cudnn解压到cuda-10.1或者10.2下，切换版本时，只需要更改根目录下.bashrc中的cuda路径设置即可。
    >
    >     

注：经实际验证，在Ubuntu18.04+CUDA10.2+TensorRT1.14.0(1.12.0同样)下，运行Github上的test程序会报错运行失败，因此安装按作者的测试环境，重新安装CUDA9.0，步骤同上。



## 安装TensorFlow

> 通过pip安装TensorFlow，因为TensorRT要求为7.0.0.11，此版本要求TensorFlow1.14.0，如果pip安装不指定版本，会自动安装最新版本2.x，并不兼容TensorRT，因此需要指定TensorFlow的版本。
>
> ```
> pip3 install tensorflow-gpu==1.14.0 (x)
> pip3 install tensorflow-gpu==1.12.0
> ```
>
> 检验TensorFlow是否安装成功
>
> ```
> python3
> >>> import tensorflow as tf
> >>> tf.__version__
> '1.14.0' (x)
> '1.12.0'
> ```
>
> 如果输出1.14.0则说明安装成功。



注：测试时由于CUDA版本的问题，换回CUDA9.0的同时也按作者的测试环境重新安装了1.12.0版本的TensorFlow。

## 安装TensorRT

注：经过多次尝试，pip的安装方法无论用什么版本都会提示找不到版本无法安装，在TensorRT的[官方文档](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing)里也没有pip直接安装的方法。文档中Debian和RPM的安装方法要求CUDA Toolkit和cudnn都用Debian或RPM的方法安装。因为之前CUDA和cudnn选择了多版本共存的安装方式，所以这里只能选择手动解压Tar格式文件安装。TensorRT 7 [下载地址](https://developer.nvidia.com/nvidia-tensorrt-7x-download)，选择Tar格式的安装包。

1. 解压压缩包

    ```
    tar -zxvf TensorRT-7.0.0.11.Ubuntu-18.04.x86_64-gnu.cuda-10.2.cudnn7.6.tar.gz
    ```

2. 添加TensorRT lib路径到环境变量，注意<>内改成自己对应的绝对路径

    ```
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:<TensorRT-7.0.0.11/lib>
    ```

3. 安装Python 3.6版本的TensorRT wheel文件

    ```
    sudo pip3 install tensorrt-7.0.0.11-cp36-none-linux_x86_64.whl
    ```

4. 安装Python3.6版本的UFF wheel文件

    ```
    sudo pip3 install uff-0.6.5-py2.py3-none-any.whl
    ```

5. 安装Python graphurgeon wheel文件

    ```
    sudo pip3 install graphsurgeon-0.4.1-py2.py3-none-any.whl
    ```

6. 参照[说明文档](https://github.com/NVIDIA/TensorRT/tree/master/samples/opensource/sampleMNIST)，运行sample检验是否安装成功（可选）

    ```
    cd TensorRT-7.0.0.11/sample/sampleMNIST
    make
    cd TensorRT-7.0.0.11/data/mnist
    python3 download_pgms.py
    cd TensorRT-7.0.0.11/bin
    ./sample_mnist
    ```

    如果输出手写数字的图像，则通过检验。



至此CUDA Toolkit，cudnn，TensorFlow和TensorRT都安装完毕，进入bisenetv2的目录，安装其他依赖项。

```
cd bisenetv2-tensorflow
sudo pip3 install -r requirements.txt
```

