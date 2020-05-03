# 基础代码

## 定义一个JavaScript

```javascript
<script>
    //创建vue实例
    var vm =  new Vue({
        el: "#app" //指定这个vue控制的对象
        ,data: { //data中，存放的是vue控制元素里的数据
            msg:"第一个代码！"
            ,msg2: "<h1>第二个代码</h1>"
            ,value2:"测试2"
        }
        ,methods:{
            show:function(){
                alert("methods 里面定义的是方法，v-on指令绑定的事件会在这个里面寻找方法");
            }
        }
    });
</script>
```

## 显示vue中的data数据

```html
<!--new 的vue的实例会控制这个id元素里的所有内容-->
<div id="app">
    <!-- v-cloak 使用此指令，能够解决浏览器网速过慢，插值表达式闪烁问题（先不显示值，显示表达式）-->
    <p v-cloak>{{msg}}</p>
    <!--v-text 效果和插值类似，但是没有闪烁问题-->
    <p v-text="msg"></p>
    <!--v-html 能够解析msg2字符串中的html元素-->
    <div v-html="msg2"></div>
    <!--v-bind会绑定属性，这时,value显示的是vue里面的值-->
    <input type="button" v-bind:value="value2">
    <!--v-bind另一种写法 ：-->
    <input type="button" :value="value2" v-on:click="show">
    <!--v-on另一种写法 @-->
    <input type="button" value="value3" @click="show">
</div>
```

## 事件修饰符

```html
<div id="app">
    <div @click="di">
        <!--如果没有stop这个修饰符，那么，点击button，则会先触发bt点击事件，后触发di点击事件
        加了stop（阻止冒泡）后，只会触发bt点击事件
        -->
        <input value="冒泡" type="button" @click.stop="bt">
        <!--prevent可以阻止默认事件，如a标签的链接，加上后，就不会跳转，只会触发点击事件-->
        <a href="www.baidu.com" @click.stop.prevent="linkbd">打开网页</a>
    </div>
    <!--capture 能捕获事件机制，如此时，点击bt，先执行的是di点击事件-->
    <div @click.capture="di">
        <input value="冒泡" type="button" @click.stop="bt">
    </div>
    <!--self只有点击di才会触发di点击事件，而不能通过冒泡触发-->
    <div @click.self="di">
        <!--once使事件只执行一次-->
        <input value="冒泡" type="button" @click.once="bt">
    </div>
</div>
```

## 双向绑定

```html
<!--    v-bind只能单向绑定，只从js->表单
    v-model 用于双向绑定，当更改inputvalue时，vm里的data中msg也高于改变
    但 v-model只能用于表单元素中
-->

<input type="text" v-model="msg">
```

## vue使用样式

**用v-bind方式绑定，使用数组方式**

```html
<style type="text/css">
    .red{
        color: brown;
    }
    .fon{
        font-size: 30px;
        font-family: 微软雅黑;
    }
</style>
```

```html
<p :class="['red', 'fon']">哈哈哈哈</p>
```

**三元表达式方式**

```html
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
            flag:false
        }
    })
</script>
```

```html
<p :class="['red', flag?'fon':'']">哈哈哈哈</p>
```

**对象方式**

```html
<p :class="{fon:flag, red:flag}">啊啊</p>
```

**内联样式**

```html
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
            sty1:{color: 'brown',  'font-size': '30px'}
        }
    })
</script>
```

```html
<p :style="sty1">aaa</p>
```

## v-for遍历

