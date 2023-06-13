# Draggable

![](../../.gitbook/assets/draggable-large.svg)

使用`useDraggable`钩子将DOM节点转换为可拖拽的源，可以在 [可拖拽的](../droppable/)容器上拾取、移动和放置。

## Usage

`useDraggable`钩子对你的应用应该如何构建并没有特别的固执己见。

### Node ref

但是，您至少需要将`useDraggable`钩子返回的`setNodeRef`函数传递给DOM元素，以便它可以访问底层DOM节点并跟踪它，以[检测](../context-provider/collision-detection-algorithms-ZH.md) 与其他 [可拖放的](../droppable/)元素的碰撞和交集。

```jsx
import {useDraggable} from '@dnd-kit/core';
import {CSS} from '@dnd-kit/utilities';


function Draggable() {
  const {attributes, listeners, setNodeRef, transform} = useDraggable({
    id: 'unique-id',
  });
  const style = {
    transform: CSS.Translate.toString(transform),
  };
  
  return (
    <button ref={setNodeRef} style={style} {...listeners} {...attributes}>
      /* Render whatever you like within */
    </button>
  );
}
```
总是尝试在你的应用上下文中使用最有[语义的](https://developer.mozilla.org/en-US/docs/Glossary/Semantics) DOM元素。查看我们的[无障碍指南](../../guides/accessibility.md)，了解更多关于如何帮助为屏幕阅读器提供更好体验的信息。

### Identifier


`id`参数是一个字符串，应该是唯一标识符，这意味着在给定的 [`DndContext`](../context-provider/) 提供程序中不应该有其他**可拖动**元素共享相同的标识符。

### Listeners

`useDraggable`钩子要求你将`listeners`附加到想要成为拖动激活器的DOM节点上。

虽然我们可以手动将这些监听器附加到提供给`setNodeRef`的节点上，但实际上强制用户手动附加监听器有许多关键优势。

#### Flexibility

虽然许多拖放库需要暴露“拖动句柄”的概念，但使用`useDraggable`钩子创建一个拖动句柄非常简单，只需手动将监听器附加到一个不同的DOM元素上，而不是设置为可拖动源DOM节点:

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

当将监听器附加到与可拖动节点不同的元素上时，请确保将属性也附加到已附加监听器的相同节点上，以便它仍然是[可访问的](../../guides/accessibility.md)。

You can even have multiple drag handles if that makes sense in the context of your application:

如果在你的应用上下文中有意义，你甚至可以有多个拖动句柄:

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


这种策略还意味着我们能够使用[React合成事件](https://reactjs.org/docs/events.html)，这最终导致了与手动将事件监听器附加到每个单独的节点相比的性能提升。

为什么？因为React并不是为每个可拖动的DOM节点附加单独的事件监听器，而是为我们在`document`上监听的每种事件类型附加一个事件监听器。一旦点击了一个可拖动的节点，文档上的React监听器就会将SyntheticEvent分派回原始处理程序。


### Transforms&#x20;


为了实际看到你的可拖动项目在屏幕上移动，你需要使用CSS移动项目。你可以使用内联样式，CSS变量，甚至CSS-in-JS库将`transform`属性作为CSS传递给你的可拖动元素。

为了性能原因，我们强烈建议你使用**`transform`** CSS属性在屏幕上移动你的可拖动项目，因为其他定位属性，如**`top`**，**`left`**或**`margin`**可能会导致昂贵的重绘。了解更多关于[CSS transforms](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)。


在项目开始被拖动后，`transform`属性将被填充为你需要在屏幕上移动项目的`translate`坐标。`transform`对象遵循以下形状: `{x: number, y: number, scaleX: number, scaleY: number}`

`x`和`y`坐标表示从可拖动元素的原点开始拖动时的增量。

`scaleX`和`scaleY`属性表示拖动的项目与当前位于其上的可放置容器之间的比例差异。这对于构建需要适应当前位于其上的可放置容器大小的可拖动项目的界面很有用。

`CSS`帮助程序完全是可选的;它是一个方便的帮助程序，用于生成[CSS transform ](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)字符串，等效于手动构造字符串:

```javascript
CSS.Translate.toString(transform) ===
`translate3d(${translate.x}, ${translate.y}, 0)`
```

### 属性

`useDraggable`钩子****为可拖动的项目提供了一组合理的默认属性。我们建议你将它们附加到要附加可拖动监听器的HTML元素上

我们鼓励你手动附加你认为在你的应用上下文中有意义的属性，而不是在不考虑是否有意义的情况下使用它们。

例如，如果你要将`useDraggable` `listeners`附加到的HTML元素已经是语义`button`，虽然这样做是无害的，但没有必要添加`role="button"`属性，因为这已经是默认的角色。

| 属性                     | 默认值                       | 说明                                                                                                                                                                                                                                                                                                                                                 |
|------------------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `role`                 | `"button"`                | <p>如果可能，我们建议你使用语义<code>&#x3C;button></code>元素为打算附加可拖动监听器的DOM元素设置。 </p><p></p><p>如果这是不可能的，请确保包含默认值<code>role="button"</code>属性。</p>                                                                                                                                                                                                                 |
| `tabIndex`             | `"0"`                     | <br/>为了让可拖动元素接收键盘焦点，如果它们不是原生交互元素(如HTML按钮元素)，则**需要**将`tabindex`属性设置为0。因此，`useDraggable`钩子默认设置`tabindex="0"`属性。                                                                                                                                                                                                                                      |
| `aria-roledescription` | `"draggable"`             | 虽然`draggable`是一个合理的默认值，但我们建议您将此值定制为适合您正在构建的用例的值。                                                                                                                                                                                                                                                                                                   |
| `aria-describedby`     | `"DndContext-[uniqueId]"` | 每个可拖动元素都有一个唯一的`aria-describedby ID`，它指向[屏幕阅读器说明](../context-provider/#screen-reader-instructions)的指令，当可拖动元素获得焦点时，该指令将被读出。 |


要了解更多关于使可拖动接口可访问的最佳实践，请阅读完整的无障碍指南:

{% content-ref url="../../guides/accessibility.md" %}
[accessibility.md](../../guides/accessibility.md)
{% endcontent-ref %}

### Recommendations

#### `touch-action`


我们强烈建议你为所有可拖动元素指定`touch-action` CSS属性。


> The **`touch-action`** CSS property sets how an element's region can be manipulated by a touchscreen user (for example, by zooming features built into the browser).\
> \
> Source: [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action)

一般来说，我们建议你将`touch-action`属性设置为`none`，以防止在移动设备上滚动。

{% hint style="info" %}
对于[Pointer Events,](../sensors/pointer.md)在与可拖动元素交互时，没有办法阻止触摸设备上浏览器的默认行为。使用`touch-action: none;`是阻止指针事件滚动的唯一方法。
{% endhint %}

{% hint style="info" %}
进一步说，使用`touch-action: none;`是目前阻止iOS Safari中的滚动的唯一可靠方法，无论是触摸事件还是指针事件。
{% endhint %}

如果你的可拖动项目是可滚动列表的一部分，我们建议你使用拖动句柄，并将`touch-action`仅设置为`none`，以便列表的内容仍然可以滚动，但是从拖动句柄启动拖动不会滚动页面。

{% hint style="info" %}

只要指针或触摸事件已经被初始化，一旦`pointerdown`或`touchstart`事件被初始化，`touch-action`值的任何更改都将被忽略。从`auto`到`none`的元素的`touch-action`值的编程更改，指针或触摸事件已经被初始化，将不会导致用户代理中止或抑制该事件的任何默认行为，只要该指针处于活动状态（有关详细信息，请参阅[Pointer Events Level 2 Spec](https://www.w3.org/TR/pointerevents2/#determining-supported-touch-behavior)）。
{% endhint %}

## Drag Overlay

`<DragOverlay>`组件提供了一种方法来渲染一个可拖动的叠加层，该叠加层从正常的文档流中删除，并相对于视口进行定位。

![](<../../.gitbook/assets/dragoverlay (1).png>)

要了解更多关于如何使用拖动叠加层的信息，请阅读深入指南:

{% content-ref url="drag-overlay.md" %}
[drag-overlay.md](drag-overlay.md)
{% endcontent-ref %}
