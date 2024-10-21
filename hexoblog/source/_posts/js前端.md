---
title: js前端
categories: front-end
img: /images/js前端.jpg
tags:
  - js
  - css
  - html
abbrlink: 28075
date: 2024-05-01 14:18:41
---

## Js 前端

### 内置对象

浏览器内置**window**对象

浏览器里面的js引擎内置了js 变量 `window` 

**history**对应 就是 当前浏览器的 浏览历史对象

// 返回上次浏览地址，等于点击浏览器的返回按钮
history.back();

// 也是 返回上次浏览地址
history.go(-1);

// 前进到返回前面的地址，等于点击浏览器的前进按钮
history.forward();

**location**对应 就是 当前浏览器的 地址对象

// 设置当前浏览地址
location.href = '/main.html'

// 重新加载网页，等于刷新网页
location.reload()


**localStorage**可以看作一个 本地键值对数据库，

它是当前网站的 前端存储，关闭浏览器窗口后也不会丢失数据

// 存入/修改 数据
localStorage.setItem('myCat', 'Tom');

// 获取数据
const cat = localStorage.getItem('myCat');

// 清除localStorage 数据
localStorage.clear();

**sessionStorage** 也是一个 本地键值对数据库，

它也是当前网站的 前端存储

和 `localStorage` 的差别是， 它的数据是和session相关的。

关闭浏览器窗口后，数据就会丢掉

// 清除 sessionStorage 数据
sessionStorage.clear();

// 存入/修改 数据
sessionStorage.setItem('myCat', 'Tom');

// 获取数据
const cat = sessionStorage.getItem('myCat');

### CSS定位

方法1：在浏览器开发者模式的elements中，Ctrl+F搜索栏输入css表达式
方法2：在浏览器开发者模式的console中，按如下格式验证表达式

```txt
$("css表达式")   # 注：表达式中存在引号，则使用单引号，'$'可更换为$$
```

css选择器语法

基础选择器

| 选择器      | 格式               | 示例             | 示例说明                         |
| ----------- | ------------------ | ---------------- | -------------------------------- |
| 选择全部    | *                  | *                | 选择全部元素                     |
| 标签选择器  | html标签           | p                | 选择所有\<p>元素                 |
| ID选择器    | #id属性值          | #su              | 选择所有id='su'的元素            |
| 类选择器    | .class属性值       | .s_btn           | 选择所有class='s_btn'的元素      |
| 属性选择器1 | [属性名]           | [type]           | 选择所有带type属性的元素         |
| 属性选择器2 | [属性名='属性值']  | [type="submit"]  | 选择所有type="submit"的元素      |
| 属性选择器3 | [属性名*='属性值'] | [type*="submit"] | 选择所有type包含"submit"的元素   |
| 属性选择器4 | [属性名^='属性值'] | [type^="submit"] | 选择所有type以"submit"开头的元素 |

组合选择器

| 选择器       | 格式           | 示例     | 示例说明                       |
| ------------ | -------------- | -------- | ------------------------------ |
| 标签指定属性 | 标签加属性描述 | input#su | 选择所有id='su'的\<input>元素  |
| 并集         | 元素1,元素2    | div,p    | 选择所有\<div>和\<p>元素       |
| 父子         | 元素1>元素2    | div>p    | 选择所有父级是\<div>的\<p>元素 |
| 后代         | 元素1 元素2    | div p    | 选择\<div>中的所有\<p>元素     |
| 相邻         | 元素1+元素2    | div+p    | 选择\<div>同级后的相邻\<p>元素 |
| 同级         | 元素1~元素2    | div~p    | 选择\<div>同级后的所有\<p>元素 |

伪属性选择器

伪属性选择器是指元素在html中实际并不存在该属性，是由css定义的拓展描述属性

| 选择器         | 格式                 | 示例                  | 示例说明                                             |
| -------------- | -------------------- | --------------------- | ---------------------------------------------------- |
| 唯一子元素     | :only-child          | p:only-child          | 选择所有\<p>元素且该元素是其父级的唯一一个元素       |
| 第一子元素     | :first-child         | p:first-child         | 选择所有\<p>元素且该元素是其父级的第一个元素         |
| 最后子元素     | :last-child          | p:last-child          | 选择所有\<p>元素且该元素是其父级的最后一个子元素     |
| 顺序选择器     | :nth-child(n)        | p:nth-child(2)        | 选择所有\<p>元素且该元素是其父级的第二个子元素       |
| 顺序类型选择器 | :nth-of-type(n)      | p:nth-of-type(2)      | 选择所有\<p>元素且该元素是其父级的第二个\<p>元素     |
| 倒序选择器     | :nth-last-child(n)   | p:nth-last-child(2)   | 选择所有\<p>元素且该元素是其父级的倒数第二个子元素   |
| 倒序类型选择器 | :nth-last-of-type(n) | p:nth-last-of-type(2) | 选择所有\<p>元素且该元素是其父级的倒数第二个\<p>元素 |

