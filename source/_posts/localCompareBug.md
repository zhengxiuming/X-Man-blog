---
title: 使用 localCompare 排序 在华为微信端出现的bug
catalog: true
date: 2019-01-09 11:20:09
subtitle:
header-img:
tags:
- bug
catagories:
- JavaScript
---
### 需求及出现的问题
需求：把一系列数组根据数组中的value的首字母进行排序。
问题：在华为微信端会出现只循环一次，然后执行报错。其他浏览器和其他手机都没有问题。

### 出现问题的代码

```
let arr = [{spell:'A'},{spell:'B'},{spell:'C'}....];
arr.sort((a,b)=>{
  return a.spell.localeCompare(b.spell)
})
```
### 如何解决

```
arr.sort((a,b)=>{
  if(a.spell>b.spell){
    return 1;
  }else if(a.spell<b.spell){
    return -1;
  }else{
    return 0;
  }
})
```
