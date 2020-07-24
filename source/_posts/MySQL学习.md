---
title: MySQL学习
date: 2020-07-24 16:26:00
tags:
categories:
- 数据库
- MySQL
---

**本文记录了学习MySQL的笔记，主要涉及MySQL本身特有的操作，SQL通用笔记不在本处记录**

## Windows下启动已经安装的MySQL

<!-- more -->

### 命令行启动、登录MySQL

+ 找到MySQL安装后对应的服务名称，一般设置为`MySQLXX`
+ CMD下输入`net start [MySQL服务名]`，可能需要管理员启动
+ 启动成功后进入mysql.exe所在目录，CMD输入`mysql -u [用户名] -p`，随后输入密码即登录了MySQL命令行。