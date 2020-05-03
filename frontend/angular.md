# 起步

先学习typescript

## cli开发工具

- 安装nodejs，最起码是6以上， npm 3以上

```shell
# 安装cli
npm i -g @angular/cli 
# 新建项目
ng new angulardemo01
# 进入项目
cd angulardemo01
# 安装依赖
npm i
# 启动项目
ng serve
```

## angular目录结构

src/app --组件

src/assets --静态资源文件

src/environments --环境相关文件

browserslist --angular支持浏览器的配置

app.module.ts文件结构

```typescript
//这个文件是angular的根模块，告诉angular如何组装应用

//浏览器解析模块
import { BrowserModule } from '@angular/platform-browser';

//核心模块
import { NgModule } from '@angular/core';

//根组件
import { AppComponent } from './app.component';
//NgModule装饰器，接收一个元数据对象，告诉angular如何编译和启动应用
@NgModule({
  //当前项目运行的组件
  declarations: [
    AppComponent
  ],
  //当前模块运行依赖的其他模块
  imports: [
    BrowserModule
  ],
  //配置项目所需要的服务
  providers: [],
  //指定应用的主视图（根组件） 通过引导根AppModule来启动应用
  bootstrap: [AppComponent]
})

export class AppModule { }

```

组件组成结构

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root', //使用这个组件的名称
  templateUrl: './app.component.html', //组件html
  styleUrls: ['./app.component.css'] //组件css
})
export class AppComponent {
  title = 'angulardemo01'; //定义属性

  constructor(){
    //构造函数
  }
}
```

# 基本使用

## 创建组件

在components目录下创建news组件，执行这个命令，就可以直接创建组件咯

```shell
ng g component components/news
```

## 定义数据

```typescript
  //指定类型的定义
test:string='';
  //不指定类型的定义
test1='';
//指定类型为任意类型
public test2:any=123
```

## 动态数据获取

- 绑定属性

```html
<div title="我是一个div">

</div>
<!--如果想从ts属性中获取动态的值
 则需要用[]抱起来获取-->
<div [title]="test">

</div>
```

- 绑定属性，并且解析属性里面的html

```typescript
//在ts里面定义一个html字符串
 context="<h2>这是一个html</h2>"
```

```html
<span [innerHTML]="context"></span>
```

- 数组使用

```typescript
 //ts里面定义一个数组
arr=[1,2,3,4,5];
//第二种方式定义数组
arr1:any[]=[1,2,3,4,5];
  //第三种
arr2:Array<any>=[1,2,3,4,5];
```

```html
<!--使用ngFor循环数组， let i=index表示将当前索引赋值给i-->
<ul>
    <li *ngFor="let item of arr; let i=index">{{item}}--{{i}}</li>
</ul>
```

## 静态资源使用

- 本地资源

只有将文件放入assets目录下，就能直接访问到

```html
<img src="assets/image/mmexport.jpg" />
```

## 动态改变class

在css文件中

```css
.red {
    color: red;
}
.blue {
    color: blue;
}
```

在html中，使用[ngclass]控制

```html
<div [ngClass]="{'red': true, 'blue':false}">
    颜色控制
</div>
```

```html
<!--如果不加单引号，则他会识别为数据-->
<div [ngStyle]="{'color': 'red'}">
    颜色控制
</div>
```

## 管道

相当于一个过滤器

- 自带管道

## 事件

键盘事件

$event表示当前键盘事件对象

```html
<input  (keydown)="keydown($event)"/>
```

```typescript
  keydown(e) {
    e.target;//获取当前dom节点
    console.log(e);
  }
```

## 双向绑定mvvm

引入模块

import { FormsModule } from '@angular/forms';

在html中

```html
<!--[]表示绑定属性，()表示绑定事件-->
<input [(ngModel)]="test" type="text" />
{{test}}
```

- select更改

```html
<nz-select formControlName="companyId" nzPlaceHolder="请选择公司"
           (ngModelChange)="selectCompany($event)">
    <nz-option *ngFor="let item of companyList" [nzLabel]="item.name"
               [nzValue]=item.id></nz-option>
</nz-select>
```



## 示例

```html
<h2>人员登记系统</h2>


