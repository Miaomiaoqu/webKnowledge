1、什么是mvvm以及mvvm原理
1）MVVM是Model-View-ViewModel的简写，即模型-视图-视图模型。模型指的是后端传递的数据。视图指的是所看到的页面。视图模型mvvm模式的核心，它是连接view和model的桥梁。在MVVM的框架下视图和模型是不能直接通信的，它们通过ViewModel来通信。
2）ViewModel通常要实现一个observer观察者，当数据发生变化，ViewModel通过数据绑定能够监听到数据的这种变化，然后通知到对应的视图做自动更新
3）当用户操作视图，ViewModel通过DOM事件监听也能监听到视图的变化，然后通知数据做改动，从而实现了数据的双向绑定。并且MVVM中的View 和 ViewModel可以互相通信。
2、mvvm 和 mvc 区别？
mvc 中 Controller 演变成 mvvm 中的 viewModel。mvvm主要解决了mvc中大量的 DOM 操作使页面渲染性能降低，加载速度变慢，影响用户体验。且当 Model 频繁发生变化，开发者需要主动更新到 View 。
3、vue的优点是什么？
(1)低耦合。视图（View）可以独立于Model变化和修改，一个ViewModel可以绑定到不同的"View"上，当View变化的时候Model可以不变，当Model变化的时候View也可以不变。
(2)可重用性。你可以把一些视图逻辑放在一个ViewModel里面，让很多view重用这段视图逻辑。
(3)独立开发。开发人员可以专注于业务逻辑和数据的开发（ViewModel），设计人员可以专注于页面设计。
(4)可测试。界面素来是比较难于测试的，而现在测试可以针对ViewModel来写。

4、vue数据双向绑定原理？(vue的原理)

采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，也就是绑定了data中数据属性的DOM元素，触发相应的监听回调。
主要实现三部分：
1）实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
2）实现一个指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
3）实现一个订阅者Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图
4）MVVM作为数据绑定的入口，整合以上三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果。
5、上面三者的具体实现 
（1）实现Observer
利用Obeject.defineProperty()来监听属性变动，将data里面所有的属性值遍历一遍，添加上set和get访问器，这样在设置data的属性值的时候，会触发set方法，那么set方法主要有两个作用，一是改变data里面的属性值，二是发出数据变化的通知。
（2）实现一个Compile
模板本质是字符串 
有逻辑，如 v-if、 v-for 等，转换为 html 渲染页面，必须用 JS 才能实现
因此，模板最重要转换成一个 JS 函数（render 函数）
render 函数执行之后，返回的是 vnode
Compile中首先将template或el编译成render函数，render函数返回一个虚拟DOM对象（将模板转为 render 函数的时候，实际是先生成的抽象语法树（AST），再将抽象语法树转成的 render 函数）
     1、浏览器不认识vue指令，所以要对模板里面的vue指令进行编译，将里面的所有节点劫持下来，在新建的文档碎片(documentflagment)里面进行编译，编译完成之后再插入到挂载目标节点。
2、如果挂载目标元素有子节点，那么对第一个子节点进行编译
3、模板编译器函数会判断元素节点是什么类型，表单元素和文本节点分别处理，因为表单节点的value会由用户手动输入，所以要给表单元素添加监听器，根据输入的value改变data里的属性值，再更新到视图上，这是双向的数据流。而文本节点，只要改变data里面的值，节点随着改变就行，只是单向的数据流
（3）实现一个订阅者Watcher
    1、data里的每个属性都有一个dep对象，dep对象里可以有很多订阅者（watcher），也就是绑定了该数据属性的所有DOM元素，因为dep数组里存的全是watcher对象，所以在dep发布变更通知的时候，会调用该watcher的update方法，来更新该watcher对应的节点值。（也就是说模板上可以有多处元素绑定data里的同一个属性值，dep是依赖于data里面的属性的，只有一个添加订阅者的方法和一个发布变化通知的方法）
