# Drag Overlay

The `<DragOverlay>` component provides a way to render a draggable overlay that is removed from the normal document flow and is positioned relative to the viewport.

![](../../.gitbook/assets/dragoverlay.png)

## 何时应该使用拖动覆盖？

根据你的使用情况，你可能回去使用拖动覆盖而不是转换原始的可拖动源元素，该元素连接到[`useDraggable`](usedraggable.md)钩子：

* 如果你想要**在拖动时显示预览**可拖动源的位置，你可以在拖动时更新可拖动源的位置，而不会影响拖动覆盖。
* 如果你的项目需要**在拖动时从一个容器移动到另一个容器**，我们强烈建议你使用`<DragOverlay>`组件，这样可拖动的项目就可以在拖动时卸载它的原始容器，而不会影响拖动覆盖。
* 如果你的可拖动项目在**可滚动的容器**中，我们也建议你使用`<DragOverlay>`，否则你需要自己将可拖动的元素设置为`position: fixed`，这样该元素就不会受到其滚动容器的溢出和堆叠上下文的限制，可以在不受其容器滚动位置影响的情况下移动。
* 如果你的`useDraggable`项目在**虚拟列表**中，你绝对会想使用拖动覆盖，因为原始的拖动源在拖动时可以卸载，而虚拟容器被滚动。
* 如果你想要**平滑的下降动画**，而不需要自己构建它们。

## 用法

你可以在`<DragOverlay>`的子元素中渲染任何有效的JSX

`<DragOverlay>`组件应该**始终保持挂载**，以便它可以执行下降动画。如果你有条件地渲染`<DragOverlay>`组件，下降动画将不起作用。

