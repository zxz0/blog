---
layout: post
title:  "VPS初始设置"
date:   2019-12-14 09:30:00 -0800
categories: O&M
tags: DevOps VPS Security
description: 虚拟服务器必要的一些安全，性能有关的初始设置
---

因为打折，忍不住剁手 * 2。慢慢收拾残局。使用的CloudCone（KVM构架），操作系统CentOS 7.6（`cat /etc/redhat-release 	# CentOS Linux release 7.6.1810 (Core)`）。

1. 修改默认的SSH端口22
```sh
	vim /etc/ssh/ssh_config 	# for ssh
	vim /etc/ssh/sshd_config	# for ssh daemon
	# uncomment # Port 22, change 22 to the port you want. better [49152, 65539]
	service sshd restart 		# restart sshd. now can only login thru the specified port
```
通过
```sh
	ssh -p port_you_specified username@ip
```
测试登陆。
如果系统开启了iptables防火墙，还需要设置允许指定的端口。

2. 创建普通用户，给予sudo权限：
```sh
adduser username
passwd username
# need to repeat twice to verify
usermod -aG wheel username 	# in CentOS, members of wheel group have sudo privileges
# or gpasswd -a username wheel
# su - username to substitute user to check
```
通过切换到用户，sudo测试，如：
```sh
sudo ls -la /root
```
session第一次执行sudo需要输入当前用户（非root）密码

3. 限制普通用户的su切换
```sh
	vim /etc/pam.d/su 	
	# uncomment: #auth required pam_wheel.so use_uid, so that only member in wheel group can execute su
```
这样，不属于wheel组的成员账号切换（到root）时，系统会拒绝：即使输入了正确的root密码，也会提示密码不正确

4. 禁止root远程SSH登陆
```sh
vim /etc/ssh/sshd_config
# change PermitRootLogin yes -> PermitRootLogin no
service sshd restart
```
不影响已连接的SSH。之后需要，可以普通用户远程登录，之后su切换到root；或者sudo执行需要root权限的命令。
往后，我们都使用新建的，有sudo权限的用户进行远程登录，以及各种常规操作。

5. 生成SSH Key pair，UI界面添加公钥到服务器（详见上文：[GitLab设置SSH key，以及相关知识]({{ site.baseurl}}/practice/2019/09/10/Gitlab-SSH-key.html)）
	- 添加完毕后重新启动，还是不可以使用
	- 按照页面上的指示，SSH去机器，运行指令：
```sh
ssh root@173.82.151.128 	# connect via SSH. since we disabled root login, we need to login as normal user, then su root
curl -o cc-ikey -L web.cloudc.one/sh/key && sh cc-ikey some_string 	# install the ssh key
```
注意到在CloudCone即使有多个instance/vps，某个vps的UI界面也只有一个SSH Key。要继续添加，可以通过User / SSH Keys. \
仔细查看cc-ikey脚本，发现就是从cloudcone数据库用给定的id拿到UI界面中的public key，放到authorized_keys最后面；根据参数，确定是否关闭root远程登录。由于脚本必须要用root用户，所以无法配置别的用户的SSH key登录。此处使用直接copy到`~/.ssh/authorized_keys`的做法，手动添加SSH key到服务器。


6. disable密码登录
	- 编辑`/etc/ssh/sshd_config`文件，加入/置`PasswordAuthentication no`，保存
	- 重启ssh服务：
```sh
sudo service ssh restrat 	# Ubuntu or Debian
sudo service sshd restart 	# CentOS / Fedora
```

