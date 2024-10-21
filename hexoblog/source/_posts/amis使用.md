---
title: amis使用
categories: front-end
img: /images/amis.jpg
tags: amis
abbrlink: 3751
date: 2024-07-13 18:05:56
---
## amis介绍

amis 是一个低代码前端框架，它使用 JSON 配置来生成页面,类似 Vue/jQuery外链代码就能使用

亮点
用 JSON 写页面有什么好处

+ 不需要懂前端
+ 不受前端技术更新的影响
+ 享受 amis 的不断升级
+ 可以 完全 使用 可视化页面编辑器 来制作页面
+ 支持扩展
+ 容器支持无限级嵌套

amis 不适合做什么？

+ 大量定制 UI
+ 极为复杂或特殊的交互

## 上手

amis 有两种使用方法：
JS SDK，可以用在任意页面中
React，可以用在 React 项目中

SDK 版本适合对前端或 React 不了解的开发者，它不依赖 npm 及 webpack

[文档介绍](https://aisuda.bce.baidu.com/amis/zh-CN/docs/start/getting-started)

## SDK

下载方式：
github 的 [releases](https://github.com/baidu/amis/releases)，文件是 sdk.tar.gz。
使用 npm i amis 来下载，在 node_modules\amis\sdk 目录里就能找到

```html
    <link rel="stylesheet" href="sdk.css" />
    <link rel="stylesheet" href="helper.css" />
    <link rel="stylesheet" href="iconfont.css" />
    <!-- 这是默认主题所需的，如果是其他主题则不需要 -->
    <!-- 从 1.1.0 开始 sdk.css 将不支持 IE 11，如果要支持 IE11 请引用这个 css，并把前面那个删了 -->
    <!-- <link rel="stylesheet" href="sdk-ie11.css" /> -->
```

### 整体架构

```js
amis.embed(
  '#root',
  {
    // amis schema
  },
  {
    // 这里是初始值 props
  },
  // 注意是第四个参数
  {
    theme: 'antd'
  }
);
```

切换主题

jssdk 版本默认使用 `sdk.css` 即云舍主题，如果你想使用仿 Antd，请将 css 引用改成 `antd.css`。同时 js 渲染地方第四个参数传入 `theme` 属性。如上

### 初始值

可以通过 props 里的 data 属性来赋予 amis 顶层数据域的值

```js
let amis = amisRequire('amis/embed');
let amisJSON = {
  type: 'page',
  body: {
    type: 'tpl',
    tpl: '${myData}'
  }
};
let amisScoped = amis.embed('#root', amisJSON, {
  data: {
    myData: 'amis'
  },
  context: {
    amisUser: {
      id: 1,
      name: 'test user'
    }
  }
});
```

3.1.0 开始可以传入 context 数据，无论哪层都可以使用到这个里面的数据。

### 控制 amis 的行为

`amis.embed` 函数还支持以下配置项来控制 amis 的行为，比如在 fetcher 的时候加入自己的处理逻辑

```js
let amisScoped = amis.embed(
  '#root',
  amisJSON,
  {
    // 这里是初始 props，一般不用传。
    // locale: 'en-US' // props 中可以设置语言，默认是中文
  },
  {
    // 下面是一些可选的外部控制函数
    // 在 sdk 中可以不传，用来实现 ajax 请求，但在 npm 中这是必须提供的
    // fetcher: (url, method, data, config) => {},
    // 全局 api 请求适配器
    // 另外在 amis 配置项中的 api 也可以配置适配器，针对某个特定接口单独处理。
    //
    // requestAdaptor(api) {
    //   // 支持异步，可以通过 api.mockResponse 来设置返回结果，跳过真正的请求发送
    //   // 此功能自定义 fetcher 的话会失效
    //   // api.context 中包含发送请求前的上下文信息
    //   return api;
    // }
    //
    // 全局 api 适配器。
    // 另外在 amis 配置项中的 api 也可以配置适配器，针对某个特定接口单独处理。
    // responseAdaptor(api, payload, query, request, response) {
    //   return payload;
    // }
    //
    // 用来接管页面跳转，比如用 location.href 或 window.open，或者自己实现 amis 配置更新
    // jumpTo: to => { location.href = to; },
    //
    // 用来实现地址栏更新
    // updateLocation: (to, replace) => {},
    //
    // 用来判断是否目标地址当前地址。
    // isCurrentUrl: url => {},
    //
    // 用来配置弹窗等组件的挂载位置
    // getModalContainer: () => document.getElementsByTagName('body')[0],
    //
    // 用来实现复制到剪切板
    // copy: content => {},
    //
    // 用来实现通知
    // notify: (type, msg) => {},
    //
    // 用来实现提示
    // alert: content => {},
    //
    // 用来实现确认框。
    // confirm: content => {},
    //
    // 主题，默认是 default，还可以设置成 cxd 或 dark，但记得引用它们的 css，比如 sdk 目录下的 cxd.css
    // theme: 'cxd'
    //
    // 用来实现用户行为跟踪，详细请查看左侧高级中的说明
    // tracker: (eventTracker) => {},
    //
    // Toast提示弹出位置，默认为'top-center'
    // toastPosition: 'top-right' | 'top-center' | 'top-left' | 'bottom-center' | 'bottom-left' | 'bottom-right' | 'center'
  }
);
```

同时返回的 `amisScoped` 对象可以获取到 amis 渲染的内部信息

`getComponentByName(name)` 用于获取渲染出来的组件

通过 `amisScoped.getComponentByName('page1.form1').getValues()` 来获取到所有表单的值，需要注意 page 和 form 都需要有 name 属性。

通过 `amisScoped.getComponentByName('page1.form1').setValues({'name1': 'othername'})` 来修改表单中的值

### 调用 amis 动作

通过`amisScoped.doAction(actions, ctx)`来调用 amis 中的通用动作和目标组件的动作。了解事件动作机制可以查看[事件动作](https://aisuda.bce.baidu.com/amis/zh-CN/docs/concepts/event-action)。参数说明如下：

- `actions`：动作列表，支持执行单个或多个动作
- `ctx`：上下文，它可以为动作配置补充上下文数据

可以通过 amisScoped 对象的 updateProps 方法来更新下发到 amis 的属性。

可以通过 amisScoped 对象的 udpateSchema 方法来更新更新内容配置

### 多页模式

默认 amis 渲染是单页模式，如果想实现多页应用，请使用 [app 渲染器](https://aisuda.bce.baidu.com/amis/zh-CN/components/app)。

### Hash 路由

默认 JSSDK 不是 hash 路由，如果你想改成 hash 路由模式，请查看此处代码实现。只需要修改 `env.isCurrentUrl`、`env.jumpTo` 和 `env.updateLocation` 这几个方法，并在路由切换的时候，通过 amisScoped 对象的 `updateProps` 方法，更新 `location` 属性即可。

[参考](https://github.com/baidu/amis/blob/master/examples/app/index.jsx)

### 销毁

如果是单页应用，在离开当前页面的时候通常需要销毁实例，可以通过 unmount 方法来完成。

```ts
amisScoped.unmount();
```

## vue

可以基于 SDK 版本封装成 component 供 vue 使用，具体请参考示例：https://github.com/aisuda/vue2-amis-demo

## react

初始项目请参考 https://github.com/aisuda/amis-react-starter。

如果在已有项目中，React 版本需要是 `>=16.8.6`，mobx 需要 `^4.5.0`。

amis 1.6.5 及以上版本支持 React 1

## 示例

文件：sdk:包含amis的样式文件

hello.html:跟sdk在同一目录下，引入sdk下的amis文件，编辑json即可

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="UTF-8" />
    <title>amis demo</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, maximum-scale=1"
    />
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
    <link rel="stylesheet" href="./sdk/sdk.css" />
    <link rel="stylesheet" href="./sdk/helper.css" />
    <link rel="stylesheet" href="./sdk/iconfont.css" /> 
    <!-- 这是默认主题所需的，如果是其他主题则不需要  -->
    <!-- 从 1.1.0 开始 sdk.css 将不支持 IE 11，如果要支持 IE11 请引用这个 css，并把前面那个删了 -->
    <!-- <link rel="stylesheet" href="sdk-ie11.css" /> -->
    <!-- 不过 amis 开发团队几乎没测试过 IE 11 下的效果，所以可能有细节功能用不了，如果发现请报 issue -->
    <style>
      html,
      body,
      .app-wrapper {
        position: relative;
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
      }
    </style>
  </head>
  <body>
    <div id="root" class="app-wrapper"></div>
    <script src="./sdk/sdk.js"></script>
    <script type="text/javascript">
      (function () {
        let amis = amisRequire('amis/embed');
        // 通过替换下面这个配置来生成不同页面
        let amisJSON = {
  "type": "page",
  "toolbar": [
    {
      "type": "form",
      "panelClassName": "mb-0",
      "title": "",
      "body": [
        {
          "type": "select",
          "label": "区域",
          "name": "businessLineId",
          "selectFirst": true,
          "mode": "inline",
          "options": [
            "北京",
            "上海"
          ],
          "checkAll": false
        },
        {
          "label": "时间范围",
          "type": "input-date-range",
          "name": "dateRange",
          "inline": true,
          "value": "-1month,+0month",
          "inputFormat": "YYYY-MM-DD",
          "format": "YYYY-MM-DD",
          "closeOnSelect": true,
          "clearable": false
        }
      ],
      "actions": [],
      "mode": "inline",
      "target": "mainPage",
      "submitOnChange": true,
      "submitOnInit": true
    }
  ],
  "body": [
    {
      "type": "grid",
      "columns": [
        {
          "type": "panel",
          "className": "h-full",
          "body": {
            "type": "tabs",
            "tabs": [
              {
                "title": "消费趋势",
                "tab": [
                  {
                    "type": "chart",
                    "config": {
                      "title": {
                        "text": "消费趋势"
                      },
                      "tooltip": {},
                      "xAxis": {
                        "type": "category",
                        "boundaryGap": false,
                        "data": [
                          "一月",
                          "二月",
                          "三月",
                          "四月",
                          "五月",
                          "六月"
                        ]
                      },
                      "yAxis": {},
                      "series": [
                        {
                          "name": "销量",
                          "type": "line",
                          "areaStyle": {
                            "color": {
                              "type": "linear",
                              "x": 0,
                              "y": 0,
                              "x2": 0,
                              "y2": 1,
                              "colorStops": [
                                {
                                  "offset": 0,
                                  "color": "rgba(84, 112, 197, 1)"
                                },
                                {
                                  "offset": 1,
                                  "color": "rgba(84, 112, 197, 0)"
                                }
                              ],
                              "global": false
                            }
                          },
                          "data": [
                            5,
                            20,
                            36,
                            10,
                            10,
                            20
                          ]
                        }
                      ]
                    }
                  }
                ]
              },
              {
                "title": "账户余额",
                "tab": "0"
              }
            ]
          }
        },
        {
          "type": "panel",
          "className": "h-full",
          "body": [
            {
              "type": "chart",
              "config": {
                "title": {
                  "text": "使用资源占比"
                },
                "series": [
                  {
                    "type": "pie",
                    "data": [
                      {
                        "name": "BOS",
                        "value": 70
                      },
                      {
                        "name": "CDN",
                        "value": 68
                      },
                      {
                        "name": "BCC",
                        "value": 48
                      },
                      {
                        "name": "DCC",
                        "value": 40
                      },
                      {
                        "name": "RDS",
                        "value": 32
                      }
                    ]
                  }
                ]
              }
            }
          ]
        }
      ]
    },
    {
      "type": "crud",
      "className": "m-t-sm",
      "api": "/amis/api/mock2/sample",
      "columns": [
        {
          "name": "id",
          "label": "ID"
        },
        {
          "name": "engine",
          "label": "Rendering engine"
        },
        {
          "name": "browser",
          "label": "Browser"
        },
        {
          "name": "platform",
          "label": "Platform(s)"
        },
        {
          "name": "version",
          "label": "Engine version"
        },
        {
          "name": "grade",
          "label": "CSS grade"
        }
      ]
    }
  ]
};
        let amisScoped = amis.embed('#root', amisJSON);
      })();
    </script>
  </body>
</html>
```


