##概述  
  React 刚出来不久，组件还比较少，不像 jquery 那样已经有很多现成的插件了，当时就自己写了一个基于 React 的轮播组件，当然只是一个小 demo，刚刚有用 es6 的语法重新改了改，就想着写一个小教程给新手，如何实现一个 React 的小组件。
  
先放上仓库地址，可以先 clone 来看看代码：https://github.com/TongchengQiu/react-slider。
  react-slider 是一个图片轮播的组件，支持的配置有 图片（必须好不好，要不然轮播毛）、轮播图片的速度、是否自动轮播、自动轮播的时候鼠标放上去是否暂停、自动轮播速度、是否需要前后箭头、是否需要 dot （我不知道怎么表述好，反正意思你懂）。

## 第一步 需求
首先，写一个组件必须先考虑改组件的需求有哪些，支持的配置需要哪些。如上已经说了改组件的需求：
- 轮播的图片
- 配置轮播图片切换的速度
- 可配置是否自动轮播
- 可配置自动轮播的时候鼠标放上去是否暂停
- 可配置自动轮播的速度
- 可配置是否需要前后箭头
- 可配置是否需要 dot （我不知道怎么表述好，反正意思你懂）
这一步先到此为止～～～

## 第二步 构建项目
这里我们是使用 React 框架，当然也是用它的好搭档 webpack 来构建自动化流程咯～😏
不懂 webpack 的配置可以看我的博客关于 webpack 使用的文章，谢谢～😄
这是项目开发目录的文件结构：

<pre>
src
├── Slider              #Slider组件
|   |
|   ├──SliderItem          
|   |   ├──SliderItem.jsx
|   |   └──SliderItem.scss
|   |
|   ├──SliderDots        
|   |   ├──SliderItem.jsx
|   |   └──SliderItem.scss
|   |
|   ├──SliderArrows         
|   |   ├──SliderItem.jsx
|   |   └──SliderItem.scss
|   |
|   ├──Slider.jsx         
|   |
|   └──Slider.scss
|
├──images              #存放demo用的图片文件
|
└── index.js           #demo的入口文件
</pre>

## 第三步 基于需求的开发
这里我们开发组件的模式是按照需求驱动型，回到第一步的需求，我们先不管组件内代码，先写出我们想怎么样配置使用组件：
``` go
const IMAGE_DATA = [
  {
    src: require('./images/demo1.jpg'),
    alt: 'images-1',
  },
  {
    src: require('./images/demo2.jpg'),
    alt: 'images-2',
  },
  {
    src: require('./images/demo3.jpg'),
    alt: 'images-3',
  },
];

render(
  <Slider
    items={IMAGE_DATA}
    speed={1.2}
    delay={2.1}
    pause={true}
    autoplay={true}
    dots={true}
    arrows={true}
  />,
  document.getElementById('root')
);
```
可以看到，在使用 Slider 组件的时候，根据需求，我们可以传入这些属性来配置组件。

一个``` items``` 数组，决定了需要轮播的内容，items 里面每个元素都是一个对象，有 src 和 alt 属性，分别是轮播图片的 src 地址和 alt 内容；
```speed``` 是图片切换的时候的速度时间，需要配置一个 number 类型的数据，决定时间是几秒；
```autoplay``` 是配置是否需要自动轮播，是一个布尔值；
```delay``` 是在需要自动轮播的时候，每张图片停留的时间，一个 number 值；
```pause``` 是在需要自动轮播的时候，鼠标停留在图片上，是否暂停轮播，是一个布尔值；
```dots``` 是配置是否需要轮播下面的小点；
```arrows``` 是配置是否需要轮播的前后箭头；

## 第四步 编写组件
首先我们来考虑一下需要多少个组件，最外层的 Slider 组件是毋庸置疑的了。

根据需求我们可以分出一个 SliderItem 组件，一个 SliderDots，一个 SliderArrows 组件，分别是 轮播每个图片的item，轮播下面dots的组件，左右箭头组件。
我们先来编写每个小组件，组件是对外封闭独立的，所以它只需要对外暴漏属性的配置即可。

### SliderItem
因为 SliderItem 是展示轮播图片的每个内容的，所以它需要的属性有，src 和 alt，因为之前我们已经把每个轮播项写成一个对象了，所以把 src 和 alt 作为属性放在 item，我们只需要一个 item 属性即可；它还需要决定它在父组件中大小，因为在未使用组件的时候我们是不知道轮播项的数目的，所以它的宽度需要根据父组件传给它count（图片数目）来计算。所以它需要的属性有：

