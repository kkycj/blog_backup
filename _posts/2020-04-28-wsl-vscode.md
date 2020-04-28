---
layout: post
title: "在WSL下用VS Code编辑个人网页项目并同步到GitHub"
categories:
  - Linux
tags:
  - Ubuntu
  - Windows Subsystem for Linux(WSL)
  - GitHub
  - VSCode
---

之前的文章中提到我们用Win10下的Ubuntu子系统作为管理个人网站的系统平台，但是由于WSL（Windows Subsystem for Linux）中默认的权限在码代码或者使用git的过程中会出现很多问题，频繁的更改权限也很麻烦。使用VS Code中的WSL插件，就可以很方便的解决这个问题，快乐地改代码。

### 用VS Code编辑个人网页内容并用git同步到GitHub

---

#### 安装VS Code

- 首先下载并安装Win10下的[VS Code](https://code.visualstudio.com/)

    并且打开VS Code，在扩展插件中安装Remote - WSL

    ![remote-wsl](/assets/images/2020-04-28-wsl-vscode/remote-wsl.png)

- 然后进入WSL系统，在终端输入

    ```
    code
    ```

    系统会自动下载并解压Linux版的VS Code

    解压完毕后，会从Win10的窗口打开VS Code，并在右下角显示WSL: Ubuntu-18.04

        ![wsl-corner](/assets/images/2020-04-28-wsl-vscode/wsl-corner.png)

    之后便可以直接访问子系统下的目录，进行操作了

####  通过git上传同步到GitHub仓库

- 首先在VS Code中安装GitHub扩展插件

    ![github-ext](/assets/images/2020-04-28-wsl-vscode/github-ext.png)

- 如果没有在git中修改过全局的用户信息，进入WSL的终端界面

    ```
    git config --global user.name your_username
    git config --global user.email your_email
    ```

- 修改完成后就可以将编写的文件同步到GitHub仓库了

    - 可以用`ctr+shift+c`的快捷键（应该是中文键盘的，日文键盘用`ctrl+shift+@`）通过VS Code的内置终端，用git命令上传同步

    - 也可以通过`ctrl+shift+p`调出VS Code的控制台，输入commit搜索，选择commit all命令

        ![vscode-commit](/assets/images/2020-04-28-wsl-vscode/vscode-commit.png)

    - 添加完commit后，左下角的同步按钮会显示有一条可上传的变化，点击后，在控制台输入GitHub的账号密码，即可将修改后的内容push到对应的分支上
        ![sync-corner](/assets/images/2020-04-28-wsl-vscode/sync-corner.png)

#### 注意

最好不要通过windows的C盘路径去找WSL所在的目录，然后通过直接用VS Code打开那个目录去改代码。这样的话，VS Code修改过的文件权限全都会变成Windows下用户的权限，WSL是不能访问的，用Win10下的git修改过WSL目录下的项目后，WSL终端的git命令也都没有权限去访问这个项目的git信息，很坑爹。我用了最笨的方法去解决这个问题，备份过项目的内容后，用sudo删除掉整个文件夹，把之前修改访问权限的所有文件都删除，恢复最初的权限状态，然后重新在WSL中克隆GitHub上的仓库，然后按照本文的方法配置即可。