# 键盘

键盘传感器响应[键盘事件](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)。如果没有定义，它是[DndContext](../context-provider/)提供程序使用的默认传感器之一。

{% hint style="warning" %}
为了使键盘传感器正常工作，必须能够接收`useDraggable`[监听器](../draggable/usedraggable.md#listeners)接收的激活器元素。要了解更多信息，请阅读深入的[无障碍指南](../../guides/accessibility.md)。
{% endhint %}

### Activator

键盘激活器是`onKeyDown`事件处理程序。如果[`event.code`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code)属性与传递给键盘传感器的`keyboardCodes`选项的`start`键之一匹配，则初始化键盘传感器。

默认情况下，激活键盘传感器的键是`Space`和`Enter`。

### 选项

#### 键盘码

这个选项代表与拖动`start`、`cancel`和`end`事件相关联的键。`keyboardCodes`选项遵循以下接口：

```typescript
type KeyboardCode = KeyboardEvent['code'];

interface KeyboardCodes {
  start: KeyboardCode[];
  cancel: KeyboardCode[];
  end: KeyboardCode[];
};
```

默认值是：

```javascript
const defaultKeyboardCodes = {
  start: ['Space', 'Enter'],
  cancel: ['Escape'],
  end: ['Space', 'Enter'],
};
```

您可以自定义这些值，但请记住，[ARIA的第三条规则](https://www.w3.org/TR/using-aria/#3rdrule)要求用户**必须**能够使用**两个**键（在Windows上为`enter`或在macOS上为`return`）和`space`键来激活与可拖动小部件相关联的操作。要了解更多信息，请阅读深入的[无障碍指南](../../guides/accessibility.md)。

请记住，如果您更新这些值，还应该使用[`<DndContext>`](../context-provider/)的`screenReaderInstructions`属性自定义屏幕阅读器说明，因为屏幕阅读器说明假定键盘传感器使用默认的键盘快捷方式初始化。

`move`键盘代码不是可自定义的选项，因为它们由[坐标获取函数](keyboard.md#coordinates-getter)处理。要自定义它们，请编写自定义坐标获取函数。

#### 坐标获取器

默认情况下，当拖动时按下任何箭头键时，键盘传感器将以`25`像素的任何给定方向移动。

这是一个合理的默认值，可能适合也可能不适合您正在构建的用例。

`getNextCoordinates`选项可用于定义自定义坐标获取函数，该函数将最新的键盘`event`以及当前坐标传递给该函数：

```javascript
function customCoordinatesGetter(event, args) {
  const {currentCoordinates} = args;
  const delta = 50;
  
  switch (event.code) {
    case 'Right':
      return {
        ...currentCoordinates,
        x: currentCoordinates.x + delta,
      };
    case 'Left':
      return {
        ...currentCoordinates,
        x: currentCoordinates.x - delta,
      };
    case 'Down':
      return {
        ...currentCoordinates,
        y: currentCoordinates.y + delta,
      };
    case 'Up':
      return {
        ...currentCoordinates,
        y: currentCoordinates.y - delta,
      };
  }

  return undefined;
};
```

虽然上面的示例相当简单，但您可以构建复杂的坐标获取器来支持高级用例。[Sortable](../../presets/sortable/)预设使用`getNextCoordinates`选项在键盘传感器之上构建，并根据按下的箭头键将活动的可排序项移动到其新索引。

### 滚动行为

这个选项代表[滚动行为](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollTo)应该在滚动到新坐标时使用。默认值是`smooth`，这导致滚动容器平滑地滚动到新坐标。

另一个可能的值是`auto`，这导致滚动容器直接滚动到新坐标，而没有任何动画。

