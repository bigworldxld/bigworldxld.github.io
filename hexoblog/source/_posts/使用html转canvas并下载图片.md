---
title: 使用html转canvas并下载图片
img: /images/html2canvas.jpg
categories: front-end
tags:
  - javascript
  - html
abbrlink: 37458
date: 2022-11-07 15:23:13
---

## 单一图片转换

html2canvas.js可以将html代码转换为一个canvas，我们可以直接下载这个canvas来实现将浏览器一部分转为图片并下载，很多业务场景下的切图功能是多次的.

将已知的某个div直接切成图片并下载图片

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
	</head>
	<script type="text/javascript" src="./jquery.min.js"></script>
	<script type="text/javascript" src="./html2canvas.min.js"></script>
	<script type="text/javascript" src="./divToCanvas.js"></script>
	<body>
		<div id="container" style="width: 500px;height: 500px;">
			<div style="width: 300px;height: 300px;background-color: #1890FF;display: inline-block;">
				我是一个中间div
			</div>
			<div style="width: 200px;height: 300px;background-color: #ffaaff;display: inline-block;position: absolute;">
				我是一个右边div
			</div>
			<div style="width: 500px;height: 200px;background-color: #ff557f;">
				我是一个下方div
			</div>
		</div>
		<button onClick="transition()">测试按钮</button>
	</body>
	<script>
	
	</script>
</html>

```

```js
function transition() {
	// 第一个是容器的id，这里html2canvas是通过容器的id自动获取宽高
	// 第二个参数是回调函数，切图完毕后会将canvas的实例直接返回给你（高版本是另外一种）
	html2canvas($('#container'), {
		onrendered:function (canvas){
			// toDataURL方法是将canvas转换成base64位图片，第二个参数是图片质量（0 - 1）
			var dataUrl = canvas.toDataURL('image/png', 1.0);
			// 简单的将转换后的图片下载下来即可
			const link = document.createElement('a')
			link.setAttribute('download', '我是一个div')
			link.href = dataUrl
			link.click()
		}
	})
}
```

![图1](https://img-blog.csdnimg.cn/8c8d306fb2714635b4f9aea9e4d2b097.png)

## 多张图片转换

要生成多个不同，但是同属于一种类型的图片（细微差别，如二维码、文字等不同，其他相同），这样如果直接更改同一个div的话，会由于html2canvas的异步导致生成的图会永远为最后一个，所以我们直接通过代码制作多个不同div并销毁掉即可

```heml
		<!--外部容器 -->
		<div id="container"></div>
		<button onClick="createDiv()">测试按钮</button>

```

```js
//多个切图的话，得从数据层面入手，做多个不同id的div
function transitionMore(data) {
	// 第一个是容器的id，这里传入的data带上参数id
	// 第二个参数是回调函数，切图完毕后会将canvas的实例直接返回给你（高版本是另外一种）,异步操作
	html2canvas($(data.id), {
		onrendered: function(canvas) {
			var dataUrl = canvas.toDataURL('image/png', 1.0);
			const link = document.createElement('a')
			link.setAttribute('download', data.name)
			link.href = dataUrl
			link.click()
			//业务场景下生成的div一般不显示，且为了二次生成的div内部内容正常被替换，得把整个div置空
			document.getElementById(data.id.split('#')[1]).outerHTML = ''
		}
	})
}

//执行一个制作div的方法
function createDiv() {
	// 制作假数据title:标题、desc:简介、color:简介背景颜色、key:一个随机数
	const row = [{
			title: 'Im a title with top',
			desc: '这里是测试生成图片0的内容',
			color: '#1890ff',
			key: Math.random() * 100000000000000000
		},
		{
			title: 'Im a title with top by one',
			desc: '这里是测试图片1的内容码',
			color: '#ffff7f',
			key: Math.random() * 100000000000000000
		},
		{
			title: 'Im a title with top by two',
			desc: '这里是测试图片2的内容',
			color: '#aaffff',
			key: Math.random() * 100000000000000000
		},
		{
			title: 'Im a title with top by three',
			desc: '这里是测试图片3的内容',
			color: '#aaff00',
			key: Math.random() * 100000000000000000
		}
	]
	// 获取外部容器
	const divBox = document.getElementById('container')
	// 异步使用for循环顺序执行正常
	for (let i = 0; i < row.length; i++) {
		// 制作一个div
		let div = document.createElement('div')
		// 将div放入外部容器
		divBox.appendChild(div)
		// 唯一id
		div.id = `id${i}`
		// style主要是为了使多个div重叠，且不出问题
		div.style =`width: 300px;height: 300px;background-color: white;position:absolute;z-index: -1`
		//创造一个头部div
		let titleDiv = document.createElement('div')
		div.appendChild(titleDiv)
		titleDiv.style = `margin-top: 8px;width: 300px;height:50px;text-align:center;font-size:18px`
		titleDiv.innerHTML = row[i].title
		// 创造一个中间div
		let bodyDiv = document.createElement('div')
		div.appendChild(bodyDiv)
		bodyDiv.style = `margin-top: 8px;width: 300px;height:150px;text-align:center;font-size:16px;background-color:${row[i].color}`
		bodyDiv.innerHTML = row[i].desc
		// 创造一个key值div
		let keyDiv = document.createElement('div')
		div.appendChild(keyDiv)
		keyDiv.style = `margin-top: 8px;width: 300px;height:40px;text-align:center;font-size:14px;background-color:${row[i].color}`
		keyDiv.innerHTML = row[i].key
		// 把需要传的值做好
		const data = {
			id: `#id${i}`,
			name: `${row[i].key}.png`,
		}
		// 执行多元下载方法
		transitionMore(data)
	}
}
```

图2
![图2](https://img-blog.csdnimg.cn/2c9b4a49a5c448f4accd8e8666e98e2a.png)