```html
<div id="app">
    <!--遍历一个数组，i可以不写-->
    <p v-for="(item, i) in list">{{item}} == 索引值 {{i}}</p>
    <!--遍历对象数组-->
    <p v-for="(item, i) in list2">{{item.a}} == {{item.b}} 索引值 {{i}}</p>
    <!--遍历对象-->
    <p v-for="(item, key, i) in list3">{{item}} == {{key}} 索引值 {{i}}</p>
    <!--遍历数字-->
    <p v-for="count in 10">{{count}}</p>
    <label>
        id:<input type="text" v-model="id">
    </label>
    <label>
        name:<input type="text" v-model="name">
    </label>
    <label>
        <input type="button" value="添加" @click="adds">
    </label>
    <div>
        <!--关于v-for key的问题 如果没有key(用bind绑定)，则添加时，选中的值位置不会变-->
        <p v-for="ite in list2" :key="ite.id">
            <input type="checkbox" :value="ite.id">{{ite.id}} -- {{ite.name}}
        </p>
    </div>
</div>
<script src="../lib/vue.js"></script>
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
            list:['aa', 'b', 'v']
            ,list2:[
                {id:'1', name:'我'},{id:'2', name:'他'}
            ]
            ,list3:{
                a:'aa'
                ,b:'bb'
                ,c:'cc'
            }
            ,name:''
            ,id:''
        }
        ,methods:{
            adds(){
                this.list2.unshift({id:this.id, name:this.name});
            }
        }
    })
</script>
```

## v-if和v-show（显示和隐藏）

```html
<div id="app">
    <input value="显示/隐藏" type="button" @click="flag=!flag">
    <!--v-if但flag为false时直接删除元素， v-show直接隐藏，display：none-->
    <h3 v-if="flag">v-if</h3>
    <h3 v-show="flag">v-show</h3>
</div>
<script src="../lib/vue.js"></script>
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
            flag:true
        }
    })
</script>
```

## 键盘事件

**使用默认键盘操作**

```html
<input type="text" class="form-control" v-model="name" @keyup.enter="add">
```

**使用键盘操作码**

```html
<input type="text" class="form-control" v-model="name" @keyup.13="add">
```

**自定义操作**

```html
<input type="text" class="form-control" v-model="name" @keyup.f2="add">
```

```html
<script>
    Vue.config.keyCodes.f2=13;
</script>
```
## 自定义指令

```html
<input type="text" class="form-control" v-model="keywords" v-focus v-color="'blue'" >
```

```html
<script>
    //自定义指令，不需要加v-，使用的时候加v-
    Vue.directive('focus', {
        bind:function(el){ //这些称钩子函数
            //绑定元素的时候执行一次（内存中）,在函数第一个参数为el，表示该元素的原生dom
        }
        ,inserted:function(el){
            //元素插入dom时（页面上）执行一次
            el.focus();
        }
        ,updated:function(){ //vnode更新时执行，会多次触发

        }
    })
    //自定义指令，改变颜色
    Vue.directive('color', {
        bind:function(el){
            el.style.color='red';
        }
    })
    var vm = new Vue({
        el: "#app"
        ,data:{
            produce:[
                {id:1,name:'苹果', date:new Date()}
                ,{id:2,name:'小米', date:new Date()}
            ]
            ,id:''
            ,name:''
            ,keywords:''
        },directives:{ //自定义私有指令
            'color':{
                bind:function(el,binding){
                    el.style.color=binding.value;
                }
            }
            ,'fonsize':function(){ //这个表示下面的代码写在了bind和update中，属于简写
                el.style.fontWeight=900
            }

        }
    })
</script>
```

## 全局过滤器

过滤器只能作用在{{}}或者v-bind中，通常用于对字符的处理

**无参数**

```html
<p>{{ msg | nameFilter}}</p>
```

```html
<script>
    // 语法 ： Vue.filter('过滤器名称', function(|前面传入的参数){ })
    Vue.filter('nameFilter', function(msg){
        return msg.replace(/爱/g, '想');
    })
   
</script>
```

**有参数**

```html
<p>{{ msg | nameFilter1('某某')}}</p>
```

```html
<script>
    //第二个参数为语法中传入的参数
    Vue.filter('nameFilter1', function(msg, arg){
        return msg.replace(/爱/g, '想'+arg);
    })
</script>
```

**多次调用**