2、watcher原型里面的update方法，会在new Watcher()和dep.notify()的时候调用。new Watcher()对应模板编译，dep.notify()对应数据监听，所以数据监听和模板编译就是通过watcher连接起来的。
6、vue-router两个模式
（1）hash 
   vue-router默认hash模式，hash模式的原理是onhashchange事件。hash值为#后面的内容。hash模式的特点在于hash出现在url中，但是不会被包括在HTTP请求中，对后端没有影响，不会重新加载页面。，仅hash符号之前的内容会被包含在请求中，如http://www.abc.com,因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回404错误。
（2）History
      pushState API来完成URL跳转二无须重新加载页面。history模式下，前端的URL必须和实际向后端发起请求的URL一致。如htttp://www.abc.com/book/id。如果后端缺少对/book/id 的路由处理，将返回404错误。
7、虚拟dom
用 JS 模拟 DOM 结构。
将DOM 对比操作放在 JS 层，提高效率，提高重绘性能
virtual DOM分为2个步骤：
1.createElement(): 用 JavaScript对象(虚拟树) 描述 真实DOM对象(真实树)
2.patch(oldNode, newNode) : 对比新旧两个虚拟树的区别，收集差异，将差异应用到真实DOM树
第一次渲染，将 vnode 缓存下来
8、vue的diff算法
1、频繁的DOM 操作会降低性能，因此尽量减少 DOM 操作
2、找出本次 DOM 必须更新的节点来更新，其他的不更新
3、vdom 中应用 diff 算法是为了找出需要更新的节点
9、vue的key的作用
key来给每个节点做一个唯一标识，Diff算法就可以正确的识别每个节点，高效的更新虚拟DOM。从而尽可能高效的渲染元素而不是从头渲染，防止默认的“就地复用”策略
10、keepalive
keep-alive组件用来缓存组件,避免多次加载相应的组件,减少性能消耗。比如列表页,去了详情页回来,还是在原来的页面。
keep-alive 指令可以将切换出去的组件保留在内存中，可以保留它的状态或避免重新渲染。
11、watch和comouted区别
同：computed和watch都是观察页面的数据变化的。
异：computed只有当页面数据变化时才会计算，当数据没有变化时，它会读取缓存。而watch每次都需要执行函数，methods也是每次都需要执行。
数据变化时执行异步操作，这个时候使用watch是合适的
12、Vuex
（1）vuex进行全局共享数据的管理，项目越来越大以后，每个组件内部的一些数据，可能需要共享给其他组件，组件之间通信的问题。音乐播放、登录状态、加入购物车。
使用时，在 main.js 创建一个store，并注入vue实例中。Vue组件从store中读取数据，若是store中的数据发生改变，依赖这个数据的组件也会发生更新。
（2）vuex有哪几种属性？
　1、state：用来存放组件之间共享的数据。一般会在组件的计算属性（computed）获取state的数据（因为，计算属性会监控数据变化，一旦发生改变就会响应）。在根实例中注册了store 后，用 this.$store.state 来访问；对应vue里面的data；存放数据方式为响应式，vue组件从store中读取数据，如数据发生变化，组件也会对应的更新。
  2、getters：看成是store的计算属性，对state的数据进行筛选，过滤，它的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
3、mutations：实际改变状态(state)中数据的唯一方式，是通过 提交(commit) 一个 mutation中的方法。mutations方法必须是同步方法！
4、actions和mutations的区别
　1.Actions 提交的是 mutations，而不是直接变更状态。也就是说，actions会通过mutations，让mutations帮他提交数据的变更。在组件中使用是$store.dispath('')进行派发
　2.Action 可以包含任意异步操作。ajax、setTimeout、setInterval不在话下
5、module:将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter,声明模块后，放到store的modules对象中，使用模块对应的属性名获取
defineProperty实现一个双向绑定







