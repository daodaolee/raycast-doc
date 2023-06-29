# getProgressIcon

代表任务、项目等进度的图标。

![Progress Icon example](../../.gitbook/assets/utils-progress-icon.png)

## 签名

```ts
function getProgressIcon(
  progress: number,
  color?: Color | string,
  options?: {
    background?: Color | string;
    backgroundOpacity?: number;
  }
): Image.Asset;
```

* `progress` 是 0 到 1 之间的数字（0 表示未开始，1 表示完成）。
* `color` 是 Raycast `Color` 或颜色的十六进制表示形式。默认情况下它将是 `Color.Red`。
* `options.background` 是 Raycast `Color` 或进度图标背景颜色的十六进制表示形式。默认情况下，如果 Raycast 的外观较暗，则为 `white`；如果外观较亮，则为 `light`。
* `options.backgroundOpacity` 是进度图标背景的不透明度。默认情况下，该值为 `0.1`。

返回可在 Raycast 期望的地方使用的 [Image.Asset](../../api-can-kao/user-interface/icons-and-images.md#image.asset)。

## Example

```tsx
import { List } from "@raycast/api";
import { getProgressIcon } from "@raycast/utils";

export default function Command() {
  return (
    <List>
      <List.Item icon={getProgressIcon(0.1)} title="Project" />
    </List>
  );
}
```