作为一个经验法则，尝试在可拖动的组件之外渲染`<DragOverlay>`，并遵循[展示组件模式](drag-overlay.md#presentational-components)以保持良好的关注点分离。

相反，你应该根据条件来渲染传递给`<DragOverlay>`的子元素：

{% tabs %}
{% tab title="App.jsx" %}
```jsx
import React, {useState} from 'react';
import {DndContext, DragOverlay} from '@dnd-kit/core';

import {Draggable} from './Draggable';


/**
 * <Item> 和 <ScrollableList> 的实现细节对于这个例子来说不是很重要，因此被省略了。
 */
function App() {
  const [items] = useState(['1', '2', '3', '4', '5']);
  const [activeId, setActiveId] = useState(null);
  
  return (
    <DndContext onDragStart={handleDragStart} onDragEnd={handleDragEnd}>
      <ScrollableList>
        {items.map(id =>
          <Draggable key={id} id={id}>
            <Item value={`Item ${id}`} />
          </Draggable>
        )}
      </ScrollableList>
      
      <DragOverlay>
        {activeId ? (
          <Item value={`Item ${activeId}`} /> 
        ): null}
      </DragOverlay>
    </DndContext>
  );
  
  function handleDragStart(event) {
    setActiveId(event.active.id);
  }
  
  function handleDragEnd() {
    setActiveId(null);
  }
}
```
{% endtab %}

{% tab title="Draggable.jsx" %}
```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function Draggable(props) {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <li ref={setNodeRef} {...listeners} {...attributes}>
      {props.children}
    </li>
  );
}
```
{% endtab %}
{% endtabs %}

## 模式

### 展示组件

虽然这是一个可选的模式，但我们建议你打算使可拖动的组件成为[展示组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)，这些组件与`@dnd-kit`解耦。

使用这种模式，创建一个你打算在拖动覆盖中渲染的组件的展示版本，以及一个可拖动的版本，该版本渲染展示组件。

#### 包装节点

如你所见，我们可以创建小的抽象组件，它们渲染一个包装节点，并使任何在可拖动中渲染的子节点：

{% tabs %}
{% tab title="Draggable.jsx" %}
```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function Draggable(props) {
  const Element = props.element || 'div';
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <Element ref={setNodeRef} {...listeners} {...attributes}>
      {props.children}
    </Element>
  );
}
```
{% endtab %}
{% endtabs %}

使用这个模式，我们可以在`<Draggable>`和`<DragOverlay>`中渲染我们的展示组件：

{% tabs %}
{% tab title="App.jsx" %}
```jsx
import React, {useState} from 'react';
import {DndContext, DragOverlay} from '@dnd-kit/core';

import {Draggable} from './Draggable';

/* The implementation details of <Item> is not
 * relevant for this example and therefore omitted. */

function App() {
  const [isDragging, setIsDragging] = useState(false);
  
  return (
    <DndContext onDragStart={handleDragStart} onDragEnd={handleDragEnd}>
      <Draggable id="my-draggable-element">
        <Item />
      </Draggable>
      
      <DragOverlay>
        {isDragging ? (
          <Item />
        ): null}
      </DragOverlay>
    </DndContext>
  );
  
  function handleDragStart() {
    setIsDragging(true);
  }
  
  function handleDragEnd() {
    setIsDragging(false);
  }
}
```
{% endtab %}
{% endtabs %}

#### 引用转发


使用[引用转发模式](https://reactjs.org/docs/forwarding-refs.html) ，我们可以将`<Draggable>`组件的`ref`传递给我们的展示组件：

```jsx
import React, {forwardRef} from 'react';

const Item = forwardRef(({children, ...props}, ref) => {
  return (
    <li {...props} ref={ref}>{children}</li>
  )
});
```

这样，你就可以创建两个版本的组件，一个是展示组件，一个是可拖动的，渲染展示组件**不需要额外的包装元素**：

```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function DraggableItem(props) {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <Item ref={setNodeRef} {...attributes} {...listeners}>
      {value}
    </Item>
  )
});
```

### 传送门

默认情况下，拖动覆盖不是在传送门中渲染的。相反，它是在它被渲染的容器中渲染的。

如果你想在不同的容器中渲染`<DragOverlay>`，请从`react-dom`中导入`createPortal`帮助程序：

```jsx
import React, {useState} from 'react';
import {createPortal} from 'react-dom';
import {DndContext, DragOverlay} from '@dnd-kit/core';

function App() {
  return (
    <DndContext>
      {createPortal(
        <DragOverlay>{/* ... */}</DragOverlay>,
        document.body,
      )}
    </DndContext>
  );
}
```

## Props

```typescript
{
  adjustScale?: boolean;
  children?: React.ReactNode;
  className?: string;
  dropAnimation?: DropAnimation | null;
  style?: React.CSSProperties;
  transition?: string | TransitionGetter;
  modifiers?: Modifiers;
  wrapperElement?: keyof JSX.IntrinsicElements;
  zIndex?: number;
}
```

### 子元素

你可以在`<DragOverlay>`的子元素中渲染任何有效的JSX。但是，请确保在拖动覆盖中渲染的组件不使用`useDraggable`钩子。

首先渲染`<DragOverlay>`的`children`，而不是首先渲染`<DragOverlay>`，否则将不会使用drop动画。

### 类名和内联样式

如果你想自定义`<DragOverlay>`的子元素渲染到的[包装元素](drag-overlay.md#wrapper-element)，请使用`className`和`style`属性：

```jsx
<DragOverlay
  className="my-drag-overlay"
  style={{
    width: 500,
  }}
>
  {/* ... */}
</DragOverlay>
```

### 拖动动画

使用`dropAnimation`属性来配置drop动画。

```typescript
interface DropAnimation {
  duration: number;
  easing: string;
}
```

这个`duration`选项应该是一个数字，以`毫秒`为单位。默认值是`250`毫秒。`easing`选项应该是一个字符串，表示一个有效的[CSS缓动函数](https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function)。默认的缓动是`ease`。

```jsx
<DragOverlay dropAnimation={{
  duration: 500,
  easing: 'cubic-bezier(0.18, 0.67, 0.6, 1.22)',
}}>
  {/* ... */}
</DragOverlay>
```

要禁用drop动画，请将`dropAnimation`属性设置为`null`。

```jsx
<DragOverlay dropAnimation={null}>
  {/* ... */}
</DragOverlay>
```

{% hint style="warning" %}
`<DragOverlay>`组件应该始终保持挂载状态，以便它可以执行drop动画。如果你有条件地渲染`<DragOverlay>`组件，drop动画将不起作用。
{% endhint %}

### 修饰符

修饰符允许您动态修改传感器检测到的运动坐标。它们可以用于各种用例，您可以通过阅读[修饰符](../modifiers.md)文档来了解更多信息。

例如，您可以使用修饰符将`<DragOverlay>`的移动限制在窗口的边界内：

```jsx
import {DndContext, DragOverlay} from '@dnd-kit';
import {
  restrictToWindowEdges,
} from '@dnd-kit/modifiers';

function App() {
  return (
    <DndContext>
      {/* ... */}
      <DragOverlay modifiers={[restrictToWindowEdges]}>
        {/* ... */}
      </DragOverlay>
    </DndContext>
  )
}
```

### 过渡

默认情况下，`<DragOverlay>`组件没有任何过渡，除非由[`Keyboard` sensor](../sensors/keyboard.md)激活。使用`transition`属性创建一个函数，该函数根据[activator event](../sensors/#activators)返回过渡。默认实现是：

```javascript
function defaultTransition(activatorEvent) {
  const isKeyboardActivator = activatorEvent instanceof KeyboardEvent;

  return isKeyboardActivator ? 'transform 250ms ease' : undefined;
};
```

### 包装元素

默认情况下，`<DragOverlay>`组件在`div`元素中渲染你的元素。如果你的可拖动元素是列表项，你将想要更新`<DragOverlay>`组件来渲染一个`ul`包装器，因为没有父`ul`包装一个`li`项是无效的HTML：

```jsx
<DragOverlay wrapperElement="ul">
  {/* ... */}
</DragOverlay>
```

### `z-index`

`zIndex`属性设置拖动覆盖的[z-order](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)。默认值是`999`，出于兼容性的原因，但我们强烈建议您使用较低的值。
