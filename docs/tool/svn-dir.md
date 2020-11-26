---
title: SVN 目录使用规范
date: 2018-04-02 11:16:11
categories: SVN
tags: [SVN]
toc: true
---
SVN 目录使用规范。
<!-- more -->

## 规范 1（推荐）

|目录名称|说明|
|:-|:-|
|trunk|trunk 是树干，主分支，日常开发进行的地方。|
|branches|branches 是分支。是用来做并行开发的，这里的并行是指和 trunk 进行比较，完成后一般会被合并到 trunk 中。一般有 3 类：【1.】一些阶段性的 release 版本，这些版本是可以继续进行开发和维护的，如 2.1 或 2.x （ 2 系列版本的最新代码）。【2.】为不同用户客制化的版本，也可以放在分支中进行开发，如 2.2_dev 。【3.】某个版本的bug修复，如： 2.1_bugfix 。|
|tags|tags 是标记，一般是只读的，这里存储阶段性的发布版本，只是作为一个里程碑的版本进行存档，如： 2.1 ，2.1.1 。|

## 规范 2

|目录名称|稳定程度|权限|说明|
|:-|:-|:-|:-|
|branches|开发分支，不稳定|开发 team 有权限|有开发任务时，从 trunk 打分支到 branches ，分支命名以日期为前缀，如： 20170101 ，可再加上_本次分支主要内容，如： _monitor 。（如果 trunk 分支在测试且证明极度不稳定，想取稳定分支，从 tags 取）。开发且自测完成时，由研发 Leader 合并到主干 trunk ，测试从 trunk 发包进行测试。一般有 3 类：1.】准备发布的分支（进行生产环境的测试、准备） Release Branch ，如 BUG-1.0_235 ( copy from tag/tag_release_1.0 , bug 版本号为235)。【2.】Bug 修复的分支（进行某编号的 bug 修复） Bug fix branch ，如 RB-1.1 ( 1.1 版本的 Release Branch )。【3.】新技术实验性分支（将某个新技术引进项目） Experimental branch ，如 TRY-1.0_PHP7 ( copy from tag/tag_release_1.0 ，PHP7 实验技术)。这些都要根据需要最终 merge 到 trunk 里面。|
|trunk|主干分支，趋于稳定|开发 Leader 有权限|最新趋于稳定版本代码存放地。开发 Leader 有权限从开发分支 merge 代码到主干，然后质量部进行测试，测试通过由运维部打上线分支到 tags 。研发 leader 要控制 trunk 的时序性。（也就是说尽量避免一个 brances 合并到 trunk 进行测试之后，在没有完成测试前又合并一个分支，导致测试返工。）|
|tags|上线分支，稳定|运维有权限|方便回滚和记录。以版本号命名，如 1.1、1.2 。|


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)