- item (有 src 和 alt 属性)
- count （轮播项总数目，计算每个轮播项的宽度）

```go
import React, { Component } from 'react';

export default class SliderItem extends Component {
  constructor(props) {
    super(props);
  }

  render() {
    let { count, item } = this.props;
    let width = 100 / count + '%';
    return (
      <li className="slider-item" style={{width: width}}>
        <img src={item.src} alt={item.alt} />
      </li>
    );
  }
}
```

【let width = 100 / count + '%'; 】即是计算它占父组件的宽度百分比。

### SliderDots
对于 SliderDots 组件，我们需要一个 count（轮播项总数目）来决定显示几个 dot，还需要一个 nowLocal 属性来判断哪个 dot 对应当前显示的轮播项，点击每个 dot 的是否需要一个回调函数 turn 来做出响应。所以它需要的属性有：

- count（轮播项总数目）
- nowLocal（当前的轮播项）
- turn（点击 dot 回调函数）

```go
import React, { Component } from 'react';

export default class SliderDots extends Component {
  constructor(props) {
    super(props);
  }

  handleDotClick(i) {
    var option = i - this.props.nowLocal;
    this.props.turn(option);
  }

  render() {
    let dotNodes = [];
    let { count, nowLocal } = this.props;
    for(let i = 0; i < count; i++) {
      dotNodes[i] = (
        <span
          key={'dot' + i}
          className={"slider-dot" + (i === this.props.nowLocal?" slider-dot-selected":"")}
          onClick={this.handleDotClick.bind(this, i)}>
        </span>
      );
    }
    return (
      <div className="slider-dots-wrap">
        {dotNodes}
      </div>
    );
  }
}
```
代码可以看出，我们用 for 循环根据 count 来决定了需要多少个 dot，然后在每个 dot 绑定函数传入一个 i 值，并且如果这个 dot 对于当前显示的轮播项，就多加一个class slider-dot-selected。每个 dot click 绑定的函数还需要计算需要向前或者向后移动多少个轮播项，然后回调 turn 函数。

### SliderArrows
对于 SliderArrows 组件，我们只需要一个 turn 函数作出回调。废话不多少，代码如下：
```go
import React, { Component } from 'react';

export default class SliderArrows extends Component {
  constructor(props) {
    super(props);
  }

  handleArrowClick(option) {
    this.props.turn(option);
  }

  render() {
    return (
      <div className="slider-arrows-wrap">
        <span
          className="slider-arrow slider-arrow-left"
          onClick={this.handleArrowClick.bind(this, -1)}>
          &lt;
        </span>
        <span
          className="slider-arrow slider-arrow-right"
          onClick={this.handleArrowClick.bind(this, 1)}>
          &gt;
        </span>
      </div>
    );
  }
}
```
这个组件下面有两个箭头，分别是向前和向后，回调的 turn 是 1 和 －1。

### Slider 组件
```go
import React, { Component } from 'react';

require('./Slider.scss');

import SliderItem from './SliderItem/SliderItem';
import SliderDots from './SliderDots/SliderDots';
import SliderArrows from './SliderArrows/SliderArrows';

export default class Slider extends Component {
  constructor(props) {
    super(props);
    this.state = {
      nowLocal: 0,
    };
  }

  // 向前向后多少
  turn(n) {
    var _n = this.state.nowLocal + n;
    if(_n < 0) {
      _n = _n + this.props.items.length;
    }
    if(_n >= this.props.items.length) {
      _n = _n - this.props.items.length;
    }
    this.setState({nowLocal: _n});
  }

  // 开始自动轮播
  goPlay() {
    if(this.props.autoplay) {
      this.autoPlayFlag = setInterval(() => {
        this.turn(1);
      }, this.props.delay * 1000);
    }
  }

  // 暂停自动轮播
  pausePlay() {
    clearInterval(this.autoPlayFlag);
  }

  componentDidMount() {
    this.goPlay();
  }

  render() {
    let count = this.props.items.length;

    let itemNodes = this.props.items.map((item, idx) => {
      return <SliderItem item={item} count={count} key={'item' + idx} />;
    });

    let arrowsNode = <SliderArrows turn={this.turn.bind(this)}/>;

    let dotsNode = <SliderDots turn={this.turn.bind(this)} count={count} nowLocal={this.state.nowLocal} />;

    return (
      <div
        className="slider"
        onMouseOver={this.props.pause?this.pausePlay.bind(this):null} onMouseOut={this.props.pause?this.goPlay.bind(this):null}>
          <ul style={{
              left: -100 * this.state.nowLocal + "%",
              transitionDuration: this.props.speed + "s",
              width: this.props.items.length * 100 + "%"
            }}>
              {itemNodes}
          </ul>
          {this.props.arrows?arrowsNode:null}
          {this.props.dots?dotsNode:null}
        </div>
      );
  }
}

Slider.defaultProps = {
  speed: 1,
  delay: 2,
  pause: true,
  autoplay: true,
  dots: true,
  arrows: true,
  items: [],
};
Slider.autoPlayFlag = null;
```
- 在这里我们先是 import 依赖，以及 需要的 子组件。
- Slider 有一个状态 nowLocal，是表明当前轮播的第几项。
- 前面一直讲 turn 函数，我们就先分析一下这个函数：
```go
turn(n) {
  console.log();
  var _n = this.state.nowLocal + n;
  if(_n < 0) {
    _n = _n + this.props.items.length;
  }
  if(_n >= this.props.items.length) {
    _n = _n - this.props.items.length;
  }
  this.setState({nowLocal: _n});
}
```
它传入一个 参数 n ，决定向前或者向后移动多少个轮播项，向前和向后分别对于 － 和 ＋。

