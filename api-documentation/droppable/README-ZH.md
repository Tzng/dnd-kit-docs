# Droppable

![](../../.gitbook/assets/droppable-large.svg)

使用`useDroppable`钩子将DOM节点设置为可放置区域，以便可以放置 [可拖动的](../draggable/) 元素。

## Usage

`useDroppable`钩子对你的应用应该如何构建并没有特别的固执己见。

但是，你至少需要将`useDroppable`钩子返回的`setNodeRef`函数传递给一个DOM元素，以便它可以注册底层DOM节点并跟踪它以检测与其他可拖动元素的碰撞和交叉。

{% hint style="info" %}

如果对`ref`的概念不熟悉，我们建议你先查看React文档网站上的[Refs and the DOM article](https://reactjs.org/docs/refs-and-the-dom.html#adding-a-ref-to-a-dom-element)。

{% endhint %}

```jsx
import {useDroppable} from '@dnd-kit/core';


function Droppable() {
  const {setNodeRef} = useDroppable({
    id: 'unique-id',
  });
  
  return (
    <div ref={setNodeRef}>
      /* Render whatever you like within */
    </div>
  );
}
```

你可以设置任意多的可放置区域，只要确保它们都有一个唯一的`id`，以便它们可以被区分。每个可放置区域都需要有自己的唯一节点，所以请确保不要尝试将单个可放置区域连接到多个`ref`上。

设置多个可放置区域，只需根据需要多次使用`useDroppable`钩子即可。

```jsx
function MultipleDroppables() {
  const {setNodeRef: setFirstDroppableRef} = useDroppable({
    id: 'droppable-1',
  });
  const {setNodeRef: setsecondDroppableRef} = useDroppable({
    id: 'droppable-2',
  });
  
  return (
    <section>
      <div ref={setFirstDroppableRef}>
        /* Render whatever you like within */
      </div>
      <div ref={setsecondDroppableRef}>
        /* Render whatever you like within */
      </div>
    </section>
  );
}
```
如果你需要动态渲染可放置容器的列表，我们建议你创建一个可重用的可放置组件，并根据需要多次渲染该组件：

```jsx
function Droppable(props) {
  const {setNodeRef} = useDroppable({
    id: props.id,
  });
  
  return (
    <div ref={setNodeRef}>
      {props.children}
    </div>
  );
}

function MultipleDroppables() {
  const droppables = ['1', '2', '3', '4'];
  
  return (
    <section>
      {droppables.map((id) => (
        <Droppable id={id} key={id}>
          Droppable container id: ${id}
        </Droppable>
      ))}
    </section>
  );
}
```
更多关于`useDroppable`钩子的使用细节，请参阅API文档部分：

{% page-ref page="usedroppable.md" %}

