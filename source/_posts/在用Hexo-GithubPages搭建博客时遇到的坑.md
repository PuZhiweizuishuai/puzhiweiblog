---
title: 在用Hexo+GithubPages搭建博客时遇到的坑
date: 2018-05-12 21:59:43
tags: [Hexo,
      GitHub,
      博客]
categories: "学习记录"
copyright: true
top: 100
---

<img src="http://7xjjdc.com1.z0.glb.clouddn.com/Hexotite.png" alt="hexo+github"/>
{% note class_name %}**开发时踩坑记录** {% endnote %}
<!-- more --> 
这不是一篇教程，主要还是记录了我从0搭建过程中遇到的坑
另外网上教程那么多，我何必多此一举呢？
## 开发这个博客时主要参考：

<a href="https://hexo.io/zh-cn/docs/index.html">Hexo中文文档</a>
<a href="http://theme-next.iissnan.com/getting-started.html">Next主题官方文档</a>
<a href="https://zhuanlan.zhihu.com/p/26625249">GitHub+Hexo 搭建个人网站详细教程</a>
<a href="https://www.jianshu.com/p/f054333ac9e6">Hexo+Next主题美化</a>

## 解决Hexo d命令报错

```
FATAL bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument

```

或者显示没有发现git仓库，当时忘截图了，大概就是这个意思

将原来_config.yml中的deploy中repo格式改为下列形式

```
deploy:
 type: git
 repo:git@github.com:your_github_user_name/your_github_user_name.github.io.git
 branch: master
```
### 重新生成ssh key并添加到github

```
$ ssh-keygen -t rsa -C 948805382@qq.com（换成你的邮箱地址）
```
接着出现的一些步骤都可以回车跳过： 

复制ssh密匙要将C:\Users\puzhiwei\.ssh\id_rsa.pub中内容全部复制到GitHub

添加过程见参考资料3

### 验证ssh key：
```
$ ssh -T git@github.com
```
可能会先弹出让你输入yes的选项，输入yes后，出现

```
Hi puzhiweizuishuai! You've successfully authenticated, but GitHub does not provide shell access.
```
即链接成功


### 初始化本地git仓库

设置Git的user name和email：
```
$ git config --global user.name "puzhiweizuishuai"(换成你的用户名)
$ git config --global user.email "948805382@qq.com"（换成你的邮箱地址）
```

在本地的hexo init生成的文件夹中初始化git仓库：

```
$ git init
```

做完以上这些步骤，说明你的仓库可以使用ssh方式来上传下载代码，而不需要输入用户名和密码了

### 网站部署

```
$ hexo clean  //清除缓存文件db.json和已生成的静态文件public
$ hexo g   //生成网站静态文件到默认设置的public文件夹
$ hexo d   //部署网站到设定的仓库
```

## 菜单栏汉化

第一次以为直接在hexo的配置文件里改就行，如将home改成主页，

<img src="https://github.com/PuZhiweizuishuai/PzhiweiBlogPhoto/blob/master/%E8%8F%9C%E5%8D%95.JPG" />

后来发现这样是错误的查找官方文档后将hoex配置文件里的language文件修改为

```
language: zh-Hans
```

但菜单栏还是不改变
最后查找资料发现要将主题中language文件修改为zh-CN
并将hoex修改为

```
language: zh-CN

```
才能变成中文


## 去掉低栏的logo和注意事项

用Next主题搭建起来的博客，在底部会有一个特别烦人的官方logo
<img src="http://7xjjdc.com1.z0.glb.clouddn.com/Nextlogo.JPG" />

对于强迫症的我必须要改掉、改掉、改掉！

### 解决方案

1.首先，找到 \themes\next\layout\_partials\下面的footer.swig文件，打开会发现，如下图的语句：

<img src="http://7xjjdc.com1.z0.glb.clouddn.com/%E4%BF%AE%E6%94%B9logo.jpg" />


第一个框 是下面侧栏的“日期 XXX”如果想加东西，一定要在双大括号外面写。如：xxx{{config.author}},当然你要是想改彻底可以变量都删掉，看个人意愿。

第二个，是图一当中 “由Hexo驱动” 的Hexo链接，先给删掉防止跳转，如果想跳转当然也可以自己写地址，至于中文一会处理。注意删除的时候格式不能错，只把
```
<a>...</a>
```
标签这部分删除即可，留着两个单引号'',否则会出错哦。

