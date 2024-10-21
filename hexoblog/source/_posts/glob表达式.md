---
title: glob表达式
categories: tools
img: /images/glob.jpg
tags: glob
abbrlink: 58369
date: 2024-08-10 21:07:42
---

## glob表达式

(glob expressions)通配符：

*匹配文件路径中的0个或多个字符，但*不会匹配路径分隔符，除非路径分隔符出现在末尾。

** 匹配路径中的0个或多个目录及其子目录，如果出现在末尾，也能匹配文件。

? 匹配文件路径中的一个字符(不会匹配路径分隔符)。

[...] 匹配方括号中出现的字符中的任意一个，当方括号中第一个字符为 ·^ 或 ! 时，则表示不匹配方括号中出现的其他字符中的任意一个。
!(pattern|pattern|pattern) 匹配任何与括号中给定的任一参数都不匹配的。

?(pattern|pattern|pattern) 匹配括号中给定的任一参数0次或1次。

+(pattern|pattern|pattern) 匹配括号中给定的任一参数1次或多次。

*(pattern|pattern|pattern) 匹配括号中给定的任一参数0次或多次。

@(pattern|pattern|pattern) 匹配括号中给定的任一参数1次。

## **glob实例**

* 能匹配 a.js , x.y , abc , abc/ ，但不能匹配 a/b.js

*.* 能匹配 a.js , style.css , a.b , x.y

*/*/*.js 能匹配 a/b/c.js , x/y/z.js ，不能匹配 a/b.js , a/b/c/d.js

** 能匹配 abc , a/b.js , a/b/c.js , x/y/z , x/y/z/a.b ，能用来匹配所有的目录和文件

**/*.js 能匹配 foo.js , a/foo.js , a/b/foo.js , a/b/c/foo.js

a/**/z 能匹配 a/z , a/b/z , a/b/c/z , a/d/g/h/j/k/z

a/**b/z 能匹配 a/b/z , a/sb/z ，但不能匹配 a/x/sb/z ，因为只有单 ** 单独出现才能匹配多级目录

?.js 能匹配 a.js , b.js , c.js

a?? 能匹配 a.b , abc ，但不能匹配 ab/ ，因为它不会匹配路径分隔符

[xyz].js 只能匹配 x.js , y.js , z.js ，不会匹配 xy.js , xyz.js 等，整个中括号只代表一个字符

[^xyz].js 能匹配 a.js , b.js , c.js 等，不能匹配 x.js , y.js , z.js

当有多种匹配模式时可以使用数组：

// 使用数组的方式来匹配多种文件
gulp.src([ 'js/*.min.js', 'sass/*.min.css' ])
使用数组的方式还有一个好处就是可以很方便的使用排除模式，在数组中的单个匹配模式前加上 ! 即是排除模式，它会在匹配的结果中排除这个匹配，要注意一点的是不能在数组中的第一个元素中使用排除模式：

// 使用数组的方式来匹配多种文件
gulp.src(['*.js','!b*.js']) // 匹配所有js文件，但排除掉以b开头的js文件
gulp.src(['!b*.js',*.js]) // 不会排除任何文件，因为排除模式不能出现在数组的第一个元素中

还可以使用展开模式。展开模式以花括号作为定界符，根据它里面的内容，会展开为多个模式，最后匹配的结果为所有展开的模式相加起来得到的结果。展开的例子如下：

a{b,c}d 会展开为 abd,acd

a{b,}c 会展开为 abc,ac

a{0..3}d 会展开为 a0d , a1d , a2d , a3d

a{b,c{d,e}f}g 会展开为 abg , acdfg , acefg

a{b,c}d{e,f}g 会展开为 abdeg , acdeg , abdeg , abdfg

## hexo目录

| 设置           | 描述                                                         | 默认值           |
| :------------- | :----------------------------------------------------------- | :--------------- |
| `source_dir`   | Source folder. Where your content is stored                  | `source`         |
| `public_dir`   | Public folder. Where the static site will be generated       | `public`         |
| `tag_dir`      | 标签文件夹                                                   | `tags`           |
| `archive_dir`  | 归档文件夹                                                   | `archives`       |
| `category_dir` | 分类文件夹                                                   | `categories`     |
| `code_dir`     | Include code 文件夹，`source_dir` 下的子目录                 | `downloads/code` |
| `i18n_dir`     | 国际化（i18n）文件夹                                         | `:lang`          |
| `skip_render`  | 匹配到的文件将会被不做改动地复制到 `public` 目录中。 您可使用 [glob 表达式](https://github.com/micromatch/micromatch#extended-globbing)来匹配路径。 |                  |

例如：

```ini
skip_render: "mypage/**/*"
# 将会直接将 `source/mypage/index.html` 和 `source/mypage/code.js` 不做改动地输出到 'public' 目录
# 你也可以用这种方法来跳过对指定文章文件的渲染
skip_render: "_posts/test-post.md"
# 这将会忽略对 'test-post.md' 的渲染

