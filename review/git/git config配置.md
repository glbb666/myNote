### 前言

写这个笔记是因为，我需要在本地配置多个账户，一个公司的，一个自己的。在配置的过程中，我发现要多次使用`git config`命令，所以我就去学习了一下。

> 查询配置信息

1. 列出当前配置：`git config --list`;
2. 列出repository配置：`git config --local --list`;
3. 列出全局配置：`git config --global --list`;
4. 列出系统配置：`git config --system --list`;

> 配置用户信息

1. 配置用户名：`git config user.name "your name"`;
2. 配置用户邮箱：`git config user.email "youremail@github.com"`;

也可以加上以下参数

```
--global 全局配置
--local 仓库配置
（什么都不加） 相当于配置当前信息用户信息，但是用户信息是基于仓库的，所以相当于--local
```

> 其他配置

1. 配置解决冲突时使用哪种差异分析工具，比如要使用vimdiff：`git config --global merge.tool vimdiff`;
2. 配置git命令输出为彩色的：`git config --global color.ui auto`;
3. 配置git使用的文本编辑器：`git config --global core.editor vi`;