<div class="people_list">
  <ul>
    <li>姓 名：<input type="text" id="username" [(ngModel)]="peopleInfo.username" value="fonm_input" /></li>
    <li>性 别：   
      <input type="radio" value="1" name="sex" id="sex1" [(ngModel)]="peopleInfo.sex"> <label for="sex1">男 </label>　　　
      <input type="radio" value="2" name="sex"  id="sex2" [(ngModel)]="peopleInfo.sex"> <label for="sex2">女 </label>
    </li>
   <li>     
    城 市：
      <select name="city" id="city" [(ngModel)]="peopleInfo.city">
         <option [value]="item" *ngFor="let item of peopleInfo.cityList">{{item}}</option>
      </select>
    </li>
    <li>    
        爱 好：               
        <span *ngFor="let item of peopleInfo.hobby;let key=index;">
            <input type="checkbox"  [id]="'check'+key" [(ngModel)]="item.checked"/> <label [for]="'check'+key"> {{item.title}}</label>
            &nbsp;&nbsp;
        </span>   
     </li>
     <li>     
       备 注：                
       <textarea name="mark" id="mark" cols="30" rows="10" [(ngModel)]="peopleInfo.mark"></textarea>         
     </li>
  </ul>
  <button (click)="doSubmit()" class="submit">获取表单的内容</button>
  <br>
  <br>
  <br>
  <br>
  <pre>
    {{peopleInfo | json}}
  </pre>
</div>
```

# 服务（公共的方法）

组件里面是没法调用其他组件的方法的，如果想要调用，只能调用公共方法

```shell
#先执行成服务的命令生成服务
ng g service services/storage
```

在服务中写入方法

```typescript
export class StorageService {

  constructor() { }

  public setValue(value){
    localStorage.setItem('1',value);
  }
  public getLog(){
    return localStorage.getItem('1');
  }
}
```



在app.module.ts里面引入服务

```typescript
import { StorageService } from './services/storage.service';
 //配置项目所需要的服务
providers: [StorageService],
```

在对应的js里面使用

```typescript
import { StorageService } from '../../services/storage.service';

//构造方法中使用
  constructor(storage:StorageService) {
      console.log(storage.getLog());
  }
```

# 设置别名

```json
"paths": {
    "@shared/*": [ "src/app/shared/*" ],
    "@service/*": [ "src/app/service/*" ]
}
```

访问示例

```typescript
import {Api} from '@service/Api';
import {Global} from '@service/global.ts';
```



# 操作原生dom

- 原生dom操作

```typescript
  //angular生命周期函数，操作dom在这里操作
  ngAfterViewInit(): void {
    //获取jsdom节点
    let dom:any = document.getElementById('divid');
  }
```

- ViewChild获取dom

在html中

```html
<div #mybox></div>
```

在ts中

```typescript
//引入viewchild
import { Component, OnInit, NgModule, ViewChild } from '@angular/core';

 @ViewChild('mybox', {static: false}) mybox:any;
  constructor(storage:StorageService) {
      console.log(storage.getLog());
  }

//使用dom
//angular生命周期函数，操作dom在这里操作
ngAfterViewInit(): void {
    //获取jsdom节点
    //let dom:any = document.getElementById('divid');
    console.log(this.mybox.nativeElement);
}
```

## 调用子组件方法

- 建立一个新组件head，在ts中写入方法

```typescript
  runlog(){
    console.log('run log');
  }
```

- 在news组件html中引入组件，并定义id=head

```html
<app-head #head></app-head>
<p>我是一个news组件</p>
```

- 在ts中，使用装饰器来定义head

```typescript
@ViewChild('head', {static: false}) head:any;
```

- 在ts中，调用子组件方法

```typescript
  runLog() {
    this.head.runlog();
  }
