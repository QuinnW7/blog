+++
date = '2025-12-03T10:00:00+08:00'
draft = true
title = '2025.12.03'
+++
<!--more-->

工作：
- codeact接入nexus
- 更新了python3.14版本
问题：
- 如何先将框架简单跑起来？
- 如何接入新agent?
  - 1、tools如何接入nexus?
    - 有哪些tool需要接入
  - 2、agents如何接入nexus?

- 如何自测？





Mcdocs 可以生成项目文档，看着还挺简单的

```python

# 安装pyenv


# 1. 安装 Python 3.14.1
pyenv install 3.14.1
# (注：如果这一步提示缺少依赖，说明第一阶段有遗漏，需补装后重新运行)

# 2. 设置为全局默认
pyenv global 3.14.1

# 3. 验证
python --version  
# 预期输出：Python 3.14.1

# 确认 pip 版本
pip --version  # 应指向 /usr/local/pyenv/...

# 安装 Jupyter
pip install jupyter

# 刷新可执行文件映射
pyenv rehash

[Service]
Type=simple
User=jupyteruser                # 保持原用户不变
WorkingDirectory=/home/jupyteruser

# 【新增】必须添加环境变量，否则找不到 3.14 的库
Environment="PATH=/usr/local/pyenv/versions/3.14.1/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

# 【修改】指向新的绝对路径
ExecStart=/usr/local/pyenv/versions/3.14.1/bin/jupyter notebook --no-browser --ip=0.0.0.0

Restart=on-failure
RestartSec=5

sudo systemctl daemon-reload
sudo systemctl restart jupyter
sudo systemctl status jupyter  # 检查是否为 active (running)
```