7. 使用/开启防火墙：
CentOS有叫做firewalld的防火墙，工具firewall-cmd可以帮助设置防火墙。基本原则：关闭所有不必要的端口。
```sh
sudo yum install 	# yum is package management tool. usage: yum install/remove/update/list/search/info/check-update [package_name]
sudo systemctl start filewalld 	# firewalld can make modification without dropping current connections
# start adjusting policies for the default zone. When we reload out firewall, this will be the zone applied to our interfaces. (NOT TRUE! current normal user connection will freeze and throw "packet_write_wait: Connection to ip_address port ssh_port: Broken pipe" somehow. I opened a root user session, not evicted somehow. maybe need to config this before change default ssh port? next time see if it crash when 2 normal user session are available)
systemctl status firewalld 	# see current firewall status
sudo firewall-cmd --permanent --add-service=ssh 	# add ssh service. --permanent will save the change to config file
sudo firewall-cmd --permanent --add-port=valid_port/tcp 	# if the default SSH port (22) changed
# can add other services here, eg. http, https (the corresponding default port will be automatically added)
sudo firewall-cmd --permanent --list-all 	# see config in file
sudo firewall-cmd --list-all 	# see current loaded config
sudo firewall-cmd --reload 	# reload with new config file, apply the changes
sudo firewall-cmd --query-port=61187/tcp 	# test the port is open
sudo systemctl enable firewalld 	# start firewall at boot
```

8. 设置时间同步
```sh
sudo timedatectl list-timezones 	# to list all. space and b to page down/up, q to exit
sudo timedatectl set-timezone region/timezone 	# set the right one with server
sudo timedatectl 	# confirm the updated timezone
sudo yum install ntp 	# Network Time Protocol (NTP) synchronization.
sudo systemctl start ntpd 	# start service for current session
sudo systemctl enable ntpd 	# start service at boot
```

9. 添加swap file：
当RAM不够的时候，Linux会把释放的空间临时保存到Swap空间中。虚拟内存。如果RAM不够大，或者不想应用crash，就用。尤其推荐db使用。
```sh
free -m 	# see how much we have for swap 
sudo fallocate -l 4G /swapfile 	# can set 1:1 to RAM
sudo chmod 600 /swapfile	# don't let other users to see the content
sudo mkswap /swapfile 	# tell the system to format the file for swap
sudo swapon /swapfile 	# tell the system it can use the swap file for current session
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab' 	# modify the system file to do it automatically at boot
```
----
#### 参考资料：
- [手把手教你怎么使用云服务器](https://mp.weixin.qq.com/s/MQqasjPs4Y-OCjQLuFj4ew)
- [How To Configure SSH Key-Based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
- [Downloading Java JDK on Linux via wget is shown license page instead](https://stackoverflow.com/questions/10268583/downloading-java-jdk-on-linux-via-wget-is-shown-license-page-instead)
- [Not able to install oracle jdk on CentOS machine using wget](https://stackoverflow.com/questions/44213454/not-able-to-install-oracle-jdk-on-centos-machine-using-wget)
- [How do I provide a username and password to wget?](https://askubuntu.com/questions/29079/how-do-i-provide-a-username-and-password-to-wget)
- [Linux下载及安装jdk1.8](https://blog.csdn.net/luck_zz/article/details/80348592)
- [The Bash Shell Startup Files](http://www.linuxfromscratch.org/blfs/view/svn/postlfs/profile.html)
- [What is the good way of setting JAVA_HOME system wide in Linux? /etc/profile or /etc/profile.d/custom.sh?](https://stackoverflow.com/questions/10117943/what-is-the-good-way-of-setting-java-home-system-wide-in-linux-etc-profile-or)
- [Java开发环境不再需要配置classpath！](https://juejin.im/post/5ce67fa1f265da1b6a346d16)
- [Linux 修改SSH端口 和 禁止Root远程登陆](https://blog.csdn.net/tianlesoftware/article/details/6201898)
- [网络安全系列之十三 Linux中su与sudo的安全设置](https://blog.51cto.com/yttitan/1568305)
- [How To Create a Sudo User on CentOS [Quickstart]](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-centos-quickstart)
- [Initial Server Setup with CentOS 7](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)
- [Additional Recommended Steps for New CentOS 7 Servers](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-centos-7-servers)
- [20 Linux YUM (Yellowdog Updater, Modified) Commands for Package Management](https://www.tecmint.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/)
- [怎样修改 CentOS 7 SSH 端口](https://sebastianblade.com/how-to-modify-ssh-port-in-centos7/)
- [An Introduction to Firewalld](https://www.liquidweb.com/kb/an-introduction-to-firewalld/)
- [Swap交换分区概念](https://www.cnblogs.com/kerrycode/p/5246383.html)
- [Centos7 防火墙 firewalld 实用操作](https://www.cnblogs.com/stulzq/p/9808504.html)