```

# 组件之间的通信

## 父组件给子组件传值

父组件操作

在ts中定义父组件数据

```typescript
title:string='父组件数据';
```

父组件中html传入

```html
<app-head #head [title]='title'></app-head>
```

子组件接收

```typescript
//引入input
import { Component, OnInit, Input } from '@angular/core';
//定义传入参数，这个title要和父组件属性一致
export class HeadComponent implements OnInit {
  @Input() title:any;
```

在html中使用

```html
<p>头部组件：{{title}}</p>
```

## 调用父组件方法

在父组件中，传入定义好的的方法

```html
<app-head #head [title]='title' [run]="run"></app-head>

```

```typescript
run(){
alert('我是父组件run方法');
}
```

子组件接收并且调用方法

```typescript
@Input() run:any;

runparent(){
    this.run();
}
```

## 将整个父组件传给子组件

父组件传入this

```html
<app-head #head [title]='title' [run]="run" [news]='this'></app-head>
```

子组件接收

```typescript
@Input() news:any;
runparent(){
    alert(this.news.context);
}
```

## 子组件向父组件广播数据

在子组件

```typescript
//引入Output、EventEmitter
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

//广播数据
outerMsg(){
    this.outer.emit('子组件广播的数据');
}
```

在父组件接收数据

```html
<app-head #head [title]='title' [run]="run" [news]='this' (outer)="outerChild($event)"></app-head>

```

```typescript
outerChild(msg){
	alert(msg);
}
```

ps:**在没有关系的组件之间，可以使用服务或者localstorage来进行传值**

# Rxjs

## 异步数据调用

- 回调函数方式

```typescript
//在服务的ts中定义一个异步方法 
getCallBack(res){
    //1000毫秒后将数据放入回调方法中
    setTimeout(() => {
       res("异步方法传过来的数据");
    }, 1000);
  }
```

```typescript
//用过回调函数调用异步方法
run(){
  this.storage.getCallBack((data)=>{
      //通过回调函数接收数据
      alert(data);
  })
}
```

- Promise方式（ES6）

```typescript
//服务主公定义promise方法
getPromise(){
    return new Promise((resolve)=>{
        setTimeout(() => {
            resolve("promise传过来的数据");
        }, 1000);
    });
}
```

```typescript
//通过then的方式调用
run(){
    this.storage.getPromise().then((data)=>{
        alert(data);
    });
}
```

- rxjs 方式

```typescript
//在服务中引入rxjs的模块
import { Observable } from 'rxjs';

getRxjs(){
    return new Observable(observer => {
        setTimeout(() => {
            observer.next('rxjs传过来的数据');
            //如果产生错误
            //observer.error('error');
        }, 1000);
    });
}
```

```typescript
run(){
  //通过订阅的方式获取数据
  this.storage.getRxjs().subscribe(res=>{
      alert(res);
  });
}
```

## Rxjs取消订阅

```typescript
run(){
  //通过订阅的方式获取数据
  var rxjs= this.storage.getRxjs().subscribe(res=>{
      alert(res);
  });
  setTimeout(() => {
    //如果1000毫秒还没执行方法，则取消订阅
    rxjs.unsubscribe();
  }, 1000);
}
```

## Rxjs一直获取数据

```typescript
getIntervalRxjs(){
    let count=0
    return new Observable(observer => {
        //每隔1000毫秒返回一个数据
        setInterval(() => {
            count++
            observer.next('rxjs传过来的数据'+count);
        }, 1000);
    });
}
```

```typescript
runIntervalRxjs(){
  //通过订阅的方式获取数据（一直在获取）
  this.storage.getIntervalRxjs().subscribe(res=>{
      console.log(res);
  });
}
```

## Rxjs fiter map

```typescript
//在组件中引入
import { map, filter } from 'rxjs/operators';

//调用
runMapRxjs(){
  //通过订阅的方式获取数据（一直在获取）
  //map:对数据进行处理后返回
  this.storage.getIntervalRxjs().pipe(map(value=>{
    return value+"哈哈";
  })).subscribe(res=>{
      console.log(res);
  });
}
runFilterRxjs(){
  //通过订阅的方式获取数据（一直在获取）
  //filter:对数据进行过滤
  this.storage.getIntervalRxjs().pipe(filter((value:any)=>{
    if(value%2==0){
      return true;
    }
  })).subscribe(res=>{
      console.log(res);
  });
}
```

# 请求数据

## http get

- app.module.ts中引入服务HttpClientModule

```typescript

import { HttpClientModule } from '@angular/common/http';

//当前模块运行依赖的其他模块
imports: [
    BrowserModule,
    FormsModule,
    HttpClientModule,
],	
```

- 调用ts引入模块HttpClient

```typescript
import { HttpClient} from '@angular/common/http';

constructor(public storage:StorageService,
            public http:HttpClient) {
    console.log(storage.getLog());
}
//调用
testPost(){
  this.http.get('http://127.0.0.1:8080/TestHttp/get').subscribe(res=>{
    console.log(res);
  });
}
```

## http post

```typescript
//引入HttpHeaders
import { HttpClient, HttpHeaders } from '@angular/common/http';

//执行
testPost(){
    
    const headOptions = {
        headers: 
        new HttpHeaders({'Content-Type': 'application/json',
                         'token':"aaaaaaaaaaaaaaa",}),
    }
    //地址，body, head
    this.http.post('http://127.0.0.1:8080/post',{username: 'zhangsan'},headOptions).subscribe(res=>{
        console.log(res);
    });
}

```

## axios整合

```shell
# 安装
npm install axios
```

## get



```typescript
//在服务中引入模块
import Axios from 'axios';

//将异步数据封装到promise中
getAxios(api){
    return new Promise(resolve=>{
        Axios.get(api).then((res)=>{
            resolve(res);//将返回值放入异步方法中
        })
    }); 
}
```

# 路由

建立一个带路由的项目

```shell
## 跳过安装
ng new angulardemo02 --skip-install
```

可以看到，多了个app-routing.module.ts路由配置模块

并且在app.module.ts中也已经引入路由模块

在根html页面引入<router-outlet></router-outlet>标签

## 基本使用

建立两个模块，在app-routing.module.ts中配置地址

```typescript
import { HomeComponent } from './component/home/home.component';
import { NewsComponent } from './component/news/news.component';

const routes: Routes = [
  {path:'home', component:HomeComponent},
  {path:'news', component:NewsComponent},
  //匹配不到路由进入的路由
  {path:'**', redirectTo:'/home'}
];

```

访问<http://localhost:4200/home>时能看到根页面<router-outlet></router-outlet>标签已加载

在页面上链接

```html
<div>
  <a routerLink='/home'>home</a>
  <a [routerLink]="['/news']">news</a>
</div>
<router-outlet></router-outlet>
```

## 路由选中

```html
<a routerLink='/home' routerLinkActive="active">home</a>
<a routerLink='/news' routerLinkActive="active" >news</a>
```

## 路由传值

### get方式

在路由跳转传入对象

```html
<li *ngFor="let item of items">
    <!--   使用queryParams属性来传递值  -->
    <a [routerLink]="['/home']" [queryParams]="{ id:item }" routerLinkActive="active">home:{{item}}</a>
</li>
<a [routerLink]="['/home']" [queryParams]="{ id:id }">home:{{id}}</a>
```

获取值

```typescript
//引入模块
import { ActivatedRoute } from '@angular/router';

