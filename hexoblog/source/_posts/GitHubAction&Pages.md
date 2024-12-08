---
title: GitHub Action&Pages
img: https://bigworldxld.github.io/picx-images-hosting/images/githubAction&Pages.webp
categories: tools
tags:
  - github
  - yaml
abbrlink: 23237
date: 2024-11-17 14:36:17
---



## GitHub Pages

您可以在新仓库或现有仓库中创建 GitHub Pages 站点

GitHub Pages 是通过 GitHub 托管和发布的公共网页。 启动和运行的最快方法是使用 Jekyll 主题选择器加载预置主题。 然后，您可以修改 GitHub Pages 的内容和样式

Jekyll 是一个静态站点生成器，内置 GitHub Pages 支持

 Jekyll 使用 Markdown 和 HTML 文件，并根据您选择的布局创建完整静态网站。 Jekyll 支持 Markdown 和 Lick，这是一种可在网站上加载动态内容的模板语言。

### 在 GitHub Pages 网站中配置 Jekyll

可以通过编辑 `_config.yml` 文件来配置大多数 Jekyll 设置，例如网站的主题和插件。 

对于 GitHub Pages 站点，有些配置设置不能更改

```yaml
lsi: false
safe: true
source: [your repo's top level directory]
incremental: false
highlighter: rouge
gist:
  noscript: false
kramdown:
  math_engine: mathjax
  syntax_highlighter: rouge
```

默认情况下，Jekyll 不会构建以下文件或文件夹：

- 位于名为 `/node_modules` 或 `/vendor` 的文件夹中
- 以 `_`、`.` 或 `#` 开头
- 以 `~` 结尾
- 被配置文件中的 `exclude` 设置排除

如果想要 Jekyll 处理其中任何文件，可以使用配置文件中的 `include` 设置。

**使用 Jekyll 在本地测试 GitHub Pages 站点**

先决条件

在使用 Jekyll 测试站点之前，您必须：