```html
<!--输出：我想你想你啊想你啊2    过滤器可以多次从左到右调用-->
<p>{{ msg | nameFilter | nameFilter2('2')}}</p>
```

**私有过滤器**

```html
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{msg:'我爱你爱你啊爱你啊'}
        ,filters:{//私有过滤器
            myFilter(msg,msg2){
                return msg+msg2;
            }
        }
    })
</script>
```

## 过度类名

在进入/离开的过渡中，会有 6 个 class 切换。

1. `v-enter`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
2. `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
3. `v-enter-to`: **2.1.8版及以上** 定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter` 被移除)，在过渡/动画完成之后移除。
4. `v-leave`: 定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
6. `v-leave-to`: **2.1.8版及以上** 定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave` 被删除)，在过渡/动画完成之后移除。



```html
<style type="text/css">
    /*enter:动画进入前的样式， leave-to动画离开后的样式*/
    .v-enter,
    .v-leave-to{
        opacity:0; /*可以看到元素显示时透明度有0 变成1，在元素消失时透明度由1变成0*/
        transform: translateX(100px);/*可以看到元素显示时位移由100变成0，在元素消失时位移由0变成100*/
    }
    .v-enter-active,
    .v-leave-active {
        /*动画作用的元素范围，时间，动画函数*/
        transition: all .8s ease;
    }
</style>
```
```html
<transition>
    <h3 v-if="flag">哈哈哈哈哈</h3>
</transition>
<!--如果不想过度动画的class不时v-开头。则可如此自定义，那么，此class可以以mytr-开头-->
<transition name="mytr">
    <h3 v-if="flag">哈哈哈哈哈</h3>
