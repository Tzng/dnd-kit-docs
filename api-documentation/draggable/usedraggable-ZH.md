# useDraggable

![](../../.gitbook/assets/draggable.png)

## 参数

```typescript
interface UseDraggableArguments {
  id: string | number;
  attributes?: {
    role?: string;
    roleDescription?: string;
    tabIndex?: number;
  },
  data?: Record<string, any>;
  disabled?: boolean;
}
```

### 标识符

参数`id`是一个字符串或数字，应该是唯一标识符，这意味着在给定的[`DndContext`](../context-provider/)提供程序中不应该有其他 **draggable**元素共享相同的标识符。

如果您正在构建一个同时使用`useDraggable`和`useDroppable`钩子的组件，它们可以共享相同的标识符，因为可拖动元素与可拖动元素容器存储在不同的键值存储中。

### 数据

`data`参数用于高级用例，在这些用例中，您可能需要访问在事件处理程序、修饰符或自定义传感器中有关可拖动元素的额外数据。

例如，如果您正在构建可排序的预设，您可以使用`data`属性在可排序列表中存储可拖动元素的索引，以便在自定义传感器中访问它。

```jsx
const {setNodeRef} = useDraggable({
  id: props.id,
  data: {
    index: props.index,
  },
});
```

另一个更高级的例子是，data参数可以在可拖放节点和可拖放容器区域之间创建关系，例如，指定可拖放节点的可拖放节点容器的类型:

```jsx
import {DndContext, useDraggable, useDroppable} from '@dnd-kit/core';

function Droppable() {
  const {setNodeRef} = useDroppable({
    id: 'droppable',
    data: {
      type: 'type1',
    },
  });

  /* ... */
}

function Draggable() {
  const {attributes, listeners, setNodeRef, transform} = useDraggable({
    id: 'draggable',
    data: {
      supports: ['type1', 'type2'],
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

    if (over && active.data.current.supports.includes(over.data.current.type)) {
      // do stuff
    }
  }
}
```

### 禁用

