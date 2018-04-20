---
title: Ubuntu 下 python 使用 matplotlib 显示中文
date: 2018-04-20 15:20:15
link: 
categories:
- python
photos:
tags: 
- python
- matplotlib
---

Windows 下有中文字体，直接设置参数即可，Linux 需要自己添加中文字体。

开发环境：python 2

## 查看 matplotlib 是否有中文字体

在 python 命令提示符下输入，找到文件位置

```python
import matplotlib
matplotlib.matplotlib_fname()
# u'/usr/local/lib/python2.7/site-packages/matplotlib/mpl-data/matplotlibrc'
```

进入字体目录

```shell
cd /usr/local/lib/python2.7/site-packages/matplotlib/mpl-data
cd fonts/ttf/
```

查看是否有中文字体，如：`SimHei.ttf`

如果没有可以 [download](https://fontzone.net/font-download/simhei) 或从 windows (C:\Windows\Fonts) 中获取

## 在脚本中添加配置信息

```python
# 用 SimHei 字体显示中文
matplotlib.rcParams['font.sans-serif'] = ['SimHei']
# 这个用来正常显示负号
matplotlib.rcParams['axes.unicode_minus'] = False
```