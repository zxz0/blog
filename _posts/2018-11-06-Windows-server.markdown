---
layout: post
title:  "家用路由器下Windows系统主机做服务器"
date:   2018-11-06 06:08:10 +0800
categories: Ops
tags: Windows Server PC Router Ops
description: 配置通过路由器连接到ISP的PC作为服务器
---
1. 绑定内网ip（参考路由器指南）
2. 设置端口转发（参考路由器指南）
3. 目标主机上运行服务器（参考具体语言/框架/项目指南）
4. Windows防火墙设置（添加出入allow规则，注意和已有规则的冲突：如果一个服务被block且allow，最终会被block）
注：路由器默认设置为禁止ping