---
title: linux教程
date: 2021-10-13 08:11:39
# 分类：大栏目，只写1个
categories: Linux笔记
# 标签：多个关键词，数组格式
tags:
  - Linux
  - 运维
permalink: linux-tutorial/
---
[狂神说Linux笔记](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483879&idx=1&sn=96181b566c35679e48db6bd26fb59a2c&scene=19#wechat_redirect)


在需要时可以快速恢复。如果是直接关闭虚拟机的话，每次启动虚拟机都会耗费很多时间。

2、硬件参数的设置
建议：先关闭虚拟机，再修改配置

image-20210811201840311
image-20210811201840311

3、快照和系统恢复
当系统出现严重错误怎么办？

1、重装系统

优点：操作简单

缺点：Ubuntu系统的重装会比较耗时。原来系统的配置，文件以及数据没了。

2、系统快照：VMware提供的系统功能

当系统出现问题的时候可以使用最近一次的快照进行恢复。

Linux系统操作
​ Linux可以用作个人桌面（办公，看视频，听音乐…),但其主要还是用于服务器环境。常用应用：文件管理器、命令行终端、文本编辑器

1、文件系统
对windows来说，每个分区有一个盘符。每一个盘符实际上是一个分区partition。

在Linux下没有C:等盘符概念。Linux使用统一的目录树结构。

none
/
/home/w #用户目录
/root
/bin
/mnt
/user
/etc
1.1、用户目录
用户目录，即用户自己的目录。如用户w用户目录为/home/w 。Linux系统上可以支持多个用户，每个用户有一个目录。特例：超级用户root，其用户目录为/root。

权限机制：对普通用户来说，他能操作的目录只有用户目录。root用户没有限制可以操作任何文件和目录。

2、Linux常用命令
none
ls/cd/pwd #目录切换和查看
mkdir/rmdir #目录的创建和删除
cp/rm/mv #复制 删除 移动
tar/zip/unzip #压缩 解压
ln...
查看目录ls
ls,即list,列出目录下的所有项：如 查看当前目录 ls, 查看/home/w目录ls /home/w

详细模式查看 ls -l /home/w
其中 -l 为参数 参数一般义 -开头

none
w@ubuntu21:~$ ls -l /home/w
总用量 36
drwxr-xr-x 2 w w 4096  8月 11 11:43 公共的
drwxr-xr-x 2 w w 4096  8月 11 11:43 模板
drwxr-xr-x 2 w w 4096  8月 11 11:43 视频
drwxr-xr-x 2 w w 4096  8月 11 11:43 图片
drwxr-xr-x 2 w w 4096  8月 11 11:43 文档
drwxr-xr-x 2 w w 4096  8月 11 11:43 下载
drwxr-xr-x 2 w w 4096  8月 11 11:43 音乐
drwxr-xr-x 2 w w 4096  8月 11 11:43 桌面
drwxr-xr-x 3 w w 4096  8月 11 11:44 snap
在输入命令和路径时，按tab建可以自动补全。如 ls /ho -> ls /home

按上下键可以翻阅输入历史的历史命令

显示当前位置pwd
pwd,即print working directory 显示当前工作目录

切换目录cd
cd，change directory 切换目录

切换到用户主目录：直接cd

切换到某个目录：cd /home/w/snap

none
几个特殊目录
~ 代表当前用户的主目录
. 代表当前目录
.. 代表上一节目录
cd ~ 切换到用户主目录,和cd一样
cd ~/snap 切换到主目录下的snap目录
cd ../www 切换到上级目录，再到www子目录

在ls命令中,也可以使用~ ...表示路径
ls ~
ls ./www
还有更复杂的 ./hello/abc/.../123/./other/test.xml
目录操作
mkdir，即make directory创建目录

mkdir abc

mkdir -p abc/123/test

使用-p参数，可以将路径的层次目录全部创建

rmdir，即remove directory删除目录

rmdir abc 如果目录非空，则删除失败

rm ,即 remove 删除文件或目录

rm -rf abc 删除abc,和子项一起删除

其中,r表示recursion(递归删除)，f表示force (强制删除)

rm
rm -rf /* 删除根目录下的所有东西（慎用)
cp,即copy复制文件或者目录

cp -rf snap snap2

强制递归复制snap，如果snap2不存在则会创建snap2,如果snap2存在则会将sanp的内容复制到snap2下。

mv,即move ,移动文件或者目录（重命名)

move hello helloworld

rm/cp/mv这三个命令对文件同样适用

归档：备份
在Linux系统重要的程序或者文件需要备份，首先将其打包为一个文件（tar包），让后在将tar文件备份到某个位置。

tar,即tape(磁带) archive(档案) 档案打包

创建档案包

none
tar -cvf example.tar example
其中c,表示create创建档案

v,表示verbose显示详情

f,表示file

也可以将多个目录打包 tar -cvf xxx.tar file1 file2 file3

还原档案包（解压压缩包)

none
tar -xvf example.tar
tar -xvf example.tar -C outdir
其中,-C参数指定目标目录(C大写表示切换一个目录)，默认解压到当前目录下
上面的tar格式并没有压缩，体积较大。所以可以通过归档并压缩

归档并压缩
none
tar -czvf example.tar.gz example 参数z表示压缩 先归档在使用gz压
解压缩
tar -xzvf example.tar.gz
tar -xzvf example.tar.gz -C outdir
软链接，相当于Windows下的快捷方式
使用ln命令（link）来创建软链接

ln -s source link 其中，-s表示soft软链接（默认为硬)，除了软链接还有硬链接。

如：ln -s example example2

删除软链接对原文件没有影响。

删除原文件，则软链接失效。

以ls -l查看文件详情时，可以看到文件目标路径,查看当前文件是否是软链接

比如，ls -l /

可以发现，/bin 实际指向/user/bin

none
lrwxrwxrwx   1 root root          7  8月 11 10:41 bin -> usr/bin
宿主机和虚拟机之间的拷贝粘贴
一般情况下，文本和文件都可以拷贝