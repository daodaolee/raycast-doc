# getAvatarIcon

当您没有头像时代表头像的图标。生成的头像将根据名字的首字母生成，并具有色彩丰富但一致的背景。

![头像图标示例](../../.gitbook/assets/utils-avatar-icon.png)

## 签名

```ts
function getAvatarIcon(
  name: string,
  options?: {
    background?: string;
    gradient?: boolean;
  }
): Image.Asset;
```

* `name` 是主题名称的字符串。
* `options.background`是用作背景颜色的颜色的十六进制表示。默认情况下，该项将从精心挑选的一组颜色中选择随机但一致的（例如，相同的名称具有相同的颜色）颜色，以很好地匹配 Raycast。
* `options.gradient`是一个布尔值，用于选择背景是否应该有轻微的渐变。默认情况下，它会。

返回可在 Raycast 期望的地方使用的 [Image.Asset](../../api-can-kao/user-interface/icons-and-images.md) 。

## 例子

```tsx
import { List } from "@raycast/api";
import { getAvatarIcon } from "@raycast/utils";

export default function Command() {
  return (
    <List>
      <List.Item icon={getAvatarIcon("John Doe")} title="John Doe" />
    </List>
  );
}
```
