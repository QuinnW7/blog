+++
date = '2025-12-02T10:00:00+08:00'
draft = false
title = 'PYTHON'
+++

<!--more-->


环境：
语法+结构+导入
解释型语言，所以需要安装解释器pip，包管理器

内置电池：python自带的标准库 import 第三方库

虚拟环境venv(模拟沙箱)
- 放在当前项目下
```bash
python3 -m venv venv
source venv 只针对当前shell 

python3 -m venv --system-site-packages venv #在虚拟环境中使用系统范围的包

deactiatve
```
pip包：
```bash
pip install -r requirements.txt
```


shebang:
#！/python路径：可以执行这个文件 ./a.py


静态分析工具：pylint/Mypy

测试框架：Pytest、nose2和unittest。还有一个很有前途的新项目叫作 ward
```python
class A:
    pass

def __init__(self,...):
    pass
```


项目结构和代码导入：
源代码应该写在一个独立的包中
项目+包组成
模块：一个py文件
包：包含一个或多个模块的目录，要包含_init_.py文件
在导入模块时，实际上是在运行它，从而执行模块中的import语句

































基本结构
数据和流程

高级概念
超越代码

