---
title: hexo删除历史大文件
img: /images/博客数据备份.jpg
categories: blog
tags: git
abbrlink: 52951
date: 2023-09-30 16:58:48
---

## hexo删除历史大文件

在git hexo仓库中有一个超过100MB的文件，并且你不小心Commit后，会出现如下报错

remote: error: Trace: 0224239f8e73847812271f660f9977951bd9387c22167deff41b61998441cf36
remote: error: See https://gh.io/lfs for more information.
remote: error: File mymusic/XXX.mp3 is 169.72 MB; this exceeds GitHub's file size limit of 100.00 MB

remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To github.com:bigworldxld/bigworldxld.github.io.git
 ! [remote rejected]   HEAD -> master (pre-receive hook declined)
error: failed to push some refs to 'github.com:bigworldxld/bigworldxld.github.io.git'

当git仓库中添加&删除了大文件后, 历史记录仍然有该文件, 超过100MB的限制，而无法提交，寻找了一下这个文件，发现已经被删除，但是commit中还是有此文件的提交记录。

### 安装 git filter-repo

使用pip安装git filter-repo去消除历史文件

```shell
pip install git-filter-repo
```

执行清理XXX文件

若出现执行git命令报错：fatal: not a git repository (or any of the parent directories): .git

问题分析

没有初始化git本地版本管理仓库，在打开文件的子目录层级中不包含.git文件，所以无法执行git命令。解决方案

跳到包含git.sh的文件目录下，执行git初始化 ，进行初始化

```bash
git init
```

### 执行清理shared/log/cron.log文件

```shell
git filter-repo --invert-paths --path "shared/log/cron.log" --force
```

git gc:

```shell
git gc --prune=now
```


删除.deploy_git文件

### 重新提交

```bash
hexo clean
hexo g
hexo d
```

### 博客备份

hexo的封面图片和音乐信息需要提前备份好，避免清数据没有了！！！

在git push推送时出现remote: fatal: pack exceeds maximum allowed size问题，经过排查，是由于出现大文件，解决方法：把没用的大文件删除，然后进行分批次上传文件并进行提交即可。

remote: fatal: pack exceeds maximum allowed size
方法一：
增加Git的HTTP传输缓冲区大小。可以通过以下命令来增加限制：

git config --global http.postBuffer 524288000  # 设置为500 MiB

或者在当前仓库中设置：

git config http.postBuffer 524288000
方法二：
最近使用git pull代码时总遇到这个问题，记录下解决办法，仅供参考！这个问题主要还是因为git add中有大文件存在，最直接的办法是用git lfs上传。

最近使用git pull代码时总遇到这个问题，记录下解决办法，仅供参考！

这个问题主要还是因为git add中有大文件存在，最直接的办法是用git lfs上传。

１.安装git-lfs

```shell
sudo apt-get install git
git lfs install
```

2. 将需要上传的文件加入git lfs track，取消用git lfs untrack

```shell
# 加入git lfs track，会在执行文件夹下产生.gitattributes
git lfs track "*.pth"
git add .gitattributes
git commit -m "add lfs track file" .gitattributes
# 取消git lfs untrack
git lfs untrack ".pth"
```

Git LFS 其他命令
显示当前被 lfs 追踪的文件列表

```bash
git lfs ls-files 
```

查看现有的文件追踪模式

```bash
git lfs track 
```

查看当前 git lfs 版本

```bash
git lfs version 
```

取消 LFS 的全局配置

```bash
git lfs uninstall
```

### 博客备份恢复

下载原始数据包

输入命令git clone -b main url (目前Git最新版本默认都是main，老版本是master)，这里也可以选择你要拉取的分支到本地。

1.进入hexo根目录
2.直接把箭头指向的`.deploy_git`和`public`直接删除！

先依次执行：

```bash
hexo clean
hexo g
hexo d
```

在将public下需要的图片和音乐文件上传

hexo-renderer-marked 3.1.0 引入了一个新的选项，其允许你无需使用 asset_img 标签插件就可以在 markdown 中嵌入图片
https://github.com/hexojs/hexo-renderer-marked
如需启用：
```yml
_config.yml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```
### 常见hexo错误

hexo Template render error: (unknown path) Error: expected end of comment, got end of file

FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Template render error: (unknown path)
  Error: expected end of comment, got end of file

某一篇博客中可能存在"’{ #“字符串，去掉它，或者去掉”#".