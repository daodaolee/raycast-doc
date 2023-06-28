# Colors

任何可以在组件属性中传递颜色的地方，都可以传递：

* 标准 [Color](colors.md#color)
* [动态](https://developers.raycast.com/api-reference/user-interface/colors#color.dynamic) Color
* [原始](https://developers.raycast.com/api-reference/user-interface/colors#color.raw) Color

## API 参考

### Color

标准颜色。使用这些颜色以保持一致性。&#x20;

颜色自动适应 Raycast 主题（浅色或深色）。

#### 例子

```typescript
import { Color, Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item title="Blue" icon={{ source: Icon.Circle, tintColor: Color.Blue }} />
      <List.Item title="Green" icon={{ source: Icon.Circle, tintColor: Color.Green }} />
      <List.Item title="Magenta" icon={{ source: Icon.Circle, tintColor: Color.Magenta }} />
      <List.Item title="Orange" icon={{ source: Icon.Circle, tintColor: Color.Orange }} />
      <List.Item title="Purple" icon={{ source: Icon.Circle, tintColor: Color.Purple }} />
      <List.Item title="Red" icon={{ source: Icon.Circle, tintColor: Color.Red }} />
      <List.Item title="Yellow" icon={{ source: Icon.Circle, tintColor: Color.Yellow }} />
      <List.Item title="PrimaryText" icon={{ source: Icon.Circle, tintColor: Color.PrimaryText }} />
      <List.Item title="SecondaryText" icon={{ source: Icon.Circle, tintColor: Color.SecondaryText }} />
    </List>
  );
}
```

#### 枚举成员

| 名称            | 深色 Theme                                                 | 浅色 Theme                                            |
| ------------- | -------------------------------------------------------- | --------------------------------------------------- |
| Blue          | ![](../../.gitbook/assets/color-dark-blue.png)           | ![](../../.gitbook/assets/color-blue.png)           |
| Green         | ![](../../.gitbook/assets/color-dark-green.png)          | ![](../../.gitbook/assets/color-green.png)          |
| Magenta       | ![](../../.gitbook/assets/color-dark-magenta.png)        | ![](../../.gitbook/assets/color-magenta.png)        |
| Orange        | ![](../../.gitbook/assets/color-dark-orange.png)         | ![](../../.gitbook/assets/color-orange.png)         |
| Purple        | ![](../../.gitbook/assets/color-dark-purple.png)         | ![](../../.gitbook/assets/color-purple.png)         |
| Red           | ![](../../.gitbook/assets/color-dark-red.png)            | ![](../../.gitbook/assets/color-red.png)            |
| Yellow        | ![](../../.gitbook/assets/color-dark-yellow.png)         | ![](../../.gitbook/assets/color-yellow.png)         |
| PrimaryText   | ![](../../.gitbook/assets/color-dark-primary-text.png)   | ![](../../.gitbook/assets/color-primary-text.png)   |
| SecondaryText | ![](../../.gitbook/assets/color-dark-secondary-text.png) | ![](../../.gitbook/assets/color-secondary-text.png) |

## 类型

### Color.ColorLike

```typescript
ColorLike: Color | Color.Dynamic | Color.Raw;
```

支持的颜色类型的联合类型。&#x20;

使用 [原始颜色](https://developers.raycast.com/api-reference/user-interface/colors#color.raw) 时，将对其进行调整以实现与 Raycast 用户界面的高对比度。要禁用颜色调整，您需要切换到使用 [动态颜色](colors.md#api-can-kao)。但是，我们建议保留颜色调整，除非您的扩展依赖于精确的颜色。

#### 例子

```typescript
import { Color, Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item title="Built-in color" icon={{ source: Icon.Circle, tintColor: Color.Red }} />
      <List.Item title="Raw color" icon={{ source: Icon.Circle, tintColor: "#FF0000" }} />
      <List.Item
        title="Dynamic color"
        icon={{
          source: Icon.Circle,
          tintColor: {
            light: "#FF01FF",
            dark: "#FFFF50",
            adjustContrast: true,
          },
        }}
      />
    </List>
  );
}
```

### Color.Dynamic

动态颜色根据活动的 Raycast 主题应用不同的颜色。&#x20;

使用 [动态颜色](https://developers.raycast.com/api-reference/user-interface/colors#color.dynamic) 时，将对其进行调整以实现与 Raycast 用户界面的高对比度。要禁用颜色调整，可以将 `adjustmentContrast` 属性设置为 `false`。但是，我们建议保留颜色调整，除非您的扩展依赖于精确的颜色。

#### 例子

```typescript
import { Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item
        title="Dynamic Tint Color"
        icon={{
          source: Icon.Circle,
          tintColor: {
            light: "#FF01FF",
            dark: "#FFFF50",
            adjustContrast: false,
          },
        }}
      />
      <List.Item
        title="Dynamic Tint Color"
        icon={{
          source: Icon.Circle,
          tintColor: { light: "#FF01FF", dark: "#FFFF50" },
        }}
      />
    </List>
  );
}
```

#### 属性

| 名称                                      | 描述                   | Type      |
| --------------------------------------- | -------------------- | --------- |
| dark<mark style="color:red;">\*</mark>  | 深色主题中使用的颜色。          | `string`  |
| light<mark style="color:red;">\*</mark> | 浅色主题中使用的颜色。          | `string`  |
| adjustContrast                          | 启用浅色和深色主题颜色的动态对比度调整。 | `boolean` |

### Color.Raw

颜色也可以是简单的字符串。您可以使用以下任意颜色格式：

* HEX, e.g `#FF0000`
* 短 HEX, e.g. `#F00`
* RGBA, e.g. `rgb(255, 0, 0)`
* RGBA 百分比, e.g. `rgb(255, 0, 0, 1.0)`
* HSL, e.g. `hsla(200, 20%, 33%, 0.2)`
* 关键字, e.g. `red`