  constructor(public router:ActivatedRoute) { }

ngOnInit() {
    //通过订阅获取值
    this.router.queryParams.subscribe(res=>{
        console.log(res);
    })
}
```

### 动态路由

在路由配置处配置参数

```typescript
{path:'news/:id', component:NewsComponent},
```

传值

```html
<a [routerLink]="['/news', id]" routerLinkActive="active" >news</a>

```

接收值

```typescript
import { ActivatedRoute } from '@angular/router';

constructor(public router:ActivatedRoute) { }

ngOnInit() {
    this.router.params.subscribe(res=>{
        console.log(res);
    })
}
```

## js实现路由跳转

### 动态路由调整

```typescript
import { Router } from '@angular/router';
//实现动态路由跳转并且传值
goNews(){
    this.routers.navigate(['/news/', '12345']);
}
```

### get传值跳转路由

```typescript
import { Router, NavigationExtras } from '@angular/router';

gohome(){
    let queryParams:NavigationExtras={
        queryParams:{'id':"1234"}
    }
    this.routers.navigate(['/home/'],queryParams);
}
```

# 父子路由

建立父子路由

```typescript
import { WelcomeComponent } from './component/home/welcome/welcome.component';
import { SettingComponent } from './component/home/setting/setting.component';

const routes: Routes = [
  { path:'home', 
    component:HomeComponent,
    children:[
      {path:'welcome',component:WelcomeComponent},
      {path:'setting',component:SettingComponent},
      {path:'**',redirectTo:'welcome'}
    ],},
  {path:'news/:id', component:NewsComponent},
  //匹配不到路由进入的路由
  {path:'**', redirectTo:'/home'}
];
```

使用路由

```html
<div class="content">
<div class="left">
    <a [routerLink]="[ '/home/welcome']" routerLinkActive="active">欢迎首页</a>
    <br>
    <br>
    <a [routerLink]="[ '/home/setting']" routerLinkActive="active">系统设置</a> 
</div> 
<div class="right"> 
    <router-outlet></router-outlet> 
</div> 
</div>
```

# 自定义模块

创建模块

```shell
ng g module module/user
```

在其下创建组件

```shell
ng g component module/user/component/userlist
```

在user模块下创建自定义user的根组件

```shell
ng g component module/user
```

## 暴露自定义模块的根组件

将根组件export

```typescript
import { UserComponent } from './user.component';

@NgModule({
  //user模块的组件
  declarations: [UserlistComponent, UserComponent],
  //暴露组件，让其他组件可以使用(一般是user的根组件)
  exports:[UserComponent],
  imports: [
    CommonModule
  ]
})
export class UserModule { }
```

在根组件中引用，之后，就能在根组件中使用这个自定义的模块了

```typescript
import { UserComponent } from './module/user/user.component';

  imports: [
    BrowserModule,
    AppRoutingModule,
    UserComponent,
  ],
```

## 通过路由挂载

创建模块并且创建路由

```shell
ng g module module/user1 --routing
```

创建子模块的根组件

```shell
ng g component module/user1
```

app根路由配置子路由,这样实现了懒加载，只有地址匹配到user1才会加载user1的模块

```typescript
const routes: Routes = [
   //指向子模块的路由模块
  {path:"user1", loadChildren:"./module/user1/user1.module#User1Module"}
];

```

