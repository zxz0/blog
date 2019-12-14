---
layout: post
title:  "树莓派从零开始设置记录"
date:   2019-10-21 10:33:00 +0800
categories: Infrastructure
tags: ARM Computer DevOps
description: 想用（一时冲动入手的）树莓派做个服务器，本文是总结为此所做出的努力
---

### 已有知识：
- 学过中学物理（电流电路……）
- 玩过51单片机（只会用C点亮LED灯）
- 会C，Python和Bash
- 了解了一点SSH

### 过程：
在查资料的过程中，大概理解了，树莓派就是一个性能尚可的带了很多接口的ARM构架的微型主机。server，只是一个能发挥其部分作用的应用方向。好像更多的是当作智能硬件来用？

那么首先，需要操作系统：
- 在[官网](https://www.raspberrypi.org/)下载基于Debian的Linux操作系统[Raspbian](https://www.raspberrypi.org/downloads/raspbian/)
	- 尝试下载zip，但是速度太慢 -> 试着下载种子 -> 发现没有种子下载器 -> do research，Mac上多人推荐Folx -> 下载Folx本身速度很慢 -> 算了，直接从官网下载最精简版Raspbian Buster Lite的zip。
	- 检查checksum，官网给出的是SHA-256:a50237c2f718bd8d806b96df5b9d2174ce8b789eda1f03434ed2213bbca6c6ff，在终端运行指令:
	```sh
	  download=$(shasum -a 256 2019-09-26-raspbian-buster-lite.zip)
	  [ ${download%% *} = "a50237c2f718bd8d806b96df5b9d2174ce8b789eda1f03434ed2213bbca6c6ff" ] && echo 1 || echo 0 	# %% return content of the variable with the longest occurrence of substring deleted from the back of the variable. ' *' matched ' 0orMoreChars'
	```
	确定符合。
- 烧录系统：
	- 下载[balenaEtcher](https://www.balena.io/etcher/)（Windows上好像是用Win32DiskImager比较多）
	- 解压zip文件，得到img镜像
	- micro SD卡转接插入电脑，选择，烧录
		- 最后烧录成2个分区，FAT32的root分区（内核），以及EXT4分区（操作系统）（Window上会显示磁盘没有格式化：系统不认识；Mac压根不会显示）

	此处，发现其实SD卡内本来烧录好了full的系统，但是我直接写了，并没有先格式化，不知道会对后续有何影响

- 检查系统烧录情况：
	- 用micro HDMI连上支持HDMI口的显示屏
	- 连接电源，树莓派自动开机（暂时不连其他外设，以免开机失败）

确认系统正常工作

至此，树莓派就变成了一台Linux主机。

因为还买了外壳，所以接下来装外壳：
- 壳组转本身按照模糊的视频教程，以及商品照片来就可以，参考每个板的凸凹，以及合适程度确认
- 附带了散热片
	- 按照大小，分别装在CPU（风扇下方，方形的），内存芯片（中间，最长的），USB管理芯片（靠USB口，方的）上
- 附带了一个风扇，带红白线，到一个头上
	- 风扇电源红线（接正极）那边接在离板中心近的5v power的GRIO口上（GRIO口横排在上，上排的左边数起第2个口。物理引脚BOARD编码：4），黑线那边（接负极）自然地卡在旁边的地线GND（GRIO口横排在上，上排的左边数起第3个口。物理引脚BOARD编码：6）上，形成5V电压（电势差）
	- 风扇有logo的那边朝外，因为想从CPU抽热气（给的风扇logo面是凹，凸进凹出。可以想象凹的那面“舀”空气，把空气往外送/让空气变稀薄从而高压吹低压）（过后可以统计温度数据，最终决定装入风还是出风）

进行server所需的设置：
- SSH设置：（自2016年9月开始，raspberry默认关闭ssh连接）
	- 在micro SD卡的boot分区里新建一个名为“ssh”的空白文件
	- 或者，在机器上执行命令`sudo raspi-config`后，选择5 Interfacing Options - P2 SSH
	- 再或者，直接在机器上
	- 验证可以正常登陆：
		1. 用树莓派连上路由器的一个LAN口，电脑连同一个路由器的另一个LAN口，保证在同一子网下。
		2. 从路由器设置页面得知树莓派ip，或者树莓派接上显示屏，`ifconfig`，结合`networksetup -listallhardwareports`确定ip（eth0代表有线网络，wlan0代表无线网络）
		3. ssh pi@raspberry_pi_ip_address 看是否成功（密码：raspberry）
- 或者可以使用telnet，xrdp，VNC等方式
- Windows可能没有预装SSH，可以用PuTTY(注意，默认设置是禁用小键盘，在Terminal - Features中修改)或者安装OpenSSH进行连接

ok，已经验证可以SSH登陆了（`ssh username@ip`），接下来就是要真正接入网络了，为了顺便复习一下计算机网络，单独写在下一篇文章内。

验证完过后，进行一些SSH有关的安全设置：
- 添加SSH key（可以参照上一篇文章:[GitLab设置SSH key，以及相关知识]({{ site.baseurl}}/practice/2019/09/10/Gitlab-SSH-key.html)）<!-- notice the link might change! -->
- 可以改SSH的端口（默认22）

一些基本设置：
- 修改默认的密码：
	- root登陆，输入passwd，就可以修改
	- 修改别的用户的，passwd xx（用户名），按照提示即可修改
- 安装vim：自带的vi很不方便（插入模式）
	- `sudo apt-get install vim`

固定ip的话，可以直连树莓派端进行设置(/etc/dhcpcd.conf文件中)，也可以在路由器处设置然后连接树莓派。此处，在路由器设置界面把Mac和ip绑定了，做了端口转发。

使用Django快速建立项目进行测试：
- Raspberry自带Python2.7和Python3.4
- `find / -name pip*` 尝试查找pip，发现没有
- 安装pip：尝试：
	- ```sh
	curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
	python get-pip.py
	``` 但是无法安装pip3
	- `sudo apt-get install python3-pip` 成功安装pip3
- `pip install Django`
- 为了让django-admin可以使用，还要加入环境变量：`export PATH=$PATH:/home/pi/.local/bin`。这是暂时的，当下生效；还需要在系统开机文件中添加这个语句，才能保证之后每次开机之后PATH变量都带这个directory：`test`
- `django-admin startproject mysite` 创建项目
- `python manage.py runserver` 运行项目，默认端口8000
- 尝试从同一子网的机器直接access，发现不行，虽然ping和ssh都可以，猜测原因：服务器禁止访问
```sh
vim hello_world/settings.py
ALLOWED_HOSTS = ['192.168.1.10'] 	# add this
```
- 还是不可以，猜测是主机防火墙（iptables）设置问题？使用ufw开启端口，注意也要开启22的ssh端口（未验证）

一些问题：
- 速度有点慢？
	- 做research的时候，知道了是千兆网口，但是I/O比较弱，跟不上，因为是micro SD卡读写。SSD会快。
	- 内存/硬盘不够？所以买大内存，大micro SD卡（32G以上好像需要reformat以及分区）
	- 选择读写速度快，稳定的micro SD卡（最低class2，2MB/s；最高3，30MB/s。作为参考，SSD速度，是三位数……最高可以以G为单位）
	- 换了4G内存，32G SD卡，依旧比较慢
	- 继续research，比较其他server，发现延迟差不多，但是就是测速很慢
	- 是在某个节点，推测线路是瓶颈？
	- ipip.net，工具TraceRoute或者tracet 查看路由以及延迟
	- ping.pe或者mtr查看各站丢包情况
	- 国内网络情况复杂，不同的运营商之间慢，因为超售，国际慢（比如电信必须专用CN2线路才快，所以要找到的好的线路/机房）
	- BBR加速（谷歌的拥塞控制算法。Linux的4.9版内核已经使用该算法）

Linux上的一些命令的复习：
- grep (Global Regular Expression Print)：管道正则搜索。 `grep 'test' d*`
- wget：下载
- scp：拷贝。eg. `scp work@192.168.0.10:/home/work/source.txt work@192.168.0.11:/home/work/ 	#把192.168.0.10机器上的source.txt文件拷贝到192.168.0.11机器的/home/work目录下`

----
#### 参考资料：
- [树莓派的操作系统介绍](https://blog.csdn.net/huayucong/article/details/48107267)
- [Linux shell 命令多行结果赋值给变量](https://blog.csdn.net/hongweigg/article/details/77948664)
- [Linux字符串截取命令](https://www.cnblogs.com/fetty/p/4857158.html)
- [What Does the Percent Sign in Linux Shell Strings Do?](https://www.howtogeek.com/270333/what-does-the-percent-sign-in-linux-shell-strings-do/)
- [linux下的字符串比较](https://blog.csdn.net/shuai0845/article/details/86532142)
- [如何识别HDMI、[mini]HDMI、[micro]HDMI接口](http://www.jjlog.com/fenxiang/207.html)
- [树莓派散热片安装与风扇接线方法/ 和自动温控方法](http://www.raspigeek.com/index.php?c=read&id=126&page=1)
- [树莓派4的GPIO接口介绍](https://www.basemu.com/raspberry-pi-4-gpio-pinout.html)
- [树莓派GPIO入门](https://blog.csdn.net/coolwriter/article/details/75577092)
- [树莓派的GPIO编程](https://www.cnblogs.com/vamei/p/6751992.html)
- [机箱风扇方向 机箱风扇怎么安装最靠谱？](https://product.pconline.com.cn/itbk/diy/power/1707/9645570.html)
- [谈一谈CPU散热与机箱风扇](https://zhuanlan.zhihu.com/p/63818978)
- [三秒钟区分电脑风扇进风端和出风端](https://zhuanlan.zhihu.com/p/53743408)
- [树莓派3B+ 安装系统](https://blog.csdn.net/kxwinxp/article/details/78370913)
- [无屏幕和键盘配置树莓派WiFi和SSH](http://shumeipai.nxez.com/2017/09/13/raspberry-pi-network-configuration-before-boot.html)
- [树莓派新系统SSH连接被拒绝的解决方法](http://shumeipai.nxez.com/2017/02/27/raspbian-ssh-connection-refused.html)
- [使用Raspi-config配置工具来设置树莓派](https://blog.csdn.net/huayucong/article/details/52966502)
- [linux查看本机IP、gateway、dns](https://blog.csdn.net/zdwzzu2006/article/details/6928803)
- [mac的ifconfig命令](https://blog.csdn.net/m0_37556444/article/details/83351952)
- [树莓派查看ip地址（命令ifconfig）和退出ping](https://blog.csdn.net/naibozhuan3744/article/details/84963024)
- [树莓派远程连接的四种方式（最全）](https://blog.csdn.net/wuli_dear_wang/article/details/84446168)
- [修改linux用户密码（passwd）](https://blog.csdn.net/u011630575/article/details/49821281)
- [How to install pip with Python 3?](https://stackoverflow.com/questions/6587507/how-to-install-pip-with-python-3)
- [How to set your $PATH variable in Linux](https://opensource.com/article/17/6/set-path-linux)
- [树莓派搭建django](https://blog.csdn.net/Calvin_zhuyue/article/details/67677491)
- [iptables 配置实践](https://wsgzao.github.io/post/iptables/)
- [【树莓派】配置树莓派防火墙](https://www.cnblogs.com/haochuang/p/6214534.html)
- [Writing your first Django app](https://docs.djangoproject.com/en/2.2/intro/tutorial01/)
- [path.md](https://gist.github.com/nex3/c395b2f8fd4b02068be37c961301caa7)
- [linux grep命令](https://www.cnblogs.com/end/archive/2012/02/21/2360965.html)
- [SSH远程拷贝文件](https://blog.csdn.net/qq_31010431/article/details/54285664)
- [树莓派3b+从0开始：（2）SD卡的配置](https://www.jianshu.com/p/59df0009316e)
- [从Linux服务器下载文件夹到本地](https://blog.csdn.net/QianZhaoVic/article/details/79031359)
- [树莓派手动指定静态IP和DNS 终极解决大法](https://blog.csdn.net/u013178472/article/details/78574878)
- [外网访问树莓派](https://tech.itabas.com/2016/09/08/raspberry/access-raspberry/)
- [用ping ，mtr ，traceroute 进行网络丢包分析](https://blog.csdn.net/lqxandroid2012/article/details/79696318)
- [Micro SD 卡你真的会选么？](https://zhuanlan.zhihu.com/p/28580807)
- [树莓派之SSH连接经验](https://blog.csdn.net/xxNull/article/details/78280581)
- [树莓派常用软件及服务](http://wiki.jikexueyuan.com/project/raspberry-pi/software.html)
- [每天学习一个命令: mtr 查看路由网络连通性](http://einverne.github.io/post/2017/11/mtr-usage.html)
- [树莓派如何开启谷歌的BBR TCP加速](https://blog.csdn.net/lynxzong/article/details/89810538)
- [开启TCP BBR拥塞控制算法](https://www.liuboping.com/%E5%BC%80%E5%90%AFtcp-bbr%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95/)
- [[科普] 为什么你用的VPN网速那么慢？](https://fangeqiang.com/1378.html)
- [树莓派安装mysql并开启远程访问(开启3306端口)](https://blog.csdn.net/farYang/article/details/50788795)
- [同一局域网内的其他电脑访问我的电脑本地的网站](https://blog.csdn.net/lamp_yang_3533/article/details/52154695)