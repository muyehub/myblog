---
layout:     post
title:      "Gerrit概念说明及使用"
date:       2020-09-23 18:35:52 +0800
author:     "max"
---

### Gerrit介绍

#### Gerrit简介

Gerrit, 一种开放源代码的代码审查软件, 使用网页界面. 利用网页浏览器, 同一个团队的软件开发者, 可以相互审阅彼此修改后的代码, 决定是否能够提交, 回退或是继续修改. 它使用版本控制系统Git作为底层.

它分支自Rietveld, 作者为Google公司的Shawn Pearce, 原先是为了管理Android项目而产生. 这个软件的名称, 来自于荷兰家具设计师赫里特·里特费尔德(Gerrit Rietveld).

因为对访问控制表(ACL)相关的修正, 没有被集成进Rietveld, 之后Gerrit就由Rietveld分支出来, 形成独立软件项目.

最早它是由Python写成, 在第二版后, 改成用Java与SQL. 使用Google Web Toolkit来产生前端的JavaScript.

#### 为什么需要Gerrit

首先, 代码审查可以帮助程序员了解系统功能, 从整体掌控代码质量, 其次, 通过代码审核可以及时止损, 构建更加健壮的系统代码.

#### 代码审核的建议(来自[程序员客栈](https://www.zhihu.com/question/20046020/answer/555007256))

1. 对事不对人, 大家都是同事, 在一个团队工作和气最重要. 不要在Code Review中说"你写的什么垃圾"这种话, 你可以说"这个变量名不是很好理解, 咱们换成xxx是不是更好"

2. 每个Review至少给一条正面评价. Gerrit中有对代码点赞的功能, 可以时不时的使用一下.

3. 保证发布的代码和评审意见的可读性.

4. 用工具进行基础问题的自动化检查. 用Tab还是空格, 用两个空格还是四个空格, 缩进风格是使用K&R还是Allman. 这些问题可以使用php code sniffer解决, 团队应该把精力放在代码规范, 代码性能优化等地方.

5. 全员参加Code Review, 并设定各部分负责人.

6. 每个代码PR(Pull Request)内容一定要少. Code Review效果和质量与PR代码量成反比, 提交的代码越多, Code Review的效果就越差. 所以要经常Code Review, 保证每个PR代码的量要少, 最多不超过300行/PR.

7. 在写新代码之前, 先Review掉需要评审的代码. 不要堆积Review, 有PR产生一定要尽快Review, 否则时间拖的长了以后Review的过程就会比较艰难.

8. 不要在Review中讨论需求, Review就是Review. 要明确Code Review是完善代码, 始终要以代码质量为中心要素.

### Gerrit使用

#### 账号登录与查看设置

首先, 需要在LDAP系统中发放账号, 拿到自己的账号后进行登录.

登录后在账号中点击Settings设置, 找到新增SSH kyes的设置, 将自己机器上的公钥添加进去

![SSH keys设置](https://upload-images.jianshu.io/upload_images/1656074-c0e5e4483b5c40c8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后到仓库中选择属于自己的项目clone到本地

![git项目列表](https://upload-images.jianshu.io/upload_images/1656074-cbd5f77ab4a78c37.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意: 红框中的两个项目为默认项目, 是新建仓库时需要继承权限来用的, 不要动

![默认项目](https://upload-images.jianshu.io/upload_images/1656074-c65ca4a43345fdce.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


克隆完毕后需要在项目目录下修改和账户相同的邮箱与真实的姓名, 否则提交时会校验失败

```
git config user.email muyege@gmail.com

git config user.name muyege
```

加--global是修改全局的, 如果项目很多, 为避免一个个项目修改麻烦, 可以加这个参数.

#### 本地仓库设置和提交

因为Gerrit的设置, 一般用户是没有直接提交到主分支或者开发分支的权限的, 必须提交到引用分支先进行代码审核, 审核通过后才能merge到对应分支. 所以, 在本地操作代码库的时候会出现push不上去的问题, 这时候我们需要在项目下设置Gerrit提供给我们的引用分支.

一般情况下在Gerrit权限设置里有三种类型的分支:

refs/* : 这是管理员才有的权限, 表示当前项目下的所有分支, 拥有此权限可以随意push到任何远端分支

refs/heads/* : 这是不需要经过code review的分支, 与 refs/* 同样权限

refs/for/* : 这是需要经过code review的分支, 提交到此分支上的代码需要经过code review, 通过后才能合并到正式分支上.

我们在提交代码审核的时候需要将代码提交到gerrit为我们提供的引用分支上, 比如说: 我在dev分支上操作, 本来要push的命令需要由 git push origin dev 改为 git push origin HEAD:/refs/for/dev.

如果嫌每次写这么长不方便的话, 可以在项目下做如下配置, git config remote.origin.push 'refs/heads/\*:refs/for/\*', 这样每次提交的时候就会自动提交到引用分支上了.

#### 代码审核

当有代码需要审核时会在CHANGES标签中看到待审核的纪录

![审核状态](https://upload-images.jianshu.io/upload_images/1656074-cd61a43a015eef8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Open表示打开的代码审核

Merged表示已经通过且已合并的代码审核

Abandoned表示未通过或遗弃的代码审核

![审核页面](https://upload-images.jianshu.io/upload_images/1656074-bce72e437220832f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

审核页面, 有 +1 和 +2 两个权限.

只有有 +2 权限的同学才能够把审核通过的代码 submit 到真正的远端分支中.

+1 权限的同学, 多为与你一个组, 或者做同一个业务的同学, 他们对你的代码先进行一次review

当+2 的同学审核通过后, submit 了代码后, 本次Review 就结束了.

> 然后回到你自己的本地分支, 执行以下 git pull --rebase 同步一下远端分支. 然后继续进行后续工作.

当评审不通过时, 需要修改代码后再次提交评审, 这里分两种情况, 第一种情况是评审人员没有ABANDON的情况下, 可以修改代码后使用以下命令再次提交

```

git add file

git coomit --amend --no-edit

```

第二种情况是评审人员已经将此次评审ABANDON了, 那么就需要重新走正常流程提交评审.

#### 冲突解决

在实际开发中, 可能其他同学提交的代码与自己修改的代码修改了同一处地方, 从而产生了冲突.

比如：

A同学提交了 a.txt 文件, 在最后一行添加了而一些内容并且审核通过已经 submit了. 此时, B同学也在a.txt文件中的最后一行添加了内容, 但是没有先做 git pull --rebase 操作, 直接推到gerrit平台上则会产生冲突.

此时: 需要先到本地进行 git pull --rebase 操作, 更新远端代码, 解决冲突! 具体步骤如下:

1> git pull --rebase

2> 找到冲突文件, 并解决冲突, 解决完后执行 git add 文件名

3> 继续执行git rebase --continue, 如果有冲突文件继续解决.

4> 待所有冲突解决之后, 执行git commit --amend  --no-edit 命令, 不需要对commit msg信息进行任何修改

5> 最后执行git push origin xxx, 将分支代码提交即可.
