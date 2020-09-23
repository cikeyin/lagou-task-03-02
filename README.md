##### Vue.js 源码剖析-响应式原理、虚拟 DOM、模板编译和组件化

#####  —、简答题
###### 1、请简述Vue首次渲染的过程。
答：
(1)  进行Vue初始化，初始化实例成员和静态成员。

(2)  初始化结束后，调用new Vue()。在构造函数中调用this._init()，该方法为Vue的入口，在此方法中调用入口文件中的vm.$mount()方法。

(3) vm.$mount()方法通过compileToFunctions()用来将模板转换成render函数，并保存到options.render中。

(4) 调用runtime文件中的vm.$mount()方法，运行时版本会在mountComponent()方法中重新获取el。mountComponent()方法中会进以下操作。
- 触发beforeMount；
- 定义updateComponent；
- 创建Watcher实例；
- 触发mounted；
- 返回vm。


###### 2、请简述Vue响应式原理。
(1) 在入口方法中调用initData()方法将data注入到vue实例中。

(2) 调用observe()方法将data对象转换成响应式的对象。observe(value)的功能为：
- 判断value是否是对象，如果不是对象直接返回
- 判断value对象是否有__ob__，如果有说明这个对象已经执行过响应化的处理，直接返回
- 如果没有，创建observer对象
- 返回observer对象

(3) Observe类的功能有：
- 给value 对象定义不可枚举的__ob__属性，记录当前的observer对象
- 为数组做响应式处理
- 为对象做响应式处理，调用walk方法,遍历每一个属性调用defineReactive()方法

(4) defineReactive()
- 为每一个属性创建dep对象
- 如果当前属性的值是对象，调用observe
- 定义getter。收集依赖，返回属性的值
- 定义setter。保存新值；如果新值是对象，调用observe；派发更新，调用dep.notify()


(5) 收集依赖
- 在watcher对象的get方法中调用pushTarget记录Dep.target属性
- 访问data中的成员的时候，在defineReactive的getter中收集依赖
- 把属性对应的watcher对象添加到dep的subs数组中
- 如果属性的值也是对象，给childOb收集依赖，目的是子对象添加和删除成员时发送通知

(6) 数据发生变化时Watcher的执行过程
- 数据发生变化时调用dep.notify()发送通知，调用wacher对象的update()方法
- update()方法中调用queueWatcher()判断wacher是否被处理，如果没有的活添加到queue队列中，并调用flushSchedulerQueue()
- flushSchedulerQueue()的执行过程： 触发beforeUpdate钩了函数；调用wacher.run()；清空上—次的依赖;触发actived沟子函数;触发updated钩子函数



###### 3、请简述虚拟DOM中Key的作用和好处。
答：Key的特殊属性主要用在Vue的虚拟DOM算法，在新旧node对比时辨识VNode。方便让VNode 在 diff 的过程中找到对应的节点，然后成功复用。使用key，它会基于key的变化重新排列元素顺序，并且会移除key不存在的元素。
设置key，可以减少dom的操作，减少diff和渲染所需的时间，提高性能。



###### 4、请简述 Vue中模板编译的过程。
答：模板编译的主要目的是将模板（template）转换成渲染函数（render）。由于编写VNode比较复杂，编写html模板比较简单，所以我们在开发时选择编写html模板，再通过编译器将模板转换成返回VNode的render函数。

(1)  模板编译入口函数compileToFunctions（template,...）。先从缓存中加载编译好的render函数；缓存中没有调用complie(template, options)。

(2)  complie(template, options)方法中先合并options，再调用编译模板方法baseCompile(template.trim(),finalOptions)。

(3)  baseCompile方法完成编译模板的核心内容。
 - parse()方法。把template转换成 AST tree
- optimize()方法。标记AST tree 中的静态sub trees；检测到静态了树，设置为静态，不需要在每次重新渲染的时候重新生成节点；patch阶段跳过静态子树
- generate()方法。将AST tree生成js的创建代码。

(4) compileToFunctions方法中通过调用createFunction()把上—步中生成的字符串形式js代码转换为函数；render和staticRenderFns初始化完毕后，挂载到vue 实例的options对应的属性中。