第三个框也是最后一个了，这个就是更改图一后半部分“主题-Next.XX”,这个比较爽直接将
```
<a>..</a>
```
都删掉，同样中文“主题”一会处理，删掉之后在上一行 ‘-’后面可以随意加上你想显示的东西


2.接下来，处理剩余的中文信息。找到这个地方\themes\next\languages\ 下面的语言文件zh-CN.yml（这里以中文为例，有的习惯用英文的配置文件，道理一样，找对应位置即可）
这个是我修改后的：
<img src="http://7xjjdc.com1.z0.glb.clouddn.com/logo%E4%BF%AE%E6%94%B9.JPG" />

```
footer:
  powered: " %s "
  theme: 吐槽
```

## 添加地址栏图片

加入后就可以在浏览器的标签栏或者是收藏夹里面现实网站的缩略图标了。 
在themes/next/的_config.yml中配置：

```
Put your favicon.ico into `hexo-site/source/` directory.
 favicon: /images/favicon.ico
```

然后把图标放到根目录的source/images/下面。

## 添加文章末尾的版权声明

### 在路径\themes\next\layout\_macro中添加passage-end-tag.swig文件，其内容为：

```
{% if theme.passage_end_tag.enabled %}+
<br/>
<div style="border: 1px solid black" weight="50%" height="50%">
<div style="margin-left:10px">
<span style="font-weight:blod">版权声明</span>
<br/>
<p style="font-size: 10px;line-height: 30px">

由
<a href="https://puzhiweizuishuai.github.io" style="color:#258FC6">
史上最帅社会主义接班人
</a>
创作并维护的<a href="https://puzhiweizuishuai.github.io" style="color:#258FC6">不挂英语</a><br/>
博客采用<a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" style="color:#258FC6">
创作共用保留署名-非商业-禁止演绎4.0国际许可证</a>。<br/>
本文首发于<a href="https://puzhiweizuishuai.github.io/" style="color:#258FC6">不挂英语</a> 
博客（ <a href="https://puzhiweizuishuai.github.io/" style="color:#258FC6">https://puzhiweizuishuai.github.io/</a> ），
版权所有，侵权必究。</p>
</div>
</div>
```
### 修改 post.swig 文件

在\themes\next\layout\_macro\post.swig中，post-body之后，post-footer之前添加如下代码：

```
<div>
  {% if not is_index %}
    {% include 'passage-end-tag.swig' %}+
  {% endif %}
</div>
```

### 在主题配置文件中添加字段

在主题配置文件 _config.yml中添加以下字段开启此功能：

```
# 文章末尾添加“本文结束”标记
passage_end_tag:
  enabled: true
```

## 换电脑了怎么办

我查找了网上很多教程，最后采用了新建一个仓库，保存原文件的做法
具体如下
由于在建立博客时账号设置什么的你应该已经搞定了，我就不多说了
接下来只必要的和可能遇到的错误

建立远程库
这个很简单，我也就不写了
直接从建立本地库开始

1.找到博客所在的文件夹

2.git init （在本机上新建一个git仓库）

3.git add -A (将文件的修改，文件的删除，文件的新建，添加到暂存区,即保存所有的修改)

4.git remote add origin xxxxxxxxx xxxxxx 就是你仓库的地址，具体的地址可以去Github上copy。关联远程仓库。

5.git commit -m “firstCommit” （提交文件，将暂存区的内容提交至Git本地数据库）

6.git pull --rebase origin master 更新远程更新到本地

7.git push origin master（git push -u origin master） 将本地repo于远程的origin的repo合并，第一次用-u，系统要求输入账号密码
此时可能会出现如下错误

```
To github.com:PuZhiweizuishuai/puzhiweiblog.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:PuZhiweizuishuai/puzhiweiblog.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

解决方案为：git push origin master -f，强行让本地分支覆盖远程分支。

8. 后期执行
```
git add .   // 添加文件到版本库（只是添加到缓存区），.代表添加文件夹下所有文件 

git commit -m "first commit" // 把添加的文件提交到版本库，并填写提交备注

git push origin master  // 第一次推送后，直接使用该命令即可推送修改
```
即可完成仓库更新


这样换电脑后直接克隆这个仓库里的文件，重新安装运行环境就行了


## End
其他问题机本上参考官方文档就可以解决，我就不多写了，毕竟太懒，有什么问题可以在评论里回复我

***

