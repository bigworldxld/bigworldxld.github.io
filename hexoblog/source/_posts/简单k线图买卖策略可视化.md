---
title: 简单k线图买卖策略可视化
date: 2022-09-05 21:53:51
img: /images/量化金融.jpg
tags: 
	- finance
	- pyecharts
---

记录最近用到的金融买卖策略框架，在给定时间段k线图上叠加显示买点、卖点、收益等信息，用pyecharts可视化标注符合业务逻辑的买点和买点信息以及收益率。

## 前提

事先可了解[pyecharts官网](https://gallery.pyecharts.org/#/)

由于在[K 线图 Candlestick](https://gallery.pyecharts.org/#/Candlestick/README)上，要标注买卖点，收益率等等信息，是不连续的，需要再上面叠加[涟漪散点图 EffectScatter](https://gallery.pyecharts.org/#/EffectScatter/README),使用overlap方法叠加两种图层，详细代码如下：

```python
.add_xaxis(buy_date)
            .add_yaxis("买入操作", buy_price, symbol=SymbolType.ARROW,tooltip_opts=opts.TooltipOpts( formatter="{a} <br/>时间：{b}<br/>详情：{c}"))
            .add_xaxis(sell_date)
            .add_yaxis("卖出操作",sell_price,tooltip_opts=opts.TooltipOpts( formatter="{a} <br/>时间：{b}<br/>详情：{c}"))
```

散点图中，其中{a}{b}{c},分别代表，系列名，数据名，数据名+数据值

此外还要添加MarkPointItem参数去遍历输出每一个卖点操作的收益率

，具体关键代码如下：

```python
.set_series_opts(
            markpoint_opts=opts.MarkPointOpts(
                data=[
                    opts.MarkPointItem(coord=[sell_date[i], sell_price[i]], name="收益率(%)", value=reward[i]) for i in range(len(sell_date))
                    # opts.MarkPointItem(type_="min", name="最小值"),
                ],
            )
        )
```

## 效果

买卖点信息展示



{% asset_img 买卖时间.png This is an example image %}

收益率展示

{% asset_img 收益率.png This is an example image %}