由于[钩子不能被有条件地调用](https://reactjs.org/docs/hooks-rules.html)，所以如果需要临时禁用`draggable`元素，请使用`disabled`参数并将其设置为`true`。

### 属性

`attributes`属性的默认值是有意义的默认值，应该涵盖广泛的用例，但是没有放之四海而皆准的解决方案。

您最了解自己的应用程序，我们鼓励您只手动附加您认为在应用程序上下文中有意义的属性，而不是在不考虑这样做是否有意义的情况下使用它们。

例如，如果要将`useDraggable``listeners`附加到的HTML元素已经是一个本地HTML按钮元素(尽管这样做是无害的)，则不需要添加`role="button"`属性，因为这已经是`button`的默认角色。

#### 角色

ARIA`"role"`属性允许您显式地定义元素的角色，从而将其目的传达给辅助技术。

`"button"`是`"role"`属性的默认值。

如果它在您正在构建的上下文中有意义，我们建议您利用原生HTML`<button>`元素用于可拖动元素。

{% hint style="info" %}

#### 角色说明

可以使用`roleDescription`参数为你的应用程序定制屏幕阅读器体验。例如，如果您正在构建一个可排序的产品列表，您可能会想要将`roleDescription`值设置为类似于`"sortable product"`的值。

**标记索引**

`tabindex`属性规定了焦点在整个文档中移动的顺序。

* 本机交互元素，如[按钮](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)，[锚标签](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)和[表单控件](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormControlsCollection)具有默认的`tabindex`值`0`。

* 用于交互并接收键盘焦点的自定义元素需要显式分配`tabindex="0"`(例如，`div`和`li`元素)

换句话说，为了让您的可拖动元素接收键盘焦点，如果它们不是本机交互元素(例如HTML`button`元素)，则它们**需要**将`tabindex`属性设置为`0`。

出于这个原因，`useDraggable`钩子默认设置`tabindex="0"`属性。

## 属性

```typescript
{
  active: {
    id: UniqueIdentifier;
    node: React.MutableRefObject<HTMLElement>;
    rect: ViewRect;
  } | null;
  attributes: {
    role: string;
    tabIndex: number;
    'aria-diabled': boolean;
    'aria-roledescription': string;
    'aria-describedby': string;
  },
  isDragging: boolean;
  listeners: Record<SyntheticListenerName, Function> | undefined;
  node: React.MutableRefObject<HTMLElement | null>;
  over: {id: UniqueIdentifier} | null;
  setNodeRef(HTMLElement | null): void;
  setActivatorNodeRef(HTMLElement | null): void;
  transform: {x: number, y: number, scaleX: number, scaleY: number} | null;
}
```

### Active

#### `active`

如果在[`DndContext`](../context-provider/)提供程序中当前有一个活动的可拖拽元素，并且使用了`useDraggable`钩子，那么`active`属性将被定义为该可拖拽元素的相应`id`、`node`和`rect`。

否则，`active`属性将被设置为`null`。


#### `isDragging`

如果当前正在拖动的可拖动元素是当前使用`useDraggable`的元素，则`isDragging`属性将为`true`。否则，`isDragging`属性将为false。

在内部，`isActive`属性只是检查`active.id === id`。

### Listeners

`useDraggable`钩子要求您将`listeners`附加到您希望成为启动拖动的激活器的DOM节点。

虽然我们可以将这些侦听器手动附加到提供给`setNodeRef`的节点，但实际上强制消费者手动附加侦听器有许多关键优势。

#### Flexibility

虽然许多拖放库都需要公开“拖动处理”的概念，但使用`useDraggable`钩子创建拖放句柄非常简单，只需手动将侦听器附加到一个不同的DOM元素上，而不是设置为可拖放源DOM节点的DOM元素上:

```jsx
import {useDraggable} from '@dnd-kit/core';


function Draggable() {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: 'unique-id',
  });
  
  return (
    <div ref={setNodeRef}>
      /* Some other content that does not activate dragging */
      <button {...listeners} {...attributes}>Drag handle</button>
    </div>
  );
}
```

当将侦听器附加到与可拖动节点不同的元素时，请确保还将属性附加到已附加侦听器的同一节点，以便仍然[可以访问该节点](../../guides/accessibility.md)。

如果在你的应用上下文中是有意义的，你甚至可以有多个拖拽句柄:

```jsx
import {useDraggable} from '@dnd-kit/core';


function Draggable() {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: 'unique-id',
  });
  
  return (
    <div ref={setNodeRef}>
      <button {...listeners} {...attributes}>Drag handle 1</button>
      /* Some other content that does not activate dragging */
      <button {...listeners} {...attributes}>Drag handle 2</button>
    </div>
  );
}
```

#### 性能

这种策略也意味着我们能够使用[React synthetic events](https://reactjs.org/docs/events.html)，这最终会比手动将事件侦听器附加到每个单独的节点上带来更好的性能。

为什么?因为React不是为每个可拖动的DOM节点附加单独的事件侦听器，而是为我们在`document`上侦听的每种类型的事件附加一个事件侦听器。一旦点击其中一个可拖动的节点，React在文档上的侦听器就会将SyntheticEvent分派回原始处理程序。
### Node

**`setNodeRef`**

为了使`useDraggable`钩子正常工作，它需要将`setNodeRef`属性附加到您打算将其转换为可拖动元素的HTML元素上，以便@dnd-kit可以测量该元素以计算碰撞:

```jsx
function Draggable(props) {
  const {setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <button ref={setNodeRef}>
      {/* ... */}
    </button>
  );
}
```

请记住，`ref`应该分配给您希望变得可拖动的外部容器，但这并不一定需要与侦听器附加到的容器一致。

#### **`node`**

对`setNodeRef`传递的当前节点的[ref](https://reactjs.org/docs/refs-and-the-dom.html)

### Activator

**`setActivatorNodeRef`**

侦听器可能附加到与`setNodeRef` 附加到的节点不同的节点上。

一个常见的例子是实现一个拖拽句柄并将监听器附加到拖拽句柄上:

```jsx
function Draggable(props) {
  const {listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <div ref={setNodeRef}>
      {/* ... */}
      <button {...listeners}>Drag handle</button>
    </div>
  );
}
```

当激活器节点与可拖动节点不同时，我们建议在激活器节点上设置激活器节点ref:

```jsx
function Draggable(props) {
  const {listeners, setNodeRef, setActivatorNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <div ref={setNodeRef}>
      {/* ... */}
      <button ref={setActivatorNodeRef} {...listeners}>Drag handle</button>
    </div>
  );
}
```

这有助于@dnd-kit更准确地处理自动焦点管理，并且传感器也可以访问它以增强激活约束。

焦点管理由@ ddn -kit自动处理。当激活器事件是Keyboard事件时，焦点将自动恢复到激活器节点的第一个可聚焦节点。

如果没有通过setActivatorNodeRef设置激活器节点，焦点将自动恢复到通过setNodeRef注册的可拖动节点的第一个可聚焦节点上。

如果没有通过`setActivatorNodeRef`设置激活器节点，焦点将自动恢复到通过`setNodeRef.`注册的可拖动节点的第一个可聚焦节点上。


### Over

#### **`over`**

如果您想要更改可拖动元素的外观以响应它被拖动到不同的可放置容器中，请检查`over`值是否已定义。

如果可拖动元素被移动到可放置区域，则无论可拖动元素是否处于活动状态，都会为所有可拖动元素定义`over`属性。

如果您想要在可拖动元素被移动到可放置区域时仅对活动可拖动元素进行更改，请检查`isDragging`属性是否为`true`。

### Transform


在拖动项目被拾起后，`transform`属性将被填充为您需要在屏幕上移动项目的`translate`坐标。

`transform`对象遵循以下形状:



```typescript
{
  x: number;
  y: number;
  scaleX: number;
  scaleY: number;
}
```

`x`和`y`坐标表示从可拖动元素的原点开始拖动以来的增量。

`scaleX`和`scaleY`属性表示当前正在拖动的元素与当前正在拖动的元素之间的比例差异，如果可拖动项目需要根据其所在的可放置区域的大小动态调整大小，则这可能很有用。