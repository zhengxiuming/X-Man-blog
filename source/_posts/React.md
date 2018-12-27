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
以上只是最基本的调用使用，还要在组件销毁时把 BScroll 实例卸载

## 三、也可以封装成一个 Scroll 组件，把内容放在这个组件里
```
import React from 'react';
import ReactDOM from 'react-dom';
import {PropTypes} from 'prop-types'
import styles from './index.less';

class Scroll extends React.Component{
  constructor (props){
    super(props);
  }
  static defaultProps = {
    click: true, //页面是否可以点击
    refresh: false, // 刷新Scroll
    onScroll: null, // scroll 回调事件,
    scrollTo: null, // 滚动到固定位置
    initScrollTop:null, //滚动到固定位置之后，回调重置scrollTop
    pullingDown: null, // 下拉刷新 回调
    pullingUp: null, // 上拉加载 回调函数
  }
  componentDidUpdate(){
    //数据更新之后，要调用refresh方法一下。
    if(this.bScroll && this.props.refresh === true){
      this.bScroll.refresh()
    }
    //返回顶部时
    if(this.bScroll && this.props.scrollTo === 0 && this.bScroll.y !== 0){
      this.bScroll.scrollTo(0,this.props.scrollTo)
      this.props.initScrollTop()
    }
  }
  componentDidMount(){
    this.scrollView = ReactDOM.findDOMNode(this.refs.scrollView);
    if(!this.bScroll){
      console.log(this.props.pullDownRefresh,this.props.pullUpLoad);
      this.bScroll = new BScroll(this.scrollView,{
        probeType:3,
        click: this.props.click
      })
      // 滑动时间
      if(this.props.onScroll){
        this.bScroll.on("scroll", (scroll) => {
          this.props.onScroll(scroll)
        })
      }
      // 下拉刷新
      if(this.props.pullingDown){
        this.bScroll.on("touchEnd", (pos) => {
          if(pos.y > 200){
            this.props.pullingDown()
          }
        })
      }
      // 上拉加载
      if(this.props.pullingUp){
        this.bScroll.on("scrollEnd", () => {
          if(this.bScroll.y <= (this.bScroll.maxScrollY + 50)){
            this.props.pullingUp()
          }
        })
      }
    }
  }
  componentWillUnmount(){
    this.bScroll.off('scroll');
    this.bScroll = null;
  }
  // 捕获错误
  componentDidCatch(error,info){
    console.log(`componentDidCatch:${error}+${info}`);
  }
  render(){
    return (
      <div className={styles.scrollView} ref="scrollView">
        {this.props.children}
      </div>
    )
  }
}
Scroll.propTypes = {
  click: PropTypes.bool,
  refresh: PropTypes.bool,
  pullingDown:PropTypes.func,
  initScrollTop:PropTypes.func,
  pullingUp:PropTypes.func,
  onScroll: PropTypes.func
}
export default Scroll;

```
### 调用组件
```
import React from 'react';
import Scroll from 'Scroll'
class Content extends React.Component{
  constructor(){
    this.state = {
      refresh: false
    }
  }
  //数据有更新要调用refresh方法
  componentWillReceiveProps(){
    this.setState({
      refresh:true
    })
  }
  //实时滑动事件
  handleScroll(e){
    console.log(e)
  }
  render(){
    return(
      <Scroll 
      refresh={this.state.refresh}
      onScroll={(e)=>{this.handleScroll(e)}}
      
      >
        <div>
          ....
        </div>
      </Scroll>
    )
  }
}

export default Content;
```