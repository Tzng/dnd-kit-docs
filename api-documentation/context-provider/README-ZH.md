# DndContext

## 应用程序结构

### 内容提供器

为了你的[Droppable](../droppable/)和[Draggable](../draggable/)组件能够相互交互，你需要确保使用它们的React树的一部分嵌套在一个父级`<DndContext>`组件中。`<DndContext>`提供程序利用[React Context API](https://reactjs.org/docs/context.html)在可拖动和可放置组件和钩子之间共享数据。

>React context提供了一种方法，可以在组件树中传递数据，而无需在每个级别手动传递props。 
> 
>因此，使用[`useDraggable`](../draggable/usedraggable.md)、[`useDroppable`](../droppable/usedroppable.md)或[`DragOverlay`](../draggable/drag-overlay.md)的组件需要嵌套在`DndContext`提供程序中。
 
它们不需要是直接的后代，但是，树中的某个地方需要有一个父级`<DndContext>`提供程序。

```jsx
import React from 'react';
import {DndContext} from '@dnd-kit/core';

function App() {
  return (
    <DndContext>
      {/* Components that use `useDraggable`, `useDroppable` */}
    </DndContext>
  );
}
```

### 嵌套

你也许也会在其他`<DndContext>`提供程序中嵌套`<DndContext>`提供程序，以实现独立于彼此的嵌套可拖动/可放置接口。

```jsx
import React from 'react';
import {DndContext} from '@dnd-kit/core';

function App() {
  return (
    <DndContext>
      {/* Components that use `useDraggable`, `useDroppable` */}
      <DndContext>
        {/* ... */}
        <DndContext>
          {/* ... */}
        </DndContext>
      </DndContext>
    </DndContext>
  );
}
```

当嵌套`DndContext`提供程序时，请记住，`useDroppable`和`useDraggable`钩子只能访问该上下文中的其他可拖动和可放置节点。

如果多个`DndContext`提供程序监听相同的事件，则事件将由包含由该事件激活的[传感器](../sensors/)的第一个`DndContext`捕获，类似于[DOM中的事件冒泡](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture)。

## Props

```typescript
interface Props {
  announcements?: Announcements;
  autoScroll?: boolean;
  cancelDrop?: CancelDrop;
  children?: React.ReactNode;
  collisionDetection?: CollisionDetection;
  layoutMeasuring?: Partial<LayoutMeasuring>;
  modifiers?: Modifiers;
  screenReaderInstructions?: ScreenReaderInstructions;
  sensors?: SensorDescriptor<any>[];
  onDragStart?(event: DragStartEvent): void;
  onDragMove?(event: DragMoveEvent): void;
  onDragOver?(event: DragOverEvent): void;
  onDragEnd?(event: DragEndEvent): void;
  onDragCancel?(): void;
}
```

### 事件处理器

你可以从上面的属性列表中看到，`<DndContext>`发出了许多不同的事件，你可以监听并决定如何处理。

你可以监听的主要事件有:

#### `onDragStart`

当满足[传感器](../sensors/)的[激活约束](../sensors/#concepts)的拖动事件发生时，会触发，以及拾起的可拖动元素的唯一标识符。

#### `onDragMove`

随时触发，当[可拖动](../draggable/)项目被移动时。根据激活的[传感器](../sensors/#activators)，例如，当[指针](../sensors/pointer.md)移动时或[键盘](../sensors/keyboard.md)移动键被按下时，这可能会发生。

#### `onDragOver` 

当[可拖动](../draggable/)项目移动到[可放置](../droppable/)容器时，会触发，以及该可放置容器的唯一标识符。

#### `onDragEnd` 

在可拖动项目被放下后触发。

此事件包含有关活动可拖动`id`的信息以及可拖动项目是否被放下`over`的信息。

如果在拖动项目被放下时没有检测到[碰撞](collision-detection-algorithms-ZH.md)，则`over`属性将为`null`。如果检测到碰撞，则`over`属性将包含被放下的可放置的`id`。
{% hint style="info" %}

重要的是要理解`onDragEnd`事件**不会将**[**可拖动**](../draggable/) **项目移动到**[**可放置**](../droppable/) **容器中。**

相反，它提供了有关哪个可拖动项目被放下以及在放下时是否在可放置容器上的**信息**。

由`DndContext`的**消费者**决定如何处理该信息以及如何对其进行反应，例如，通过更新\(或不\)其内部状态以响应事件，以便在不同的父可放置容器中声明性地呈现项目。
#### `onDragCancel`

如果取消拖动操作，则会触发，例如，如果用户在拖动可拖动项目时按下`escape`。
### Accessibility

有关可拖动和可放置组件的可访问性的详细信息和最佳实践，请阅读可访问性部分：
{% page-ref page="../../guides/accessibility.md" %}

#### Announcements

使用`announcements`属性来自定义屏幕阅读器在拖动项目被拾起、移动到可放置区域以及放下时在实时区域中宣布的公告。

默认公告是:
```javascript
const defaultAnnouncements = {
  onDragStart(id) {
    return `Picked up draggable item ${id}.`;
  },
  onDragOver(id, overId) {
    if (overId) {
      return `Draggable item ${id} was moved over droppable area ${overId}.`;
    }

    return `Draggable item ${id} is no longer over a droppable area.`;
  },
  onDragEnd(id, overId) {
    if (overId) {
      return `Draggable item was dropped over droppable area ${overId}`;
    }

    return `Draggable item ${id} was dropped.`;
  },
  onDragCancel(id) {
    return `Dragging was cancelled. Draggable item ${id} was dropped.`;
  },
}
```
虽然这些默认公告是合理的默认值，应该涵盖大多数简单的用例，但是您最了解您的应用程序，我们强烈建议您自定义这些公告，以提供更适合您正在构建的用例的屏幕阅读器体验。

### 屏幕阅读器说明

使用`screenReaderInstructions`属性来自定义屏幕阅读器在焦点移动时读取的说明

### 自动滚动

使用可选的`autoScroll`布尔属性来暂时或永久禁用此`DndContext`中使用的所有传感器的自动滚动。

可以使用传感器的静态属性`autoScrollEnabled`在传感器的个别基础上禁用自动滚动。例如，[键盘传感器](../sensors/keyboard.md)在内部管理滚动，因此将静态属性`autoScrollEnabled`设置为`false`。

### 碰撞检测

使用`collisionDetection`属性来自定义用于检测`DndContext`提供程序中可拖动节点和可放置区域之间碰撞的碰撞检测算法。

默认的碰撞检测算法是[矩形交叉](collision-detection-algorithms-ZH.md#rectangle-intersection)算法。

中文：内置的碰撞检测算法是:

* [矩形的交集](collision-detection-algorithms-ZH.md#rectangle-intersection)
* [最近中心](collision-detection-algorithms-ZH.md#closest-center)
* [最近角](collision-detection-algorithms-ZH.md#closest-corners)

您还可以构建自定义碰撞检测算法或组合现有算法。

要了解更多信息，请阅读碰撞检测指南:

{% page-ref page="collision-detection-algorithms.md" %}

### 传感器

传感器是一种抽象，用于检测不同的输入方法，以便启动拖动操作，响应运动并结束或取消操作。

`DndContext`使用的默认传感器是[Pointer](../sensors/pointer.md)和[Keyboard](../sensors/keyboard.md)传感器。

要了解如何自定义传感器或如何将不同的传感器传递给`DndContext`，请阅读传感器指南:
{% page-ref page="../sensors/" %}

### 编辑器

修饰符允许您动态修改传感器检测到的运动坐标。它们可用于各种用例，例如:
* 限制运动到单个轴
* 限制运动到可拖动节点容器的边界矩形
* 限制运动到可拖动节点的滚动容器边界矩形
* 应用阻力或限制运动

要了解如何使用修饰符，请阅读修饰符指南:

{% page-ref page="../modifiers.md" %}

### 布局测量

中文：您可以通过使用`layoutMeasuring`属性配置`DndContext`应该何时以及如何测量其可放置元素。

`frequency`参数控制应该如何频繁地测量布局。默认情况下，布局测量设置为`optimized`，它只根据`strategy`测量布局。

指定以下策略之一:

* `LayoutMeasuringStrategy.WhileDragging`：默认行为，仅在拖动开始后测量可放置元素。

* `LayoutMeasuringStrategy.BeforeDragging`：在拖动开始之前和拖动结束之后测量可放置元素。

* `LayoutMeasuringStrategy.Always`：在拖动开始之前、拖动开始之后以及拖动结束之后测量可放置元素。

* `LayoutMeasuringStrategy.Always`：在拖动开始之前、拖动开始之后以及拖动结束之后测量可放置元素。
示例用法:

```jsx
import {DndContext, LayoutMeasuringStrategy} from '@dnd-kit/core';

<DndContext layoutMeasuring={{strategy: LayoutMeasuringStrategy.Always}} />
```