```txt
表达式4：input:focus
表达式5：input:enabled
表达式6：input:checked
表达式7: input:not([id])
表达式4表示查找当前获取焦点的input页面元素
表达式5表示查找可操作的input页面元素
表达式6表示查找处于勾选状态的checkbox页面元素
表达式7表示查找所有无id属性的input页面元素
```



### DOM 文档对象模型

在浏览器内部（内存中）创建一个文档对象模型 (document object model) 简称 `DOM` 对象 ， 对应 树状的 文档结构

只包含一段文本字符串的节点是 `文本节点（Text node）`

任何 DOM对象都是一个 节点 ， 包括 元素节点、 注释节点、 文本节点、属性节点

节点类型	节点值

ELEMENT_NODE	1

ATTRIBUTE_NODE	2

TEXT_NODE	3

CDATA_SECTION_NODE	4

PROCESSING_INSTRUCTION_NODE	7

COMMENT_NODE	8

DOCUMENT_NODE	9

DOCUMENT_TYPE_NODE	10

DOCUMENT_FRAGMENT_NODE	11

F12打开 DevTools Console，执行

```javascript
var link = document.querySelector('a')
```

就可以通过一个css选择表达式 获取到 标签名为 `a` 的第一个 的DOM元素对象

可以通过该对象的属性，来改变对象的内部内容。

```javascript
link.textContent = '白月'
link.href = 'https://www.byhy.net'
```

// 根据id属性获取元素对象
document.getElementById('password')

// 返回 body元素对象
document.body

// 返回 head元素对象
document.head

// 返回 当前窗口的标题
document.title

// 返回 当前窗口的url地址
document.URL

### 获取元素对象

`querySelector` 、 `querySelectorAll` 方法，参数是 CSS 选择表达式

//  tag名选择
document.querySelector('textarea')

//  id选择
document.querySelector('#result')
// 还可以 使用 getElementById
document.getElementById('result')

//  范围选择
document.querySelector('div pre')

// 返回数组
document.querySelectorAll('p')

// 数组里面的第2个元素
document.querySelectorAll('p')[1]

// 遍历
ps = document.querySelectorAll('p')
for (let p of ps) { 
  console.log(p) 
}

### 事件和处理

在 id 为 go 的元素上，使用 `addEventListener` 方法，注册了点击的事件， 当这种事件发生的时候，就回调执行函数 salaryStats 来处理这个事件,

```javascript
  <script>    
  function salaryStats(event){
    alert('执行salaryStats') 
  }
  // 注册事件回调函数
  document.querySelector('#go').addEventListener("click", salaryStats );
  </script>
```

官方文档：https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API#html_dom_api_interfaces

```text
a 对应的DOM对象类型是 `HTMLAnchorElement
p 对应的DOM对象类型是 `HTMLParagraphElement
textarea 对应的DOM对象类型是 `HTMLTextAreaElement
button 对应的DOM对象类型是 `HTMLButtonElement
```

textarea文本框里面的内容 对应的是属性 `value` ，如下所示

document.querySelector('#salary').value

### 改变网页内容

给某个节点添加新节点，比如

```JavaScript
document.querySelector('div')
  .insertAdjacentHTML("beforeend", "<p id='test1'>薪资20K以上的有：xxx、yyy</p>")
```

`insertAdjacentHTML` 方法就是在DOM 对象的某个位置插入新的html节点。

第2个参数 就是要插入的节点对应的HTML 内容

第1个参数 指定插入节点的位置，这里 `beforeend` 就是 指 插入节点作为 当前节点的 最后一个子节点

要删除元素，可以 先获取该元素，然后调用remove方法

```JavaScript
document.getElementById("test1").remove()
```

还可以这样删除

```Javascript
document.getElementById("myDiv").outerHTML="";
```

// 改变id为result元素的文本颜色为红色 

document.querySelector('#result').style.color = 'red' 

// 改变button按钮文本 

let addOneBtn = document.querySelector('button') 

addOneBtn.innerText = addOneBtn.innerText === '隐藏' ? '添加' : '隐藏';

控制元素是否显示，可以根据元素的CSS样式里面的 `display` 属性 ，

当 `display` 的值为 `none` ，元素不显示

比如

```JavaScript
let pnEle = document.querySelector('.paginator')
if(pd.pagecount === 0){
  pnEle.style.display = 'none'
  return
}
```

### 继承关系

一个链接a元素对象 它的类型是 HTMLAnchorElement ，对应的继承关系如下

```txt
EventTarget <- Node <- Element <- HTMLElement <- HTMLAnchorElement
```