</transition>
```

# 动画

## 自定义动画

### 使用animate.css自定义动画

```html
<link rel="stylesheet" href="../lib/animate.css">
```

**第一种写法**

使用bounce这两个class，下面的元素class为公共的，
在h3写入class相当于在enter（enter-active-class="bounce animated"）和leave中都写了animated, duration表示动画时间

~~~html
<transition enter-active-class="bounce" leave-active-class="bounceOut" :duration="200">
    <h3 v-if="flag" class="animated">哈哈哈哈哈</h3>
</transition>
~~~

指定入场离场时间

~~~html
<transition enter-active-class="bounce" leave-active-class="bounceOut" :duration="{enter:200, leave:600}">
    <h3 v-if="flag" class="animated">哈哈哈哈哈</h3>
</transition>
~~~

## 半场动画

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>$Title$</title>
    <style>
        .ball {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background-color: blue;
        }
    </style>
</head>
<body>
<div id="app">
    <input type="button" value="变" @click="flag=!flag" />
    <!--定义半场动画对应的方法-->
    <transition
        @before-enter="beforeEnter"
        @enter="enter"
        @after-enter="afterEnter"
    >
        <div class="ball" v-if="flag" ></div>
    </transition>

</div>
<script src="../lib/vue.js"></script>
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
            flag:false
        },methods:{
            //el为原生的
            beforeEnter(el){
                //动画还未开始，让其原地
                el.style.transform="translate(0,0)";
            },enter(el,done){
                el.offsetWidth; //实现渐变，玄学代码，不要问,问就是不知道
                el.style.transform="translate(100px,300px)";
                el.style.transition = 'all 0.8s ease'; //过度时长，过度行为
                done(); //回调函数，让after立即执行
            },afterEnter(el){
                this.flag = false
            }
        }

    })
</script>
</body>
</html>
~~~

# 组件

**第一种方式**

~~~html
<my-f1></my-f1>

<script>
    //第一种方式
    var f1 = Vue.extend({
        template:"<h3>第一种方式</h3>"
    });
    //如果使用了驼峰命名法，则在书写标签时，应该变成-小写这种方式
    Vue.component("myF1", f1);
</script>
~~~

**第二种方式**

~~~html
<myf2></myf2>

<script>
    Vue.component("myf2", {
        template:"<h3>第二种方式 {{msg }}</h3>"
        ,data:function(){
            return {
                msg:"组件自定义data,data必须是方法,方法中必须返回对象"
            }
        }
    });
</script>
~~~

**第三种方式**

~~~html
<myf3></myf3>
~~~

~~~html
<template id="my3">
    <div>
        <h3>第三种方式</h3>
        <h2>自定义组件只能顶层元素只能是唯一一个</h2>
    </div>
</template>
~~~

~~~html
<script>
    //第三种方式,在html定义标签
    Vue.component("myf3",{
        template:"#my3"
    })
</script>
~~~

## 组件自由切换

~~~html
<a href="" @click.prevent="componentId='myf6'">m6</a>
    <a href="" @click.prevent="componentId='myf7'">m7</a>
    <transition mode="out-in"><!--度度动画包裹组件,加一个模式，先出场，再入场，否则会同时执行-->
        <component :is="componentId"></component>
    </transition>
~~~

~~~java
<script>
    Vue.component("myf6",{
        template:"<h3>组件6</h3>"
    })
    Vue.component("myf7",{
        template:"<h3>组件7</h3>"
    })
</script>
~~~

## 组件之间父子传值问题

**方法等变量不能有大写**

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>$Title$</title>
</head>
<body>
<div id="app">
    <!--通过：bind的方式，将msg值传入子组件 -->
    <myf1 :parentmsg="msg" @show="test"></myf1>
</div>
<script src="../lib/vue.js"></script>
<script>
    var vm = new Vue({
        el: "#app"
        ,data:{
           msg:'父元素数据'
        },methods:{
            test(a,b){
                this.msg="子组件传来了"+a+"--"+b;
            }
        },components:{
            myf1:{
                data(){ //组件data
                    return{

                    }
                }, template:"<h3 @click=\"childClick\">子组件通过标签绑定的属性值，取到父组件的值--{{parentmsg}}</h3>"
                ,props:['parentmsg'] //通过属性，取到parent值，注意属性值必须小写
                ,methods:{
                    childClick(){
                        //此处的show必须是组件中v-on绑定的值，调用父方法，进行传值
                        this.$emit("show", "1","2")
                    }
                }
            }
        }
    })
</script>
</body>
</html>
~~~

# 路由

## 路由基础

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<!--new 的vue的实例会控制这个id元素里的所有内容-->
<div id="app">
	<!--router-link 默认渲染为一个a 标签 -->
	<router-link to="/login" tag="span">登录</router-link>
	<router-link to="/register">注册</router-link>
	
	<!-- 这是 vue-router 提供的元素，专门用来 当作占位符的，将来，路由规则，匹配到的组件，就会展示到这个 router-view 中去 -->
    <!-- 所以： 我们可以把 router-view 认为是一个占位符 -->
	<transition mode="out-in">
		<router-view></router-view>
	</transition>
</div>
<script src="../lib/vue.js"></script>
<script src="../lib/vue-router.js"></script>
<script>
	//    组件模板对象
    var login = {
        template:'<h1>登录组件</h1>'
    }
	var register = {
		template:'<h1>注册组件</h1>'
	}
	 var routerObj = new VueRouter({
//        route// 这个配置对象中的 route 表示 【路由匹配规则】 的意思
        routes:[// 路由匹配规则
            // 每个路由规则，都是一个对象，这个规则对象，身上，有两个必须的属性：
            //  属性1 是 path， 表示监听 哪个路由链接地址；
            //  属性2 是 component， 表示，如果 路由是前面匹配到的 path ，则展示 component 属性对应的那个组件
            // 注意： component 的属性值，必须是一个 组件的模板对象， 不能是 组件的引用名称；
            {path:'/',redirect:'/login'},//重定向
            {path:'/login',component:login },
            {path:'/register',component:register },
//            {path:'/',component:login},
        ],
        linkActiveClass:'myactive'
    });
	
    //创建vue实例
    var vm =  new Vue({
        el: "#app" ,//指定这个vue控制的对象
		router: routerObj
    });	
</script>
</body>
</html>

```

