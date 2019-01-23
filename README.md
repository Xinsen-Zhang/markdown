# 利用 Vue 搭建简单的 markdown 编写环境及代码高亮和目录生成
## 全部代码见[鑫森的 github]()
## 效果
<div align='center'>
    <img src='https://img2018.cnblogs.com/blog/1591103/201901/1591103-20190123120512932-865336460.png' alt='整体页面的效果' width='400'>
</div>
### 代码区的高亮
### 目录的生成

## 开发流程
1. 静态页面的制作
2. 静态页面的拆分(Vue 组件实现)
3. 渲染好 Markdown 界面
4. 目录的生成

## 静态页面的制作

### 静态依赖
* Bootstrap
* Jquery
* Popper

## 组件编写

### 组件的关系
<div align='center'>
    <img src='https://img2018.cnblogs.com/blog/1591103/201901/1591103-20190123133400575-740211859.png' alt='组件间的关系' width='400'>
</div>

### 组件通信与状态管理
* 个人感觉使用发布订阅机制可以非常灵活并且不是很熟悉这个机制. 因此强制自己使用这个机制进行 Vue 的组件间通信
#### 通信工具
* pubsub.js
    * `npm i --save-dev pubsub-js`
    * 导入
        * `import PubSub from 'pubsub-js'`
    * 使用
        * `PubSub.publish('contentChange', content)`
            * 参数说明
                1. `contentChange`
                    * 发布的消息名称,相当于事件触发
                2. `content`
                    * 发布消息时的具体内容
                    * 与事件触发的不同之处就在于此,能够携带信息进行传递 
        * `PubSub.subscribe('contentChange', cb)`
            * 参数说明
                1. `contentChange`
                    * 订阅的消息名称,相当于事件监听是监听的事件名称
                2. `cb`
                    * 回调函数,相当于事件监听是具体的事件操作
                    * 回调函数有两个参数`msg`和`data`
                    * 订阅的消息的真实内容就在第二个参数`data`中

### 功能实现
#### markdown 解析
##### 思路
* 通过订阅事件`contentChange`, 当编辑器的内容发生变化的时候实时获取 markdown 文本
* 利用`showdown` 库将 markdown 内容进行解析
##### markdown 解析工具
* showdown.js
    * `npm i --save-dev showdown-js`
    * 导入
        * `import {showdown} from 'showdown-js'`
    * render
        * `var converter = new showdown.Converter()`
        * `convert.makeHTML('markdown context')`

##### 部分代码

```javascript
const contentChangeHandler = (vueComponent, content) => {
    PubSub.subscribe('contentChange', (msg, data) => {
        if (vueComponent) {
            vueComponent.content = data
            vueComponent.content = converter.makeHtml(data)
        } else {
            content = data
            content = converter.makeHtml(data)
        }
        if (vueComponent) {
            generateCatlog()
        }
    })
    return content
}
```
* 这是剥离出来的 markdown 渲染内容
* 由于使用了组件的 data
    * 函数传入了两个参数
        * 当在 data 内时
            * 第一个参数传入空值`null`或者`false`
            * 第二个参数随意传入一个变量
            * 将返回的内容设置为 data
        * 在订阅消息的回调函数中
            * 第一个参数传入当前的 Vue 组件, 即`this`
            * 第二个参数此时无效

#### 代码高亮
##### 思路
* 对使用 showdown 渲染出来的代码块直接使用 highlight 的高亮方法即可
##### 代码高亮显示工具
* highlight.js
    * `npm i --save-dev highlight`
    * 导入
        * `import hljs from 'highlight.js'`
        * `import 'highlight.js/styles/atom-one-dark.css'`
    * highlight
        * `hljs.highlightBlock(element)`
            * 参数说明
                1. element 是需要高亮显示的 dom 元素
                2. element里的代码块 是要被`<pre><code>`和`</code></pre>`包裹的代码块
                3. 默认可以自动识别, 也可以才`<code>`标签中通过`className` 来指定具体的高亮策略
```html
<pre>
    <code class='python'>
import numpy as np
from matplotlib import pyplot as plt
x = np.arange(0,100,0.01)
y = np.sin(x)
plt.plot(x, y)
plt.show()
    <code>
</pre>
```

##### 部分代码
```javascript
const highlightCode = () => {
    const preEl = document.querySelectorAll('pre')
    preEl.forEach((element) => {
        hljs.highlightBlock(element)
    })
}
``` 
* 在 markdown 内容解析成功之后直接调用上述函数就能使代码块高亮显示


#### 目录的生成
##### 思路
* 在`showdown`解析好的 html 中获得所有的 h 标签及内容
    * 怎么获得呢?
    * 想到的方法是通过正则表达式进行获取
* 考虑到生成内容之后可以通过锚点快速跳转因此需要设置id
    * 为了保证 id 是惟一的,所以使用标题的text 作为 id
    * 那如果 id 里面有中文怎么办
        * 通过`escape(str)`将中文转码
* 最后将数据通过`refreshCatlog`发布给`markdownCatlog`组件
* 在组件中使用正则表达式剥离出 标题级别,文本内容和 id 属性
* 利用剥离出来的内容制作`h`标签,并且在标签中内嵌`a`标签来达到锚点跳转的效果
* 为了视觉上的效果,不同等级的标签会有不同的缩进
* 为了标签显示的不是很 pussy, 对 h 的大小稍微进行了调整

##### 部分代码
```javascript
const updateHead = (elementArray) => {
    elementArray.forEach((element) => {
        element.id = escape(element.innerText)
    })
}
```
* 设置markdown 解析好的 html 页面里面的`h`标签 id 值被设置且为题
* 解决了中文的问题
* 只需要通过选择器和遍历执行`updateHead`函数即可

```javascript
if (!headInfo) {
        return false
    }
    let details = []
    headInfo.forEach((element, index) => {
        element.match(/<h(\d) id="(.*)?">(.*)?</g)
        details.push({
            head: RegExp.$1,
            id: RegExp.$2,
            detail: RegExp.$3
        })
    }, this)
    let catlog = ''
    details.forEach((element) => {
        let textSize = parseInt(element.head) + 2
        if (textSize > 6) {
            textSize = 6
        }
        let a = `<h${element.head} style='text-indent: ${element.head}em' class= "h${textSize}"
        ><a href="#${element.id}" class= 'text-white'>${element.detail}</a></h${element.head}>`
        catlog = catlog + a
    }, this)
```
* headInfo 为`refreshCatlog`发布过来的消息
* this 指向的是`markdownCatlog` 组件

### 效果图
#### 代码的高亮

<div align='center'>
    <img src='https://img2018.cnblogs.com/blog/1591103/201901/1591103-20190123123826377-1347279543.png' alt='代码的高亮' width='400'>
</div>

#### 目录的生成

<div align='center'>
    <img src='https://img2018.cnblogs.com/blog/1591103/201901/1591103-20190123123927078-904245874.png' alt='目录的生成' width='400'>
</div>


