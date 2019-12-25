---
layout: post
title:  "GitLab设置SSH key，以及相关知识"
date:   2019-09-10 10:42:00 -0700
categories: Practice
tags: Security SSH GitLab GitHub Practice
description: 已有GitHub的SSH key的情况下，新增GitLab的SSH key，以及一些SSH的相关背景知识
---
### GitLab设置SSH key（在本地已有SSH key的情况下）
环境：macOS 10.14.6
（因为是Mac，所以预装了OpenSSH client。Windows得先装这个再进行以下操作）
1. 生成SSH key：
    ```sh
    $ ssh-keygen -t ed25519 -C "注册用的email, ..."
    ```
    - ed25519这个数字签名算法是最安全，performance最好的（验证、签名、key生成的速度都快，签名公钥都很小）。如果server支持就用尽量这个。
    - 默认且最常用的是RSA。推荐加上-b 4096参数生成4096 bits的key；旧的RSA不安全（仅一轮MD5 hash），推荐在6.5版本之后，加上-o用新方法encode。
        ```sh
        ssh-keygen -o -t rsa -b 4096 -C "注册用的email, ..."
        ```
    - -C 参数是comment，可不写。可以是本机信息，因为注释会包含在最后的公钥中。
    - -f 参数指定file name
2. 在被保存SSH key的file的时候，不要用之前GitHub重复的文件（或者说，任何已有的SSH key的文件，否则之前的就会被覆盖 or 退出）。
    - 默认名字是id_签名算法。如果之前是一路enter，那么就是id_rsa
    - 最好保持路径一样：在默认文件夹：~/.ssh下
3. 在被问password（passphrase）的时候，为了安全，可以使用。这是为了在本机硬盘上保护私钥。
4. 后台启动ssh-agent，将 SSH 私钥添加到 ssh-agent 并将密码存储在密钥链中：
    ```sh
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/new_ssh_key
    ```
    - 如果没有设置passphrase，可以省略这一步；如果有passphrase而省略这一步，每次连接都会询问passphrase。
    - ssh-agent创建socket，等待SSH connections。用户只需输入一次passphrase，之后ssh-agent会处理一切。
    - 如果需要删除，使用`ssh-add -D ~/.ssh/new_ssh_key`，只对手动加入的有效
    - 如果不指定文件，就是加入默认文件夹：.ssh下的所有available keys
5. 创建/修改配置文件，保证本机能够根据host选择对应的私钥：
    ```sh
    vim ~/.ssh/config
    (a)
    # GitHub
    Host github.com
      Preferredauthentications publickey
      IdentityFile ~/.ssh/id_rsa

    # Private GitLab instance
    Host gitlab.company.com
      Preferredauthentications publickey
      IdentityFile ~/.ssh/new_ssh_key
    (esc + : + wq)
    ```
    - 如果之前有使用密码保护私钥，可以用`UseKeychain yes`将passphrases都存在Keychain中
    - 如果是macOS Sierra 10.12.2及以上版本的，可以直接指定`AddKeysToAgent yes`从而省略上一步（ssh-agent）的手动配置
6. 复制公钥到剪切板：
    ```sh
    pbcopy < ~/.ssh/new_ssh_key.pub
    ```
    - 如果手动copy，要注意copy所有内容：以ssh-ed25519 / ssh-rsa开始，以使用的email结尾。
7. 添加公钥到远程服务器：
    - 大概有可以用的图形界面（GitLab: setting - SSH Keys - Key）
    - 如果生成的时候用了-C，那么注释会出现在Title下。如果没有？写个好辨认的title，比如，home，或者Work Laptop。
    - 如果没有，需要手动添加，则：
    ```sh
    ssh-copy-id -i identity_file username@remote_host
    ```
        - 需要输入远程用户登陆密码
        - 命令完成后，公钥就会被放到remote account的~/.ssh文件夹下，名为authorized_keys
    - 如果没有，可以使用`cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`替代（显示，管道，如果不存在则创建文件夹，加在文件最后）
    - 或者可以手动复制：
    ```sh
    cat ~/.ssh/id_rsa.pub
    # log in to remote server
    mkdir -p ~/.ssh
    echo public_key_string >> ~/.ssh/authorized_keys
    ```
    - 完了之后，为了安全，可以设置文件夹为仅当前用户读写执行，文件为仅当前用户读写：
    ```sh
    chmod 600 ~/.ssh/authorized_keys
    chmod 700 ~/.ssh
    ```