## 路由选中

使选中路由的默认class替换为myactive class

```js
linkActiveClass:'myactive'
```



## 路由传值

**第一种**

```html
<!-- 如果在路由中，使用 查询字符串，给路由传递参数，则 不需要修改 路由规则的 path 属性 -->
    <router-link to="/login?id=10&name=zs">登录</router-link>
    <router-link to="/register">注册</router-link>

    <router-view></router-view>
```

```html
<script>
    var login = {
        template:'<h1>登录 --- {{  $route.query.id }} --- {{ $route.query.name }}</h1>',
        data(){
          return {
              msg:'123'
          }
        },
        created(){// 组件的生命周期钩子函数
//            console.log(this.$route);
            console.log(this.$route.query.id)
        }
    }
    var register = {
        template:'<h1>注册</h1>'
    }
    var router = new VueRouter({
        routes:[
            {path:'/login',component:login},
            {path:'/register',component:register},
        ]
    })
    var vm = new Vue({
        el:'#app',
        data:{},
        methods:{},
        router
    })
</script>
```

**第二种**

```html
 <!-- 如果在路由中，使用 查询字符串，给路由传递参数，则 不需要修改 路由规则的 path 属性 -->
    <router-link to="/login/12/ls">登录</router-link>
    <router-link to="/register">注册</router-link>

    <router-view></router-view>
```

```html
<script>
    var login = {
        template:'<h1>登录 --- {{ $route.params.id }} --- {{ $route.params.name }}</h1>',
        data(){
            return {
                msg:'123'
            }
        },
        created(){// 组件的生命周期钩子函数
//            console.log(this.$route);
            console.log(this.$route.params.id)
        }
    }
    var register = {
        template:'<h1>注册</h1>'
    }
    var router = new VueRouter({
        routes:[
            {path:'/login/:id/:name',component:login},
            {path:'/register',component:register},
        ]
    })
    var vm = new Vue({
        el:'#app',
        data:{},
        methods:{},
        router
    })
</script>
```

## 监听文本框值变化

**键盘事件监听**

如果first和last改变，则full也跟着改变

```html
<input type="text" v-model="first" @keyup="firstUp" />
<input type="text" v-model="last" @keyup="lastUp" />
<input type="text" v-model="full" />
```

```html
<script>
    //创建vue实例
    var vm =  new Vue({
        el: "#app" //指定这个vue控制的对象
        ,data:{
			first:""
			,last:""
			,full:""
		},methods:{
			firstUp(){
				this.full=this.first + "|" + this.last;
			},lastUp(){
				this.full=this.first + "|" + this.last;
			}
		}
    });
</script>
```

**watch监视**

当first改变时，打印日志为：
新值：1 原始值
新值：11 原始值1

```html
<input type="text" v-model="first" />
```

```html
<script>
    //创建vue实例
    var vm =  new Vue({
        el: "#app" //指定这个vue控制的对象
        ,data:{
			first:""
			,last:""
			,full:""
		},watch:{
			'first': function(newVal, oldVal){
				console.log("新值："+newVal+" 原始值"+ oldVal);
			}
		}
    });
</script>
```

## 使用computer返回动态值

full是一个动态的属性，由computer进行计算后返回，只要所使用的data中的数据变化，则就会重新计算

```html
<div id="app">
	<input type="text" v-model="first" />
	<input type="text" v-model="last" />
	<input type="text" v-model="full" />
</div>
<script src="../lib/vue.js"></script>
<script>
    //创建vue实例
    var vm =  new Vue({
        el: "#app" ,//指定这个vue控制的对象
		data:{
			first:"",
			last:""
		},computed:{
			'full':function(){
				return this.first + "|" + this.last;
			}
		}
    });
</script>
```

# nrm使用和安装

