---
title: Linux 常用命令 - 打包、压缩、解压缩
date: 2019-01-06 15:44:40
categories: Linux
tags: [Linux, 命令]
toc: true
---
通过本文学习 Linux 常用命令-打包、压缩、解压缩：tar 。
<!-- more -->

## 名词区分

- 打包：将一大堆文件或目录变成一个总的文件（ tar 命令）
- 压缩：将一个大的文件通过一些压缩算法变成一个小文件（ gzip，bzip2 等）

Linux 中很多压缩程序只能针对一个文件进行压缩，因此当你想要压缩一大堆文件时，你得将这一大堆文件先打成一个包（ tar 命令），然后再用压缩程序进行压缩（ gzip，bzip2 命令）。

习惯上以 .tar 后缀代表 tar 包，用 xxx.tar.gz 或 .tgz 代表 gzip 压缩过的 tar 文件，用 .tar.bz2 代表 bzip2 压缩过的 tar 文件。

## 语法

- tar [主选项 + 辅选项] 文件或目录

使用该命令时，主选项必须有，且仅有一个，如：tar -xzvf mysql-5.6.42-linux2.6-x86_64.tar.gz

## 主选项

使用该命令时，主选项必须有，且仅有一个。

- -c：<create> 新建一个压缩文档，即打包
- -x：<extract> 解压文件
- -t：<list> 查看压缩文档里的所有内容
- -r：<append> 向压缩文档里追加文件
- -u：<update> 更新原压缩包中的文件

## 辅助选项

- -z：具有 gzip 属性，一般格式为 xxx.tar.gz 或xx.tgz
- -j：具有 bzip2 属性，一般格式为 xx.tar.bz2
- -Z：具有 compress 属性,一般格式为 xx.tar.Z
- -v：显示操作过程
- -f：使用文档名，在f之后要立即接文档名，不要再加其他参数
- -C：打包/压缩时可将当前目录更改为指定的目录，详见下文

## 打包/压缩

- tar -cvf img.tar img1 img2 --> 注：将当前目录下 img1 和 img2 两个文件夹打包成 img.tar ，仅打包不压缩
- tar -czvf img.tar.gz img1 img2 --> 注：将当前目录下 img1 和 img2 两个文件夹打包成 img.tar.gz ，打包后，以 gzip 压缩
- tar -cjvf img.tar.bz2 img1 img2 --> 注：将当前目录下 img1 和 img2 两个文件夹打包成 img.tar.bz2 ，打包后，以 bzip2 来压缩
- tar -cvf img.tar -C /usr/local aaa --> 注：将当前目录改为 /usr/local ，并将 /usr/local 下的aaa目录打包到 img.tar

## 不解压的情况下查看

- tar -tvf img.tar --> 注：查看当前目录下 img.tar 中的所有内容

## 解压

- tar -xvf img.tar --> 注：将 img.tar 解压到当前目录
- tar -xvf img.tar img1 --> 注：将 img.tar 解压到当前目录，但只减压 img.tar 中的 img1 文件夹
- tar -xvf img.tar -C /usr/local --> 注：将当前目录改为 /usr/local ，并将 img.tar 解压到 /usr/local 目录

## 更新

- tar -uvf img.tar img1 --> 注：将 img1 文件夹更新到 img.tar 中

## 追加

- tar -rvf img.tar img3 --> 注：将 img3 文件夹追加到 img.tar 中

## C 参数

-C dir 参数的作用在于改变工作目录，其有效期为该命令中下一次 -C dir 参数之前。

- tar -cvf img.tar -C /usr/local aaa --> 注：将当前目录改为 /usr/local ，并将 /usr/local 下的 aaa 目录打包到 img.tar
- tar -xvf img.tar -C /usr/local --> 注：将当前目录改为 /usr/local ，并将 img.tar 解压到 /usr/local 目录

## 解压方法总结

- *.tar 用 tar –xvf 解压
- *.gz 用 gzip -d或者 gunzip 解压
- *.tar.gz和*.tgz 用 tar –xzf 解压
- *.bz2 用 bzip2 -d 或者用 bunzip2 解压
- *.tar.bz2 用 tar –xjf 解压
- *.Z 用 uncompress 解压
- *.tar.Z 用 tar –xZf 解压
- *.rar 用 unrar x 解压，需先安装
- *.zip 用 unzip 解压，需先安装


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)