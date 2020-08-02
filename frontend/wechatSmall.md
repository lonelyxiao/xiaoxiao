# 环境准备

## 注册



## 获取Openid

登陆公众平台后

开发->开发者设置->获取openid

## 建立项目

使用微信开发者工具，新建项目

## 新建的项目结构

pages

​	index  首页

​	index.js 首页的逻辑文件

​	index.json

app.js 项目入口文件

app.json 全局配置文件

app.wxss 全局样式文件

## 全局配置app.json

### 如何在项目中新增文件

在app.json中，新增一个配置路径，开发者工具自动生成文件

```json
"pages": [
    "pages/index/index",
    "pages/logs/logs",
    "pages/demo01/demo01"
  ],
```

如何让项目一进入就进入新建的demo01,只需要将demo01放到数组的第一个即可

### window

用于设置小程序的状态栏、导航条、标题、窗口背景色

### 底部tab

```json
 "tabBar": {
    "list": [{
      "pagePath": "pages/index/index",
      "text": "个人中心",
      "iconPath": "icon/center.png",
      "selectedIconPath": "icon/selectcenter.png"
    },{
      "pagePath": "pages/demo01/demo01",
      "text": "设置",
      "iconPath": "icon/set.png",
      "selectedIconPath": "icon/selectset.png"
    }
  ]
},
```

## 页面配置文件

每个路径配置下都有json文件

他的配置，和全局json配置一样，能改变局部的顶部样式

# 数据渲染

## for循环

默认情况

wx:for-item="item" wx:for-index="index"

wx:key="id"，id是数组的唯一值

也可以

wx:key="*this", *this标识数组的当前项

```js
data: {
      list:[
        {id: 1, name: 'xiaoxiao'},
        {id:2, name :'xiao1'}
      ]
  },
```

```html
<view >
    <view wx:for="{{list}}" wx:for-item="item" wx:for-index="index" wx:key="id">
        下标：{{index}}
        数据：{{item.name}}
    </view>
</view>
```

对象也是可以循环的

wx:for-item="对象值" wx:for-index="对象属性"

## block

当需要循环某些数据，就可以使用这个标签，当页面渲染时，会将这个标签去掉

```html
<view >
    <block wx:for="{{list}}" wx:for-item="item" wx:for-index="index" wx:key="id">
        下标：{{index}}
        数据：{{item.name}}
    </block>
</view>
```

## 数据绑定

当text数据改变时，触发handleinput方法

```html
<input type="text" bindinput="handleinput"/>
```

```js
handleinput(e){
    console.log(e)
}
```

当点击button时，触发按钮，并且能获取data中的值

```html
<button bindtap="handletap" data-operation="{{12}}">+</button>
```

```js
handletap(e) {
    console.log(e);
}
```

# WXSS样式

rpx 能够根据屏幕是适配

当屏宽（设计稿）375px时

1rpx=0.5px

假如设计稿存在一个宽度100px，做适配则可以这样

page px = 750 rpx

100px=750rpx*100/page

可以利用css样式calc计算，这里设计稿是375. 注意rpx不能空格

```css
view {
    width: calc(750rpx * 100 / 375);
    height: 200rpx;
    background-color: aquamarine;
}
```

## 样式引入

在wxss文件中

```css

@import "../../style/common.wxss";
```