nrm 是一个 `npm` 源管理器，允许你快速地在 `npm` 源间切换，能够使我们使用国内的一些源

**安装命令**

```cmd
>npm install -g nrm
```

**列表命令**

```cmd
>nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

**使用淘宝源**

```cmd
>nrm use taobao
```

**添加源**

```cmds
nrm add registry http://192.168.10.127:8081/repository/npm-public/
```

# webpack

1 能够处理js文件的相互依赖的关系

2 能够处理js的兼容问题

npm install webpack -g 命令进行安装

## 文件结构

```js
dist 发布的文件
webpack.config.js
src 

	css

	image

	js

	mian.js  js的入口文件

	index.html
```



**初始化**

在对应的项目上执行行命令

```cmd
D:\git\mybook\StuVue\webpack>npm init -y
```

得到package.json文件

**安装js文件**

```cmd
D:\git\mybook\StuVue\webpack>npm install jquery -S
```

得到node_modules文件

**在main.js中导入jquery**

import ** from ** 时ES6的导入模块方法

相当于 const $ = require('jquery')

```js
//导入jquery,从node_modules导入jquery模块，放入$中
import $ from 'jquery'
```

然后在main.js中写js代码

```js
$(function(){
	$('li').css('backgroundColor', '#D97634');
})
```

将main.js进行转化

```cmd
D:\git\mybook\StuVue\webpack>webpack .\src\main.js -o .\dist\bundle.js
```

就可以在html中引用bundle.js了

## webpack.config.js

在项目根目录建立webpack.config.js这个文件

在根目录下命令行输入

webpack

webpack会去解析webpage.config.js得到配置文件中导出的配置对象，然后执行配置对象

```js
const path = require('path');
//npm i html-webpack-plugin -D
//热加载页面会只加载修改页面
const htmlWebpackPlugin = require('html-webpack-plugin');

//使用node模块操作，向外暴漏一个配置对象
module.exports = {
  //入口文件
  entry: path.join(__dirname, './src/main.js'),
  //出口文件
  output: {
    path: path.join(__dirname, './dist'),
    filename: 'bundle.js'
  },
  plugins: [
		//指定模板页面
		new htmlWebpackPlugin({
			template: path.join(__dirname, './src/index.html'),
			//指定生成的页面
			filename: 'index.html'
		})
	]
}
```

## 自动打包编译

当前项目根目录安装webpack-dev-server

```html
npm i webpack-dev-server -D
```

遇到找不到类似文件的错误删除models文件夹

执行

```cmd
npm i
```

webpack-dev-server强制要求在本地项目安装webpage

```cmd
npm i webpack -D
npm i webpack-cli -D
```

遇到类似错误

npm ERR! Refusing to install package with name "webpack" under a package

表明目录下有一个同名 "webpack" 文件，于是找到目录下 package.json 文件，将文件中 "name": "webpack" 改为其他名字即可

在package.json文件中修改如下

```js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server"
  }
```

执行命令

```cmd
npm run dev
```

出现如下

```cmd
｢wds｣: Project is running at http://localhost:8080/
｢wds｣: webpack output is served from /
```

这时，需要访问编译的js需要如此，因为server打包没有写入实际磁盘，只是托管内存中

```html
<script src="../bundle.js"></script>
```

## 配置package.json

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --hot" //配置dev项
  },
```

自动打开浏览器 --open

运行端口 --port 3000

指定打开内容跟路径 --contentBase src

热加载热更新(不会再更新全部的js) --hot

```cmd
webpack-dev-server --open
```

**html-webpack-plugin**

这个插件能使html在内存中使用

```cmd
npm i html-webpack-plugin -D
```

**使用方式**

在配置js中引入插件

```js
const htmlWebpackPlugin = require('html-webpack-plugin');
```

配置插件

```js
plugins: [
		//指定模板页面
		new htmlWebpackPlugin({
			template: path.join(__dirname, './src/index.html'),
			//指定生成的页面
			filename: 'index123.html'
		})
	]
```

