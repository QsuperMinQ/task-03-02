##  Vue.js 源码剖析-响应式原理、虚拟 DOM、模板编译和组件化

一、简答题

1、请简述 Vue 首次渲染的过程。

* 首先 Vue 初始化，初始化 Vue 的实例成员以及静态成员。
* 初始化结束之后，开始调用构造函数 new Vue()，在构造函数中调用了 this._init(),这个方法相当于整个Vue的入口。
* 在 _init() 中调用了 this.$mount(),共有两个 this.$mount
	*  第一个 this.$mount()是 entry-runtime-with-compiler.js 入口文件，这个方法的核心作用是把模版编译成 render 函数，首先判断一下当前是否传入了 render 选项，如果没有传入render：
  		* 首先获取 template 选项，编译成 render 函数
    	* 如果没有 template 选项，则把 el 中的内容作为模版，编译成 render 函数
    	* 模版是通过 compileToFunctions() 函数编译成 render 函数的，编译好之后，render 函数存在 options.render 中
	* 第二个 this.$mount() 是 runtime/index.js 中的方法，这个方法首先会重新获取 el, 因为如果是运行时版本的话，是没有执行 entry-runtime-with-compiler.js 这个入口中获取 el,所以，需要在这里重新获取。
* 接下来调用 mountComponent(this, el)
	* 首先判断 render 选项，如果没有 render，但传入了模版。并且当前是开发环境，则发送警告，警告运行时版本不支持编译器。
	* 接下来触发 beforeMount 这个生命周期的钩子函数，也就是开始挂载之前
	* 然后定义了 updateComponent(),在这个方法中，定义了 _render 和 _update，_render 的作用是生成虚拟 DOM，_update的作用是将虚拟DOM转换成真是DOM，并且挂载到页面上。
	* 接下来创建 Watcher 对象，在创建 Watch 时，传递了 updateComponent 这个函数，这个函数最终是在 Watcher 内部调用的。在 Watcher 创建完之后还调用了 get 方法，在 get 方法中，会调用 updateComponent()
	* 最后触发了生命周期的钩子函数 mounted ，挂载结束，返回 Vue 实例。

2、请简述 Vue 响应式原理。

* 响应式是从 Vue 的 init 方法开始的，首先调用 initState() 初始化实例的状态，在这个方法中调用 initData() 方法把 data 属性注入到 Vue 实例上，并且调用 observe() 方法把 data 转化成响应式的对象。
* observe(value) 方法是响应式的入口（位置： src/core/observer/index.js）
	* 判断 value 是否是对象，如果不是直接返回
	* 判断 value 对象是否有 _ob_ 属性，如果有直接返回（说明已经做过响应式的处理）
	* 如果没有，创建 observer 对象，并返回
* 创建 observer 类过程 (位置： src/core/observer/index.js）
	* 给 value 对象定义不可枚举的 _ob_ 属性，记录当前的 observer 对象
	* 数组的响应式处理，也就是设置数组的常用方法，包括push,pop,sort等，这些方法会改变原数组，当这些方法调用的时候，找到数组对象中的ob,ob中的 dep,调用 dep 中的 notify 方法进行通知。设置完数组的方法之后，遍历数组中的每个成员，对每个成员调用 observe(成员)，进行判断是否进行响应式处理。
	* 对象的响应式处理，调用 walk 方法，方法中遍历对象的每一个属性，对每一个属性调用 defineReactive()
* defineReactive() 方法过程
	* 为属性创建 dep 对象，dep 对象收集依赖
	* 如果当前属性值是对象，调用 observe ，进行响应式处理
	* 定义 getter，收集依赖，返回属性的值
	* 定义 setter，保存新值，如果新值是对象，调用 observe 进行响应式处理，当数据发生变化时，发送通知，调用 dep.notify()
* 收集依赖
	* 在 Watcher 对象的 get 方法中调用 pushTarget 记录 Dep.target 属性
	* 调用 data 中的成员的时候收集依赖，defineReactive 中的 getter 中收集依赖
	* 把属性对应的 watcher 对象添加到 dep 的subs 数组中
	* 给 childOb 收集依赖，目的是子对象添加和删除成员时发送通知
* Watcher 
	* 当数据发生变化时，调用 dep.notify() 方法发送通知，再调用 update() 方法，在 update() 中会调用 queueWatcher()
	* 在 queueWatcher() 中判断 watcher 是否被处理，如果没有添加到 queue 队列中，并调用 flushSchedulerQueue() 刷新队列
	* flushSchedulerQueue() 方法过程
		* 触发 beforeUpdate() 钩子函数
		* 调用 watcher.run() 过程，get() -> getter() -> updateComponent()，之后最新内容渲染完成
		* 清空上一次依赖
		* 触发 actived 钩子函数
		* 触发 updated 钩子函数

3、请简述虚拟 DOM 中 Key 的作用和好处。

* 作用：给每一个节点设置key，能够跟踪每个节点的身份，在进行比较的时候，会基于 key 的变化重新排列元素顺序，重用和重新排序现有元素，移除 key 不存在的元素。
* 好处：可以减少 dom 操作，减少 diff 和渲染所需要的时间，提升了性能。
	

4、请简述 Vue 中模板编译的过程。

* compileToFunctions(template,...) 是编译模版的入口函数，首先先从缓存中加载编译好的 render 函数，缓存中没有则调用 complie(template, options) 开始编译
* complie(template, options) 的核心是合并选项，首先合并 options，然后调用 baseCompile(template, trim(), finalOptions) 开始编译
* baseCompile 函数过程
	* 解析器 parse() 把模版转换成 AST 抽象语法树
	* 优化器 optimize() 对AST进行静态比标记，用来优化虚拟 DOM 的渲染
	* generate() 将 AST Tree 生成 js 的创建代码
* 将 render 函数 通过 createFunction 函数 转换为 一个可以执行的函数
*  render 和 staticRenderFns 初始化完毕，挂载到 Vue 实例的 options 对应的属性中
