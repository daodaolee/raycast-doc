# Environment

Environment API 对于获取有关命令运行的设置的上下文非常有用。您可以获得有关扩展和命令本身以及 Raycast 的信息。此外，还注入了一些路径，有助于构建与资源相关的文件路径命令。

## API 参考

### environment

包含环境值，例如 Raycast 版本、扩展信息和路径。

#### 例子

```typescript
import { environment } from "@raycast/api";

export default async function Command() {
  console.log(`Raycast version: ${environment.raycastVersion}`);
  console.log(`Extension name: ${environment.extensionName}`);
  console.log(`Command name: ${environment.commandName}`);
  console.log(`Command mode: ${environment.commandMode}`);
  console.log(`Assets path: ${environment.assetsPath}`);
  console.log(`Support path: ${environment.supportPath}`);
  console.log(`Is development mode: ${environment.isDevelopment}`);
  console.log(`Appearance: ${environment.appearance}`);
  console.log(`Text size: ${environment.textSize}`);
  console.log(`LaunchType: ${environment.launchType}`);
}
```

#### 属性

| 名称                                               | 描述                                  | 类型                                        |
| ------------------------------------------------ | ----------------------------------- | ----------------------------------------- |
| appearance<mark style="color:red;">\*</mark>     | Raycast 应用程序使用的外观。                  | `"light"` 或 `"dark"`                      |
| assetsPath<mark style="color:red;">\*</mark>     | 扩展的资源目录的绝对路径。                       | `string`                                  |
| commandMode<mark style="color:red;">\*</mark>    | 启动命令的模式，在 package.json 中指定          | `"no-view"` 或 `"view"` 或 `"menu-bar"`     |
| commandName<mark style="color:red;">\*</mark>    | 已启动命令的名称，在 package.json 中指定         | `string`                                  |
| extensionName<mark style="color:red;">\*</mark>  | 扩展名，在 package.json 中指定              | `string`                                  |
| isDevelopment<mark style="color:red;">\*</mark>  | 指示该命令是否是开发命令（相对于 Store 中安装的命令）。     | `boolean`                                 |
| launchType<mark style="color:red;">\*</mark>     | 命令的启动类型（用户启动或后台）。                   | [`LaunchType`](environment.md#launchtype) |
| raycastVersion<mark style="color:red;">\*</mark> | Raycast 主应用程序的版本                    | `string`                                  |
| supportPath<mark style="color:red;">\*</mark>    | 扩展支持目录的绝对路径。使用它来读取和写入与您的扩展或命令相关的文件。 | `string`                                  |
| textSize<mark style="color:red;">\*</mark>       | Raycast 应用程序使用的文本大小。                | `"medium"` 或 `"large"`                    |
| canAccess<mark style="color:red;">\*</mark>      | 返回用户是否有权访问给定的 API。                  | `(api: unknown) => boolean`               |

## environment.canAccess

检查用户是否可以访问特定的API。

#### 签名

```typescript
function canAccess(api: any): bool;
```

#### 例子

```typescript
import { AI, showHUD, environment } from "@raycast/api";
import fs from "fs";

export default async function main() {
  if (environment.canAccess(AI)) {
    const answer = await AI.ask("Suggest 5 jazz songs");
    await Clipboard.copy(answer);
  } else {
    await showHUD("You don't have access :(");
  }
}
```

#### 返回

一个布尔值，指示运行命令的用户是否有权访问 API。

### getSelectedFinderItems

从 Finder 获取选定的项目。

#### 签名

```typescript
async function getSelectedFinderItems(): Promise<FileSystemItem[]>;
```

#### 例子

```typescript
import { getSelectedFinderItems, showToast, Toast } from "@raycast/api";

export default async function Command() {
  try {
    const selectedItems = await getSelectedFinderItems();
    console.log(selectedItems);
  } catch (error) {
    await showToast({
      style: Toast.Style.Failure,
      title: "Cannot copy file path",
      message: String(error),
    });
  }
}
```

#### 返回

使用 [选定的文件系统项](https://developers.raycast.com/api-reference/environment#filesystemitem) 解析的 Promise。如果 Finder 不是最前面的应用程序，则 Promise 将被 rejected。

### getSelectedText

获取最前面的应用程序的选定文本。

#### 签名

```typescript
async function getSelectedText(): Promise<string>;
```

#### 例子

```typescript
import { getSelectedText, Clipboard, showToast, Toast } from "@raycast/api";

export default async function Command() {
  try {
    const selectedText = await getSelectedText();
    const transformedText = selectedText.toUpperCase();
    await Clipboard.paste(transformedText);
  } catch (error) {
    await showToast({
      style: Toast.Style.Failure,
      title: "Cannot transform text",
      message: String(error),
    });
  }
}
```

#### 返回

使用所选文本解析的 Promise。如果在最前面的应用程序中没有选择任何文本，则 Promise 将被 rejected。

## 类型

### FileSystemItem

保存有关文件系统项的数据。使用 [getSelectedFinderItems](https://developers.raycast.com/api-reference/environment#getselectedfinderitems) 方法检索值。

#### 属性

| 名称                                     | 描述    | 类型       |
| -------------------------------------- | ----- | -------- |
| path<mark style="color:red;">\*</mark> | 项目的路径 | `string` |

### LaunchType

指示命令启动的类型。使用它来检测命令是否已从后台启动。

#### 枚举成员

| 名称            | 描述            |
| ------------- | ------------- |
| UserInitiated | 通过用户交互定期启动    |
| Background    | 按时间间隔安排并从后台启动 |
