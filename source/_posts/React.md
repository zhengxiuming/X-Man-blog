---
title: React中使用better-scroll
catalog: true
date: 2018-08-31 16:04:47
subtitle: 'better-scroll'
header-img: "img/header_img/sky.jpg"
tags:
- React
---
# React中使用better-scroll

---------

## 一、better-scroll滚动无效的原因
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

## 三、封装成一个 Scroll 组件，把内容放在这个组件里
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
    click: true, //页面是否可以点击,
    tap: true,
    refresh: false, // 刷新Scroll
    onScroll: null, // scroll 回调事件,
    scrollTo: null, // 滚动到固定位置
    initScrollTop:null, //滚动到固定位置之后，回调重置scrollTop
    pullingDown: null, // 下拉刷新 回调
    pullingUp: null, // 上拉加载 回调函数
    scrollToEle: null,
  }
  componentDidUpdate(){
    if(this.bScroll && this.props.refresh === true){
      this.bScroll.refresh()
    }
    if(this.bScroll && this.props.scrollTo === 0 && this.bScroll.y !== 0){
      this.bScroll.scrollTo(0,this.props.scrollTo)
      this.props.initScrollTop()
    }
    if(this.bScroll && this.props.scrollToEle){
      this.bScroll.scrollToElement(this.props.scrollToEle,200)
      this.props.initScrollTop()
    }
  }
  componentDidMount(){
    this.scrollView = ReactDOM.findDOMNode(this.refs.scrollView);
    if(!this.bScroll){
      console.log(this.props.pullDownRefresh,this.props.pullUpLoad);
      this.bScroll = new BScroll(this.scrollView,{
        probeType:3,
        click: this.props.click ? this.iScrollClick(): false,
        taps: this.props.tap
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
  /**
   * 解决ios上需要点击两次才能触发点击事件
   *  */
  iScrollClick(){
    if (/iPhone|iPad|iPod|Macintosh/i.test(navigator.userAgent)) return false;
    if (/Chrome/i.test(navigator.userAgent)) return (/Android/i.test(navigator.userAgent));
    if (/Silk/i.test(navigator.userAgent)) return false;
    if (/Android/i.test(navigator.userAgent)) {
       var s=navigator.userAgent.substr(navigator.userAgent.indexOf('Android')+8,3);
       return parseFloat(s[0]+s[3]) < 44 ? false : true
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
index.less
```
.scrollView{
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}
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
> 在IOS端遇到的兼容性问题

 better-scroll 默认会阻止浏览器的原生click事件，我们需要将click属性设为true。
 问题来了，在IOS端和浏览器模拟器中，即使我们设为false也是可以触发click事件的。如果设为true，我们在IOS端和浏览器模拟器中就需要点击两次才能触发click事件。
 根据以上特性我们只能根据设备的和浏览器版本信息，动态设置click的值。
 
 ```
 iScrollClick(){
    if (/iPhone|iPad|iPod|Macintosh/i.test(navigator.userAgent)) return false;
    if (/Chrome/i.test(navigator.userAgent)) return (/Android/i.test(navigator.userAgent));
    if (/Silk/i.test(navigator.userAgent)) return false;
    if (/Android/i.test(navigator.userAgent)) {
       var s=navigator.userAgent.substr(navigator.userAgent.indexOf('Android')+8,3);
       return parseFloat(s[0]+s[3]) < 44 ? false : true
    }
  }
 ```

> 最后

我想说的是不管我们使用css的-webkit-overflow-scrolling:touch还是better-scroll（其他iScroll.js等其他第三方库），都会有想不到的问题，实际中我们还是要针对我们自己的项目来选择使用什么方式。