8. 验证一下之前的SSH key没有被破坏（此处是GitHub），以及新的SSH key是否添加成功
    ```sh
    ssh -T git@github.com
    ssh -T git@gitlab.company.com
    ```
    - 可以加参数-v看到详细验证过程

---

### 相关背景知识
#### SSH
用于加密登陆的网络协议（即使被截获，内容也不会泄露），有多种实现。主要用于远程登录（safely administering remote servers）。在2个party间建立安全的连接，互相验证，传递commands和output。

#### SSH工作流程：
1. 建立安全信道：此处使用对称加密（双方共享一个key，用于加密和解密；或者一堆关系简单可以推导的key）。
    - 无论用那种验证方法，都要先建立安全信道
    - 这个key是sesson-base，用于保证之后所有的传输的安全
    - 可以设置使用不同的symmetrical cipher systems：client有个preference list，最后双方共同选择的是，client的list上尽量靠前的，server也支持的。
    - 大概过程是：
        1. 双方共享一个大质数；
        2. 双方约定encryption generator (eg. AES)；
        3. 双方各自决定一个秘密的质数，当作私钥；
        4. 使用这个私钥，还有约定好的相同的encryption generator，和共享的大质数生成公钥。这个公钥可以随意公开，并且公钥无法推导出对应的私钥；
        5. 双方交换公钥；
        6. 各方有自己的私钥和对方的公钥，以及一开始共享的大质数，可以计算出一个shared secret key。双方各自计算，但是结果是一样的。
    - 最后得到的共同的secret key就是上面所说的sesson-based key，用于双方加密解密接下来的通信
2. 验证用户：使用非对称加密（单方向发送数据需要一对associated key：公钥可以随意分享，用于加密；私钥用于解密对应公钥加密的内容。）
    - 用户名密码：传统方法，像普通的计算机登陆。密码被加密后传送，但是长度有限，可以暴力破解。
    - SSH key：更安全。
        - client生成一对key pair。
        - server存公钥（加密），client都有（私钥解密）。
        - server通过验证client有对应私钥验证client身份。
        - client通过第一次连接的时候获取的fingerprint人工手工验证server，只要这个fingerprint不变化，就不会再bother到人（或者在config文件中设置StrickHostKeyChecking no，自动把新的host加入known_hosts，否则第一次连接的时候会询问；UserKnownHostsFile /dev/null，不警告新的或者fingerprint变化的host。）
        - 具体验证过程：
            1. client发送自己的key pair的ID给server；
            2. server在对应用户目录下的~/.authorized_keys中查找到对应的公钥；
            3. 如果found，server生成随机数字，用公钥加密；
            4. server发送这个加密后的信息（当然，要使用先前建立好的安全信道，所以还有一道加密。不过这个相当于当前layer下的一层了，可以当作透明，不考虑）；
            5. client如果有对应的私钥，就可以成功解密，得到original number；
            6. client结合解密后的数字，以及shared session key，计算出MD5值；
            7. client发送这个MD5值到server，作为答案；
            8. server一样使用shared session key，和自己有的original number计算MD5值。如果一样，则认为client有provate key，验证成功。

另外，还会使用hash保证data integrity。

---
#### 参考资料：
- [github/gitlab同时管理多个ssh key](https://xuyuan923.github.io/2014/11/04/github-gitlab-ssh/)
- [github和gitlab公用，ssh key 配置](https://www.jianshu.com/p/b9f686dfbdb2)
- [生成新 SSH 密钥并添加到 ssh-agent](https://help.github.com/cn/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
- [GitLab and SSH keys](https://gitlab.com/help/ssh/README#types-of-ssh-keys-and-which-to-choose)
- [Understanding the SSH Encryption and Connection Process](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process)
- [How To Configure SSH Key-Based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
- [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
- [Git连接GitLab远程仓库](https://www.cnblogs.com/gavincoder/p/10054532.html)
- [SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
- [Diffie-Hellman密钥是如何交换的](https://blog.csdn.net/qq_40870418/article/details/78829769)
- [what's the purpose of ssh-agent?](https://unix.stackexchange.com/questions/72552/whats-the-purpose-of-ssh-agent)
- [How to remove an ssh key?](https://stackoverflow.com/questions/25464930/how-to-remove-an-ssh-key)
- [Upgrade Your SSH Key to Ed25519](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54)