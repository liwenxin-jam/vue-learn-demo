# vue-learn-demo
## vue2 and vue3 学习记录项目

基于[【尚硅谷】Web前端迅速上手Vue教程丨vue3.0入门到精通](https://www.bilibili.com/video/BV1Zy4y1K7SH)，记录相关知识点，达到备忘录的目的

## 风格指南

- [风格指南](https://cn.vuejs.org/v2/style-guide/)推荐

## 环境配置

- 如出现下载缓慢请配置npm淘宝镜像

```bash
npm config set registry https://registry.npm.taobao.org
npm view vue versions
```

- Vue-Cli脚手架隐藏了所有webpack相关的配置，若想查看具体的webpack配置

```
vue inspect > output.js
```

- vue-devtools 

  谷歌插件[Vue.js devtools](https://chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg?utm_source=chrome-ntp-icon)，注意beta版才支持vue3(vuex不支持)，以前的只支持vue2

  1. 开启允许访问文件网址（支持本地调试）

## vm与vc

- vm是实例对象，如const vm = new Vue({ data: {} }); data可以是一个普通对象/函数对象
- vc是组件实例对象，如Vue.component(options)、Vue.extend(options); data必须是一个函数对象。
- 一个重要的内置关系：VueComponent.prototype.__ proto __  ===  Vue.prototype  (让组件实例对象vc可以访问Vue原型上的属性、方法)

```js
// vc组件实例跟下边vm实例对象区别：因为组件是可复用的 Vue 实例，所以它们与 new Vue 接收相同的选项，例如 data、computed、watch、methods 以及生命周期钩子等。仅有的例外是像 el 这样根实例特有的选项。
const vm = new Vue({
  el: '#app'
})
const vm = new Vue().$mount('#app');
```

## 指令

- v-html

  - 指令作用：向指定节点中渲染包含html结构的内容

  - 与插值语法的区别：

    a. v-html会替换掉节点中所有的内容，{{xxx}}不会

    b. v-html可以识别html结构，区别v-text指令

  - 注意：v-html有安全性问题

    a. 在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击

    b. 一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！

  - [Cookie-Editor](https://chrome.google.com/webstore/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm?utm_source=chrome-ntp-icon)

  ```js
  document.cookie // 只显示非http-only的 
  ```

- v-cloak指令（没有值）:

  1. 本质是一个特殊属性，Vue实例创建完比并接管容器后，会删掉v-cloak属性
  2. 使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题

- 指令函数何时会被调用？（区别指令对象不同函数时机）

  1. 指令与元素成功绑定时
  2. 指令所在的模板被重新解析时（任意数据变化）

- 注意：

  1. 指令函数无法使用data数据，它的this是指向window，可以考虑把data某个属性传参进去，如果考虑的是数据传递，可以考虑借助dataset方式传递
  2. 指令名如果是多个单词，要使用kebab-case(短横线)命名方式，不要用camelCase(小驼峰)命名方式（注意：定义组件名称(属性)也建议用kebab-case，大写组件名(属性)必须vue-cli脚手架才行，直接引入vue不行，自闭合组件标签同理，多个只会渲染一个）
  3. 其它自带指令：v-once(只渲染一次)、v-pre(跳过编译)

```scss
[v-cloak] {
	display: none; // 常规配套使用
  visibility: hidden;
}
```

```js
// <span v-big='n'></span>
// <input type="text" v-fbing:value="n">

directives: {
  // 1.指令函数，有bind和update效果
	big(element, binding) {
		element.innerText = binding.value * 10
	}
  
  // 2.指令对象
	fbing: {
    // 指令与元素成功绑定时
		bind(element, binding) {
      element.value = binding.value
    }
    // 指令所在元素被插入页面时，需要在此阶段做特殊处理
		inserted(element, binding) {
      element.focus()
    }
    // 指令所在的模板被重新解析时（任意数据变化）
		update(element, binding) {}
	}
}
// 全局指令
// Vue.directive('fbing', function() {})  
// Vue.directive('fbing', { })  
```

## 过滤器

- 定义：对要显示的数据进行特定格式式再显示（适用于一些简单逻辑的处理）,vue3取消了，推荐用方法或计算属性实现

- 语法：

  1. 注册过滤器：

     ```js
     Vue.filter(name, callback) 或 vc组件实例对象 { filters: {} }
     ```

  2. 使用过滤器：

     ```js
     {{ xx | 过滤器1 | 过滤器2() }} 或 v-bind:属性="xx | 过滤器名()"
     ```

- 备注：

  1. 过滤器也可以接收额外参数、多个过滤器也可以串联
  2. 并没有改变原本的数据，是产生新的对应的数据

## 修饰符

- 事件修饰符，支持链式使用，passive和prevent冲突，不能同时绑定在一个监听器上

```vue
<!-- 
.capture (捕获事件，父捕获子) 
.stop (阻止事件冒泡，子冒泡父) 
w3c的方法是e.stopPropagation()，IE则是使用e.cancelBubble = true
document.addEventListener(event, function, useCapture=false)
.self (event.target是当前操作元素才触发事件，避免捕获和冒泡事件触发) 

.prevent (阻止默认事件，e.preventDefault()，如a标签默认跳转) 
.once (只触发一次) 
.passive (事件的默认行为立刻执行，无需等待事件回调执行完毕，如@scoll，@touchmove，@wheel防止页面卡顿)
 -->
<div @scroll.passive="scroll"></div><!-- up、down或按住滚动条滚动 -->
<div @wheel.passive="wheel"></div><!-- 鼠标滚轮 -->
```

- 键盘修饰符，Vue默认提供以下9个按键别名，未提供的可以用按键原始的key值去绑定，如 caps-lock(短横线命名)

```js
/**
 正常配合keyup使用
.enter：回车键
.tab：制表键  特殊，需要配合keydown
.delete：含delete和backspace键
.esc：返回键
.space: 空格键
.up：向上键
.down：向下键
.left：向左键
.right：向右键
**/
```

- 系统修饰符（用法特殊）: ctrl、alt、shift、meta(win/command)

  1. 配合keyup使用，需要组合键释放了才能触发，例如ctrl+a

     ```vue
     <div @keyup.ctrl="handler"></div> <!-- 不会正常触发 -->
     <div @keyup.ctrl.a="handler"></div>
     ```

  2. 配合keydown正常使用

- 自定义键名，Vue.config.keyCodes.自定义键名(huiche) = 键码 （跟编码@keyup.13一样不推荐，语义化不清晰和浏览器编码不一定保持一致）

- v-model修饰符

  ```vue
  /** .lazy(转变为在change事件再同步) .number(自动将用户的输入值转化为数值类型) .trim(自动过滤用户输入的首尾空格) **/
  <input v-model.lazy="msg">
  ```

- 自定义(第三方)组件修饰符 .native

  ```vue
  <input v-model="form.name" placeholder="昵称" @keyup.enter="submit">
  <el-input v-model="form.name" placeholder="昵称" @keyup.enter.native="submit"></el-input>
  <!-- 会把click事件当成自定义事件，需要$emit('click')来触发 -->
  <my-component @click='handler'></my-component>
  <!-- 会把click事件当成原生事件，vue3取消了.native -->
  <my-component @click.native='handler'></my-component>
  ```

## props

- 单向数据流

  简单类型不允许直接改，复杂类型(引用地址)不能重新赋值，但能直接用v-model实现双向绑定(不建议)

- 不仅能传递数据，也能传递方法(不推荐)

  1. 自定义事件(除了click、keyup、keydown外) @eventName=handler <--> $emit('eventName') ,如在子组件绑定自定义事件，就由子组件来$emit派发，回调写在父组件的handler方法上

     ```js
     // 哪个vc $emit就由哪个vc解绑$off
     this.$off('eventName')  // 解绑一个自定义事件
     this.$off(['eventName1', 'eventName2'])  // 解绑多个自定义事件
     this.$off()  // 解绑所有的自定义事件
     
     // vc实例销毁会让所有自定义事件失效，但不影响默认事件，如click
     // 注意：data绑定的数据也会失去响应式效果，即事件触发了页面不响应
     this.$destroy() 
     ```

  2. this.$refs.childVc.$on('eventName', function(){ console.log(this) }) 灵活处理绑定时机

     谁派发谁监听，注意function里面的this指向this.$refs.childVc，而不是指向当前写这行代码的vc父组件实例。记住谁触发了自定义事件，事件回调当中的this就指向谁。 改写成：this.$refs.vc.$on('eventName', () => {}) 

- $attrs与$listeners

  1. inheritAttrs，默认为true，父作用域的非props的属性会自动绑定到根节点上

  ```vue
  <template>
    <component :a="a" :b="b" @cb="cb"></component>
  
   	<div>
      <!-- 当前组件不需要，传递下去 -->
  		<child v-bind="$attrs" v-on="$listeners"></child>
    </div>
  
  	<!-- 通过定义props接收数据和emit自定义事件派发出去 -->
    <grandson>
      <div @click="emit('cb', '我是grandson')">{{ b }}</div>
    </grandson>
  </template>
  ```

## 插槽

- 作用：让父组件可以向子组件指定位置插入html结构，也是一种组件间通信的方式，适用于 父组件  ===> 子组件

```vue
<!-- category分类组件 -->
<template>
	<div class="category">
    <h3>{{title}}分类</h3>
    <!-- 定义一个默认插槽 -->
    <slot :test="test" msg="handsome boy"> <!-- 给作用域插槽传递一个公共数据test、msg -->
      <span>我是默认显示值</span>
  	</slot>
    <!-- 定义一个具名插槽 footer -->
    <slot name="footer" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      test: ['1', '2', '3']
    }
  }
}
</script>

<style scoped>
  span {
    color: blue;
  }
</style>

<!-- 父组件，里面引用了category分类组件 -->
<template>
	<category title="默认显示">
    <!-- 会默认绑定在vc上的$slots，即使组件内部没有定义slot -->
    <span>默认显示</span>
  </category>

  <category title="默认插槽">
    <span>Welcome to here!<span>
    <span slot="footer">Hello，Vue！<span>
    <!-- 另一种写法，只支持 template -->
    <template v-slot:footer>Hello，Vue！<template>
    <!-- vue3推荐使用v-slot和# -->
    <template #footer>Hello，Vue！<template>
      
    <!-- 作用域插槽，只支持 template，支持解构 -->
    <template scope="data">{{ data.test }}</template>
		<template scope="{ test }">{{ test }}</template>
		<!-- 另一种写法 -->
		<template slot-scope="{ test }">{{ test }}</template>
  </category>
</template>

<style scoped>
  span {
    color: blue;
  }
</style>
```

## 全局事件总线 eventBus

- 适用于任意组件间通信。注意如果利用事件总线$on绑定自定义事件，需要自己手动在beforeDestroy钩子中触发$off解绑。常规下vc自身的$on绑定，在vm销毁下会自动清除自定义事件

```js
// 第一种
import Vue from 'vue';
export default new Vue(); // 声明一个Vue 实例

import EventBus from '@utils/eventBUS';
// EventBus.$on
Vue.prototype.$eventBus = EventBus;
// this.$eventBus.$on

// 第二种
const vm = new Vue({
  el: '#app',
  render: h => h(App),
  beforeCreate() {
    Vue.prototype.$eventBus = this // 安装全局事件总线
  }
})
```

- pubsub-js(消息订阅发布)

```js
import pubsub from 'pubsub-js'

// 订阅（监听）
this.pubsubId = pubsub.subscribe('messageName', function(msgName, data) {
  console.log('有人发布了messageName消息，messageName消息的回调执行了', msgName, data)
})
// 发布（通知）
pubsub.publish('messageName', 666)

beforeDestroy() {
  // 取消订阅，类似定时器，有次订阅都有一个唯一ID，需要借助唯一ID取消
  pubsub.unsubscribe(this.pubsubId)
}
```

## Vuex

```js
import Vue from vue;
import Vuex from vuex;
// 应用Vuex插件
Vue.use(Vuex);

// 正常是import进来的
const otherOptions = {
  namespaced: true, // 是否支持命名空间访问，默认为false
  state: {},
  getters: {},
  actions: {},
  mutations: {},
}

// 在实例化store前，必须先引入插件
export default Vuex.Stroe({
  // 用于存储数据，不建议在非mutaions方法下直接修改state，devtool无法监测
  state: {},
  // 用于将state中的数据进行加工，类似自己写computed
  getters: {}, 
  // 用于响应组件中的动作，常用于共用复杂业务和异步操作
  actions: {}, // 方法名建议小驼峰
  // 用于同步操作数据(state)，在devtool追踪数据变化
  mutations: {}, // 方法名建议大驼峰或者全大写(下划线隔开)，区别actions中的方法
  // 子模块数据，里面结构跟根节点保持一致，即同样拥有state、mutations...
  modules: {
    // otherOptions,
    a: otherOptions,
  },
})
```

1. mapState方法用于帮助我们映射state中的数据为计算属性
2. mapGetters方法用于帮助我们映射getters中的数据为计算属性
3. mapActions方法生成对应的指定方法，方法中会调用dispatch去联系action
4. mapMutations方法生成对应的指定方法，方法中会调用commit去联系mutation

```vue
<script>
  import { mapState, mapGetters, mapMutations, mapActions } from 'vuex';
  
  export default {
    computed: {
      // 第一种写法，对象形式，key和state中的名称不一致的情况
      ...mapState({ key: 'stateName' }),
      // 第二种写法，数组简写形式
      ...mapState(['stateName']),
        
      // 同理mapState
      ...mapGetters({ key: 'gettersName' }),
      ...mapGetters(['gettersName']),
      
      // 注意需要自己在模板主动给funcName传参，默认参数是事件对象event
      ...mapMutations({ funcName: 'MutationFuncName' }),
      ...mapMutations(['MutationFuncName']),
      // 同理mapMutations
    	...mapActions({ funcName: 'MutationFuncName' }),
      ...mapActions(['MutationFuncName']),
  
  		// 子模块a
  		...mapState(['a']),
      // 借助命名空间namespaced访问
      ...mapState('a', ['stateName1', 'stateName2']),
      ...mapState('b', ['stateName1', 'stateName2']),
      ...mapMutations('a', { funcName: 'MutationFuncName' }),
    },
    methods: {
      test() {
        console.log(this.$store)
        this.$store.stateName1
        // 子模块 获取state某个数据
        this.$store.a.stateName1
        // 注意getters会跟state有所区别
        this.$store.getters['a/getterName']
        
        // dispatch action方法同理
        this.$store.commit('mutationName', params)
        this.$store.commit('a/mutationName', params)
      }
    }
  }
</script>
```

## Vue-Router

- 一个路由(route)就是一组映射关系(key-value)，多个路由需要路由器(router)进行管理

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../components/home'
import About from '../components/about'

Vue.use(VueRouter)
const router = new VueRouter({
    // 默认hash，history(刷新404)需要后端配合，例如nginx
  	// 前端自己搭node静态服务器，推荐express + connect-history-api-fallback
  	mode: 'history',  
    // 支持嵌套路由，在router-view的位置上显示
    routes: [
        // 前端路由：key是路径，value是组件
        {
            name: 'home', // 命名路由，name需要唯一，路由路径过长简化访问
            path: '/home',
            meta: { title: 'XX' }, // 路由元信息，用户自定义数据
            component: Home,
        },
        {
            path: '/about',
            component: About,
            // 嵌套路由，配置子路由
            children: [
                {
                    // 非一级路由，path不需要加/
                    path: 'news',
                    componet: News,
                },
                {
                    path: 'message',
                    componet: Message,
                },
                {
                    name: 'detail',
                    // 动态路由，绑定params参数
                    path: 'detail/:id', // :id 占位符，声明接收
                    componet: Detail,
                    // 路由支持传参，有三种写法
                    // 1.该对象中所有key-value都会以props的形式传给Detail组件
                    props: { a: 1, b: '2'},
                    // 2.值为布尔值，若为真，会把该路由组件收到的所有params参数，以props的形式传给Detail组件
                    props: true,
                    // 3.值为函数，以props的形式传给Detail组件
                    props($route) { // 支持解构
                        return { id: $route.query.id, name: $route.query.name}
                    },
                }
            ]
        }
    ]
})
export default router;
```

- 每个组件都有自己的$route属性，里面存储着自己的路由信息
- 整个应用只有一个router，可以通过组件的$router属性获取全部路由

```vue
<template>
	<!-- 传统方式，用a标签实现页面跳转 -->
	<a class="active" href="./home.html">home</a>
	<a class="active" href="./aboute.html">about</a>
	<!-- 插件会自动转化成a标签 active-class指定激活高亮样式名 -->
	<router-link active-class="active" to="/home">home</router-link>
	<router-link active-class="active" to="/about">about</router-link>
	
	<!-- 不需要传递其它参数，用简写方式即可，默认是push追加历史记录，replace替换当前记录，没法后退/前进 -->
	<router-link replace active-class="active" :to="{ path: '/home'}">home</router-link>

	<!-- 二级路由 -->
	<router-link active-class="active" to="/about/news">about</router-link>

	<!-- query传参 -->
	<router-link active-class="active" to="/about/detail?id=666&title=你好">about</router-link>
	<!-- 字符串写法 -->
	<router-link active-class="active" :to="`/about/detail?id=${item.id}`">about</router-link>
	<!-- 对象写法 -->
	<router-link active-class="active" :to="{
    	path: '/about/detail',
        query: {
            id: item.id                                    
        }
    }">about</router-link>

	<!-- params传参，其它写法参照query传参 -->
	<router-link active-class="active" to="/about/detail/666">about</router-link>
	<!-- 注意：对象写法params不允许跟path，只能配置name -->
	<router-link active-class="active" :to="{
    	// path: '/about/detail', 
        name: 'detail',                                
        params: {
            id: item.id                                    
        }
    }">about</router-link>

	<!-- 指定组件的呈现位置 -->
	<router-view></router-view>
	<!-- 允许路由嵌套显示 -->
	<router-view>
        <!-- 缓存路由组件，保持挂载，不被销毁，不加include会默认把所有组件都缓存 -->
        <!-- include组件名称，配置生命周期钩子 activated(激活，mounted)、deactivated(未激活，beforeDestroy) -->
        <!-- <keep-alive include="News"> -->
        <keep-alive :include="['News', 'Message']">
			<router-view></router-view>
        </keep-alive>
    </router-view>
</template>

<script>
	export default {
        name: 'News', // 组件名称
        props: ['a', 'b'],
        mounted() {
            console.log(this.$route)
            this.$router.push("/home")
            this.$router.replace({
                path: '/about/detail',
                query: {
                    id: item.id                                    
                }
            })
            this.$router.forward() // 前进一步
            this.$router.go(-1) // 正数前进N步，负数后退N步
        }
    }
</script>
```

- 路由守卫，区分全局路由、独享路由、组件路由

```js
// 全局前置路由守卫 -- 初始化的时候和每次路由切换之前被调用
router.beforeEach((to, from, next) => {
  // 常用鉴权
  next() // 放行
})
// 全局后置路由守卫 -- 初始化的时候和每次路由切换之后被调用
router.afterEach((to, from) => {
  // 更改浏览器窗口显示标题
  document.title = to.meta.title || '默认标题'
})

// 独享路由守卫
routes: [
  {
    name: 'home', 
    path: '/home',
    component: Home,
    // 只有前置，没有后置，进入页面组件之前
    beforeEnter: (to, from, next) => {
      // 类似beforeEach对它单独做一些判断
    }
  }
]

// 组件路由守卫
export default {
  // 通过路由规则，进入该组件时被调用
  beforeRouteEnter(to, from, next) => {}
  // 通过路由规则，离开该组件时被调用
  beforeRouteLeave(to, from, next) => {}
}
```

## 动画 transition、transition-group

- 作用：在插入、更新或移除DOM元素时，在合适的时候给元素添加样式类名

```vue
<template>
	<!-- appear 刚出现就展示一次动画 -->
	<transition name="hello" appear>
    <h1 v-show="isShow">你好啊！</h1>
  </transition>
	<!-- 多个同时动画，注意需要加key -->
	<transition-group name="hello" appear>
    <h1 v-show="isShow" key="1">你好啊！</h1>
    <h1 v-show="isShow" key="2">你好啊！</h1>
  </transition>
</template>
<script>
  data() {
    return {
      isShow: true
    }
  }
</script>
<style scoped>
  /** 默认不加name，是v-enter-active **/
  .hello-enter-active {
    animation: test 0.5s linear;
  }
  .hello-leave-active {
    animation: test 0.5s linear;
  }
  @keyframes test {
    from {
      transform: translateX(-100%);
    }
    to {
      transform: translateX(0);
    }
  }
  
  /** 不管是进入还是离开都分别有三个类名 **/
  h1 {
    /** 第一种 **/
    transition: 0.5s linear;
  }
  /** 进入的起点，离开的终点 **/
  .hello-enter, .hello-leave-to {
    transform: translateX(-100%);
  }
  /** 进入的过渡，离开的过渡 **/
  .hello-enter-active, .hello-leave-avtive {
    /** 第二种 **/
    transition: 0.5s linear;
  }
  /** 进入的终点，离开的起点 **/
  .hello-enter-to, .hello-leave {
    transform: translateX(0);
  }
</style>
```

- 第三方动画库 [animate](https://animate.style/#utilities)

```vue
<template>
	<transition-group appear name="animate__animated animate__bounce" enter-active-class="animate__swing" leave-active-class="animate__backOutUp">
    <h1 v-show="isShow" key="1">你好啊！</h1>
    <h1 v-show="isShow" key="2">你好啊！</h1>
  </transition>
</template>
<script>
  import 'animate.css'
  data() {
    return {
      isShow: true
    }
  }
</script>
```

## 代理

- 接口请求方式

  1. xhr (new XMLHttpRequest()、xhr.open()、xhr.send())
  2. jQuery ($.get、$.post)
  3. axios (内核是xhr，提供$axios，vue-resource插件$http)
  4. fetch (弊端：IE不兼容，返回数据会包两层)

- 跨域方式

  1. cors (Access-Control-Allow-Origin)

  2. jsonp (script src get)

  3. vue-cli 

     - 官方文档 [devserver-proxy](https://cli.vuejs.org/zh/config/#devserver-proxy)
     - proxy可参考中间件 [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware#proxycontext-config)

     ```json
     // vue.config.js
     devServer: {
         // 任何未知请求 (没有匹配到静态文件的请求) 代理到 http://localhost:8000
         // 弊端：会优先查找根目录下public文件夹里的，找不到再到代理服务器上找
     	proxy: 'http://localhost:8000'
     }
     
     proxy: {
         // 更多的代理控制行为，跟 .env.development 配置使用 
         [process.env.VUE_APP_URL]: {
             target: '<url>', // 代理目标的基础路径
             ws: true, // 是否启用支持websockets，默认为true
             secure: false, // 使用的是http协议则设置为false，https协议则设置为true
             changeOrigin: true, // 开启代理，是否保持一致的域名访问(善意的谎言)，默认为true
             pathRewrite: { [`^${process.env.VUE_APP_URL}`]: '' }, // 重写路径
         }
     }
     ```

  4. nginx (反向代理)

## Vue3

- 实例化

```js
// vue2是Vue构造函数，vue3是createApp的工厂函数
import { createApp } from 'vue'
import App from './App.vue'

// vue2是 new Vue({ render: h => h(App) }).$mount('#app')
createApp(App).mount('#app') // 创建应用实例对象 vm
```

- Composition API(组合式API)

```vue
<template>
  <!-- vue3支持多个根节点 fragment -->
  <div>Hello，Vue3!</div>
  <div>{{ name }}</div>
</template>

<script>
  import { h, ref, reactive, computed, watch, watchEffect } from 'vue'
  
  export default {
    // setup接收参数必须先定义
    props: ['msg'],
    // setup中的context.emit触发自定义事件必须先声明，不声明当成是原生事件，如click
    emits: ['callback'],
    // 尽量不要与vue2.x配置混用，这里支持写data、methdos、computed
    // vue2方法里能访问setup配置的属性，setup无法访问vue2的属性
    // 如果有重名，setup优先
    // setup不能是一个async函数，因为返回的不再是一个对象，而是一个promise，模板看不到return对象中的属性，可以借助动态加载组件 + suspense可以做到
    // 在beforeCreate之前执行，setup里面的this是undefined
    setup(props, context) { // context支持解构{ attrs,emit,slots }
      
      // 生命周期钩子 销毁 <--> 卸载
      // vue2 beforeCreate、created、beforeMount、mounted、beforeUpdate、updated、beforeDestroy、destroyed 
      // composition API是括号内的对应关系
      // vue3 beforeCreate(setup)、created(setup)、beforeMount(onBeforeMount)、mounted(onMounted)、beforeUpdate(onBeforeUpdate)、updated(onUpdated)、beforeUnmount(onBeforeUnmount)、unmounted(onUnmounted)
      
      // 如果需要写异步接口，建议用箭头函数包装多一层
      const func = async () => {
        const { data } = await apiFunc()
      }
      
      // 简单类型 name是RefImpl 引用对象 get set
      let name = ref('xx');
      // 在js中修改需要 .value
      name.value = 'handsome boy'
      
      // 复杂类型 info是RefImpl 而info.value是Proxy引用对象(内部借助reactive) 
      let info = ref({ name: 'xx' });
      info.value.name = 'handsome boy'
      
      // 不支持简单类型reactive(666)，通过Proxy代理数据劫持，再通过Reflect反射源对象
      let info = reactive(['a', 'b', 'c']);
      // vue2无法监测到响应式，需要借助this.$set(info, 0, 'aa')
      info[0] = 'aa';
      
      // 计算属性，简写形式，支持分开写 get(){} 和 set(){}
      let fullName = computed(() => {
        return person.firstName + person.lastName;
      })
      
      // 监听属性,接收4种类型(ref、reactive、array、function) 
      // let name = ref(name) 此外监听不需要name.value
      watch(name, (newVal, oldVal) => {
      },{ immediate: true, deep: true })
      // 同时监听多个，newVal和oldVal是一个数组
      watch([a, b], (newVal, oldVal) => {
      })
      // 监听ref能够正常，监听reactive是引用类型无法正常(新旧地址一致，且deep失效)
      watch(state, (newVal, oldVal) => {
      },{ deep: true })
      watch(() => state.age, (newVal, oldVal) => {})
      watch([() => state.age], (newVal, oldVal) => {})
      // 监听对象中的某个属性需要开启深度监听，但依旧无法真正获取真正的oldVal，let obj = ref({})同理
      watch(() => state.obj, (newVal, oldVal) => {},{deep:true})
      
      // 有点类似computed原理，里面用到哪些响应式数据，当它们改变时就会触发，但不需要返回值
      watchEffect(() => {
        if (state.status) {
          // do something
        }
      })
      
      // hooks，类似vue2的mixin
      // useMousePosition.ts
      import { onBeforeUnmount, onMounted, ref } from 'vue
      export default function () {
        const x = ref(-1) ; // x 绑定为响应式数据
        const y = ref(-1);
        const clickHandler=(event:MouseEvent)=>{
          x.value = event.pageX
          y.value = event.pageY
        } 
        onMounted(()=>{
          window.addEventListener('click', clickHandlker)
        })
        onBeforeUnmount(()=>{
          window.removeEventListner('click', clickHandler)
        })
        return {
          x,
          y
        }
       }
      
      import useMousePosition from './hooks/useMousePosition'
			export default {
        setup() {
          // 这里就用了 hooks 函数, 从而提高了复用性
          const {x, y} = useMousePosition() 

          return {
            x,
            y,
          }
        }
      }
      
      // toRef、toRefs
      let person = reactive({
        name: 'xx',
        age: 18
      })
      const name1 = person.name; // xx
      // 作用：toRef创建一个ref对象，其value值指向另一个对象中的某个属性
      // 应用：要将响应式对象中的某个属性单独提供给外部使用时
      const name2 = toRef(person, 'name'); // ObjectRefImpl
      
      // 浅层次(第一层)数据的响应式
      // 应用：如果有一个对象数据，结构比较深，但变化时只是外层属性变化
      let person = shallowReactive({
        name: 'xx',
        age: 18,
        info: {
          name: 'xx',
        	age: 18,
        }
      })
      
      // 传递普通类型跟ref没区别，如果传递的是一个对象，无法实时响应对象的值，person.value是普通对象(非proxy)
      // 应用：如果有一个对象数据，后续功能不会修改该对象中的属性，而是生成新的对象来替换
      let person = shallowRef({
        name: 'xx',
      })
      
      // readonly(深只读)不允许修改属性，shallReadonly(浅只读)允许修改嵌套属性(非根属性)
      let person = readonly(reactive({
        name: 'xx',
        info: {
          name: 'xx',
        }
      }))
      
      // toRaw把响应式对象reactive处理成普通对象，没法处理ref
      state = toRaw(state)
      // markRaw，标记一个对象，不再是一个响应式对象
      // 应用：当渲染具有不可变数据源的大列表时，跳过响应式转换提高性能
      state.info = markRaw(state.info)
      
      // 创建一个自定义的ref(精装修与毛坯customRef)，并对其依赖项跟踪和更新触发进行显式控制
      const myRef(value, delay = 500) {
        let timer
        // customRef(value) === ref(value)
        return customRef((track, trigger) => {
          return {
            get() {
              // 通知Vue追踪value的变化
              track();
              return value;
            },
            set(newValue) {
              clearTimeout(timer)
              timer = setTimeout(() => {
                value = newValue;
                // 通知Vue去重新解析模板
                trigger();
              }, delay);
            },
          }
        })
      }
      let testMyRef = myRef('hello')
      
      // provide与inject 适用于祖与后代(跨级)组件间通信
      provide('car', car); // 给自己的后代组件传递数据
      // car是响应式，接收到的也是响应式，使用inject必须先provide
      let car = inject('car')
      
      // 响应式数据的判断
      isRef：检查一个值是否为一个ref对象
      isReactive：检查一个对象是否是由reactive创建的响应式代理
      isReadonly：检查一个对象是否是由readonly创建的只读代理
      isProxy：检查一个对象是否由reactive或者readonly方法创建的代理
      
      // 常用，返回一个对象，则对象中的属性、方法在模板中均可以直接使用
      return { 
        // toRefs与toRef功能一致，但可以批量创建多个ref对象
        toRefs(person),
        name
      }
      
      // 返回一个渲染函数，会覆盖template结构
      return () => h('h1', 'handsome boy')
    }
  }
</script>
```

- 新的组件

  - Fragment

    - 在Vue2中：组件必须有一个根标签
    - 在Vue3中：组件可以没有根标签，内部会将多个标签包含在一个Fragment虚拟元素中
    - 好处：减少标签层级，减小内存占用

  - Teleport(传送)

  - Suspense

    - 等待异步组件时渲染一些额外内容，让应用有更好的用户体验，底层也是两个插槽，一个用来展示异步加载内容还没回来的时候，一个用来展示真正要显示的内容

    1. 使用defineAysncComponent异步引入组件
    2. 使用suspense包裹组件，并配置好default与fallback

    ```vue
    <!-- 父组件 -->
    <template>
    	<Suspense>
        <template v-slot:default>
          <child />
        </template>
        <template v-slot:fallback>
          <h3>稍等，加载中...</h3>
        </template>
    	</Suspense>
    </template>
    
    <script>
    // import Child from './components/child' // 静态引入
    import { defineAsyncComponent } from 'vue'
    const Child = defineAsyncComponent(()=>import('./components/child') // 动态(异步)引入
                                       
    export default {
    	components: { Child },                                   
    }
    </script>
    
    <!-- child组件 -->
    <template>
    	<div class="child">
        <h3>我是Child组件</h3>
        {{ sum }}
      </div>
    </template>
    
    <script>
    import { ref } from 'vue'
                                       
    export default {
      // 借助Suspense和异步引入组件，可以async
    	async setup() {
        let sum = ref(0)
        let p = new Promise((resolve, reject)=>{
          setTimeout(()=>{ resolve({ sum })}, 1000)
        })
        return await p
      }                                  
    }
    </script>
    ```

- vue2迁移vue3改变

  - [官网](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88)

## v-for中的key的有什么作用？

- 虚拟DOM中key的作用:
  - key是虚拟DOM对象的标识，当数据发生变化时，Vue会根据【新数据】生成【新的虚拟DOM】，随后Vue进行【新虚拟DOM】与【旧虚拟DOM】的差异比较，比较规则如下：
- 对比规则：
  1. 旧虚拟DOM中找到了与新虚拟DOM相同的key:
     1. 若虚拟DOM中内容没变，直接使用之前的真实DOM！
     2. 若虚拟DOM中内容变了，则生成新的真实DOM，随后替换页面中之前的真实DOM。
  2. 旧虚拟DOM中未找到与新虚拟DOM相同的key，创建新的真实DOM，随后渲染到页面。
- 用index作为key可能会引发的问题：
  1. 若对数据进行逆序添加、逆序删除等破坏顺序操作：会产生没有必要的真实DOM更新  ==》 界面效果没问题，但效率低
  2. 如果结构中还包含输入类的DOM：会产生错误DOM更新 ==》 界面有问题
- 开发中如何选择key?
  1. 最好使用每条数据的唯一标识作为key，比如id、手机号、身份证号、学号等唯一值
  2. 如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的

## 模拟一个数据监测

```js
let data = {
	name: 'jam',
  age: '20'
}

// 对象响应式，vue2基于Object.defineProperty(obj, prop, descriptor)，vue3基于new Proxy(target, handler)
// 错误示范，死循环
Object.defineProperty(data, 'name', {
	get() {
		return data.name;
	},
	set(val) {
		data.name = val;
	}
}
                
// Vue2 创建一个监视的实例对象，用于监视data中的属性的变化
const ob = new Observer(data);
funtion Observer(obj) {
  // 汇总对象中所有的属性形成一个数组
  const keys = Object.keys(obj);
  // 遍历
  keys.forEach(k => {
    // 对属性的读取、修改进行拦截(数据劫持)
    // 存在问题：新增属性、删除属性，界面不会更新
    Object.defaineProperty(this, k, {
      get() {
        return obj[k];
      },
      set(val) {
        obj[k] = val;
      }
    }
  })
}     
// const vm = new Vue({}) 
// vm.data.name = vm._data.name // _data 实际就是一个ob实例对象
                           
// Vue3通过Proxy(代理)：拦截对象中任意属性的变化，包括：属性值的读写、属性的添加、属性的删除等
// 通过Reflect(反射)：对源对象的属性进行操作。ECMA团队正在考虑把Object里的方法往Reflect迁移
// Proxy与Reflect是对立关系，先代理再反射
const ob = new Proxy(obj, {
  // 读取
	get(target, propName) {
    // return target[propName];
    return Reflect.get(target, propName);
  },
  // 新增、修改 post put
  set(target, propName, value) {
    // target[propName] = value;
    Reflect.set(target, propName, value);
  },
  // 删除
  deleteProperty(target, propName) {
    // return delete target[propName];
    return Reflect.deleteProperty(target, propName);
  }
})
```

## 数据双向绑定响应

- Vue会监视数据的原理：

1. vue会监视data中所有层次的数据

2. 如何监测对象中的数据？

   通过setter实现监视，且要在new Vue时就传入要监测的数据。

   a. 对象中后追加的属性，Vue默认不做响应式处理

   b. 如需给后添加的属性做响应式，请使用如下API：

   ​	Vue.set(target, propertyName/index, value) 

   ​	vm.$set(target, propertyName/index, value)

3. 如何监测数组中的数据？

   通过包裹数组更新元素的方法实现，本质就是做了两件事：

   a. 调用原生对应的方法对数组进行更新

   b. 重新解析模板，进而更新页面

4. 在Vue修改数据中的某个元素一定要用如下方法：

   a. 使用这些变更方法API：push()、pop()、shift()、unshift()、splice()、sort()、reverse()，注意：变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 `filter()`、`concat()` 和 `slice()`。它们不会变更原始数组，而**总是返回一个新数组**。当使用非变更方法时，可以用新数组替换旧数组：

   b. Vue.set() 或 vm.$set()

特别注意：Vue.set()和vm.$set()不能给vm或vm的根数据对象添加属性！！！

```js
const vm = new Vue({
    data: {
        name: 'jam',
        info: {
            name: 'xx'
        },
        arrInfo: [{ name: 'xx'}],
        arr: ['吃饭', '学习', '睡觉']
    }
})

this.$set(vm, 'age', 20);  // error
this.$set(vm.data, 'age', 20);  // error
this.$set(vm.data.info, 'age', 20);  
// 数组对象，Object.defineProperty，有get set
vm.data.arrInfo[0].name = 'jam';
// 直接通过下标修改数组，界面不会自动更新
vm.data.arrInfo[0] = { name: 'jam' }; // 不能实时响应
// 如果需要唯一ID，可以借助nanoid库，生成的是21位唯一值

// 数组包装劫持7个原型的变更方法，push、pop、unshift、shift、splice、sort、reverse能够监测到数组数据变化
vm.data.arr[0] = '饿肚子'; // 不能实时响应
vm.data.arr.splice(0, 1, { name: 'jam' }); // 能实时响应
```

## 彩蛋

- 静态服务器，支持history模式 server.js

```js
// 1. npm init -y，安装好express + connect-history-api-fallback依赖
// 2. 创建static存放打包好的静态文件
// 3. node server
const express = require('express')
const history = require('connect-history-api-fallback')

const app = express()
app.use(history)
app.use(express.static(__dirname + '/static'))

app.get('/test', (req, res) => {
  res.send({
    name: 'tom',
    age: 18
  })
})

app.listen(5005, (err) => {
  if (!err) {
    conosle.log('服务器启动成功了！')
  }
}
```





