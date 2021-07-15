# 尚硅谷谷粒商城相关笔记

## 控制台模块

### 人人fast

### 人人VUE

> 导入element-ui相关的模块

1. 在main.js文件中导入相关的模块

```js
import '@/element-ui'                        
import '@/icons'                             
import '@/element-ui-theme'
```

## 网关部分



## 库存部分

- 分类属于库存管理，所属的表：gulimall_pms

> 网关配置

- 将所有请求/api/product/的地址轮询去调用pms地址 

```yaml
gateway:
  routes:
    - id: mall-product_route
      uri: lb://mall-product
      predicates:
        - Path=/api/product/**
      filters:
        - RewritePath=/api/product/?(?<segment>.*), /pms/$\{segment}
```