turn 函数内，声明一个 ```_n``` 变量表示下一个轮播的第几项。

如果```_n``` 小于 0 ，那当然是不行的，所以就会让``` _n```变成最后一项；

如果 ```_n``` 大于 items 的长度 ，那当然也是不行的，所以就会让``` _n```变成第一项；

最后改变状态 ```nowLocal``` 等于``` _n```。

- 下面是一个开始自动轮播的函数 goPlay：
```go
goPlay() {
  if(this.props.autoplay) {
    this.autoPlayFlag = setInterval(() => {
      this.turn(1);
    }, this.props.delay * 1000);
  }
}
```
如果 autoplay 是 true，则执行 setInterval 来自动调用 this.turn(1) 向前移动轮播项，this.props.delay * 1000就是根据配置的 delay 来决定多久移动一次。

这里我们需要一个 autoPlayFlag 参数来保存 setInterval 的回调，用来在鼠标停放在图片上停止自动轮播，我们把这个 autoPlayFlag 作为组件的一个属性来保存。
```go
Slider.autoPlayFlag = null;
```
然后在组件初始化 componentDidMount 的时候调用这个函数：

```go
componentDidMount() {
  this.goPlay();
}
```

- 我们还需要一个 pausePlay 函数来暂停自动轮播。
```go 
pausePlay() {
  clearInterval(this.autoPlayFlag);
}
```

- 最后就是 render 了，根据属性配置哪些函数和组件需要～
- 默认属性，这个是需要的，在使用属性的时候如果忘了一些配置项也不会出错。
```go 
Slider.defaultProps = {
  speed: 1,
  delay: 2,
  pause: true,
  autoplay: true,
  dots: true,
  arrows: true,
  items: [],
};
```
## 样式
什么，你告诉我显示在页面上的都是什么鬼？来写 scss 吧：
```css
* {
  margin: 0;
  padding: 0;
}
.slider {
  overflow: hidden;
  width: 100%;
  position: relative;

  &>ul {
    height: auto;
    overflow: hidden;
    position: relative;
    left: 0;
    transition: left 1s;
  }

  .slider-item {
    display: inline-block;
    height: auto;

    &>img {
      display: block;
      height: auto;
      width: 100%;
    }
  }

  .slider-arrow {
    display: inline-block;
    color: #fff;
    font-size: 50px;
    position: absolute;
    top: 50%;
    margin-top: -50px;
    z-index: 100;
    padding: 20px;
    cursor: pointer;
    font-weight: bold;

    &:hover {
      background: rgba(0,0,0,.2);
    }

    &.slider-arrow-right {
      right: 0;
    }
    &.slider-arrow-left {
      left: 0;
    }
  }

  .slider-dots-wrap {
    z-index: 99;
    text-align: center;
    width: 100%;
    position: absolute;
    bottom: 0;

    .slider-dot {
      display: inline-block;
      width: 6px;
      height: 6px;
      border: 3px solid #ccc;
      margin: 6px;
      cursor: pointer;
      border-radius: 20px;

      &:hover {
        border: 3px solid #868686;
      }

      &.slider-dot-selected {
        background: #ccc;
      }
    }
  }
}
```