访问<http://localhost:8080/index123.html>

在这个内存页面中，我们不需要引入编译过后的js，这个插件会自动帮我们引入

## 处理css

**main.js导入css文件**

```js
import './css/index.css'
```

**项目本地安装loader**

```cmd
>npm install style-loader css-loader -D
```



**webpack.config.js配置相关引入**

使用rules引入loader

test:正则表达式匹配css结尾的文件

use:指定第三方loader，loader调用顺序时从右到左

```js
module: {
		rules: [
			{test: /\.css$/, use:['style-loader','css-loader']},
			//另一种写法
			//{test:/\.css$/,loader:"style-loader!css-loader"}
		]
	}
```

## 处理css中的图片

如果css中含有图片

```css
#iv {
	background-image: url('../image/23018.jpg');
}
```

则需要另load url

安装组件

```cmd
>npm i url-loader file-loader -D
```

配置文件中增加正则匹配

limit：超过这个大小以base64形式显示，否则以地址显示

name：前面用hash值截取8位-原始名称.后缀

url-loader还能处理ttf等字体文件

```js
{test:/\.(jpg|png|gif|bmp)$/, use: 'url-loader?limit=7631&name=[hash:8]-[name].[ext]'}
```

## 引入bootstrap

```cmd
>npm i bootstrap -D
```

引入model中的js

```js
import 'bootstrap/dist/css/bootstrap.css'
```

## ES6新语法 class

先安装两套能将ES6高级语法转化为低级语法的包(需要安装babel-loader@7.1.5)

```cmd
>npm i babel-core babel-loader@7.1.5 babel-plugin-transform-runtime -D

> npm i babel-preset-env babel-preset-stage-0 -D
```

在配置文件添加正则规则

这里需要过滤掉modules文件下的js，只需要匹配项目上的js

```js
{test:/\.js$/,use:'babel-loader',exclude:/node_modules/}
```

在根目录建立一个.babelrc的配置文件

```json
{
	"presets":["env", "stage-0"],
	"plugins": ["transform-runtime"]
}
```

在js文件中写法如下

static关键子表示静态属性，静态属性可以通过类名直接访问

实例属性只能通过类实例访问

```js
class Person{
	static info = {name:'zs', age:12}
}
```

## render渲染vue组件

render能够将创建的组件覆盖vue渲染的div，也就是说渲染以后，没有#app这个div，只有组件了

```html
<div id="app">	
</div>
<script>
	var login = {
		template:"<h1>login组件</h1>"
	}
    //创建vue实例
    var vm =  new Vue({
        el: "#app", //指定这个vue控制的对象
        render:function(createElements){
			return createElements(login);
		}
    });
</script>
```



## webpack使用vue组件

**安装vue**

```cmd
>npm i vue -S
```

**导入vue**

```js
import Vue from 'vue'
```

**安装对应loader**

```html
>npm i vue-loader vue-template-compiler -D
```

**配置人家配置对应规则**

```js
const VueLoaderPlugin = require('vue-loader/lib/plugin');
```

```js
{ test: /\.vue$/, use: 'vue-loader' }
```

```js
plugins: [
		//指定模板页面
		new htmlWebpackPlugin({
			template: path.join(__dirname, './src/index2.html'),
			//指定生成的页面
			filename: 'index123.html'
		}),
        // make sure to include the plugin for the magic
        new VueLoaderPlugin()
	]
```

**建立login.vue文件，在其中写入对应组件**

```js
<template>
	<div>
		<h1>这是登录组件，使用 .vue 文件定义出来的</h1>
	</div>
</template>

<script>
</script>

<style>
</style>
```

**js**

```js
import Vue from 'vue'
```

**引入组件**

```js
import login from './login.vue'
```

```js
var vm = new Vue({
	el:"#app",
	data:{
		msg:'哈哈'
	},
	render:function(c){
		return c(login);
	}
})
```

