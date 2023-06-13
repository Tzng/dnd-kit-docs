# 碰撞检测算法

如果你熟悉2D游戏的制作过程，你可能会遇到碰撞检测算法的概念。

一种更简单的碰撞检测形式是在两个轴对齐的矩形之间进行碰撞检测——这意味着没有旋转的矩形。这种形式的碰撞检测通常被称为[轴对齐边界框](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D\_collision\_detection#Axis-Aligned\_Bounding\_Box)(AABB)。

内置的碰撞检测算法假设一个矩形边界框。

> 元素的边界框是尽可能小的矩形(与该元素的用户坐标系的轴线对齐)，它完全包围了元素及其后代。\
> – 来源: [MDN](https://developer.mozilla.org/en-US/docs/Glossary/bounding\_box)

这意味着即使可拖放或可放节点看起来是圆形或三角形，它们的边界框仍然是矩形的:

![](../../.gitbook/assets/axis-aligned-rectangle.png)

如果你想使用其他形状而不是矩形来检测碰撞，构建你自己的[自定义碰撞检测算法](collision-detection-algorithms-ZH.md#custom-collision-detection-strategies).

## 矩形的交集

默认情况下，[`DndContext`](./)使用**矩形相交**碰撞检测算法。

该算法的工作原理是确保矩形的4个边之间没有间隙。任何间隙都意味着不存在碰撞。

这意味着，为了让一个可拖拽的项目被认为是在一个可拖拽的区域上，两个矩形之间需要有一个交集:

![](../../.gitbook/assets/rect-intersection-1-.png)

## 最近中心

虽然矩形求交算法非常适合于大多数拖放用例，但它可能是不可原谅的，因为它要求可拖放的边界矩形和可放的边界矩形直接接触并相交。

对于某些用例，例如[可排序的](../../presets/sortable/)列表，建议使用更宽容的碰撞检测算法。

顾名思义，最近中心算法查找中心最接近活动可拖放项的边界矩形中心的可拖放容器:

![](../../.gitbook/assets/closest-center-2-.png)

## 最近角

与最近中心算法一样，最近角算法不要求可拖放矩形相交

相反，它会测量活动可拖动项的所有四个角与每个可放置容器的四个角之间的距离，以找到最近的一个。

![](../../.gitbook/assets/closest-corners.png)

距离是从可拖拽项的左上角到可拖放边界矩形的左上角、从右上到右上、从左下到左下、从右下到右下测量的。

### **我应该什么时候使用最近角算法而不是最近中心算法？**

在大多数情况下，**最近中心**算法效果很好，通常是可排序列表的推荐默认值，因为它提供了比**矩形相交算法**更宽容的体验。

通常，最近中心和最近角算法将产生相同的结果。但是，当构建可放置容器叠加在一起的界面时，例如，当构建Kanban时，最近中心算法有时会返回整个Kanban列的底层可放置区域，而不是该列中的可放置区域。

![Closest center is 'A', though the human eye would likely expect 'A2'](../../.gitbook/assets/closest-center-kanban.png)

在那些情况下，**最近角**算法是首选的，它将产生与人眼预测的结果更加一致的结果:

![Closest corners is 'A2', as the human eye would expect.](../../.gitbook/assets/closest-corners-kanban.png)

## 内部指针

顾名思义，内部指针碰撞检测算法仅在指针包含在其他可放置容器的边界矩形内时才注册碰撞。

这个碰撞检测算法非常适合于高精度的拖放界面。

{% hint style="info" %}
顾名思义，此碰撞检测算法**仅适用于基于指针的传感器**。出于这个原因，如果您打算使用`pointerWithin`碰撞检测算法，我们建议您使用[碰撞检测算法的组合](collision-detection-algorithms-ZH.md#composition-of-existing-algorithms)，以便您可以回退到不同的碰撞检测算法，以便使用键盘传感器。
{% endhint %}

## 自定义碰撞检测算法

在高级用例中，如果提供的碰撞检测算法不适合您的用例，您可能希望构建自己的碰撞检测算法。

你可以从头开始编写一个新的碰撞检测算法，也可以组合两个或多个现有的碰撞检测算法。

### 现有算法的组合

有时，您不需要从头开始构建自定义碰撞检测算法。相反，您可以组合现有的碰撞算法来增强它们。

一个常见的例子是使用`pointerWithin`碰撞检测算法。正如它的名字所暗示的，这个碰撞检测算法依赖于指针坐标，因此在使用其他传感器（如键盘传感器）时无法工作。它也是一个非常高精度的碰撞检测算法，因此当`pointerWithin`算法没有返回任何碰撞时，它有时可以帮助回退到更宽容的碰撞检测算法。

```javascript
import {pointerWithin, rectIntersection} from '@dnd-kit/core';

function customCollisionDetectionAlgorithm(args) {
  // 首先，让我们看看是否有与指针的冲突
  const pointerCollisions = pointerWithin(args);
  
  // 碰撞检测算法返回一个碰撞数组
  if (pointerCollisions.length > 0) {
    return pointerCollisions;
  }
  
  // 如果没有与指针碰撞，则返回矩形交叉点
  return rectIntersection(args);
};
```

另外一个组合现有算法的例子是，如果您希望一些可放置容器与其他容器具有不同的碰撞检测算法。

例如，如果您正在构建一个支持将项目移动到垃圾箱的可排序列表，您可能希望组合`closestCenter`和`rectangleIntersection`碰撞检测算法。

![Use the closest corners algorithm for all droppables except 'trash'.](../../.gitbook/assets/custom-collision-detection.png)

![Use the intersection detection algorithm for the 'trash' droppable.](../../.gitbook/assets/custom-collision-detection-intersection.png)

从实现的角度来看，上面例子中描述的自定义交叉算法看起来像这样：

```javascript
import {closestCorners, rectIntersection} from '@dnd-kit/core';

function customCollisionDetectionAlgorithm({
  droppableContainers,
  ...args,
}) {
  // 如果没有与指针碰撞，则返回矩形交叉点
  const rectIntersectionCollisions = rectIntersection({
    ...args,
    droppableContainers: droppableContainers.filter(({id}) => id === 'trash')
  });
  
  // 碰撞检测算法返回一个碰撞数组
  if (rectIntersectionCollisions.length > 0) {
    // The trash is intersecting, return early
    return rectIntersectionCollisions;
  }
  
  // 计算其它碰撞
  return closestCorners({
    ...args,
    droppableContainers: droppableContainers.filter(({id}) => id !== 'trash')
  });
};
```

### 构建自定义碰撞检测算法

对于高级用例或检测非矩形或非轴对齐形状之间的碰撞，您将需要构建自己的碰撞检测算法。

这里有一个[检测圆之间的碰撞](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D\_collision\_detection#Circle\_Collision)的例子，而不是矩形：

```javascript
/**
 * 按降序(从最大值到最小值)排序碰撞。
 */
export function sortCollisionsDesc(
  {data: {value: a}},
  {data: {value: b}}
) {
  return b - a;
}

function getCircleIntersection(entry, target) {
  // 抽象了计算半径的逻辑
  var circle1 = {radius: 20, x: entry.offsetLeft, y: entry.offsetTop};
  var circle2 = {radius: 12, x: target.offsetLeft, y: target.offsetTop};

  var dx = circle1.x - circle2.x;
  var dy = circle1.y - circle2.y;
  var distance = Math.sqrt(dx * dx + dy * dy);

  if (distance < circle1.radius + circle2.radius) {
    return distance;
  }

  return 0;
}

/**
 * 返回交点面积最大的圆
 */
function circleIntersection({
  collisionRect,
  droppableRects,
  droppableContainers,
}) => {
  const collisions = [];

  for (const droppableContainer of droppableContainers) {
    const {id} = droppableContainer;
    const rect = droppableRects.get(id);

    if (rect) {
      const intersectionRatio = getCircleIntersection(rect, collisionRect);

      if (intersectionRatio > 0) {
        collisions.push({
          id,
          data: {droppableContainer, value: intersectionRatio},
        });
      }
    }
  }

  return collisions.sort(sortCollisionsDesc);
};
```
要了解更多信息，请参考内置碰撞检测算法的实现。
