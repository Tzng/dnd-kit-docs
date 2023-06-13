# useDroppable

![](../../.gitbook/assets/droppable-1-.png)

## 参数

```typescript
interface UseDroppableArguments {
  id: string | number;
  disabled?: boolean;
  data?: Record<string, any>;
}
```

### 标识符

`id`参数是一个`string` 或`number`，应该是一个唯一的标识符，这意味着在给定的[`DndContext`](../context-provider/) 提供程序中，不应该有其他**可拖放**元素共享相同的标识符。

如果你正在构建一个同时使用`useDroppable`和`useDraggable`钩子的组件，它们可以共享相同的标识符，因为可拖放元素与可拖拽元素存储在不同的键值存储区。

### 禁用

由于[hooks不能有条件地调用](https://reactjs.org/docs/hooks-rules.html)，所以使用`disabled`参数，并在需要临时禁用可放置区域时将其设置为`true`。

### 数据

`data`参数是用于高级用例的，您可能需要在事件处理程序、修饰符或自定义传感器中访问有关可拖放元素的其他数据。

例如，如果您正在构建一个可排序的预设，您可以使用`data`属性将可拖放元素的索引存储在可排序列表中，以便在自定义传感器中访问它。

```jsx
const {setNodeRef} = useDroppable({
  id: props.id,
  data: {
    index: props.index,
  },
});
```

另一个更高级的例子，`data`参数可以很有用的是创建可拖动节点和可放置区域之间的关系，例如，指定可放置区域接受哪些类型的可拖动节点：

```jsx
import {DndContext, useDraggable, useDroppable} from '@dnd-kit/core';

function Droppable() {
  const {setNodeRef} = useDroppable({
    id: 'droppable',
    data: {
      accepts: ['type1', 'type2'],
    },
  });

  /* ... */
}

function Draggable() {
  const {attributes, listeners, setNodeRef, transform} = useDraggable({
    id: 'draggable',
    data: {
      type: 'type1',
    },
  });

  /* ... */
}

function App() {
  return (
    <DndContext onDragEnd={handleDragEnd}>
      /* ... */
    </DndContext>
  );
  
  function handleDragEnd(event) {
    const {active, over} = event;

    if (over && over.data.current.accepts.includes(active.data.current.type)) {
      // do stuff
    }
  }
}
```

## Properties

```typescript
{
  rect: React.MutableRefObject<LayoutRect | null>;
  isOver: boolean;
  node: React.RefObject<HTMLElement>;
  over: {id: UniqueIdentifier} | null;
  setNodeRef(element: HTMLElement | null): void;
}
```

### Node

#### `setNodeRef`

为了使`useDroppable`钩子正常工作，它需要将`setNodeRef`属性附加到您打算将其转换为可放置区域的HTML元素上：

```jsx
function Droppable(props) {
  const {setNodeRef} = useDroppable({
    id: props.id,
  });
  
  return (
    <div ref={setNodeRef}>
      {/* ... */}
    </div>
  );
}
```

#### `node`

对传递给`setNodeRef`的当前节点的[ref](https://reactjs.org/docs/refs-and-the-dom.html)

#### `rect`

对于高级用例，如果您需要可放置区域的边界矩形测量。

### Over

#### `isOver`

使用`useDroppable`钩子返回的`isOver`布尔值来更改当`draggable`元素被拖动到您的可放置容器上时显示的外观或内容。&#x20;

#### `over`


如果您想要在可拖动的元素被拖动到不同的可放置容器上时更改可放置的外观，请检查`over`值是否已定义。根据您的用例，您还可以读取可拖动项的其他可放置项的`id`，以便更改可放置组件的渲染输出。

```jsx
####
