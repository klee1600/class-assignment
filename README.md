# 2022百度前端实战训练营大作业题目2——简单MVVM框架实现

- 姓名：吴禹廷
- 学校：南京大学
- 专业：软件工程
- 年级：20级
- QQ：859917105
- 邮箱：859917105@qq.com; 201250031@smail.nju.edu.cn

### 大作业简介

本次百度前端实战训练营大作业题目2的MVVM框架，运用了多种设计模式，在实现MVVM框架的基础上，添加了诸多额外的功能。

- MVVM基本功能：
  1. 数据劫持
  2. 发布订阅模式
  3. 数据单向绑定
  4. 数据双向绑定

- 使用到的设计模式：
  1. 外观模式
  2. 构造器模式
  3. 模块模式
  4. 发布订阅模式
  5. 原型模式
  6. 命令模式


### MVVM的实现过程

- MVVM的流程设计
- [发布订阅模式](/mvvm/example/mediator/)的实现
- [发布订阅模式](../../tree/master/mvvm/example/mediator/)的实现(gitee仓库链接)
- [数据劫持](/mvvm/example/hijack/)的实现
- [数据劫持](../../tree/master/mvvm/example/hijack/)的实现(gitee仓库链接)
- [数据双向绑定](/mvvm/example/dataBinder/)的实现
- [数据双向绑定](../../tree/master/mvvm/example/dataBinder/)的实现(gitee仓库链接)
- [简易视图指令的编译过程](/mvvm/example/view/)的实现
- [简易视图指令的编译过程](../../tree/master/mvvm/example/view/)的实现(gitee仓库链接)
- [ViewModel](/mvvm/example/viewModel/)的实现
- [ViewModel](../../tree/master/mvvm/example/viewModel/)的实现(gitee仓库链接)
- [MVVM](/mvvm)的实现
- [MVVM](../../tree/master/mvvm/)的实现(gitee仓库链接)

### MVVM的实现演示

- MVVM示例的使用如下所示，包括`browser.js`(View视图的更新)、`mediator.js`(发布订阅模式)、`binder.js`(MVVM的双向数据绑定)、`view.js`(视图)、`hijack.js`(数据劫持)以及`vue.js`(MVVM实例)。

    ``` javascript
    <div id="app">
    <input type="text" b-value="input.message" b-on-input="handlerInput">
    <div>{{ input.message }}</div>
    <div b-text="text"></div>
    <div>{{ text }}</div>
    <div b-html="htmlMessage"></div>
    </div>

    <script src="./mixin.js"></script>
    <script src="./browser.js"></script>
    <script src="./mediator.js"></script>
    <script src="./binder.js"></script>
    <script src="./view.js"></script>
    <script src="./hijack.js"></script>
    <script src="./vue.js"></script>


    <script>
        let vm = new Vue({
            el: '#app',
            data: {
                input: {
                    message: 'KLEE!!!'
                },
                text: 'KLEE!',
                htmlMessage: `<button>重置</button>`
            },
            methods: {
                handlerInput(e) {
                    this.text = e.target.value
                },

                reset() {
                    this.text = "KLEE!"
                }
            }
        })
    </script>
    ```

- 初始界面效果图参看图片![](picture/step1.png)
- 在输入框中输入后效果图参看图片![](picture/step2.png)
- 点击重置按钮后效果图参看图片![](picture/step3.png)

- 网页示例代码参看[文件](/mvvm/src/index.html)

#### 初始化流程

- 创建MVVM实例对象，初始化实例对象的`options`参数
- 方法`proxyData`和`proxyMethods`将MVVM实例对象的`data`数据和函数代理到MVVM实例对象上
- `Hijack`类实现数据劫持功能（对MVVM实例与视图进行响应式数据进行监听）
- 解析视图指令，对MVVM实例与视图关联的DOM元素转化成文档碎片并进行绑定指令解析（包括但不限于`b-value`、`b-on-input`、`b-html`等指令），
- 添加数据订阅和用户监听事件，将视图指令对应的数据挂载到`Binder`类的数据绑定引擎上（数据变化时通过Pub/Sub模式通知绑定器更新视图）
- 使用发布/订阅模式代替Vue中的Observer模式
- 数据绑定引擎采用了命令模式解析视图指令，调用`update`方法对视图解析绑定指令后的文档碎片进行更新视图处理
- `Browser`采用了外观模式对浏览器进行了简单的兼容性处理

#### 响应式流程

1. 监听用户输入事件
2. 调用MVVM实例对象的数据设置方法更新数据
3. 进行数据劫持，并调用`setter`方法
4. 通过发布/订阅模式向所有订阅者发布数据变化情况
5. 订阅者接收数据更新通知，更新数据对应的视图

### 发布订阅模式的简单实现

- 参见文件 mvvm\src\mediator.js
- 其示例文件参看 mvvm\example\mediator\index.html

- 其中发布/订阅模式实例对象的使用规范如下：

