---
layout: post
title: "Linux下执行python文件出现找不到自定义模块的问题"
date: 2023-02-24 
description: "Linux下执行python文件出现找不到自定义模块的问题"
tag: python
---   


若在一个python文件中引用了自定义的其它目录下的文件，在pycharm中可以顺利通过编译，而在Linux或命令行下会出现 ModuleNotFoundError: No module named ‘xxx‘ 的报错信息，其根本原因是python的**搜索包机制**。

python没有在IDE环境下时，是从要执行的.py文件开始向下搜索包，因此对于其它目录下的文件，在Linux或命令行环境下无法被搜索。可以使用os包来引入项目的根目录，使得python编译器从根目录往下搜索文件。

```python
import sys
import os
curPath = os.path.abspath(os.path.dirname(__file__))
rootPath = os.path.split(curPath)[0]
sys.path.append(rootPath)
```

其中`os.path.dirname(__file__)`获取当前文件所在目录的目录名，`os.path.abspath`获取当前文件的绝对路径，`os.path.split`将最后一个/进行拆分，将索引为0的视为目录（路径），将索引为1的视为文件名。

在文件头加入上述代码即可解决问题。
