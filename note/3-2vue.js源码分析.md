## 任务一：vue.js 源码分析-响应式原理

> 解读源码经常问自己：我是谁，我在哪？0.0 <br/>
> 调试源码快捷键：F11进入方法，F10向下执行

* vue使用的打包工具是 Rollup 原因有：
	* Vue.js 源码的打包工具使用的是 Rollup，比 Webpack 轻量
	* Webpack 把所有文件当做模块，Rollup 只处理 js 文件更适合在 Vue.js 这样的库中使用，webpack 合适开发项目使用
	* Rollup 打包不会生成冗余的代码

* vue 的不同版本构建
	* 完整版：同时包含 编译器 和 运行时 版本
	* 编译器：用来将模版字符串编译成为 Javascript 渲染函数代码，体积大，效率低
	* 运行时：用来创建 Vue 实例，渲染并处理虚拟 DOM 等的代码，体积小，效率高。基本上就是除去编译器代码
	* UMD：UMD 版本通用的模块版本，支持多种模块方式。 vue.js 默认文件就是运行时 + 编译器的 UMD 版本
	* CommonJS(cjs)：CommonJS 版本用来配合老的打包工具比如 Browserify 或 webpack 1
	* ES Module：从 2.6 开始 Vue 会提供两个 ES Modules (ESM) 构建文件，为现代打包工具提供的版本。


* vscode 中使用flow时有红波浪线的报错，在设置中改为 json 格式，添加 "javascript.validate.enable": false, 默认为true，改为false即可

* vue 首次渲染过程
	* Vue 初始化，实例成员，静态成员
	* new Vue()
	* this._init()
	* vm.$mount()
		* src\platforms\web\entry-runtime-with-compiler.js
		* 判断是否传入 render，如果没有传入 render，把模版编译成 render 函数
		* render 渲染函数是由 compilerToFunction() 生成
		* 然后把render存到 options 中，即 options.render = render
	* vm.$mount()
		* src\platforms\web\runtime\index.js
		* 运行时版本会在 index.js 中的 $mount 方法中再次获取 el,然后调用 mountComponent()
	* mountComponent(this,el)
		* src\core\instance\lifecycle.js
		* 判断是否有 render 选项，如果没有但是传入了模版，并且当前是开发环境的话会发送警告：运行时版本不支持编译器
		* 触发 beforeMount
		* 定义 updateComponent
			* vm._update(vm.render(),...)
			* vm._render()渲染，渲染虚拟 DOM
			* vm._update()更新，将虚拟 DOM 转换成真实 DOM
		* 创建 watcher 实例
			* updateComponent 传递
			* 调用 get() 方法
		* 触发 mounted
		* return vm
	* watcher.get()
		* 创建完 watcher 会调用一次 get
		* 调用 updateComponent()
		* 调用 vm._render() 创建 VNode
			* 调用 render.call(vm._renderProxy,vm.$createElment)
			* 调用实例化时 Vue 传入的render()  或者编译 template 生成的 render()
			* 返回 Vnode
		* 调用 vm.update(vnode, ...)
			* 调用 vm._patch_(vm.$el, vnode) 挂载真实 DOM 
			* 记录 vm.$el
	
* 响应式处理 defineReactive()


## 任务二：vue.js 源码分析-虚拟 DOM 

1、 为什么使用虚拟 DOM ？

* 避免直接操作 DOM，提高开发效率
* 作为一个中间层可以跨平台
* 虚拟 DOM 不一定可以提高性能
	* 首次渲染的时候会增加开销
	* 复杂视图情况下提升渲染性能

2、h 函数

* vm.$createElement(tag, data, children, normalizeChildren)
	* tag: 标签名称或者组件对象
	* data: 描述tag,可以设置 DOM 的属性或者标签的属性
	* children: tag 中的文本内容或者子节点
* Vnode核心属性：tag,data,children,ele,key

3、createElement：把vnode转化为DOM元素，并插入DOM树，触发响应的钩子函数

4、patchVnode：对比新旧 Vnode 以及新旧 Vnode 的子节点，更新差异，如果新旧 Vnode 都有字节点，会调用 updateChildren 对比子节点的差异

5、updateChildren

* 从头和尾开始依次找到相同的子节点进行比较 patchVnode, 总共四种方式
* 在老节点的子节点中查找 newStartVnode,并进行处理
* 如果新节点比老节点多，把新增的子节点插入到DOM中
* 如果老节点比新节点多，把多余的老节点删除

6、设置 key 的作用：优化DOM操作


## 任务二：vue.js 源码剖析-模板编译和组件化

1、模板编译的作用

* Vue 2.x 使用 VNode 描述视图以及各种交互，用户自己编写 VNode 比较复杂
* 用户只需要编写类似 HTML 的代码 - Vue 模板，通过编译器将模板转换为返回 VNode 的 render 函数
* .vue 文件会被 webpack 在构建的过程中转换成 render 函数

2、模版编译的主要目的是将模版（template）转换为渲染函数（render）

3、对编译生成的 render 进行渲染的方法 

```
vm._c = (a,b,c,d) => createElement(vm,a,b,c,d,false)

```
4、对手写 render 函数进行渲染的方法

```
vm.$createElement = (a,b,c,d) => createElement(vm,a,b,c,d,true)

```

5、编译入口：compileToFunctions(template, {}, this)

6、抽象语法树AST（Abstract Syntax Tree）
	
* 使用对象的形式描述树形的代码结构
* 次数的抽象语法树是用来描述树形结构的 HTML 字符串

7、为什么要使用抽象语法树


8、注册组件方式：

* 全局组件
* 局部组件








	