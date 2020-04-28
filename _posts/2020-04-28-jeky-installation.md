---
layout: post
title: "jekyll安装以及环境配置"
categories:
  - Linux
tags:
  - Ubuntu
  - Windows Subsystem for Linux(WSL)
  - jekyll
  - 环境配置
  - GitHub
---
第一次搭建个人网页用的GitHub Pages + jekyll的框架，平时在家还是用windows系统多一些，于是打算用Win10的Ubuntu18.04子系统来作为管理个人网页的系统平台。本篇简单记录了从现有的GitHub仓库克隆到本地WSL目录并初始化的一些内容。

### jekyll 安装以及环境配置

---

全文参考jekyll[官方文档](https://jekyllrb.com/docs/)进行安装配置

#### 步骤

1. 安装ruby，[参考](https://linuxize.com/post/how-to-install-ruby-on-ubuntu-18-04/)

    ```
    sudo apt update
    sudo apt install ruby-full build-essential zlib1g-dev
    ```

    为gem添加用户的路径配置

    ```
    echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    ```

2. 安装jekyll和bundler gems

    ```
    gem install jekyll bundler
    ```

3. 从Github克隆你的Github Page个人主页仓库

    ```
    git clone https://github.com/kkycj/kkycj.github.io.git
    ```

4. 进入仓库目录，安装bundler组件

    ```
    bundler install
    ```

5. 通过bundler编译网页，使网页能在本地服务器浏览（http://localhost:4000）

    ```
    bundler exec jekyll serve
    ```

    也可以在浏览器输入127.0.0.1:4000浏览页面