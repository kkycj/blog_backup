---
layout: post
title: "Screen命令使用说明"
categories:
  - Linux
tags:
  - Ubuntu
  - 命令
  - screen
---
疫情自肃期间学校实验室不能进，通过ssh远程登陆实验室主机做PaddlePaddle的实验。为了能让PaddlePaddle的进程一直保持运行，需要用到screen命令来保持终端会话不会在推出ssh之后强制中断。P.S. 架设MC服务器的时候也经常会用到，让MC的服务器程序一直在VPS上保持运行。

### Screen命令使用说明
---

- Screen –S name 
  - 创建新会话 
  - 在会话中输入命令，启动程序
  - Ctrl +a ,d返回主窗口 

- Screen –r name 
  - 继续会话 

- Screen 中exit 
  - 退出会话 

- Screen –ls 
  - 查看当前存在会话 

- Screen –d 
  - 将指定窗口离线 
  - 如果显示matching xxx，用这个命令退出窗口 