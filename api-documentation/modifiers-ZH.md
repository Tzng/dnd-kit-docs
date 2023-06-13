# 修改器

修改器允许你动态修改传感器检测到的移动坐标。它们可以用于广泛的用例，例如:

* 将运动限制在单一轴上
* 将运动限制在可拖动节点容器的边界矩形内
* 将运动限制在可拖动节点的滚动容器边界矩形内
* 应用阻力或夹紧运动

## 安装

To start using modifiers, install the modifiers package via yarn or npm:

要开始使用修改器，请通过yarn或npm安装修改器包:

```
npm install @dnd-kit/modifiers
```

## 使用

修改器仓库包含一些有用的修改器，可以应用于[`DndContext`](context-provider/)以及[`DragOverlay`](draggable/drag-overlay.md)。

```jsx
import {DndContext, DragOverlay} from '@dnd-kit';
import {
  restrictToVerticalAxis,
  restrictToWindowEdges,
} from '@dnd-kit/modifiers';

function App() {
  return (
    <DndContext modifiers={[restrictToVerticalAxis]}>
      {/* ... */}
      <DragOverlay modifiers={[restrictToWindowEdges]}>
        {/* ... */}
      </DragOverlay>
    </DndContext>
  )
}
```

如上面的示例所示，`DndContext`和`DragOverlay`都可以有不同的修改器。
## 内置的修饰符

### 将运动限制在单一轴上

#### `restrictToHorizontalAxis`

仅将移动限制为水平轴

#### `restrictToVerticalAxis`

仅将移动限制为垂直轴

### 限制运动到容器的边界矩形

#### `restrictToWindowEdges`

限制运动到窗口的边缘。这个修改器可以防止`DragOverlay`被移出窗口的边界。

#### `restrictToParentElement`

限制运动到可拖动节点的父元素。

#### `restrictToFirstScrollableAncestor`

限制运动到可拖动节点的第一个可滚动的祖先元素。

### 对齐网格

#### `createSnapModifier`

函数创建修饰符以捕捉给定的网格大小。

```javascript
import {createSnapModifier} from '@dnd-kit/modifiers';

const gridSize = 20; // pixels
const snapToGridModifier = createSnapModifier(gridSize);
```

## 构建自定义修饰符

要构建自己的自定义修改器，请参考`@dnd-kit/modifiers`的内置修改器的实现：[https://github.com/clauderic/dnd-kit/tree/master/packages/modifiers/src](https://github.com/clauderic/dnd-kit/tree/master/packages/modifiers/src)

例如，这是一个创建修改器以捕捉网格的实现:

```javascript
const gridSize = 20;

function snapToGrid(args) {
  const {transform} = args;
  
  return {
    ...transform,
    x: Math.ceil(transform.x / gridSize) * gridSize,
    y: Math.ceil(transform.y / gridSize) * gridSize,
  };
 }
```



