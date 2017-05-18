---
layout: post
title: 创建配置git服务器
tags:
- git
- aliyun
- synchro
categories: GIT
description: 在阿里云上搭建git服务器，实现本地代码更新到站点目录。（centos）
---
在阿里云上搭建git服务器，实现本地代码更新到站点目录。（centos）



<!-- more -->
1. 安装git
```
sudo yum install git    //ubuntu上用apt-get install git。
```
2. 创建一个git用户，用来运行git服务
```
sudo adduser git
```
3. 创建证书登录：

- 收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。

- 在阿里云服务器根目录/home/git/下创建文件夹.ssh,在.ssh下创建authorized_keys文件。

- 在本地客户端的git bash中通过命令：
```
$ ssh-keygen -t rsa -C"youremail@example.com"  
```
或
```
$ ssh-keygen
```
命令生成公钥，默认在c盘：用户/.ssh中id_rsa.pub文件是公钥，用记事本打开复制粘贴到服器的/home/git/.ssh/authorized_keys文件中即可。

4.初始化服务器上git仓库
- 选定一个目录作为Git仓库，假定是/data/testgit/sample.git，在/data/testgit目录下输入命令：
```
$ sudo git init --bare sample.git
```
- 把owner改为git：
```
$ sudo chown -R git:git sample.git
```


5.禁用shell登录：(实际操作时没做这一步)
- 出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：
```
git:x:1001:1001:,,,:/home/git:/bin/bash
```
改为
```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```
- 这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

6.克隆远程仓库：
- 本地客户端的git bash输入下面命令
```
$ git clone git@server:/data/testgit/sample.git
```

7.将代码推送到服务器上
```
git add .
git commit -am "test commit"
git push origin master
```
- 如果在这里推送失败了，极有可能是因为服务器的权限问题,解决方法，修改testgit权限。
```
chown -R git:git testgit
```

8.实现自动同步到站点目录
- 自动同步功能用到的是 git 的钩子功能
- 服务器端：进入裸仓库：/data/testgit/sample.git
```
cd /data/testgit/sample.git
cd hooks
//这里我们创建post-receive文件
vim post-receive
//在该文件里输入以下内容
#!/bin/bash
git --work-tree=/data/test checkout -f
//保存退出后，将该文件用户及用户组都设置成git
chown git:git post-receive
//由于该文件其实就是一个shell文件，我们还应该为其设置可执行权限
chmod +x post-receive
```
- 本地，在上面clone出的文件中，添加文件，推送到服务器上。
- 若出现推送不成功，或者在Git推送的工程中发现推送成功 但是在/data/test目录下并没有自己的代码,这是由于文件夹的权限的原因造成的.
- 解决：对于test文件，让git用户加入这个组;并给git添加写入权限
```
cd /data/
chown -R git:git test
```

9.将本地git仓库与git服务器仓库关联（本人未操作，直接clone下来，此时与git服务器仓库已关联）
- 在本地git bash中定位到本地仓库，输入并执行下方代码 
```
$ git remote add origin git@xxx.xxx.xxx.xxx:/data/testgit/sample.git
```

> 参考资料：

- [http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)
 
- [http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)

- [http://www.jb51.net/article/98421.htm](http://www.jb51.net/article/98421.htm)


