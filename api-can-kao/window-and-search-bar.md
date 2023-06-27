# 窗口 & 搜索栏

## API 参考

### clearSearchBar

清除搜索栏中的文本。

#### 签名

```typescript
async function clearSearchBar(options: { forceScrollToTop: boolean }): Promise<void>;
```

#### 参数

| 名称      | 描述                        | 类型                              |
| ------- | ------------------------- | ------------------------------- |
| options | 可用于强制滚动到顶部。清除搜索栏后默认滚动到顶部。 | `{ forceScrollToTop: boolean }` |

#### 返回

是一个 promise，当搜索栏被清除时变为 resolve。

### closeMainWindow

关闭 Raycast 主窗口。

#### 签名

```typescript
async function closeMainWindow(options: { clearRootSearch: boolean; popToRootType?: PopToRootType }): Promise<void>;
```

#### 例子

```typescript
import { closeMainWindow } from "@raycast/api";
import { setTimeout } from "timers/promises";

export default async function Command() {
  await setTimeout(1000);
  await closeMainWindow({ clearRootSearch: true });
}
```

您可以使用 `popToRootType` 参数暂时阻止 Raycast 在 Raycast 中应用用户的 “Pop to Root Search” 首选项；例如，当您需要与外部系统实用程序交互，然后允许用户返回到视图命令时：

```typescript
import { closeMainWindow, PopToRootType } from "@raycast/api";

export default async () => {
  await closeMainWindow({ popToRootType: PopToRootType.Suspended });
};
```

#### 参数

| 名称      | 描述                                                                                                                                                                                                                           | 类型                                                                                                         |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| options | 具有以下属性的参数对象：`clearRootSearch`：清除根搜索栏中的文本并滚动到顶部；默认为 `false` `popToRootType`：定义弹出到根（[PopToRootType](https://developers.raycast.com/api-reference/window-and-search-bar#poptoroottype)）；默认值是在 Raycast 中的“Pop to Root Search”首选项 | `{ clearRootSearch: boolean; popToRootType:` [`PopToRootType`](window-and-search-bar.md#poptoroottype) `}` |

#### 返回

是一个 promise，当主窗口关闭时变为 resolve。

### popToRoot

将导航堆栈弹出回根搜索。

#### 签名

```typescript
async function popToRoot(options: { clearSearchBar: boolean }): Promise<void>;
```

#### 例子

```typescript
import { Detail, popToRoot } from "@raycast/api";
import { useEffect } from "react";
import { setTimeout } from "timers";

export default function Command() {
  useEffect(() => {
    setTimeout(() => {
      popToRoot({ clearSearchBar: true });
    }, 3000);
  }, []);

  return <Detail markdown="See you soon 👋" />;
}
```

#### 参数

| 名称      | 描述                              | 类型                            |
| ------- | ------------------------------- | ----------------------------- |
| options | 可用于清除搜索栏。默认情况下，弹出到 root 后清除搜索栏。 | `{ clearSearchBar: boolean }` |

#### 返回

是一个 promise，当 Raycast 弹出到 root 时变为 resolve。

## 类型

### PopToRootType

定义主窗口关闭时弹出到根目录的行为。

#### 枚举成员

| 名称        | 描述                                      |
| --------- | --------------------------------------- |
| Default   | 允许用户在 Raycast 中的“Pop to Root Search”首选项 |
| Immediate | 立即弹回到 root                              |
| Suspended | 防止 Raycast 弹出回根目录                       |