而 img 对应的 HTMLImageElement 对应的继承关系如下

```txt
EventTarget <- Node <- Element <- HTMLElement <- HTMLImageElement
```

每个html元素 前面的继承链都是

### EventTarget

[EventTarget](https://developer.mozilla.org//en-US/docs/Web/API/EventTarget)类型提供了如下方法

addEventListener()

用来注册处理 该对象的事件 的处理函数

```javascript
<button id='target'>测试按钮</button>
<script>     
  let targetBtn = document.querySelector('#target')

  // 注册事件回调函数
  targetBtn.addEventListener(
    "click",
    ()=>{alert('测试按钮被点击了')}
  )
</script>
```

removeEventListener()

这是 类型 EventTarget 的方法，用来取消 注册处理 该对象的事件 的处理函数

```javascript
<button id='target'>测试按钮</button>
<button id='cancel'>取消事件处理</button>
<script>     
  let targetBtn = document.querySelector('#target')
  let cancelBtn = document.querySelector('#cancel')
  function targetClicked() {
    alert('测试按钮被点击了')
  }
  targetBtn.addEventListener("click",targetClicked)
  cancelBtn.addEventListener("click", 
    ()=>{
      alert('取消测试按钮点击处理')
      targetBtn.removeEventListener("click",targetClicked)
    }
  );
</script>
```

### [Node](https://developer.mozilla.org//en-US/docs/Web/API/Node)类型

获取所有子节点

`childNodes`属性可以 获取元素的所有子节点对象

textContent 属性 用来 获取 元素内部所有的文本内容，包括不可见的部分

`parentElement` 该属性返回这个Node的父HTML元素

###  [Document](https://developer.mozilla.org/en-US/docs/Web/API/Document) 

继承关系如下

```
EventTarget <- Node <- Document
```

所以 `EventTarget` 和 `Node` 的属性方法，比如 `addEventListener()` , `childNodes` 等等 ， 对 document都是适用的

`createElement` 方法可以产生元素对象。

产生的元素对象可以通过后续讲解的方法添加到DOM中

### Element

`id` 属性可以获取/设置 该元素的id属性

`children` 属性可以获取该元素的所有子元素

children属性返回的是对象是 [HTMLCollection](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCollection) 类型，

这个类型 功能类似 数组，里面存放的是 html元素对象，

可以使用 `for of` 遍历， 也可以使用索引获取其中的元素

for (let child of source.children)  

	console.log(child.id)

 source.children[1]

`innerHTML` 可以用来获取一个元素对象 `内部` HTML文本

`outerHTML` 可以用来获取一个元素对象 `全部` HTML文本

`closest()` 方法获取符合条件的 `最接近` 的上级元素

```javascript
<div id='div1' class="type1 type2">
  <div id='div2' class="type1 type2">
    <div id='div3' class="type3">
      <div id='text'>
        内部文本
      </div> 
    </div> 
  </div> 
</div>
<script>
const inner = document.getElementById("text");
console.log('得到元素的id是', inner.closest('.type2').id)
</script>
```

得到元素的id是 div2

如果自身也匹配，则返回本元素自身

Element方法中，有好几个用来添加节点。

after() / before() / append()

`after` 是给当前元素节点，添加一些 `后续弟弟节点` ，

before 是给当前元素节点，添加一些 前置哥哥节点

append 是给当前元素节点，添加一些 子节点

参数可以是1个或者 多个节点对象

参数可以是 元素对象 （添加后成为html元素） 或者 字符串（添加后成为文本节点）

使用 after() / before() / append()，必须先使用 `document.createElement()` 创建一个元素对象

然后再调用新对象的方法，创建内容

```javascript
<body>
  <p id='target'>原来的元素</p>
</body>

<script>
  let p = document.getElementById('target')
  const new1 = document.createElement("p");
  new1.textContent = '新元素1';
  const new2 = document.createElement("p");
  new2.textContent = '新元素2';
  p.append(new1,new2,'abc');
</script>
```

可以使用 `insertAdjacentHTML` 方法直接写元素对应的html文本内容

`insertAdjacentHTML` 方法就是在DOM 对象的 `某个位置` 插入新的html节点。

- 第2个参数 就是要插入的节点对应的HTML文本

- 第1个参数 指定插入节点的位置

- `afterbegin`

  插入内容 作为 当前节点的 `第一个子节点`

- `beforeend`

  插入内容 作为 当前节点的 `最后一个子节点`

- `beforebegin`

  插入内容 作为 当前节点的 `前一个哥哥节点`

- `afterend`

  插入内容 作为 当前节点的 `下一个弟弟节点`

`remove()` 方法用来删除元素自身

document.getElementById('target1').outerHTML="";

使用 `replaceWith` 方法替换元素

```javascript
<p id='p1'>原来的内容</p>
<script>
  let p = document.getElementById('p1')
  const span = document.createElement("span");
  span.textContent = '新内容';
  p.replaceWith(span);
</script>
```

`getAttribute()` 可以用来获取元素的属性， `getAttribute()` 返回值都是字符串的形式

也可以直接通过 `元素对象.属性` 的方式获取元素的属性

`setAttribute()` 可以用来设置元素的属性

`removeAttribute()` 用来删除元素的属性

`className` 可以用来获取和设置元素的class属性

之所以不直接用 `class` 作为属性名，是为了避免和 js 的关键字 class 冲突

`classList` 属性对应一个 [DOMTokenList](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList) 类型， 有一些方便操作属性值的方法，

其中比较常用的是

- `contains` 方法

返回true和false表示是否包含指定元素

- `remove` 方法

删除某个属性，其它属性不动

- `add` 方法

添加某个属性

```javascript
<head>
  <style>
    .hide {
      display:none
    }
  </style>
</head>
<body>
<button>隐藏/显示</button>
<div id='div1' class="type1">
   元素内容
</div>
<script>
  // 注册事件回调函数
  document.querySelector('button').addEventListener(
    "click", 
    function (){
      const ele = document.getElementById("div1"); 
      if (ele.classList.contains('hide')){
        ele.classList.remove("hide");
      }
      else{
        ele.classList.add("hide");
      }
    }
  );
</script>
</body>
```

区别是：

innerText 展示的是该元素对应的可以呈现在界面上的文本内容

textContent 展示的是元素内部所有的文本内容，包括不可见的部分

style属性的值是 [CSSStyleDeclaration](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration) 类型， 类似 Object类型， 可以通过这个对象的属性来 获取、设置 元素的样式

focus方法可以让该元素获取输入焦点，前提是这种元素可以获取输入焦点

### `input`

 对应的DOM对象类型是 [HTMLInputElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)

常用的通用属性和方法有：

- value

value属性对应输入框里面的文本

- name

对应name属性

- type

对应type属性:text,number,redio,checkbox,color,range,date,time,datetime-local,email,password

- disabled

是个bool值，表示该按钮是否被禁用，禁用为true，可用为false

类型是 text，email，password，number 等这些输入框input，有如下属性

- maxLength/minLength

对应 maxLength/minLength 属性，也就是最多和最少输入的字符数量

- placeholder

对应 placeholder 属性

- readOnly

对应 readOnly 属性

- size

对应 size 属性，

- select()

全部选中输入框里面的文本

调用该方法前面通常需要先调用focus方法

- setSelectionRange(selectionStart, selectionEnd)

选中指定部分内容，

参数 selectionStart 指定了选中的开始字符索引， selectionEnd指定了选中的结尾字符索引+1， 索引从0开始计数

调用该方法前面通常需要先调用focus方法

- setRangeText(replacement, start, end)

用新字符串替换输入框中指定部分内容，

参数 replacement 指定用来替换的字符串

参数 start, end 是可选参数，

如果没有 start,end 参数，会替换用户选中部分内容，如果用户没有选中内容，则插入到光标位置

如果 有 start,end 参数， 分别指定了替换的开始字符索引 和 结尾字符索引+1， 索引从0开始计数

类型是 number 的数字输入框, 除了上述属性外，还有下面的属性

- max、min

max指定了 输入框 可输入数字最大值，超过该值时会有错误提示，

min指定了 输入框 可输入数字最小值，小于该值时会有错误提示

- step

一个数字，表示输入框数字的增减单位，

指示输入框点击上下箭头时，会按这个step为粒度增减。

当输入的数字不是该值的倍数时，会有错误提示

下面是 raido 或者 checkbox 类型的 input 常用属性

- checked

该属性值为bool类型，表示是否选中

- defaultChecked

该属性值为bool类型，表示缺省（页面刚加载完成时）是否选中

### `textarea`

 对应的DOM对象类型是 对应的DOM对象类型是 [HTMLTextAreaElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement)

常用属性有：

- value

返回/设置 文本框内的输入内容

- rows/cols

`rows` 、 `cols` 属性分别指定 输入框的行数和列数

- disabled

返回/设置 是否禁用文本框，被禁用后，就不能输入内容了

- maxLength/minLength

对应 maxLength/minLength 属性，也就是最多和最少输入的字符数量

- placeholder

对应 placeholder 属性

- wrap

返回/设置 一行文字超过文本框宽度时 如何显示，取值为

- off

  一行文字超过文本框宽度时，不换行， textarea元素出现水平滚动条

- soft

  一行文字超过文本框宽度时，剩余内容 折到下一行显示，这是缺省值

- hard

  设置该值时，必须同时指定 `cols` 属性

  这样，一行文字超过文本框宽度时，浏览器自动插入换行符 (CR+LF)，剩余内容 折到下一行显示

- select()

全部选中输入框里面的文本

调用该方法前面通常需要先调用focus方法

- setSelectionRange(selectionStart, selectionEnd)

选中指定部分内容，

参数 selectionStart 指定了选中的开始字符索引， selectionEnd指定了选中的结尾字符索引+1， 索引从0开始计数

调用该方法前面通常需要先调用focus方法

- setRangeText(replacement, start, end)

用新字符串替换输入框中指定部分内容，

参数 replacement 指定用来替换的字符串

参数 start, end 是可选参数，

如果没有 start,end 参数，会替换用户选中部分内容，如果用户没有选中内容，则插入到光标位置

如果 有 start,end 参数， 分别指定了替换的开始字符索引 和 结尾字符索引+1， 索引从0开始计数

### select 和 option

`select` 对应的DOM对象类型是 [HTMLSelectElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement)

`option` 对应的DOM对象类型是 [HTMLOptionElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptionElement)

`HTMLSelectElement` 常用属性、方法有：

- options 只读

返回 [HTMLOptionsCollection](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptionsCollection) 对象， 里面存放了所有的option对应的 HTMLOptionElement 对象

- value

返回/设置 第一个选定 option 元素的value属性。

值为 空字符串 表示未选择任何元素。

- selectedIndex

返回/设置 第一个选定 option 元素的索引数字。

值为 -1 表示未选择任何元素

- disabled

返回/设置 是否禁用选择框，被禁用后，就不能选择了

- type 只读

返回 表示支持多选与否的字符串。 当该select有multiple属性时，返回“select-multiple”； 否则，返回“select-one”

- add(item, before)

添加选项，有两个参数

- item

  新增 HTMLOptionElement 对象 或者 HTMLOptGroupElement 对象

- before

  表示在哪个索引对应的选项前插入，值为数字，插入到index对应的位置上。

  注意，索引从0开始

  该参数可选，如果没有传入值，或者为一个不存在的index，则新选项会插入到最末尾

- remove(index)

  删除选项

  参数index表示要删除选项的位置索引，值为数字

  注意，索引从0开始

`HTMLOptionElement` 常用属性、方法有：

- value

返回/设置 该 option 元素的value属性

- text

返回/设置 该 option 元素的 选项文本内容

- selected

返回/设置 该 option 元素是否被选择，是为 true，否为 false

- index 只读

返回 该 option 元素的索引值

`a` 对应的DOM对象类型是 [HTMLAnchorElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement)

常用属性有： hash host hostname href origin port protocol search

```txt
a.hash     = #pop
a.host     = www.byhy.net:80
a.hostname = www.byhy.net
a.href     = https://www.byhy.net:80/py/lang/basic/09?name=byhy&pagenum=23#pop
a.origin   = https://www.byhy.net:80
a.port     = 80
a.protocol = https:
a.search   = ?name=byhy&pagenum=23
```

其中 search 属性获取的是url参数，每个参数由键值对组成。

如果要获取其中某个参数的值，可以这样

```javascript
let params = new URLSearchParams(a.search);
let pagenum = parseInt(params.get("pagenum")); 
```

`button` 

对应的DOM对象类型是 [HTMLButtonElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement)

常用属性有：

- disabled

是个bool值，表示该按钮是否被禁用，禁用为true，可用为false

`img` 

对应的DOM对象类型是 [HTMLImageElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement)

常用属性有：

- alt

对应 img 元素的 alt属性， 当图片显示失败时（比如url错误），在图片位置显示的文本内容

- height

对应 img 元素的 高度，数字为单位，表示像素

- width

对应 img 元素的 宽度，数字为单位，表示像素

- src

对应 img 元素的 的url地址

### 事件处理

addEventListener

// 鼠标点击事件
element.addEventListener("click", handleFunc )

// 键盘事件
element.addEventListener("keydown", handleFunc )

文本输入框中，键盘按键 `Ctrl + 回车` ，就会执行script里面的函数

```javascript

<script>    
function salaryStats(event){
  // 如果ctrl键按下，并且按下了回车键
  if (event.ctrlKey && event.key==='Enter') {    
    document.querySelector('pre').innerText  = `处理结果省略...`
  }
}

// 注册事件回调函数
document.querySelector('#salary').addEventListener("keydown", salaryStats );
</script> 
```

还可以使用 元素对应 DOM 对象的事件属性 指定 事件处理函数

事件属性名是 `on`开头，后面加事件名称

比如： `onkeydown` 、 `onclick` 等等

// 键盘事件
element.onkeydown = handleFunc 

// 鼠标点击事件
element.onclick = handleFunc

```javascript
document.querySelector('#salary').onkeydown = function (event){
  if (event.ctrlKey && event.key==='Enter') {    
      document.querySelector('pre').innerText  = `处理结果省略...`
  } --匿名函数
}
```

箭头函数

```javascript
document.querySelector('#salary').onkeydown = event =>{
  if (event.ctrlKey && event.key==='Enter') {    
      document.querySelector('pre').innerText  = `处理结果省略...`
  }
}
```

上例中 `<script>` 元素包含的代码 是放在 body 的最后的

### 页面加载后才执行

```javascript
window.addEventListener('load', (event) => {
  // 执行代码
});
```

参数event 就是 load事件对象

如果不需要处理该对象，可以忽略，像这样

也可以使用 window对象的onload属性

```javascript
<head>

  <script>  
  window.onload = () => {
    document.querySelector('#salary').onkeydown = event =>{
      if (event.ctrlKey && event.key==='Enter') {    
          document.querySelector('pre').innerText  = `处理结果省略...`
      }
    }   
  };    
  </script>
</head>
```

虽然键盘事件处理代码是放在head中，

但不是立即执行，而是等页面加载完成后再执行

 event ([网址](https://developer.mozilla.org/zh-CN/docs/Web/API/Event))参数对应的就是，事件发生时，浏览器传入回调我们的函数时，传入的 `事件对象`

传入的是键盘事件对象，它的有属性

- ctrlKey

如果事件发生时，ctrl键按下，值为true，否则为false

- altKey

如果事件发生时，alt键按下，值为true，否则为false

- Shift

如果事件发生时，shift键按下，值为true，否则为false

- key

返回 事件发生时，按下按键的字符串表示，比如

Enter 对应回车键

1、2、3、4 对应数字键 1、2、3、4

a、b、c、d 对应字母键 a、b、c、d

A、B、C、D 对应字母键 A、B、C、D

事件对象类型 有很多，除了 键盘事件、鼠标按钮事件 外，还有 滚轮事件（WheelEvent）、拖拽事件（DragEvent）、游戏触控板事件（GamepadEvent）

还有的事件不是用户操作触发的，比如 页面加载完成事件、网址hash更改事件（HashChangeEvent）、websocket网络消息事件、 存储事件 等等

### 事件处理顺序

- 浏览器创建一个 click 事件对象

- 这个事件对象会先从 浏览器DOM 顶层的 window 对象一直 传递下去，直到 `触发事件的对象的父对象` ，这个过程称之为 `capture Phase（捕获阶段）`

- 然后，这个事件对象 到达触发事件的td对象，这个过程称之为 `target phase（目标阶段）`
- 然后，这个事件对象 再从触发事件的td对象，一直传递到顶层的window对象，这个过程称之为 `bubbling Phase（冒泡阶段）`

注意， click 事件是会 冒泡传播 的， 但是也有些类型的事件（比如，blur、focus）是不会冒泡的，到target 位置就结束了。

要声明注册的处理函数是在 `捕获阶段 触发执行` ，应该这样

element.addEventListener("click", e => {这里是处理代码}, true)

第3个参数如果是boolean 并且设置为true ，就表示是 Capture Phase触发行

默认情况：要声明注册的处理函数是在 `非捕获阶段（target phase 和 bubbling Phase）触发执行` ，应该没有第3个参数，或者第3个参数为false

### 事件对象 target属性 / this

`e.target` 指代了 真正触发事件的那个DOM对象而，

`e.currentTarget` 指代了当前 正在处理事件的DOM对象， 也就是当前处理函数 注册对应那个对象

在处理函数中，也可以使用 `this` ，等价于 `event.currentTarget`

比如，上面的代码可以等价改为

```javascript
function changeColor(e) {
  e.target.style.backgroundColor  = 'green'
  this.style.backgroundColor  = 'gray'
} 
```

### XHR/AJAX

户访问网页，包括网页里面内嵌的 CSS/js/图片 等等资源，通常是浏览器直接获取的，这种获取请求的HTTP被称之为 **同步请求**

而浏览器执行网页包含的js代码的过程，有时js代码还要到服务端 提交、获取数据， 这样的请求被称之为 **异步请求**

浏览器的js环境内置了一个 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 类型，简称 `XHR`

可以这样创建一个 XHR 对象

```
var xhr = new XMLHttpRequest();
```

XHR 对象 有属性方法，支持 浏览器页面代码 和 服务端之间的通信

这种方式传输数据，使用的序列化方法主要是 XML， 所以又被称之为 `AJAX (Asynchronous JavaScript And XML)` 方式

缺点是：写起来比较麻烦，所以目前并不推荐使用。

现在比较多的用法是： 使用 jQuery库 ，或者 ES6引入的 `fetch` API

比如，静态文件是 nginx 提供的服务，API则是单独开发的业务服务程序

，web服务可以使用 VSCode 的扩展 live Server，

live Server有代理配置项，可以设置转发规则

如下配置，

```javascript
"liveServer.settings.proxy": {
  "enable": true,
  "baseUri": "/api",
  "proxyUri": "http://127.0.0.1:80/api"
}
```

其中 baseUri 就是配置什么样的请求是要转发给API服务的

这里 `/api` ， 就是把 `/api` 开头的 http请求 都转给 bysms系统处理

因为 bysms系统服务的IP和端口 是 `127.0.0.1:80` ，

所以 `proxyUri` 的值为 `http://127.0.0.1:80/api` ，也可以写成 `http://127.0.0.1/api`

注意： `http://127.0.0.1:80/api` 后面的部分 `/api` 不能少

### jQuery 构建请求消息

典型的ajax()方法，接收一个settings参数，该参数是一个[PlainObject](https://api.jquery.com/Types/#PlainObject)类型， 可以把它当作 Object 来用

设定 XHR（AJAX）请求的数据 ， 包括 请求方法、url、消息头和消息体，比如，要发送 `get` 请求，可以使用这样的代码

```javascript
$.ajax({  
  type: 'GET',        
  url : '/api/mgr/signin',  
})
```

settings参数包含了两个键值对数据：

- type 是 HTTP方法

比如 GET,POST,PUT,DELETE,HEAD 等等

- url 是 请求的url地址

比如这里就是 `/api/mgr/signin` ，

注意：这里url没有前面的 `http://IP:端口`

请求的方法是get、post，还有简便写法

```javascript
$.get('/api/mgr/signin')
$.post('/api/mgr/signin',
      {
        username:'byhy',
        password:'abc'
      }
)
// 或者
$.get({        
  url : '/api/mgr/signin',  
})
$.post({     
  url : '/api/mgr/signin', 
  data: {
    username:'byhy',
    password:'abc',
  }
})
```

URLSearchParams对象的方法 `toString` ,可以将对象序列化为 urlencoded 格式

比如：

```javascript
var queryStr = new URLSearchParams({code:'6000001', time:'2022-02-23' }).toString()
$.get(`http://localhost/api/stock?${queryStr}`)
```

也可以使用 jQuery 的 [param](https://api.jquery.com/jquery.param/) 方法

```javascript
var queryStr = $.param({code:'6000001', time:'2022-02-23' })
$.get(`http://localhost/api/stock?${queryStr}`)
```

### 消息头

jQuery 发送http请求要定制一些消息头，可以通过ajax方法的settings参数里面的 `headers` 属性设置，如下

```javascript
$.ajax({  
  url: '/api/something',
  type: 'GET', 
  headers: {"X-Test-Header": "test-value"}
})
```

ajax方法的参数对象 contentType 设置消息头 contentType 的值

缺省为： `application/x-www-form-urlencoded; charset=UTF-8`

JSON 的方法 `stringify` 可以序列化 js对象 为 JSON格式的字符串

```javascript
$.ajax({
  url: '/api/mgr/signin',        
  type: 'POST',       
  contentType : 'application/json', 
  data: JSON.stringify({ username: 'byhy', password:'88888888'}),
})
```

如果使用的是jQuery， settings的 success属性函数定义了成功接收到HTTP响应消息的回调函数

可以在回调函数的第3个参数 [jqXHR](https://api.jquery.com/jQuery.ajax/#jqXHR) 对象获取响应消息的信息

```javascript
$.ajax({
  url: '/api/mgr/signin',        
  type: 'POST', 
  data: 'username=byhy&password=88888888',

  // 正确返回
  success: function(data, textStatus, xhr) { 
    // 获取状态码
    console.log(textStatus);
    // 获取所有消息头
    console.log(xhr.getAllResponseHeaders());
    // 获取某个消息头
    console.log(xhr.getResponseHeader("content-length"));
  },

  // 错误  
  error:function (xhr, textStatus, errorThrown ){
    console.error(`${xhr.status} \n${textStatus} \n${errorThrown }`)
  }
})
```

- success 请求响应成功后的回调函数

所谓成功就是返回响应消息，

回调函数被传入3个参数：

- data 从服务端返回的数据
- textStatus 返回的状态文本描述
- xhr 这是 XMLHttpRequest的扩展类型jqXHR的对象
- error 请求响应失败后的回调函数

回调函数被传入3个参数：

- xhr 这是 XMLHttpRequest的扩展类型jqXHR的对象

- textStatus 返回的错误状态文本描述

  比如："timeout", "error", "abort", 等等 "parsererror"

- errorThrown 异常对象文本

  比如："Not Found" 或者 "Internal Server Error."

响应消息体数据格式， 应该设置 ajax 参数 settings 的 `dataType` 属性说明这个data的类型

比如 设置为 json

```javascript
$.ajax({
  url: '/api/mgr/signin',        
  type: 'POST', 
  data: 'username=byhy&password=88888888',

  // 响应消息格式 预设为 json, 
  dataType: 'json', 

  // 正确返回
  success: function(data, textStatus, xhr) { 
    console.log(data)
  }
})
```

`dataType` 属性值还可以是

- xml

返回 XML 对象

- html

返回 HTML 字符串

- text

返回消息体字符串

### jQuery

jQuery库url为：

```txt
https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js
```

- [字节跳动公共库cdn](https://cdn.bytedance.com/)

`jQuery('button')` 返回的就是一个 jQuery 对象。

这个对象类似js的数组，里面包含了所有参数 css 选择器 选中的DOM对象。

jQuery 对象的 `on` 方法，用来定义事件处理，如下

```txt
jQuery('button').on('click',  function () { alert('按钮被点击') })
```

第一个参数就是事件名称，第二个参数是回调函数

```
jQuery` 还有一个简化的别名就是一个字符 `$
```

所以上面的代码也可以简写为

```txt
$('button').on('click', function () { alert('按钮被点击') })
```

可以注册处理其他类型事件

```javascript
<head>
  <script src="https://cdn.jsdelivr.net/npm/jquery/dist/jquery.min.js"></script>
</head>
<body>
  <textarea>按ctrl+回车 试试</textarea>
  <script>
  $('textarea').on('keydown', function (e) { 
    if (e.ctrlKey && e.key==='Enter')    
      alert('按下了ctrl+回车') 
  })
  </script>
<body>
```

添加元素

```javascript
// 这样创建一个html元素对象 
var ele1 = $( "<p>新元素1</p>" );
// 添加到 div#content1 ，作为最后一个子元素
ele1.appendTo( "#content1" );
// 当然也可以这样写
$( "<p>新元素1</p>" ).appendTo( "#content1" );
// 添加到 #content1 ，作为第一个子元素
$( "<p>新元素2</p>" ).prependTo( "#content1" );
// 插入到 #target2 前面，作为哥哥节点
$( "<p>新元素3</p>" ).insertBefore( "#target2" );
// 插入到 #target2 后面，作为弟弟节点
$( "<p>新元素4</p>" ).insertAfter( "#target2" );
$( "#content1" ).append( "<p>新元素1</p>" );
$( "#content1" ).prepend( "<p>新元素2</p>" );
$( "#target2" ).before( "<p>新元素3</p>" );
$( "#target2" ).after( "<p>新元素4</p>" );
```

删除元素方法很简单，先选中，然后调用 `remove` 方法，

比如：

```javascript
$( "#content1" ).remove()
```

替代元素

- 调用 `html` 方法，更新元素内部的html，

比如：

```txt
$( "#content2" ).html(`<p id='target3'>新元素2222</p>`)
```

类似 浏览器内置的 元素对象的 `innerHTML` 方法

- 调用 `text` 方法，更新元素内部的文本内容，

比如：

```txt
$( "#target2" ).text(`新元素22222`)
```

类似 浏览器内置的 元素对象的 `innerText` 方法

元素值操作

调用 `val()` 方法，可以获取或者设置一些元素的值，比如 input、select、textarea

类似 HtmlElement 的 value 属性的作用

比如

```javascript
// 获取 id 为 username 的元素的值
let username = $('#username').val()
// 设置 id 为 username 的元素的值
$('#username').val('byhy')
```
ID属性的值不要以数字开头，数字开头的ID在 Mozilla/Firefox 浏览器中不起作用
不要在属性值与单位之间留有空格（如："margin-left: 20 px" ），正确的写法是 "margin-left: 20px" 



`attr()` 可以用来获取元素的属性，类似浏览器内置的 元素对象的`getAttribute()` 方法

返回值都是字符串的形式

`attr()` 如果提供第2个参数，就是用来设置元素的属性，

类似浏览器内置的 元素对象的`setAttribute()` 方法

`removeAttr()`方法 用来删除元素的属性

类似浏览器内置的 元素对象的`removeAttribute()`

`css()`方法 用来设置元素的style里面的某个样式属性

比如，

```txt
a.css('color', 'red')
```

```javascript
<a id="hey" 
  style='color:green;font-size: 2rem;'  
  href="#">
  一个链接
</a>

<script>
  $('a').click(function () {
    $(this).css('color', 'red');
  });
</script>
```

`$(this)` 代表了 触发这个点击事件的 对象，并且封装为一个JQuery对象，这样可以方便的使用JQuery对象更便捷方法。

否则，如果我们只用 this，就得这样写

```
$('.a').click(function () {
  this.style.color = "red"
});
```