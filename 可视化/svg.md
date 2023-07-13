## 渐变色

- 编写渐变时，必须给渐变内容指定一个 id 属性，use 引用需用到。
- 建议渐变内容定义在\<defs>标签内部，渐变通常是可复用的。

1. 线性渐变，是沿着直线改变颜色。

- 第 1 步:在 SVG 文件的 defs 元素内部，创建一个\<linearGradient>节点，并添加 id 属性。
- 第 2 步:在\<linearGradient>内编写几个<stop>节点。
  ✓ 给\<stop> 节点指定位置 offset 属性和 颜色 stop-color 属性，用来指定渐变在特定的位置上应用什么颜色
  ✓ 可通过 stop-opacity 来设置某个位置的半透明度。
  ✓ stop-opacity 和 stop-color 这两个属性值，也可以通过 CSS 来指定。
- 第 3 步:在一个元素的 fill 属性或 stroke 属性中通过 ID 来引用 \<linearGradient> 节点。
  ✓ 比如:属性 fill 属性设置为 url( #Gradient2 )即可。
- 第 4 步(可选):控制渐变方向，通过 ( x1, y1 ) 和 ( x2, y2 ) 两个点控制。
  ✓ (0, 0) (0, 1)从上到下;(0, 0)(1, 0)从左到右。
  ✓ 当然也可以通过 gradientTransform 属性 设置渐变形变。比如: gradientTransform=“rotate(90)” 从上到下。

```js
<svg height='150' width='400' xmlns='http://www.w3.org/2000/svg'>
  <defs>
    <linearGradient id='grad1' x1='0%' y1='0%' x2='100%' y2='0%'>
      <stop offset='0%' style='stop-color:rgb(255,255,0);stop-opacity:1' />
      <stop offset='100%' style='stop-color:rgb(255,0,0);stop-opacity:1' />
    </linearGradient>
  </defs>

  <ellipse cx='200' cy='70' rx='85' ry='55' fill='url(#grad1)' />
</svg>

// <linearGradient>标记的 id 属性定义了这个渐变色的唯一标志
// <linearGradient>标记的 x1, x2, y1,y2 四个属性定义了渐变色的起始位置和终止位置
// 渐变色的颜色组成可以是 2 个或 2 个以上的颜色。每种颜色都使用一个<stop>标记定义。
// offset 属性用来定义渐变色的开始和结束位置,fill 属性定义了需要引用的渐变色的 ID
```

2. 径向渐变

- \<radialGradient>标记用来定义一个径向渐变色

- \<radialGradient>标记必须放置在\<defs>标记内。
- cx,cy 和 r 属性定义了辐射的最大圆，fx 和 fy 属性定义了辐射的焦点

```js
<svg height='150' width='500' xmlns='http://www.w3.org/2000/svg'>
  <defs>
    <radialGradient id='grad1' cx='50%' cy='50%' r='50%' fx='50%' fy='50%'>
      <stop offset='0%' stop-color='rgb(255,255,255)' stop-opacity='0' />
      <stop offset='100%' stop-color='rgb(0,0,255)' stop-opacity='1' />
    </radialGradient>
  </defs>

  <ellipse cx='200' cy='70' rx='85' ry='55' fill='url(#grad1)' />
</svg>
```

## 毛玻璃

### 使用 CSS 的 backdrop-filter 或 filter 属性

backdrop-filter:可以给一个元素后面区域添加模糊效果。
适用于元素背后的所有元素。为了看到效果，必须使元素或其背景至少部分透明。
filter:直接将模糊或颜色偏移等模糊效果应用于指定的元素。

```js

```

### 使用 SVG 的 filter 和 feGaussianBlur 元素(建议少用)

- \<filter>: 元素作为滤镜操作的容器，该元素定义的滤镜效果需要在 SVG 元素上的 filter 属性引用。
  ✓ x ， y, width, height 定义了在画布上应用此过滤器的矩形区域。
  ✓ x， y 默认值为 -10%(相对自身);width ，height 默认 值为 120% (相对自身) 。
- \<feGaussianBlur >:该滤镜专门对输入图像进行高斯模糊
  ✓ stdDeviation 熟悉指定模糊的程度
- \<feOffset> :该滤镜可以对输入图像指定它的偏移量。

```js
<svg width='200' height='200' xmlns='http://www.w3.org/2000/svg'>
  <filter id='blueFilter'>
    <feOffset dx='0' dy='0' />
    <feGaussianBlur stdDeviation='8'></feGaussianBlur>
  </filter>

  <image
    x='0'
    y='0'
    width='200'
    height='200'
    href='../images/avatar.jpeg'
    filter='url(#blueFilter)'
  ></image>
</svg>
```

## 形变

## stroke 描边动画

1. stroke-dasharray 为一个参数时： 其实是表示虚线长度和每段虚线之间的间距
   　　如：stroke-dasharray = ‘10’ 表示：虚线长 10，间距 10，然后重复 虚线长 10，间距 10

两个参数或者多个参数时：一个表示长度，一个表示间距
　　如：stroke-dasharray = ‘10, 5’ 表示：虚线长 10，间距 5，然后重复 虚线长 10，间距 5
　　如：stroke-dasharray = ‘20, 10, 5’ 表示：虚线长 20，间距 10，虚线长 5，接着是间距 20，虚线 10，间距 5，之后开始如此循环

2. stroke-dashoffset： 在 dasharray 模式下路径的偏移量。
   值为 number 类型，除了可以正值，也可以取负值,相对于起始点的偏移，正数相当于往左移动了 x 个长度单位，负数相当于往右移动了 x 个长度单位。

```js
<svg width='300' height='150' xmlns='http://www.w3.org/2000/svg'>
  <polyline
    class='line'
    points='100,20, 200, 20,200,100'
    fill='transparent'
    stroke='red'
    stroke-width='5'
  ></polyline>
</svg>
```

```css
.line {
  stroke-dasharray: 180;
  /* 一开始是不可见的 */
  stroke-dashoffset: 180;
  animation: moveLine 3s linear forwards;
}

@keyframes moveLine {
  100% {
    stroke-dashoffset: 0;
  }
}
```