简写

```js
render: c=> c(login);
```

## ES6 向外导出

在testExport.js中导出，这种方式只能导出一个

```js
var info = {
	name:'zs',
	age:11
}
export default info;
```

在main.js 中 接收

```
import m from './testExport.js';
console.log(m);
```

**这种方式可以导出多个**

```js
export var title = '标题';
export var content = '信息';
```

但是接收的时候必须{ } 接收， 并且名称对应，出发用 as 起别名

```js
import m, {title as t1, content } from './testExport.js';
console.log(m);
console.log(t1);
console.log(content);
```

## 路由组件使用

**安装router组件**

```cmd
npm install vue-router -S
```

**在main.js中引入vue/vue-router**

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);
```

**建立main文件夹，在其下建立两个vue组件文件**

如：

```vue
<template>
		<div>account 组件</div>
</template>

<script>
</script>

<style>
</style>
```

**在main.js中引入组件**

```js
import app from './App.vue';
import account from './main/Account.vue';
import goodslist from './main/goodslist.vue';
```

```js
var router = new VueRouter({
	routes:[
		{path: '/account', component: account},
		{path: '/goodslist', component: goodslist}
	]
})
```

```js
var vm = new Vue({
	el:"#app",
	data:{
		msg:'哈哈'
	},
	/* render:function(c){
		return c(login);
	} */
	//render: c=> c(login)
	render: c=> c(app),
	router	
});
```

**在主组件中建立对应子组件的操作**

```html
<template>
	<div>
		<div>app模板</div>
		<router-link to="/account">a</router-link>
		<router-link to="/goodslist">g</router-link>
		
		<router-view></router-view>
	</div>
</template>

<script>
</script>

<style>
</style>
```

## 路由嵌套

**建立两个子组件**

```vue
<template>
	<div>login 子组件</div>
</template>

<script>
</script>

<style>
</style>
```

```vue
<template>
	<div>regist 子 组件</div>
</template>

<script>
</script>

<style>
</style>
```

**mian.js**

```js
import login from './subcom/login.vue';
import regist from './subcom/regist.vue';
```

```js
{	
    path:"/account",
        component: account,
            children:[
                { path: 'login', component: login},
                { path : 'regist', component: regist}
            ]
}
```

**account.vue**

```vue
<template>
	<div>
		<div>account 组件</div>
		<router-link to="/account/login">login</router-link>
		<router-link to="/account/regist">regist</router-link>
		
		<router-view></router-view>
	</div>
</template>

<script>
</script>

<style>
</style>
```

## 组件中的样式

加上scoped里面的样式为当前组件的局部样式，否则为全局样式

```vue
<style scoped="scoped">
	div {
		color: red;
	}
</style>
```

## 将router抽离main.js

新建router.js文件，将相关router抽入这个文件中

```js
import VueRouter from 'vue-router';

import account from './main/Account.vue';
import goodslist from './main/goodslist.vue';
import login from './subcom/login.vue';
import regist from './subcom/regist.vue';

var router = new VueRouter({
	routes:[
		//{path: '/account', component: account},
		//增加子组件
		{	
			path:"/account",
			component: account,
			children:[
				{ path: 'login', component: login},
				{ path : 'regist', component: regist}
			]
		},
		{path: '/goodslist', component: goodslist}
	]
});

export default router;
```

export default router;表示将router对象暴露出去

在mian.js中导入

```js
import router from './router.js';
```

```js
var vm = new Vue({
	el:"#app",
	data:{
		msg:'哈哈'
	},
	/* render:function(c){
		return c(login);
	} */
	//render: c=> c(login)
	render: c=> c(app),
	router	
});

```

## 使用mui（外部css）

**将mui的dist文件复制到项目lib中，改名mui**

## Vuex

配套的公共数据管理工具，可以把一些共享的数据保存到vuex中，方便整个程序中的任何组件直接获取或者修改我们的公共数据

