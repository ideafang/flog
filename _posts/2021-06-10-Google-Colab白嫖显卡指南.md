---
layout: post
comments: false
title: "Google Colab白嫖显卡指南"
date: 2021-6-10
tags: study-log
---

>实验室现在只有一个3090，但跑深度学习的人有很多，摊下来每个人能调用的算力其实很小，希望老师能再加几块......

<!--more-->

Google Colab 是Google推出的一个类jupyter notebook的项目，可以为有深度学习需求的用户提供免费和付费的算力使用。Colab提供GPU和TPU可供选择，并且默认后台已经安装配置好pytorch以及tensorflow环境，可以直接导入项目使用。

Colab最神奇的一点在于，他为你分配的虚拟机的磁盘和GPU并不在同一个真实的机器上，也就是说，磁盘和GPU之间的通信并不是走PCIE总线，而是通过网络传输的。但磁盘和CPU之间是否在同一个现实机器上我还不清楚。

但免费的Colab使用GPU的最长时间是12小时，而且如果长时间不操作Colab界面的话，系统会判定你浪费资源自动断掉GPU的连接，就算后台在跑程序也不行（很离谱）。

Colab会根据全网目前GPU的使用情况进行分配，目前有Tesla T4和Tesla K80两种，我个人的情况是，上午一般都能分到T4，下午晚些时候到晚上都只能分到K80了，可能我这的晚上就是西半球那边白天，用的人比较多。

[Colab官网地址](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjCwrrclY3xAhXLa94KHdHBAFYQFjAAegQIBhAD&url=https%3A%2F%2Fresearch.google.com%2Fcolaboratory%2F&usg=AOvVaw38J01zt_Dlb6pQ1fe6FGrI)

# 使用方法

首先需要将待跑的项目在本地调试好，上传到Google Drive中。Colab可以将Google Drive映射为一个磁盘来使用。但由于上述通过网络传输数据给GPU会耗费大量时间，训练效果反而不如直接用CPU跑，因此后续我会将项目复制到GPU本地机器上。

首先是真·抽卡环节，看一下分到的是什么GPU
```shell
!nvidia-smi
```
然后挂载Google Drive，运行时会弹出网址进行登录验证
```shell
from google.colab import drive
drive.mount('/content/gdrive')
!ls
```
接着将项目拷贝到GPU本地机器上，避免因为网络问题导致效率很差。Colab左边的目录默认显示的是`/content/`下的文件，所以最开始需要添加content
```shell
!cp -r /content/gdrive/MyDrive/[program-name] ./
```
最后进入项目，硬train一发!
```shell
!cd [program-name] && python train.py
```
这里使用`&&`连接两个shell语句，可以在调用python文件的同时，确保项目中相对路径的语句运行正确，也就是说，此时运行python程序的当前路径就是`/content/[program-name]`。如果使用 `python /content/[program-name]/train.py`的话，python程序的当前路径就是`/`，导入本地包就会报错`No such filexxxxx`之类的。

# 小技巧
Colab页面如果长时间不操作会自动断开GPU连接导致当前程序直接remake，为了避免这种情况，可以在浏览器控制台添加一个定时点击的小脚本。

`Ctrl+Shift+i`打开控制台，输入脚本

```javascript
function ConnectButton(){
    console.log("Connect pushed"); 
    document.querySelector("#top-toolbar > colab-connect-button").shadowRoot.querySelector("#connect").click() 
}
setInterval(ConnectButton,60000);
```
执行后记下返回的`intervalId`,
如果想停止脚本的话，可以在控制台输入
```javascript
clearInterval([intervalId])
```