- 安装 [Jekyll](https://jekyllrb.com/docs/installation/)
- 创建一个 Jekyll 站点

建议使用 [Bundler](https://bundler.io/) 安装和运行 Jekyll。 Bundler 可管理 Ruby gem 依赖项，减少 Jekyll 构建错误和阻止环境相关的漏洞

### [本地构建网站](https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll#building-your-site-locally)

1. 打开Git Bash。

2. 导航到站点的发布来源。 有关详细信息，请参阅“[配置 GitHub Pages 站点的发布源](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)”。

3. 运行 `bundle install`。

4. 在本地运行您的 Jekyll 站点。

   ```shell
   $ bundle exec jekyll serve
   ```



## Github-Action

GitHub Actions 是一种持续集成和持续交付 (CI/CD) 平台，可用于自动执行生成、测试和部署管道。 你可以创建工作流，以便在推送更改到存储库时运行测试，或将合并的拉取请求部署到生产环境

**workflow 文件**

[参考](https://docs.github.com/zh/actions/writing-workflows/workflow-syntax-for-github-actions#about-yaml-syntax-for-workflows)

GitHub Actions 的配置文件叫做 workflow文件，存放在代码仓库的.github/workflows/目录下。比如写一个first.yaml文件，存储的目录就是.github/workflows/first.yaml

workflow/下的文件采用 YAML 格式，文件名可以任意取，但是后缀名统一为.yml或者yaml，比如foo.yml。一个库可以有多个 workflow 文件。GitHub 只要发现.github/workflows目录下里面有.yml文件，就会自动运行该文件。

workflow/下的文件的配置字段非常多，下面是一些基本字段
### name

- name字段是 **workflow** 的名称。如果省略该字段，默认为当前 **workflow** 的文件名。

name: GitHub Actions Test

### on

+ on字段指定触发 workflow 的条件，通常是某些事件。

on: push
上面代码指定，push事件触发 workflow。

on字段也可以是事件的数组。

on: [push, pull_request]
上面代码指定，push事件或pull_request事件都可以触发 workflow。

完整的事件列表，请查看[官方文档](https://docs.github.com/zh/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)。除了代码库事件，GitHub Actions 也支持外部事件触发，或者定时运行

**on. .**

> 指定触发事件时，可以限定分支或标签。

```yaml
on:
  push:
    branches:
      - master
```

上面代码指定，只有`master`分支发生`push`事件时，才会触发 workflow

**on.issues.types**

某些事件具有活动类型，可让你更好地控制工作流的运行时间。 使用 `on.<event_name>.types` 定义将触发工作流运行的事件活动类型。

例如，`issue_comment` 事件具有 `created`、`edited` 和 `deleted` 活动类型。 如果工作流在 `label` 事件上触发，则每当创建、编辑或删除标签时，它都会运行。 如果为 `created` 事件指定 `label` 活动类型，则工作流将在创建标签时运行，但不会在编辑或删除标签时运行

```yaml
on:
  issues:
    types:
      - opened
      - labeled
```

### jobs..name

workflow 文件的主体是jobs字段，表示要执行的一项或多项任务。

jobs字段里面，需要写出每一项任务的job_id，具体名称自定义。job_id里面的name字段是任务的说明。

jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job

> 上面代码的jobs字段包含两项任务，`job_id`分别是`my_first_job`和`my_second_job`。

### jobs..needs

needs字段指定当前任务的依赖关系，即运行顺序。

jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
上面代码中，job1必须先于job2完成，而job3等待job1和job2的完成才能运行。因此，这个 workflow 的运行顺序依次为：job1、job2、job3。

### jobs..runs-on

runs-on字段指定运行所需要的虚拟机环境。它是必填字段。目前可用的虚拟机如下。

ubuntu-latest，ubuntu-18.04或ubuntu-16.04
windows-latest，windows-2019或windows-2016
macOS-latest或macOS-10.14
下面代码指定虚拟机环境为ubuntu-18.04

runs-on: ubuntu-18.04

### jobs..steps

steps字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

jobs.<job_id>.steps.name：步骤名称。
jobs.<job_id>.steps.run：该步骤运行的命令或者 action。
jobs.<job_id>.steps.env：该步骤所需的环境变量。
下面是一个完整的 workflow 文件的范例。

## 附录

常见项目的目录结构及介绍
ghaction-github-pages/
├── .github/
│   └── workflows/
│       └── gh-pages.yml
├── .gitignore
├── LICENSE
├── README.md
└── action.yml
.github/workflows/: 包含 GitHub Actions 的工作流配置文件。
.gitignore: 指定 Git 忽略的文件和目录。
LICENSE: 项目的开源许可证。
README.md: 项目的介绍和使用说明。
action.yml: GitHub Actions 的自定义动作配置文件


“Pull Request”选项是一种用于代码协作和审查的重要功能。
“Actions” 是 GitHub 提供的一项强大的持续集成（CI）和持续交付（CD）服务
持续集成一次运行的过程，即一个 workflow；
提交代码，单元测试，代码编译，程序构建，推送服务器，完成部署

一个 workflow 由一个或多个 jobs 构成；
每个 job 由多个 step 构成；
每个 step 可以依次执行一个或多个命令，顺序执行，这些命令称为 actions；
event事件/触发器：提交代码，创建分支，创建tag，提交PR，定时执行...
GitHub Actions 的配置文件，通常存放在代码仓库的 .github/workflows 目录中。
格式：采用 YAML 格式
on: push 表示当有新的提交推送到仓库时触发 workflow
runs-on：指定运行 job 所需的虚拟机环境，将文件main.py 恢复到最近一次提交时的版本

Caches：缓存功能允许开发者在CI/CD管道中缓存依赖项、构建输出或其他文件，以减少重复下载和构建时间
Deployments：功能允许开发者将代码直接从GitHub仓库部署到各种环境，如云服务、虚拟机或容器等
Attestations（证明）：确保他们的代码在构建和部署过程中没有被篡改或损坏
Runner（运行器）：官方托管的Runner由GitHub管理和维护，适用于大多数常见的CI/CD任务。
自托管的Runner允许开发者在自己的硬件或云基础设施上运行工作流

Wiki选项是一个强大的文档编写和共享工具，它允许用户在项目页面上创建、编辑和管理文档
	
Pakages(github包管理器，javasrcipt-npm,ruby-gem,java-mvn,java-gradle,.NET-dotent CLI,Docker)
注意：如果您的项目包含动态内容（例如数据库），GitHub Pages可能无法正常工作，因为它不支持服务器端代码。在这种情况下，您可能需要使用其他托管服务，如Heroku, Netlify等，这些服务支持全栈的应用部署。

Codespaces:github项目页面点击。进入网页版的vscode,在运行和调试按钮，继续工作，选择Create New Codespace,从GitHub申请一个远程的运行环境，进行运行调试，有一定的免费额度

Git LFS:使用到LFS Server（大文件服务器）
(文件上传大小限制100MB)
本地仓库添加LFS功能：git lfs install
mp4文件需要LFS管理：git lfs track "*.mp4"
最后push即可
注意：Git LFS Data只提供了1GB的空间


参考文档：https://docs.github.com/zh/
查看使用流量额度（针对私有仓库）
setting->billing and plans->plans and usage-> 


回退到最近一次提交的main.py文件记录的位置
git checkout HEAD main.py

如果你需要完全放弃对文件的所有更改（包括已暂存的更改）可以使用 
git reset HEAD main.py 或者 git restore HEAD main.py（在较新的 Git 版本中）

从远程仓库 origin 的 main 分支拉取并合并：
git pull origin main

使用 rebase 方式从远程仓库 origin 的 develop 分支拉取并整合：
git pull --rebase origin develop

从远程仓库 origin 的所有分支拉取并合并：
git pull --all
git push <远程仓库名> <本地分支名>:<远程分支名>

将本地 main 分支推送到远程仓库 origin 的 main 分支：
git push origin main

强制推送本地 feature 分支到远程仓库 origin 的 feature 分支：
git push --force origin feature

删除远程仓库 origin 上的 old-branch 分支：        
git push --delete origin old-branch

git clone ... --depth 10 最近10次提交内容
git submodule update --init --recursive 对于一些很久的分支，更新一下子模块
git graph(左旧右新)
git reset --soft HEAD^ 回退到上一次提交，在暂存区点“-”下放到代码区，点“discard change”放弃代码区的更改
git push -f 强制将提交推送到远端库
git remote -v 获取已配置的远端仓库地址
git fetch origin --depth 10 获取远端仓库最近10个提交
git cherry-pick允许用户从一个分支中选择一个或多个提交，并将这些提交应用到另一个分支上，而无需合并整个分支
git log命令查看提交历史并获取哈希值。例如，git cherry-pick abcdef123456会将哈希值为abcdef123456的提交应用到当前分支
遇到冲突，Git会停止并等待你解决冲突。使用git add命令标记已解决的冲突文件，然后使用git cherry-pick --continue继续cherry-pick过程
如果需要放弃cherry-pick操作，可以使用git cherry-pick --abort

github加速：
uu加速器
watt toolkit（steam++）官方下载地址：https://steampp.net/
github520开源仓库：https://github.com/521xueweihan/GitHub520
Xbox下载加速器：https://github.com/skydevil88/XboxDownload
一行命令上GitHub的仓库： https://github.com/feng2208/github-hosts

微软edge浏览器使用命令（注意文件路径是否一致）：
&"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --host-rules="MAP github.com octocaptcha.com, MAP github.githubassets.com yelp.com, MAP *.githubusercontent.com githubusercontent.com" --host-resolver-rules="MAP octocaptcha.com 20.27.177.113, MAP yelp.com 199.232.240.116, MAP githubusercontent.com 199.232.176.133"

谷歌浏览器使用命令（注意文件路径是否一致）：
&"C:\Program Files\Google\Chrome\Application\chrome.exe" --host-rules="MAP github.com octocaptcha.com, MAP github.githubassets.com yelp.com, MAP *.githubusercontent.com githubusercontent.com" --host-resolver-rules="MAP octocaptcha.com 20.27.177.113, MAP yelp.com 199.232.240.116, MAP githubusercontent.com 199.232.176.133"
