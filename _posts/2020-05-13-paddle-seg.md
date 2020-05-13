---
layout: post
title: "PaddleSeg脚本可选参数"
categories:
  - Linux
tags:
  - Ubuntu
  - PaddlePaddle
  - PaddleSeg
  - Deep Learning
---
参考PaddleSeg在GitHub的说明文档，做个记录。

# PaddleSeg脚本可选参数

参考 PaddleSeg [说明文档](https://github.com/PaddlePaddle/PaddleSeg/blob/release/v0.4.0/docs/config.md)

## 命令行FLAGS 

| **FLAG**     | **用途**                         | **支持脚本**   | **默认值** | **备注**                                                     |
| ------------ | -------------------------------- | -------------- | ---------- | ------------------------------------------------------------ |
| --cfg        | 配置文件路径                     | ALL            | None       |                                                              |
| --use_gpu    | 是否使用GPU进行训练              | train/eval/vis | False      |                                                              |
| --use_mpio   | 是否使用多进程进行IO处理         | train/eval     | False      | 打开该开关会占用一定量的CPU内存，但是可以提高训练速度。 **NOTE：** windows平台下不支持该功能, 建议使用自定义数据初次训练时不打开，打开会导致数据读取异常不可见。 |
| --use_tb     | 是否使用TensorBoard记录训练数据  | train          | False      |                                                              |
| --log_steps  | 训练日志的打印周期（单位为step） | train          | 10         |                                                              |
| --debug      | 是否打印debug信息                | train          | False      | IOU等指标涉及到混淆矩阵的计算，会降低训练速度                |
| --tb_log_dir | TensorBoard的日志路径            | train          | None       |                                                              |
| --do_eval    | 是否在保存模型时进行效果评估     | train          | False      |                                                              |
| --vis_dir    | 保存可视化图片的路径             | vis            | "visual"   |                                                              |

---

## 训练 

```
python3 pdseg/train.py --use_gpu --use_mpio --use_tb --tb_log_dir ./visual --do_eval --cfg ./configs/kitti_fscnn.yaml 
```