``` javascript
let mediator = new Mediator()
// 订阅channel1
let channel1First = mediator.sub('channel1', (data) => {
  console.info('[mediator][channel1First][callback] -> data', data)
})
// 再次订阅channel1
let channel1Second = mediator.sub('channel1', (data) => {
  console.info('[mediator][channel1Second][callback] -> data', data)
})
// 订阅channel2
let channel2 = mediator.sub('channel2', (data) => {
  console.info('[mediator][channel2][callback] -> data', data)
})
// 发布channel1,向所有订阅者广播数据变化情况
mediator.pub('channel1', { name: 'KLEE' })
// 发布channel2,向所有订阅者广播数据变化情况
mediator.pub('channel2', { name: 'KLEE!!!' })
// 取消channel1标识为channel1Second的订阅
mediator.cancel(channel1Second)
// 再次发布channel1,此时只会执行channel1中标识为channel1First的回调函数
mediator.pub('channel1', { name: 'KLEE' })
```

### 数据劫持的简单实现

- 参见文件 mvvm\src\hijack.js
- 其示例文件参看 mvvm\example\hijack\index.html

- 使用了`[[ Get ]]`和`[[ Set ]]`的特性，在访问对象的属性和写入对象的属性时自动触发属性特性的调用函数，从而做到监听数据变化的目的。
- 对象的属性通过方法`Object.defineProperty(data, key, descriptor)`改变属性的特性，其中`descriptor`传入参数为特性集合。

- 以下为数据劫持实例对象的使用规范：

``` javascript
let hijack = (data) => {
    if (typeof data !== 'object') return
    for (let key of Object.keys(data)) {
        let val = data[key]
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: false,
            get() {
                console.log('[hijack][get] -> val: ', val)
                    // 和执行 return data[key] 有什么区别 ？
                return val
            },
            set(newVal) {
                if (newVal === val) return
                console.log('[hijack][set] -> newVal: ', newVal)
                val = newVal
                    // 如果新值是object, 则对其属性劫持
                hijack(newVal)
            }
        })
    }
}

let person = { name: 'KLEE2', age: 1 }
hijack(person)
    // [hijack][get] -> val:  KLEE2
person.name
    // [hijack][get] -> val:  1
person.age
    // [hijack][set] -> newVal:  KLEE
person.name = 'KLEE'

// 属性类型变化劫持
// [hijack][get] -> val:  { familyName:"KLEE2", givenName:"KLEE" }
person.name = { familyName: 'KLEE2', givenName: 'KLEE' }
    // [hijack][get] -> val:  KLEE2
person.name.familyName = 'KLEE2'

// 数据属性
let job = { type: 'javascript' }
console.log(Object.getOwnPropertyDescriptor(job, "type"))
    // 访问器属性
console.log(Object.getOwnPropertyDescriptor(person, "name"))
```

### 双向绑定的简单实现

- 参见文件 mvvm\src\binder.js
- 其示例文件参看 mvvm\example\dataBinder\index.html

- 数据绑定实例对象使用规范如下：
```javascript
let input = document.getElementById('input')
let div = document.getElementById('div')

// model
let data = { input: '' }

// 数据劫持
hijack(data)

// model -> view
data.input = 'KLEE'

// view -> model
input.oninput = function(e) {
  // model -> view
  data.input = e.target.value
}
```

### 指令编译过程的简单实现

- 实现流程：
  - 获取对应的元素，通常是`#app`
  - 将元素转换成对应的文档碎片
  - 识别文档碎片中的绑定指令并修改该指令相对应的DOM元素
  - 处理完文档碎片后重新渲染`#app`元素

- 示例：

    ``` html
    <body>
        <h1>compiler</h1>
        <div id="app">
            <input type="text" b-value="message" />
            <input type="text" b-value="message" />
            <input type="text" b-value="message" />
        </div>

        <script src="./browser.js"></script>
        <script src="./binder.js"></script>
        <script src="./view.js"></script>
        <script>
            // 模型
            let model = {
                message: 'Hello World',
                getData(key) {
                    let val = this
                    let keys = key.split('.')
                    for (let i = 0, len = keys.length; i < len; i++) {
                        val = val[keys[i]]
                        if (!val && i !== len - 1) {
                            throw new Error(`Cannot read property ${keys[i]} of undefined'`)
                        }
                    }
                    return val
                }
            }

            // 抽象视图
            new View('#app', model)
        </script>
    </body>
    ```

- 已经实现了对`b-value`, `b-text`, `b-html`, `b-on-*`等指令的简单实现
  
- 在`view.js`中实现了`#app`下的元素转化成文档碎片以及对所有子元素进行属性遍历操作（用于`binder.js`的绑定属性解析）
- 在`binder.js`处理绑定指令
- 在`browser.js`中使用外观模式对浏览器原生的事件以及DOM操作进行了再封装，从而可以做到浏览器的兼容处理。
- 其详细示例文件参看 mvvm\example\view\index.html

### 代码测试

- 应用了jest对上述实现过程进行了代码测试，已经实现了测试基本全覆盖。

### 结语

- 至此已经实现了一个简易的MVVM模型。可以模拟出多数Vue指令。