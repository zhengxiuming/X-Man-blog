---
title: 关于React中使用better-scroll及普通情况下失效的原因
catalog: true
date: 2018-08-31 16:04:47
subtitle: 'better-scroll'
header-img:
tags:
- React
---
# 关于React中使用better-scroll及普通情况下失效的原因

---------

## 一、一般情况下better-scroll滚动无效的原因
### 1.DOM层级关系
```
<div class="wrapper">
  <div class="content">
    content...
  </div>
</div>
```
依照作者的说法，wrapper容器里会只对第一个子元素有效。如果多个子元素，一定要将滚动的元素放在第一个位置。
### 2.content是否被成功添加滚动相关style
```
<div className={styles.wrapper}>
    <ul className={styles.content}>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</div>
```
在审查元素中可以看到ul上已经有对应的style属性
```
transition-timing-function: cubic-bezier(0.165, 0.84, 0.44, 1);
transition-duration: 0ms;
transform: translate(0px, 0px) scale(1) translateZ(0px);
```
### 3.wrapper高度必须大于滚动元素的高度
let scroll = new BScroll()
console.log(scroll);在控制台可以看到：
```
wrapperHeight:1246 (父级元素的高度)
scrollerHeight：3849（滚动元素高度）
hasVerticalScroll：true (是否可以滚动)

```
以上就是可以滚动的情况，wrapperHeight(1246) < scrollHeight(3849)，hasVerticalScroll为true；
如果这些数据不对，检查是否dom没有更新完就初始化BScroll了，要等dom更新完才能初始化
### 4.wrapper的样式
wrapper要给上定位
```
position: absolute;
left: 0;
top: 0;
overflow: hidden;

或者
position:relative
```

## 二、在React中使用better-scroll
在better-scroll必须在创建完真实DOM之后才能初始化better-scroll,所以在react中需要在componentDidMount或者componentDidUpdate周期中才能初始化better-scroll.
```
componentDidMount(){
    let timer = null;
    if(timer){
      clearTimeout(timer)
    }
    timer = setTimeout(()=>{
      const content = document.querySelector('#wrapper');
      let scroll = new BScroll(content,{});
      console.log(content);
    },0);
    console.log(scroll,'scroll');
}
```


