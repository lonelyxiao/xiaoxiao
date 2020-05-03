# 基础

## helloword

```vue
<template>
  <div>
    <canvas id="canvasOne" width="500" height="300">
      您的浏览器不支持 HTML5 canvas 标签。
    </canvas>
  </div>
</template>

<script>
export default {
  mounted() {
    this.initCanvas();
  },
  methods: {
    initCanvas() {
      //初始化canvas
      //获取canvas对象
      var canvasOne = document.getElementById("canvasOne");
      //需要2d环境才能操作它
      var context = canvasOne.getContext("2d");
      //绘制一个与画布一样大小方块
      //设置颜色
      context.fillStyle="#ffffaa";
      //创建一个矩形
      context.fillRect(0,0,500,300);

      //绘制文字
      //设置文字颜色
      context.fillStyle="#000000";
      //字体
      context.font="20px Sans-Serif";
      //设置字体的垂直对齐方式
      context.textBaseline="top";
      //设置文字和所在坐标
      context.fillText("HelloWord", 193, 80);

      //给矩形设置边框
      context.strokeStyle="#000000";
      //矩形的位置和高度
      //左上角x,y ,宽度，高度
      context.strokeRect(5,5,490,290);
    }
  }
};
</script>
<style>
</style>
```

