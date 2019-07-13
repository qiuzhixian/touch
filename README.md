# 移动端 touch 事件

-   [事件类型](#事件类型)
    -   [事件类型](#事件类型)
    -   [对象属性的区别](#对象属性的区别)
    -   [属性参数](#属性参数)
-   [使用场景](#使用场景)
    -   [简单的判断滑动方向](#简单的判断滑动方向)
    -   [复杂的拖拽、缩放、旋转 、长按等](#复杂的拖拽、缩放、旋转、长按等)

当手触摸移动设备的时候，会触发 touch 事件，而触摸事件主要有下面 4 大类型组成。
可以通过检查触摸事件的 TouchEvent.type 属性来确定当前事件属于哪种类型。

### 事件类型

-   **touchstart**：触摸开始的时候触发
-   **touchmove** : 触摸结束的时候触发
-   **touchend**: 手指在屏幕上滑动的时候触发
-   **touchcancel**：触摸时由于某些原因被中断时触发

### 对象属性

当我们触发事件的时候会生成一个 event 对象，以下是对象的属性列表

-   **touches**：屏幕上所有触摸点的信息
-   **targetTouches**：目标区域上所有触摸点的信息
-   **changedTouches**：当前事件触摸点的信息

### 对象属性的区别

-   第一根手指触摸屏幕，三个事件获取到的信息是相同的。
-   第二根手指触摸屏幕，touches 会获取两个触摸点的信息，如果触摸点是在同一个目标区域上的 targetTouches 也是获取到两个触摸点信息，而 chengedTouched 只保存第二个信息点。
-   若同时两根手指触摸屏幕，changedTouches 会保存两个触摸点信息。
-   第一个触摸点和最后一个触摸点离开的时候，只有 changedTouches 才会保存离开的触摸点信息。

> 从各属性的区别来看，只有 changedTouches 才会保存触摸点信息，比较适合我们计算一些位置，大小，目标等。

### 属性参数

touch 对象构成的数组，可以用过 event.touches，event.targetTouches,event.targetTouches 来取到。
我们来看下**changedTouches 对象**下属性参数，可以知道当前的位置，大小，形状，压力大小和目标。

```
// 获取event.changedTouches 输出
{
    clientX: 603.6799926757812 // 返回触摸点相对于浏览器视口左边缘的X坐标，不包括任何滚动偏移。
    clientY: 932.9600219726562 // 返回触摸点相对于浏览器视口上边缘的Y坐标，不包括任何滚动偏移。
    force: 1 // 压力大小，是从 0.0(无压力)到 1.0(最大压力)的浮点数
    identifier: 0 // 一次触摸动作的唯一标识符
    pageX: 603.6799926757812 // 返回触摸点相对于文档左边缘的X坐标。与clientX此不同，此值包括水平滚动偏移
    pageY: 932.9600219726562 // 返回触摸点相对于文档顶部的Y坐标。与clientY,此值不同，包括垂直滚动偏移
    radiusX: 30.053333282470703 // 返回椭圆的X半径，该半径最接近地限定与屏幕的接触区域。该值的大小与像素相同screenX。
    radiusY: 30.053333282470703 //返回椭圆的Y半径，该半径最接近地限定与屏幕的接触区域。该值的大小与像素相同screenY。
    rotationAngle: 0 // 它是这样一个角度值：由radiusX 和 radiusY 描述的正方向的椭圆，需要通过顺时针旋转这个角度值，才能最精确地覆盖住用户和触摸平面的接触面
    screenX: 447 // 返回触摸点相对于屏幕左边缘的X坐标。
    screenY: 527 // 返回触摸点相对于屏幕上边缘的Y坐标。
    target: body // 此次触摸事件的目标element
}
```

## 使用场景

### 简单的判断滑动方向

-   **实现方案**：用 touchend 最后滑动的坐标减去 touchstart 的开始坐标，X 坐标是正数为向右滑动，否则向左移动，Y 坐标是正数为向下移动，否则向上。

```javascript
{
	let startX = '';
	let startY = '';
	let endX = '';
	let endY = '';

	// 判断方向，Math.abs取绝对值比较，判断以哪个滑动方向为主
	function direction(startX, endX, startY, endY) {
		return Math.abs(endX - startX) >= Math.abs(endY - startY)
			? endX - startX > 0
				? '右'
				: '左'
			: endY - startY > 0
			? '下'
			: '上';
	}

	// 监听touch触发事件计算距离
	function touchHandle(event) {
		let ev = event || window.event;
		switch (ev.type) {
			case 'touchstart':
				// 触摸开始的时候触发
				startX = ev.changedTouches[0].pageX; // 开始位置的X坐标触点
				startY = ev.changedTouches[0].pageY; // 开始位置Y坐标触点
				break;
			case 'touchend':
				// 触摸结束的时候触发
				endX = ev.changedTouches[0].pageX; // 结束位置的X坐标触点
				endY = ev.changedTouches[0].pageY; // 结束位置的Y坐标触点
				console.log(direction(startX, endX, startX, endY));
				break;
		}
	}

	document.addEventListener('touchstart', touchHandle, false);
	document.addEventListener('touchend', touchHandle, false);
}
```

### 复杂的拖拽、缩放、旋转 、长按等

-   [AlloyFinger](http://www.alloyteam.com/2016/05/super-small-web-gesture-library-alloyfinger-released/) - 轻巧易用的 web 手势库

```javascript
// element为需要监听手势的dom对象
var element = document.querySelector('body');
new AlloyFinger(element, {
	pointStart: function() {
		// 手指触摸屏幕触发
		console.log('手指触摸屏幕触发');
	},
	multipointStart: function() {
		// 一个手指以上触摸屏幕触发
		console.log('一个手指以上触摸屏幕触发');
	},
	rotate: function(evt) {
		// evt.angle代表两个手指旋转的角度
		console.log('旋转的角度:' + evt.angle);
	},
	pinch: function(evt) {
		// evt.scale代表两个手指缩放的比例
		console.log('两个手指缩放的比例:' + evt.scale);
	},
	multipointEnd: function() {
		// 当手指离开，屏幕只剩一个手指或零个手指触发
	},
	pressMove: function(evt) {
		// evt.deltaX和evt.deltaY代表在屏幕上移动的距离
		console.log('X轴移动的距离:' + evt.deltaX);
		console.log('Y抽移动的距离:' + evt.deltaY);
	},
	tap: function(evt) {
		// 点按触发
		console.log('点按触发');
	},
	doubleTap: function(evt) {
		// 双击屏幕触发
		console.log('双击屏幕触发');
	},
	longTap: function(evt) {
		// 长按屏幕750ms触发
		console.log('长按屏幕750ms触发');
	},
	swipe: function(evt) {
		// evt.direction代表滑动的方向
		console.log('滑动的方向：' + evt.direction);
	},
	singleTap: function(evt) {
		// 单击
		console.log('单击');
	},
});
```

-   [hammerjs](http://hammerjs.github.io/) - 支持多点触控的手势库

## 参考

-   [Using Touch Events
    ](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events/Using_Touch_Events)
-   [触摸事件判断滑动方向](https://segmentfault.com/a/1190000015171176)
