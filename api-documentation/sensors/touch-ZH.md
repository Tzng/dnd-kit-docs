# Touch

触摸传感器响应[触摸事件](https://developer.mozilla.org/en-US/docs/Web/API/Touch\_events)。触摸事件提供了在触摸屏或触控板上解释手指或触控笔活动的能力。

### 激活器

触摸激活器是`onTouchStart`事件处理程序。如果[`event.touches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/touches)属性上没有超过一次触摸，则初始化触摸传感器。

### 激活约束

像[指针](pointer.md)传感器一样，触摸传感器有两个激活约束:

* 等距约束
* 延迟约束

这些激活约束是相互排斥的，不能同时使用。

#### 距离

距离约束订阅以下接口:

```typescript
interface DistanceConstraint {
  distance: number;
}
```

`distance`属性表示在发出拖动开始事件之前触摸输入需要移动的距离，以像素为单位。

#### Delay

延迟约束订阅以下接口:

```typescript
interface DelayConstraint {
  delay: number;
  tolerance: number;
}
```

`delay`属性表示在发出拖动开始事件之前，可拖动项目需要由触摸输入保持的持续时间，以毫秒为单位。

`tolerance`属性表示在拖动操作被中止之前容忍的运动距离，以像素为单位。如果在延迟持续时间内移动手指或触控笔，并且将容差设置为零，则拖动操作将立即中止。如果设置了更高的容差，例如，容差为`5`像素，则只有在延迟期间手指移动超过5像素时，操作才会被中止。

此属性对于触摸输入特别有用，当使用延迟约束时应考虑一些容差，因为触摸输入比鼠标输入精确度低。

### 建议

#### `touch-action`

我们强烈建议您为所有可拖动元素指定`touch-action`CSS属性。

> **`touch-action`** CSS属性设置触摸屏用户（例如，由浏览器内置的缩放功能）如何操作元素的区域。\
> \
> 来源：[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action)

一般来说，当使用触摸传感器时，我们建议您将`touch-action`属性设置为`manipulation`以用于可拖动元素。

{% hint style="info" %}
触摸事件不会像指针事件一样受到限制，可以防止页面在`touchmove`事件中滚动。
{% endhint %}

