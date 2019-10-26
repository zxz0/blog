---
layout: post
title:  "【转】跨域"
date:   2019-04-20 12:00:00 -0700
categories: Front-End
tags: Security Front-End
description: 跨域请求概念以及解决方案
---
### 总结自：
- [http://web.jobbole.com/95654/](http://web.jobbole.com/95654/)

同源策略：协议，域名(主域，子域/二级域名)，端口相同（仅仅通过“URL的首部”来识别而不会根据域名对应的IP地址是否相同来判断）。即使2个不同域名指向同一个ip地址，也非同源。  
- 限制内容：
	- Cookie、LocalStorage、IndexedDB等储存性内容
	- DOM节点
	- AJAX请求发送后，结果被浏览器拦截了
- 允许跨域加载资源的标签：
	```html
	<img src=XXX>
	<link href=XXX>
	<script src=XXX>
	```

跨域解决方案：
jsonp
