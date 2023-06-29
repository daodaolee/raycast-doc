# getFavicon

显示网站图标的图标。

收藏夹图标（收藏夹图标）是随网站附带的一个小图标，显示在浏览器地址栏、页面选项卡和书签菜单等位置。

![网站图标示例](../../.gitbook/assets/utils-favicon.png)

## 签名

```ts
function getFavicon(
  url: string | URL,
  options?: {
    fallback?: Image.Fallback;
    size?: boolean;
    mask?: Image.Mask;
  }
): Image.ImageLike;
```

* `name` 是主题名称的字符串。
* `options.fallback`是一个 [Image.Fallback](../../api-can-kao/user-interface/icons-and-images.md#image.fallback) 图标，以防找不到 Favicon。默认情况下，后备将为 `Icon.Link`。
* `options.size`是返回的图标的大小。默认情况下，它是 64 像素。
* `options.mask` 是应用于图标的 [Image.Mask](../../api-can-kao/user-interface/icons-and-images.md#image.mask) 的大小。

返回一个可以在 Raycast 需要的地方使用的 [Image.ImageLike](../../api-can-kao/user-interface/icons-and-images.md#image.imagelike)。

## 例子

```tsx
import { List } from "@raycast/api";
import { getFavicon } from "@raycast/utils";

export default function Command() {
  return (
    <List>
      <List.Item icon={getFavicon("https://raycast.com")} title="Raycast Website" />
    </List>
  );
}
```