## This also can be used to exclude posts,
skip_render: "_posts/test-post.md"
# will ignore the `source/_posts/test-post.md`.
```

## hexo包括或不包括目录和文件

使用以下选项可明确处理或忽略某些文件/文件夹。 可以使用 [glob 表达式](https://github.com/micromatch/micromatch#extended-globbing) 进行路径匹配。

`include` 和 `exclude` 选项只会应用到 `source/` ，而 `ignore` 选项会应用到所有文件夹.

| 设置      | 描述                                                      |
| :-------- | :-------------------------------------------------------- |
| `include` | 包含隐藏文件（包括名称以下划线开头的文件/文件夹，* 除外） |
| `exclude` | 排除文件或文件夹                                          |
| `ignore`  | 忽略文件/文件夹                                           |

例如：

```ini
# 处理或不处理目录或文件
include:
  - ".nojekyll"
  # 处理 'source/css/_typing.css'
  - "css/_typing.css"
  # 处理 'source/_css/' 中的任何文件，但不包括子目录及其其中的文件。
  - "_css/*"
  # 处理 'source/_css/' 中的任何文件和子目录下的任何文件。
  - "_css/**/*"

exclude:
  # 不处理 'source/js/test.js'。
  - "js/test.js"
  # 不处理 'source/js/' 中的文件、但包括子目录下的所有目录和文件。
  - "js/*"
  # 不处理 'source/js/' 中的文件和子目录下的任何文件。
  - "js/**/*"
  # 不处理 'source/js/' 目录下的所有文件名以 'test' 开头的文件，但包括其它文件和子目录下的单文件。
  - "js/test*"
  # 不处理 'source/js/' 及其子目录中任何以 'test' 开头的文件。
  - "js/**/test*"
  # 不要用 exclude 来忽略 'source/_posts/' 中的文件。
  # 你应该使用 'skip_render'。 或者在要忽略的文件的文件名之前加一个下划线 '_'。
  # - "_posts/hello-world.md" # 在这里配置是没有用的。

ignore:
  # 忽略任何一个名叫 'foo' 的文件夹。
  - "**/foo"
  # 只忽略 'themes/' 下的 'foo' 文件夹。
  - "**/themes/*/foo"
  # 对 'themes/' 目录下的每个文件夹中忽略名叫 'foo' 的子文件夹。
  - "**/themes/**/foo"
```

列表中的每一项都必须用单引号或双引号包裹起来。

`include` 和 `exclude` 并不适用于 `themes/` 目录下的文件。 如果需要忽略 `themes/` 目录下的部分文件或文件夹，可以使用 `ignore` 或在文件名之前添加下划线 `_`。

`source/_posts` 文件夹是一个例外，但该文件夹下任何名称以 `_` 开头的文件或文件夹仍会被忽略。 不建议在该文件夹中使用 `include` 规则。



[hexo](https://hexo.io/docs/)
[glob](https://www.jianshu.com/p/91eb8d81da64)


