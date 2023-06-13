# 传感器

## 概念

传感器是一种抽象，用于检测不同的输入方法，以便启动拖动操作，响应移动并结束或取消操作。

### Activators

传感器可以定义一个或多个**激活器事件**。激活器事件使用React [SyntheticEvent监听器](https://reactjs.org/docs/events.html)，这比手动向每个可拖动节点添加事件监听器具有更好的性能。

一旦检测到一个激活器事件，传感器将被初始化。

### Built-in sensors


内置传感器有:

* [指针](pointer.md)
* [鼠标](mouse.md)
* [触摸](touch.md)
* [键盘](keyboard.md)

### 自定义传感器

如果需要，您还可以实现自定义传感器来响应其他输入，或者如果内置传感器不适合您的需求。如果您构建了自定义传感器并且认为其他人会受益，请随时打开RFC拉取请求。

## 生命周期

传感器的生命周期如下:

* 检测到Activator事件，如果事件合格，则初始化传感器类。
* 传感器在初始化时手动将新的侦听器附加到输入方法。
* 一旦满足约束，传感器将分派拖动开始事件。
* 传感器在响应输入时分派拖动移动事件。
* 传感器分派拖动结束或取消事件。
* 传感器被拆下并清理手动附加的事件侦听器。

从实现的角度来看，传感器是[类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)。

它们是基于类而不是钩子的，因为它们需要立即同步实例化以立即响应用户交互，并且必须可以有条件地调用它们。

## Hooks

### useSensor

默认情况下，`DndContext`使用[Pointer](pointer.md)和[Keyboard](keyboard.md)传感器。

如果您想使用其他传感器，例如鼠标和触摸传感器，请分别使用`useSensor`钩子初始化这些传感器并使用您想要使用的选项

```jsx
import {MouseSensor, TouchSensor, useSensor} from '@dnd-kit/core';

function App() {
  const mouseSensor = useSensor(MouseSensor, {
    // Require the mouse to move by 10 pixels before activating
    activationConstraint: {
      distance: 10,
    },
  });
  const touchSensor = useSensor(TouchSensor, {
    // Press delay of 250ms, with tolerance of 5px of movement
    activationConstraint: {
      delay: 250,
      tolerance: 5,
    },
  });
}
```

### useSensors

使用`useSensor`初始化传感器时，请确保在将它们传递给`DndContext`之前将它们传递给`useSensors`：

```jsx
import {
  DndContext,
  KeyboardSensor,
  MouseSensor,
  TouchSensor,
  useSensor,
  useSensors,
} from '@dnd-kit/core';

function App() {
  const mouseSensor = useSensor(MouseSensor);
  const touchSensor = useSensor(TouchSensor);
  const keyboardSensor = useSensor(KeyboardSensor);
  
  const sensors = useSensors(
    mouseSensor,
    touchSensor,
    keyboardSensor,
  );
  
  return (
    <DndContext sensors={sensors}>
      {/* ... */}
    </DndContext>
  )
}
```

在文档的其他示例中，您还可以看到传感器在没有中间变量的情况下初始化，这与上面的语法等效：

```jsx
import {
  DndContext,
  KeyboardSensor,
  MouseSensor,
  TouchSensor,
  useSensor,
  useSensors,
} from '@dnd-kit/core';

function App() {
  const sensors = useSensors(
    useSensor(MouseSensor),
    useSensor(TouchSensor),
    useSensor(KeyboardSensor),
  );
  
  return (
    <DndContext sensors={sensors}>
      {/* ... */}
    </DndContext>
  )
}
```

