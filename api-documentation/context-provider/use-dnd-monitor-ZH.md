# useDndMonitor

`useDndMonitor`钩子可以在包裹在`DndContext`提供程序中的组件中使用，以监视针对该`DndContext`发生的不同拖放事件。
```jsx
import {DndContext, useDndMonitor} from '@dnd-kit/core';

function App() {
  return (
    <DndContext>
      <Component />
    </DndContext>
  );
}

function Component() {
  // 监视发生在父' DndContext '提供程序上的拖放事件
  useDndMonitor({
    onDragStart(event) {},
    onDragMove(event) {},
    onDragOver(event) {},
    onDragEnd(event) {},
    onDragCancel(event) {},
  });
}
```



